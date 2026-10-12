# ROLLBACK PLAN — Tích hợp rights_gate + hash_binding vào production

**Trạng thái: KẾ HOẠCH — CHƯA THỰC HIỆN. Tích hợp production CHỜ SẾP DUYỆT RIÊNG.**
**Phạm vi rollback:** file `youtube-os/production/qc_upload_private.py` (+ 1 file wrapper mới).
**Nguyên tắc:** V1 đang chạy ổn định — mọi bước tích hợp đều phải revert được về đúng
trạng thái trước đó trong < 5 phút, không mất dữ liệu, không upload nhầm.

> Lưu ý: workspace `youtube-os` KHÔNG phải git repo (đã kiểm tra 12/10) →
> mọi backup dùng file `.bak.<timestamp>`, KHÔNG dùng `git revert`.

## 0. Điều kiện tiên quyết (pre-conditions) — CHƯA HOÀN THÀNH

| # | Điều kiện | Nhãn |
|---|---|---|
| P0 | Sếp duyệt riêng bước tích hợp production (quyết định đã chốt: staging trial → regression đạt + rollback → duyệt riêng) | CHƯA HOÀN THÀNH |
| P1 | Regression staging 35/35 PASS | TESTED (xem STAGING_TRIAL_REPORT.md) |
| P2 | HMAC secret production đã có nơi lưu an toàn (Secure Vault / env var `YT_SEAL_SECRET`), KHÔNG hardcode, KHÔNG dùng secret test staging | CHƯA HOÀN THÀNH |
| P3 | Khung giờ tích hợp: ngoài giờ upload video hằng ngày của 3 kênh (tránh 11:00–22:00 giờ VN) | CHƯA HOÀN THÀNH |
| P4 | Backup toàn bộ `production/` trước khi sửa (xem B0) | CHƯA HOÀN THÀNH |

Không bắt đầu B1 nếu bất kỳ P0–P4 nào chưa xong.

## 1. Thiết kế tích hợp (để diff nhỏ nhất, rollback dễ nhất)

- **Không sửa logic** `rights_gate.py` / `hash_binding.py` — dùng nguyên bản đã TESTED.
- Tạo file wrapper mới `production/quality_gates_integration.py` chứa 3 hàm:
  `check_rights(manifest, video)`, `seal_after_qc(video, qc_report)`,
  `verify_before_upload(seal_path)` — ánh xạ verdict sang status pipeline
  (`BLOCKED`→dừng, `REVIEW`→chờ người duyệt, `ok=False`→dừng).
- `qc_upload_private.py` chỉ thêm **3 điểm chèn** (mỗi điểm ~5 dòng), không đụng
  logic QC/upload/verify hiện có:
  - **I1** — trong `upload_private()`, ngay sau `qc_video()` PASS: gọi `check_rights()`.
    `BLOCKED` → `status="RIGHTS_BLOCKED"`, return (không upload).
    `REVIEW` → `status="RIGHTS_REVIEW_AWAITING_HUMAN"`, return (không auto-upload).
  - **I2** — sau I1 PASS, trước khi load token: gọi `seal_after_qc()` → lưu seal JSON
    cạnh QC report trong `QC_DIR` (`<video>.seal.json`).
  - **I3** — ngay trước `upload_video(...)`: gọi `verify_before_upload()`.
    `ok=False` → `status="HASH_MISMATCH_BLOCKED"`, return (không upload).
- Thêm CLI arg `--manifest` (bắt buộc cho lệnh upload; lệnh `qc` độc lập không bắt buộc).

## 2. Các bước tích hợp + điểm kiểm tra

### B0. Backup — CHƯA HOÀN THÀNH
1. `cp production/qc_upload_private.py production/qc_upload_private.py.bak.<YYYYMMDD-HHMMSS>`
2. Ghi SHA-256 của file gốc vào `production/BACKUP_LOG.md` (timestamp, người làm, lý do).
3. **Điểm kiểm tra:** `diff` file .bak vs file gốc → identical; SHA-256 khớp log.

### B1. Thêm wrapper `production/quality_gates_integration.py` — CHƯA HOÀN THÀNH
- File MỚI hoàn toàn, không sửa file cũ → rủi ro ~0.
- **Điểm kiểm tra:** `python3 -c "import quality_gates_integration"` chạy được từ `production/`;
  chạy smoke test trên MP4 mẫu staging (rights PASS + seal + verify ok).

### B2. Chèn I1 (rights gate sau QC) — CHƯA HOÀN THÀNH
- Chèn ~5 dòng sau khối `if not ok: ... QC_FAIL ...` trong `upload_private()`.
- **Điểm kiểm tra:** chạy `upload_private` ở chế độ dry-run với (a) manifest thiếu →
  `RIGHTS_BLOCKED`, không gọi upload; (b) manifest hợp lệ → đi tiếp. So sánh log
  với hành vi trước B2 trên cùng input (ngoài verdict mới, mọi thứ khác giống nhau).

