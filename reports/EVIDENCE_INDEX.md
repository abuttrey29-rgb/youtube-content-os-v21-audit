# GÓI BẰNG CHỨNG KIỂM TOÁN — Sprint 24h V2.1 (12/10/2026)

**Mục đích:** mọi tuyên bố "chạy thật / PASS / LIVE VERIFIED" trong báo cáo sprint đều truy được về file bằng chứng gốc. Kiểm toán viên độc lập (sếp, GPT, hoặc người mới) chạy lại theo hướng dẫn bên dưới để tái hiện.

**Quy ước đường dẫn:** mọi đường dẫn trong file này là **tương đối từ `~/workspace/youtube-os/`** (repo root). Không copy file lớn trùng lặp — README chỉ trỏ tới vị trí gốc.

**Báo cáo được kiểm toán:** `v2_1/V2.1_SPRINT_24H_REPORT_20261013.md` (+ `v2_1/V2.1_INITIAL_REPORT_20261012.md` cho phần đối chiếu V1, `v2_1/oversight/OVERSIGHT_WAVE0_REPORT.md` cho phần oversight).

**Nguyên tắc đọc nhãn (bắt buộc khi trích dẫn):**
- **LIVE VERIFIED** = đã chạy thật trên API thật, có raw response lưu lại.
- **TESTED** = test chạy thật trên máy, chưa chạm production/API thật.
- **MOCK** = không có bằng chứng thật (ghi rõ, không tái sử dụng).
- **CHƯA HOÀN THÀNH** = mới làm một phần (ghi rõ phần nào).

---

## 1. Bản đồ tuyên bố → bằng chứng

### A. Breakout Radar — "8 kênh thật, 31 raw JSON, LIVE VERIFIED"

| Tuyên bố trong báo cáo | File bằng chứng (tương đối từ repo root) |
|---|---|
| 8 search.list thật (order=date, publishedAfter 30 ngày, đa niche/ngôn ngữ) | `v2_1/state/snapshots/breakout_radar_20261012/01_search_*.json` (8 file) |
| 179 channelId dedupe → channels.list | `v2_1/state/snapshots/breakout_radar_20261012/02_channels_batch_*.json` (6 file) |
| Deep-scan 8 kênh (playlistItems + videos) | `v2_1/state/snapshots/breakout_radar_20261012/03_playlistitems_*.json` (8 file) + `04_videos_*.json` (8 file) |
| Kết quả xếp hạng cuối | `v2_1/state/snapshots/breakout_radar_20261012/05_analysis_20261012T014403Z.json` |
| Báo cáo đọc được cho người | `v2_1/modules/breakout_radar/RADAR_RESULTS_20261012.md` |
| Code chạy | `v2_1/modules/breakout_radar/radar.py` |
| Quota 824/10.000, token FV nguyên vẹn (sha256/mtime trước=sau) | Ghi trong `RADAR_RESULTS_20261012.md` (mục quota + token integrity) |
| Velocity = "CHƯA ĐỦ DỮ LIỆU" (chỉ 1 snapshot, cấm nội suy) | `v2_1/modules/breakout_radar/RADAR_RESULTS_20261012.md` (mục velocity) |

**Chạy lại (kiểm toán viên):**
```bash
cd ~/workspace/youtube-os
python3 v2_1/modules/breakout_radar/radar.py --dry-run   # chỉ in kế hoạch quota, KHÔNG gọi API
```
Chạy thật (tốn ~824 quota units, cần credential read-only — KHÔNG chạy nếu chưa được duyệt):
```bash
python3 v2_1/modules/breakout_radar/radar.py   # KHÔNG có --dry-run = gọi API thật
```
Lưu ý: chạy thật sẽ tạo snapshot mới có timestamp khác — so sánh nội dung JSON (channelId, statistics), không so tên file.

### B. Oversight verify Radar — "8/8 MATCH, 0 lệch"

| Tuyên bố | Bằng chứng |
|---|---|
| 8/8 kênh khớp 4 field (publishedAt, subs, views, videoCount) | `v2_1/oversight/watcher/VERIFICATION_REPORT.json` |
| Watcher độc lập (không import radar.py, 1 channels.list riêng) | `v2_1/oversight/watcher/verify_radar.py` |
| Dung sai áp dụng (subs ±3, views ±max(10,2%), videoCount ±1) | Ghi trong `VERIFICATION_REPORT.json` + `v2_1/oversight/OVERSIGHT_WAVE0_REPORT.md` §1.1 |
| Token không bị đụng khi verify | `v2_1/oversight/OVERSIGHT_WAVE0_REPORT.md` §1.1 |

