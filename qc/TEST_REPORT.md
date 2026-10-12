# TEST REPORT — Video Quality Gate prototype (sprint 24h, P0.2/P0.5)

**Ngày chạy:** 2026-10-12 ~08:50 (giờ VN)
**Kết quả chạy thật (không mock gate): 20/20 PASS, 0 FAIL**
**Video thật dùng test (chỉ đọc):** `production/cinema_verdict/v2/cv_sample_02_chigurh_v2.mp4`
(57.358.829 bytes, SHA-256 `9dcb2af0c68cb01eeb1a6f7c8e304b5baecd444eeadaa9175de6ea2e70e9cff8`)
**Manifest mẫu:** `v2_1/modules/video_quality/samples/cv_v2_manifest.json`
(dựng từ `footage_sources_v2.md` có thật: 1 shot CC0 Wikimedia + 5 MG tự tạo + 1 audio bed tự tạo)

## 1. rights_gate.py — kết quả (11 case)

| # | Case | Kỳ vọng | Thực tế |
|---|---|---|---|
| 01 | Manifest hợp lệ (7 shot, license trong allowlist + evidence) | PASS | PASS |
| 02 | Thiếu file manifest | BLOCKED | BLOCKED |
| 03 | manifest = None | BLOCKED | BLOCKED |
| 04 | Shot thiếu license | BLOCKED | BLOCKED |
| 05 | Shot thiếu evidence | BLOCKED | BLOCKED |
| 06 | Evidence rỗng `{}` | BLOCKED | BLOCKED |
| 07 | License lạ ("Standard YouTube License") | REVIEW | REVIEW |
| 08 | "fair use tự nhận" | REVIEW (không auto-PASS) | REVIEW |
| 09 | Manifest hỏng JSON | BLOCKED | BLOCKED |
| 10 | Manifest không có shots | BLOCKED | BLOCKED |
| 11 | Video không tồn tại | BLOCKED | BLOCKED |

**Khẳng định chống bug P0.2:** không có đường nào trả PASS khi thiếu manifest/bằng chứng.

## 2. hash_binding.py — kết quả (8 case)

| # | Case | Kỳ vọng | Thực tế |
|---|---|---|---|
| 01 | seal + verify nguyên vẹn (video thật 57MB + QC report) | ok | ok |
| 02 | Sửa đúng **1 byte** MP4 sau seal (copy trong /tmp) | FAIL | FAIL |
| 03 | Sửa QC report sau seal | FAIL | FAIL |
| 04 | Mất file sau seal | FAIL | FAIL |
| 05 | save seal ra file JSON → load → verify | ok | ok |
| 06 | HMAC đúng secret | ok | ok |
| 07 | HMAC sai secret | FAIL | FAIL |
| 08 | Seal có HMAC nhưng verify thiếu secret | FAIL | FAIL |

## 3. End-to-end (1 case)

rights gate PASS → seal → verify: toàn OK (video thật, chỉ đọc — không sửa production).

## 4. Giới hạn còn lại (chưa làm trong prototype này)

1. **Provenance vẫn dựa trên khai báo của người làm video** — gate kiểm tra máy
   tính đầy đủ của license/evidence *như đã khai*, nhưng không thể tự xác minh
   độc lập rằng footage thật sự là CC0 (ví dụ không gọi API Wikimedia để đối chiếu).
   Đây là giới hạn cố hữu của mọi gate offline $0 — giảm rủi ro bằng yêu cầu
   evidence bắt buộc + REVIEW cho trường hợp mập mờ.
2. **Chưa tích hợp vào pipeline production** — theo ràng buộc sprint: không sửa V1.
   Tích hợp `rights_gate.gate()` + `hash_binding.seal()/verify()` vào
   `qc_upload_private.py` là bước Wave 1 (cần duyệt).
3. **HMAC secret chưa có nơi lưu an toàn** — prototype dùng secret truyền tay;
   production cần Secure Vault / env, không hardcode.
4. **Chưa xử lý video >4GB chunked streaming tối ưu** — code đã đọc theo chunk 4MB
   nên chạy được, nhưng chưa benchmark trên file >10GB.
5. **Verify chống TOCTOU giữa seal và upload** — nếu kẻ tấn công đổi file *trước*
   khi seal, gate không phát hiện (ngoài phạm vi: đó là việc của QC checklist +
   rights gate).
6. **Allowlist/REVIEW_PATTERNS là danh sách tĩnh** — cần review định kỳ khi thêm
   nguồn footage mới (Pexels/Pixabay terms đổi, v.v.).

## 5. File bàn giao

- `v2_1/modules/video_quality/rights_gate.py`
- `v2_1/modules/video_quality/hash_binding.py`
- `v2_1/modules/video_quality/samples/cv_v2_manifest.json`
- `v2_1/tests/unit/test_quality_gate_real.py`
- `v2_1/modules/video_quality/TEST_REPORT.md` (file này)

Chạy lại test: `python3 v2_1/tests/unit/test_quality_gate_real.py`
