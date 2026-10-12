# BÁO CÁO OVERSIGHT WAVE 0 — Kiểm chứng độc lập (sprint 24h)

**Ngày:** 12/10/2026 · **Deadline:** 08:40 13/10/2026 · **Chi phí:** $0 · **Không sửa V1, không write YouTube, không động token**

---

## 1. ĐÃ KIỂM CHỨNG (verified — có bằng chứng chạy thật)

### 1.1. Breakout Radar — verify độc lập 8/8 kênh: **MATCH**

- **Công cụ:** `oversight/watcher/verify_radar.py` — watcher độc lập, KHÔNG import radar.py, KHÔNG dùng lại raw API response của Radar. Chỉ lấy 8 channel_id từ analysis JSON làm *đối tượng cần re-check*, mọi field đều lấy từ 1 cuộc gọi `channels.list` RIÊNG (batched, ~1 quota unit, read-only).
- **Kết quả:** `VERIFICATION_REPORT.json` — **8/8 kênh khớp toàn bộ 4 field** (publishedAt, subscriberCount, viewCount, videoCount), 0 mismatch, 0 missing.
- **Token:** sha256 + size + mtime trước = sau chạy (file token không bị đụng), đúng chuẩn read-only tuyệt đối.
- **Dung sai áp dụng (ghi rõ trong report):** publishedAt khớp tuyệt đối sau chuẩn hóa phần thập phân giây; subs ±3, views ±max(10, 2%), videoCount ±1 — bù độ trôi thời gian giữa lúc Radar snapshot (~08:44) và lúc verify (~08:47).

### 1.2. Audit log append-only: **3 entries, chuỗi hash nguyên vẹn**

- **Công cụ:** `oversight/audit_log/audit_log.py` — `append_entry()` chỉ mở file ở mode append (1 entry = 1 dòng JSONL); mỗi entry nối hash-chain `sha256(seq|ts|actor|action|subject|detail|prev_hash)`; `verify_chain()` phát hiện sửa ngược.
- **Log:** `oversight/audit_log/audit_20261012.jsonl` — 3 entries:
  - seq 0 · radar_verify → VERIFIED 8/8 MATCH
  - seq 1 · quality_gate_ingest → TEST_REPORT.md của video_quality (claim 20/20 PASS) — **ghi nhận claim, chưa re-run độc lập** (ghi rõ trung thực trong entry)
  - seq 2 · analytics_snapshot_ingest → snapshot T0 (6 video PUBLIC, 0 like/0 comment; CTR/retention N/A)
- **Test chống sửa ngược:** copy log sang /tmp, sửa 1 ký tự ở entry seq 1 → `verify_chain` báo `hash mismatch at seq 1`; log gốc verify OK. PASS.

### 1.3. Policy gate (chặn thao tác nguy hiểm) — thiết kế + test: **11/11 PASS**

- **Công cụ:** `oversight/gates/policy.py` — hàm thuần `evaluate_publish_proposal()` (ALLOW/BLOCK + lý do). Rule fail-closed: channel_id trong registry · không trong denylist · approval_id hợp lệ · QC status PASS · hash binding đã verify · sha256 video khớp QC evidence.
- **Test:** `oversight/gates/test_policy.py` chạy thật — **11/11 PASS** (đủ điều kiện → ALLOW; sai kênh, thiếu approval, approval giả, QC thiếu hash binding, QC REVIEW/BLOCKED, sha256 lệch, thiếu QC, kênh trong denylist, thiếu channel_id → BLOCK kèm lý do).
- **NHẤN MẠNH:** chỉ test logic — **KHÔNG hook vào production, không chặn thật, không can thiệp V1.**

---

## 2. PHÁT HIỆN

- **Không phát hiện lệch** giữa số Radar công bố và số API trả về khi verify độc lập (8/8 khớp trong dung sai).
- **Không phát hiện dấu hiệu token bị ghi/persist** ở cả lần chạy Radar và lần verify.
- Ghi nhận trung thực: kết quả test của video_quality gate là **claim do worker tự báo** (20/20 PASS) — oversight mới chỉ ingest + hash evidence, chưa re-run độc lập trong Wave 0.

---

## 3. GIỚI HẠN (những gì oversight CHƯA làm trong Wave 0)

1. Chỉ verify **channel statistics 4 field** — chưa verify lại logic xếp hạng subs/ngày, ngưỡng breakout, hay 8 query search của Radar (đó là việc của watcher W2 ở Wave 1).
2. Verify là **1 snapshot thời điểm** — số liệu trôi theo giờ; cần re-verify định kỳ.
3. Dung sai (subs ±3, views ±2%) là thỏa hiệp thực tế; nếu kênh bùng nổ thật sự, drift có thể vượt ngưỡng → thiết kế báo `mismatch` chứ không tự động "hợp lý hóa".
4. Audit log hiện lưu 1 bản trong VM — chưa neo hash ra 2 nơi độc lập (theo thiết kế 6.2, làm ở Wave 0 giai đoạn ổn định).
5. Policy gate chưa gắn vào interlock thực (`auto_public: false` vẫn do config vận hành giữ, chưa có code chặn trong pipeline).

---

## 4. FILE BÀN GIAO

| File | Vai trò |
|---|---|
| `oversight/watcher/verify_radar.py` | Watcher verify Radar độc lập |
| `oversight/watcher/VERIFICATION_REPORT.json` | Bằng chứng verify 8/8 MATCH |
| `oversight/audit_log/audit_log.py` | append_entry() + verify_chain() |
| `oversight/audit_log/audit_20261012.jsonl` | 3 entries, chain verified OK |
| `oversight/gates/policy.py` | Policy ALLOW/BLOCK (thiết kế) |
| `oversight/gates/test_policy.py` | 11/11 PASS |
| `oversight/OVERSIGHT_WAVE0_REPORT.md` | File này |

**Đề xuất tiếp theo (Wave 1, cần duyệt):** re-verify Radar định kỳ + watcher W2 (re-run scoring); re-run độc lập test của video_quality; nối `policy.py` thành `interlock.guard()` thật trước mọi thao tác gây tác động; neo hash audit log ra nơi thứ 2 ngoài VM.