**Chạy lại:**
```bash
cd ~/workspace/youtube-os
python3 v2_1/oversight/watcher/verify_radar.py   # gọi ~1 quota unit, read-only; đọc 8 channel_id từ analysis JSON rồi re-check độc lập
```

### C. Video Quality — "rights_gate + hash_binding 20/20 PASS, TESTED"

| Tuyên bố | Bằng chứng |
|---|---|
| 11 case rights_gate PASS | `v2_1/modules/video_quality/TEST_REPORT.md` §1 |
| 8 case hash_binding PASS | `v2_1/modules/video_quality/TEST_REPORT.md` §2 |
| 1 case end-to-end PASS | `v2_1/modules/video_quality/TEST_REPORT.md` §3 |
| Video thật dùng test (57.358.829 bytes, SHA-256 `9dcb2af0…cff8`) | `v2_1/modules/video_quality/TEST_REPORT.md` (đầu file) — file MP4 gốc nằm ở `production/cinema_verdict/v2/` (không copy vào đây) |
| Manifest mẫu | `v2_1/modules/video_quality/samples/cv_v2_manifest.json` |
| Code gate | `v2_1/modules/video_quality/rights_gate.py`, `v2_1/modules/video_quality/hash_binding.py` |
| 6 giới hạn còn lại (provenance dựa khai báo, chưa tích hợp production, HMAC secret chưa có vault...) | `v2_1/modules/video_quality/TEST_REPORT.md` §4 — đọc kỹ trước khi trích dẫn |

**Chạy lại (không cần API, không cần mạng):**
```bash
cd ~/workspace/youtube-os
python3 v2_1/tests/unit/test_quality_gate_real.py   # kỳ vọng: 20/20 PASS, 0 FAIL
```

### D. Oversight acceptance 4 cases — "14/14 PASS" (yêu cầu nghiệm thu #2 của sếp)

| Tuyên bố | Bằng chứng |
|---|---|
| 14/14 PASS: dữ liệu bịa→bắt ×5; sai kênh→BLOCK; thiếu/giả approval→BLOCK ×2; timeout/403/quota→fail-closed ×3; static no-retry ×3 | `v2_1/oversight/tests/test_acceptance_4cases.py` (script tự chạy, in kết quả) |

**Chạy lại (không cần API):**
```bash
cd ~/workspace/youtube-os
python3 v2_1/oversight/tests/test_acceptance_4cases.py   # kỳ vọng: 14/14 PASS
```

### E. Policy gate — "11/11 PASS"

| Tuyên bố | Bằng chứng |
|---|---|
| Logic ALLOW/BLOCK fail-closed (đủ điều kiện→ALLOW; 10 case sai→BLOCK kèm lý do) | `v2_1/oversight/gates/policy.py` (thiết kế) |
| 11/11 test PASS | `v2_1/oversight/gates/test_policy.py` |
| NHẤN MẠNH: chưa hook production, không chặn thật | `v2_1/oversight/OVERSIGHT_WAVE0_REPORT.md` §1.3 |

**Chạy lại (không cần API):**
```bash
cd ~/workspace/youtube-os
python3 v2_1/oversight/gates/test_policy.py   # kỳ vọng: 11/11 PASS
```

### F. Analytics snapshot T0 — "6/9 video đo được, CTR/retention N/A trung thực, LIVE VERIFIED"

