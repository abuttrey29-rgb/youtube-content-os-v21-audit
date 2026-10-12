# STAGING TRIAL REPORT — rights_gate + hash_binding

**Ngày chạy:** 2026-10-12 (giờ VN)
**Môi trường:** staging cô lập `v2_1/staging/quality_gates/` — KHÔNG động V1/production
**Mục đích:** thử nghiệm 2 module vá kiểm toán P0.2 + P0.5 trước khi đề xuất tích hợp production
**Chi phí:** $0 (chỉ dùng stdlib Python + ffmpeg có sẵn trên VM)

## 1. Thiết lập staging — TESTED

| Hạng mục | Trạng thái | Bằng chứng |
|---|---|---|
| Tạo `v2_1/staging/quality_gates/` | TESTED | thư mục tạo mới, V1 không động |
| Copy `rights_gate.py` (không sửa) | TESTED | `diff -q` gốc vs staging: IDENTICAL |
| Copy `hash_binding.py` (không sửa) | TESTED | `diff -q` gốc vs staging: IDENTICAL |
| Copy `samples/cv_v2_manifest.json` | TESTED | `diff -q`: IDENTICAL |
| File gốc `v2_1/modules/video_quality/` | TESTED | nguyên vẹn, timestamp không đổi |
| MP4 mẫu thật `fixtures/sample_video.mp4` | TESTED | ffmpeg lavfi testsrc 3s, 320x240, H.264+AAC, 43.151 bytes, SHA-256 `02485efa008d7d802b182a93a031760d13e85da61a277552ad9f929f261fbe62` (ffprobe xác nhận `mov,mp4,m4a,3gp,3g2,mj2`) |

Không có dependency ngoài stdlib (`json`, `os`, `hashlib`, `hmac`, `datetime`, `argparse`, `subprocess`) — module staging tự đủ, không import gì từ V1.

## 2. Regression P0.2 — rights_gate.py — TESTED (22/22 PASS)

Vá: *"thiếu manifest nguồn / thiếu giấy phép / thiếu evidence → phải BLOCKED"*.
Chạy thật module staging trên manifest JSON thật + MP4 thật, không mock logic gate.

```
[PASS] R01 | thiếu file manifest | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R02 | manifest_path = None | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R03 | manifest_path rỗng | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R04 | manifest JSON hỏng | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R05 | manifest là array (sai cấu trúc) | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R06 | manifest không có shots | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R07 | shots = [] rỗng | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R08 | shot thiếu license | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R09 | shot license rỗng | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R10 | shot thiếu evidence | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R11 | shot evidence = {} rỗng | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R12 | evidence không có trường chứng minh | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R13 | manifest hợp lệ (7 shot allowlist+evidence) | kỳ vọng=PASS, thực tế=PASS
[PASS] R14 | license "fair use tự nhận" -> REVIEW (bắt buộc người duyệt) | kỳ vọng=REVIEW, thực tế=REVIEW
[PASS] R15 | license lạ "XYZ License 2026" -> REVIEW | kỳ vọng=REVIEW, thực tế=REVIEW
[PASS] R16 | license mập mờ "public domain" -> REVIEW | kỳ vọng=REVIEW, thực tế=REVIEW
[PASS] R17 | video không tồn tại -> BLOCKED | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R18 | BLOCKED+REVIEW -> BLOCKED tổng | kỳ vọng=BLOCKED, thực tế=BLOCKED
[PASS] R19 | REVIEW+PASS -> REVIEW tổng | kỳ vọng=REVIEW, thực tế=REVIEW
[PASS] R20a | CLI manifest hợp lệ -> exit 0 | kỳ vọng=0, thực tế=0
[PASS] R20b | CLI thiếu manifest -> exit 2 | kỳ vọng=2, thực tế=2
[PASS] R21 | không đường nào PASS khi thiếu manifest | kỳ vọng=True, thực tế=True

==== P0.2: 22/22 case đạt ====   (exit code 0)
```

**Kết luận P0.2:** không có đường nào trả PASS khi thiếu manifest/giấy phép/evidence.
Ghi rõ theo yêu cầu: case *"manifest có nhưng license không rõ"* → **REVIEW** (R14/R15/R16),
không phải BLOCKED cứng và KHÔNG auto-PASS — bắt buộc người duyệt, đúng thiết kế fail-closed.

## 3. Regression P0.5 — hash_binding.py — TESTED (13/13 PASS)

Vá: *"MP4 bị thay đổi sau QC thì phải phát hiện qua hash binding"* (chống TOCTOU).
Chạy SHA-256 thật trên file thật; mọi thao tác sửa file thực hiện trên BẢN COPY,
file gốc `sample_video.mp4` giữ nguyên (H12 xác nhận hash gốc không đổi sau test).

