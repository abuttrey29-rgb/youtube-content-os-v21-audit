# YouTube Content OS V2.1 — Hồ sơ kiểm toán độc lập

Public evidence package for the independent audit of **YouTube Content OS V2.1**
(sprint 24h, hoàn thành sớm 08:50 ngày 12/10/2026, deadline 08:40 13/10/2026).

## Đọc gì trước

- **[AUDIT_INDEX.md](AUDIT_INDEX.md)** — danh sách đầy đủ mọi tài liệu + link RAW trực tiếp.
- `reports/V2.1_SPRINT_24H_REPORT_20261013.md` — báo cáo bàn giao sprint (8 mục).
- `reports/V2.1_INITIAL_REPORT_20261012.md` — đối chiếu kiểm toán V1 + kiến trúc V2.1 + Independent Oversight.
- `reports/V1_GPT_INDEPENDENT_AUDIT_20261010.md` — báo cáo kiểm toán độc lập gốc của GPT (10/10/2026).

## Cấu trúc

| Thư mục | Nội dung |
|---|---|
| `reports/` | Báo cáo sprint, đối chiếu kiểm toán V1/V2.1, chỉ đạo, quyết định, điều tra config |
| `radar/` | Breakout Radar: kết quả 8 kênh + scoring, mã nguồn, 31 file JSON snapshot API gốc |
| `market_intel/` | 13 cơ hội nội dung (shortlist + ranking + công thức chấm điểm), 30 JSON snapshot |
| `analytics/` | Snapshot Analytics T0, mã nguồn `video_status.py` (phân biệt PRIVATE/SCHEDULED/PUBLIC/FAILED) |
| `qc/` | `rights_gate.py` (fail-closed), `hash_binding.py` (SHA-256 MP4↔QC), test reports, staging trial, rollback plan |
| `oversight/` | Independent Oversight: watcher, audit log (hash-chain), policy gate, 4-case acceptance tests |
| `docs/` | Cron jobs (chỉ-đọc), hướng dẫn tạo API key Radar, dossier xin scope yt-analytics.readonly |

## Quy ước trung thực dữ liệu

Mọi số liệu trong hồ sơ được gắn nhãn: **LIVE VERIFIED** (gọi API thật, có bằng chứng),
**TESTED** (chạy test thật, chưa production), **MOCK** (mô phỏng — không dùng làm bằng chứng),
**CHƯA HOÀN THÀNH** (đang chờ dữ liệu). Không nội suy số liệu thiếu.

## Làm sạch trước khi công khai

- Không có API key, OAuth token, refresh token, client secret, private key hay mật khẩu.
- Email trong JSON snapshot API (email liên hệ công khai của kênh YouTube) đã che thành `[EMAIL-REDACTED]`.
- Email tài khoản quản trị đã che thành `[EMAIL-REDACTED]`.