| Tuyên bố | Bằng chứng |
|---|---|
| Snapshot T0 (6/9 video PUBLIC đo được; 3 còn SCHEDULED/PRIVATE) | `v2_1/state/snapshots/analytics_20261012/snapshot_20261012_T0.json` |
| Báo cáo đọc được | `v2_1/modules/analytics/ANALYTICS_SNAPSHOT_20261012.md` |
| Code collector | `v2_1/modules/analytics/collector.py` |
| CTR/impressions/retention/watch time = N/A (thiếu scope yt-analytics.readonly) | `v2_1/modules/analytics/ANALYTICS_SNAPSHOT_20261012.md` — xem thêm `v2_1/docs/YT_ANALYTICS_READONLY_SCOPE_DOSSIER.md` |
| Phân biệt PRIVATE/SCHEDULED/PUBLIC/FAILED (yêu cầu nghiệm thu #3) | `v2_1/modules/analytics/video_status.py` — 7/7 unit + 3/3 API thật (2 PUBLIC, 1 SCHEDULED đúng) |

**Chạy lại:**
```bash
cd ~/workspace/youtube-os
python3 v2_1/modules/analytics/collector.py   # gọi API thật, read-only; cần credential — KHÔNG chạy nếu chưa được duyệt
```

### G. Market + Competitor Intel — "13 cơ hội cho 3 kênh, LIVE VERIFIED (phần đề xuất: TESTED)"

| Tuyên bố | Bằng chứng |
|---|---|
| 6 search bổ sung thật | `v2_1/state/snapshots/market_intel_20261012/01_search_sup_*.json` |
| channels/videos bổ sung | `v2_1/state/snapshots/market_intel_20261012/02_channels_batch_*.json`, `03_playlistitems_*.json`, `04_videos_*.json` |
| 13 cơ hội xếp hạng + công thức chấm điểm + lịch hành động 7 ngày | `v2_1/modules/market_intel/OPPORTUNITY_RANKING_20261012.md` |
| Code search bổ sung | `v2_1/modules/market_intel/supplement_search.py` |
| Tổng quota sprint ~1.447/10.000 | Ghi trong báo cáo sprint §5 |

**Chạy lại:**
```bash
cd ~/workspace/youtube-os
python3 v2_1/modules/market_intel/supplement_search.py   # gọi API thật — KHÔNG chạy nếu chưa được duyệt
```

### H. Audit log — "3 entries, chain nguyên vẹn, tamper test PASS"

| Tuyên bố | Bằng chứng |
|---|---|
| 3 entries (radar_verify, quality_gate_ingest, analytics_snapshot_ingest) | `v2_1/oversight/audit_log/audit_20261012.jsonl` |
| append_entry() + verify_chain() | `v2_1/oversight/audit_log/audit_log.py` |
| Test chống sửa ngược (sửa 1 ký tự → phát hiện mismatch) | `v2_1/oversight/OVERSIGHT_WAVE0_REPORT.md` §1.2 |

**Chạy lại (không cần API):**
```bash
cd ~/workspace/youtube-os
python3 -c "
import sys; sys.path.insert(0, 'v2_1/oversight/audit_log')
from audit_log import verify_chain
print(verify_chain('v2_1/oversight/audit_log/audit_20261012.jsonl'))  # kỳ vọng: OK / True
"
```

### I. Đối chiếu kiểm toán V1 (GPT audit vs code hiện tại)

| Tuyên bố | Bằng chứng |
|---|---|
| Đồng ý 12/13 phát hiện P0/P1; 24 module phân loại LIVE/INTEGRATION/MOCK/DEMO/DOCS-ONLY | `v2_1/V2.1_INITIAL_REPORT_20261012.md` (mục 1–2) |
| Claim "64/64 test" của `auto_publish/`: đếm thực tế 155 asserts, 100% FakeYouTubeGateway → **MOCK** | `v2_1/V2.1_SPRINT_24H_REPORT_20261013.md` §1 |
| `qc_upload_private.py` thiếu provenance check máy + thiếu hash binding | `v2_1/V2.1_SPRINT_24H_REPORT_20261013.md` §1; prototype vá ở mục C |
| `channel_status.json` STALE (FV `OAUTH_PENDING` dù đã upload 8–12/10); `youtube-os/config/` bị xóa sau snapshot | `v2_1/V2.1_SPRINT_24H_REPORT_20261013.md` §1 |
| V1 không bị động trong sprint (verify bằng đọc code) | `v2_1/V2.1_SPRINT_24H_REPORT_20261013.md` §5 |

**Chạy lại (xác minh V1 nguyên vẹn):** so byte từng file V1 với snapshot 10/10 10:54 theo phương pháp ghi trong `V2.1_INITIAL_REPORT_20261012.md` (mục 1). Không có script tự động duy nhất — kiểm toán viên đọc mục đó và chạy lệnh diff tương ứng.

### J. Quyết định của sếp sau sprint (nguồn phê duyệt)

| Quyết định | Bằng chứng |
|---|---|
| 5 quyết định (cron read-only; staging rights_gate/hash_binding; chuẩn bị xin yt-analytics.readonly; CHƯA duyệt 13 cơ hội; API key riêng cho Radar) + yêu cầu bổ sung + nguyên tắc | `v2_1/DECISIONS_20261012_0911.md` (nguyên văn ý sếp) |
| Chỉ đạo sprint 24h + 3 yêu cầu nghiệm thu bổ sung | `v2_1/DIRECTIVE_SPRINT_24H_20261012.md` |
| Chỉ đạo V1→V2.1 gốc | `v2_1/DIRECTIVE_V2.1_20261012.md` |

---

## 2. Thứ tự chạy lại cho kiểm toán độc lập (khuyến nghị)

**Nhóm 1 — không cần API, không cần credential, chạy ngay (xác minh TESTED):**

```bash
cd ~/workspace/youtube-os
python3 v2_1/tests/unit/test_quality_gate_real.py      # C: kỳ vọng 20/20 PASS
python3 v2_1/oversight/tests/test_acceptance_4cases.py  # D: kỳ vọng 14/14 PASS
python3 v2_1/oversight/gates/test_policy.py             # E: kỳ vọng 11/11 PASS
```

**Nhóm 2 — đọc bằng chứng tĩnh (không chạy gì):**

1. Mở `v2_1/oversight/watcher/VERIFICATION_REPORT.json` → đếm 8 entries, kiểm tra `match: true` từng kênh (B).
2. Mở 1–2 file trong `v2_1/state/snapshots/breakout_radar_20261012/` → xác nhận là raw JSON response của YouTube API (có `kind`, `etag`, `pageInfo`), không phải dữ liệu tự bịa (A).
3. Mở `v2_1/state/snapshots/analytics_20261012/snapshot_20261012_T0.json` → đếm video, đối chiếu privacyStatus với báo cáo (F).
4. Mở `v2_1/oversight/audit_log/audit_20261012.jsonl` → 3 dòng JSONL, mỗi dòng có `prev_hash` nối chuỗi (H).
5. Đọc `v2_1/modules/video_quality/TEST_REPORT.md` §4 (giới hạn) trước khi trích dẫn bất kỳ claim "an toàn bản quyền" nào (C).

**Nhóm 3 — chạy thật API (chỉ khi được duyệt, tốn quota):**

```bash
cd ~/workspace/youtube-os
python3 v2_1/modules/breakout_radar/radar.py --dry-run   # A: an toàn, không gọi API
python3 v2_1/oversight/watcher/verify_radar.py           # B: ~1 quota unit
# KHÔNG chạy các lệnh gọi API đầy đủ nếu chưa có phê duyệt cron/quota:
# python3 v2_1/modules/breakout_radar/radar.py
# python3 v2_1/modules/analytics/collector.py
# python3 v2_1/modules/market_intel/supplement_search.py
```

---

## 3. Những gì KHÔNG có trong gói này (trung thực ghi nhận)

- **Velocity Radar thật:** chưa có — cần snapshot thứ 2 (đề xuất re-scan 19/10, chờ duyệt cron). Mọi con số "tăng trưởng" hiện tại đều là suy luận từ 1 snapshot, đã ghi rõ "CHƯA ĐỦ DỮ LIỆU".
- **Analytics 24h/72h/7d:** chưa có — mới điểm T0 (12/10). T1 13/10, T2 15/10, T3 19/10 chờ duyệt.
- **CTR/impressions/retention/watch time:** N/A — chờ scope `yt-analytics.readonly` (xem `v2_1/docs/YT_ANALYTICS_READONLY_SCOPE_DOSSIER.md`).
- **Oversight re-run độc lập test của video_quality:** Wave 0 mới ingest claim 20/20, chưa re-run độc lập (ghi rõ trong `OVERSIGHT_WAVE0_REPORT.md` §2).
- **Audit log neo ngoài VM:** chưa làm — log hiện 1 bản trong VM.
- **Token integrity:** chứng cứ là sha256/mtime ghi trong từng báo cáo module — kiểm toán viên muốn tự xác minh cần đọc báo cáo module tương ứng (không có file hash độc lập tập trung).

---

## 4. Liên hệ tài liệu

- Báo cáo sprint (nguồn mọi tuyên bố): `v2_1/V2.1_SPRINT_24H_REPORT_20261013.md`
- Báo cáo đối chiếu V1 + kiến trúc: `v2_1/V2.1_INITIAL_REPORT_20261012.md`
- Báo cáo oversight: `v2_1/oversight/OVERSIGHT_WAVE0_REPORT.md`
- Quyết định sếp: `v2_1/DECISIONS_20261012_0911.md`
- Dossier scope Analytics: `v2_1/docs/YT_ANALYTICS_READONLY_SCOPE_DOSSIER.md`
- Hướng dẫn API key Radar: `v2_1/docs/API_KEY_RADAR_SETUP_GUIDE.md`