### B3. Chèn I2 (seal sau QC) — CHƯA HOÀN THÀNH
- **Điểm kiểm tra:** sau 1 lần chạy QC PASS, file `<video>.seal.json` xuất hiện trong
  `QC_DIR`, `verify` thủ công trên seal → ok=True.

### B4. Chèn I3 (verify trước upload) — CHƯA HOÀN THÀNH
- **Điểm kiểm tra:** sửa 1 byte bản copy MP4 sau seal → chạy pipeline → dừng ở
  `HASH_MISMATCH_BLOCKED`, KHÔNG có call upload nào (kiểm tra bằng log + không có
  video mới trên kênh). Test trên video thử, KHÔNG dùng video thật của kênh.

### B5. Chạy thử end-to-end 1 video thử (PRIVATE) — CHƯA HOÀN THÀNH
- Upload 1 video thử nghiệm lên 1 kênh (PRIVATE), đi qua đủ I1→I2→I3.
- **Điểm kiểm tra:** status `DONE_PRIVATE`; seal verify ok; rights verdict PASS
  trong upload JSON.

## 3. Điều kiện trigger rollback

Rollback NGAY khi bất kỳ điều nào xảy ra (không cần chờ hết bước):

| Mã | Điều kiện | Mức |
|---|---|---|
| T1 | Bất kỳ điểm kiểm tra nào ở B0–B5 FAIL | rollback bước đó |
| T2 | Video thật của kênh bị BLOCKED oan (false positive) do gate — tức manifest hợp lệ nhưng verdict sai | rollback B2 (I1) |
| T3 | Upload bị chặn oan do hash mismatch giả (file không đổi nhưng verify FAIL) | rollback B4 (I3) |
| T4 | Pipeline crash/exception mới xuất hiện ở 3 điểm chèn (traceback không có trước B1) | rollback bước gây lỗi |
| T5 | Sếp ra lệnh dừng / không duyệt tiếp | rollback toàn bộ về B0 |

Riêng T2/T3: trước khi rollback, giữ lại log + manifest + seal liên quan để debug
sau (không xóa evidence).

## 4. Thủ tục rollback từng bước (ngược thứ tự, mỗi bước < 5 phút)

**RB-B4/B3/B2 (gỡ điểm chèn I3/I2/I1):**
1. Mở `production/qc_upload_private.py`, xóa đúng khối code đã chèn ở bước tương ứng
   (mỗi khối được đánh dấu comment `# QUALITY_GATE I1/I2/I3 — <ngày>` khi chèn).
2. Xóa import wrapper nếu không còn điểm chèn nào dùng.
3. **Verify:** `python3 -m py_compile production/qc_upload_private.py` → OK;
   chạy `qc` độc lập trên 1 video thật → kết quả giống trước tích hợp.

**RB-B1 (gỡ wrapper):**
1. `mv production/quality_gates_integration.py /tmp/rollback_gq_<timestamp>.py`
   (giữ lại để điều tra, không xóa hẳn).
2. **Verify:** `grep -n "quality_gates_integration" production/qc_upload_private.py` → không còn tham chiếu.

**RB-B0 (khôi phục toàn bộ — dùng khi T5 hoặc nhiều bước lỗi):**
1. `cp production/qc_upload_private.py.bak.<timestamp> production/qc_upload_private.py`
2. Xóa wrapper theo RB-B1.
3. **Verify:** SHA-256 file đã khôi phục == SHA-256 trong `BACKUP_LOG.md`;
   chạy 1 lệnh `qc` độc lập → PASS như cũ.

**Sau mọi rollback:** ghi vào `production/BACKUP_LOG.md`: thời gian, mã trigger
(T1–T5), bước đã rollback, người thực hiện, kết quả verify. Báo sếp theo chế độ
báo cáo thường trực (vấn đề ảnh hưởng tiến độ → báo ngay).

## 5. Ma trận phê duyệt

| Việc | Người duyệt | Ghi chú |
|---|---|---|
| Bắt đầu tích hợp (P0) | Sếp | duyệt riêng, sau khi đọc STAGING_TRIAL_REPORT.md này |
| Mỗi bước B1–B5 đi tiếp | Em (trong phạm vi đã duyệt) | dừng + báo sếp ngay nếu trigger T1–T4 |
| Rollback từng bước (T1–T4) | Em tự thực hiện, báo sếp sau | trong phạm vi vận hành đã duyệt |
| Rollback toàn bộ / dừng hẳn (T5) | Sếp | |
| Bật lại tích hợp sau rollback | Sếp duyệt lại | kèm nguyên nhân + fix đã test lại ở staging |

## 6. Nhãn tổng hợp

| Hạng mục | Nhãn |
|---|---|
| Staging trial + regression 35/35 | TESTED |
| Kế hoạch rollback này (viết xong, chưa chạy) | TESTED (tài liệu) |
| Mọi bước B0–B5, P2–P4, RB-* | CHƯA HOÀN THÀNH — chờ sếp duyệt riêng mới thực hiện |