```
[PASS] H01 | seal + verify MP4 nguyên vẹn | kỳ vọng ok=True, thực tế ok=True
       verify reason: OK: MP4 và QC report nguyên vẹn so với thời điểm seal
       chi tiết: flip 1 byte tại offset 21575 (ad -> ae), hash seal=02485efa008d7d80...
[PASS] H02 | sửa 1 byte MP4 sau seal -> FAIL (BLOCKED ở tầng gọi) | kỳ vọng ok=False, thực tế ok=False
       verify reason: MP4 ĐÃ BỊ ĐỔI sau QC (hash không khớp)
[PASS] H03 | sửa 1 byte QC report sau seal -> FAIL | kỳ vọng ok=False, thực tế ok=False
       verify reason: QC report ĐÃ BỊ ĐỔI sau QC (hash không khớp)
[PASS] H04 | mất file MP4 sau seal -> FAIL | kỳ vọng ok=False, thực tế ok=False
       verify reason: MP4 mất sau khi seal: .../fixtures/h04_video.mp4
[PASS] H05 | save/load seal JSON -> verify ok | kỳ vọng ok=True, thực tế ok=True
[PASS] H06 | HMAC đúng secret -> ok | kỳ vọng ok=True, thực tế ok=True
[PASS] H07 | HMAC sai secret -> FAIL | kỳ vọng ok=False, thực tế ok=False
       verify reason: chữ ký HMAC của seal không khớp — seal bị sửa
[PASS] H08 | seal có HMAC nhưng thiếu secret -> FAIL | kỳ vọng ok=False, thực tế ok=False
[PASS] H09 | file seal bị sửa 1 ký tự (có HMAC) -> FAIL | kỳ vọng ok=False, thực tế ok=False
[PASS] H10a | CLI seal+verify nguyên vẹn | kỳ vọng=True, thực tế=True
[PASS] H10b | CLI verify sau sửa 1 byte | kỳ vọng exit 2, thực tế exit 2
[PASS] H11 | seal file không tồn tại -> raise | kỳ vọng=True, thực tế=True
[PASS] H12 | file gốc nguyên vẹn sau test | kỳ vọng=True, thực tế=True

==== P0.5: 13/13 case đạt ====   (exit code 0)
```

**Kết luận P0.5:** sửa đúng 1 byte trong MP4 sau seal → verify FAIL (`ok=False`,
reason "MP4 ĐÃ BỊ ĐỔI sau QC (hash không khớp)"); MP4 nguyên vẹn → `ok=True`.
Ánh xạ nghiệp vụ: `ok=False` ở tầng pipeline = **BLOCKED / không upload** (fail-closed).
Case H09 chứng minh thêm: kẻ sửa file seal JSON cũng bị phát hiện khi có HMAC.

## 4. Tổng hợp

| Hạng mục | Kết quả | Nhãn |
|---|---|---|
| Staging cô lập (copy byte-identical, gốc nguyên vẹn) | xong | TESTED |
| Regression P0.2 (rights_gate fail-closed) | 22/22 PASS, exit 0 | TESTED |
| Regression P0.5 (hash binding TOCTOU) | 13/13 PASS, exit 0 | TESTED |
| Chạy lại test | `cd v2_1/staging/quality_gates && python3 test_regression_p02.py && python3 test_regression_p05.py` | TESTED |
| Tích hợp vào `production/qc_upload_private.py` | CHƯA thực hiện (chờ sếp duyệt riêng) | CHƯA HOÀN THÀNH |
| HMAC secret cho production (Secure Vault / env) | CHƯA thiết lập | CHƯA HOÀN THÀNH |
| Benchmark file >10GB | CHƯA làm (code đọc chunk 4MB, lý thuyết chạy được) | CHƯA HOÀN THÀNH |
| Xác minh provenance độc lập (gọi API nguồn footage) | ngoài phạm vi gate offline $0 | CHƯA HOÀN THÀNH |

## 5. Giới hạn đã biết (giữ nguyên từ TEST_REPORT.md gốc)

1. Provenance dựa trên khai báo của người làm video — gate kiểm tra máy tính đầy đủ
   của license/evidence *như đã khai*, không tự xác minh độc lập footage thật sự là CC0.
2. HMAC secret production cần nơi lưu an toàn (không hardcode, không truyền tay).
3. Allowlist/REVIEW_PATTERNS tĩnh — cần review định kỳ khi thêm nguồn footage mới.

## 6. Ghi chú toàn vẹn

- Trong quá trình trial, tooling đã sinh `__pycache__/` trong thư mục gốc
  `v2_1/modules/video_quality/` (artifact cache, không phải sửa source).
  Đã xóa để trả cây gốc về đúng trạng thái ban đầu; `diff -q` xác nhận 2 file
  `.py` gốc byte-identical với bản staging, timestamp không đổi.
- Đã verify độc lập: `import rights_gate` với `sys.path.insert(0, staging)`
  resolve đúng về bản staging (không lẫn bản gốc) — test thực sự chạy trên
  module staging.

## 7. Đề xuất bước tiếp theo

Regression đạt (35/35). Theo quyết định sếp đã chốt (staging trial → production chỉ sau
regression đạt + rollback, duyệt riêng): phương án rollback chi tiết đã viết tại
`ROLLBACK_PLAN.md` trong cùng thư mục. Tích hợp production CHỜ SẾP DUYỆT RIÊNG —
không tự thực hiện.
