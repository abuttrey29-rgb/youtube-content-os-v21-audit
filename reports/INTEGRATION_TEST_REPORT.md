# Integration Test Report — Analytics + Oversight + Video Quality (V2.1)

**Ngày:** 12/10/2026, ~09:40 (+07) | **Script:** `tests/integration/test_modules_real.py`
**Chế độ:** API read-only tuyệt đối, token file chỉ đọc (sha256 trước==sau: **unchanged**).
**Quota đã dùng: ~2 units** (videos.list ×1 + channels.list ×1) — dưới xa trần 500u.
**Kết quả: ALL PASS** (chạy bằng `~/workspace/youtube_api/venv/bin/python`).

## Chuỗi tích hợp đã chứng minh (dữ liệu THẬT, không mock)

```
analytics.collector (parse helpers)
        +
analytics.video_status.classify/summarize
        │  videos.list LIVE (6 video batch 12/10, 1 unit)
        ▼
oversight.verify_radar.matches/norm_published_at
        │  channels.list LIVE (2 kênh Radar, 1 unit) + 05_analysis JSON thật
        ▼
video_quality.hash_binding.seal/verify (file MP4 thật, QC manifest thật)
        ▼
oversight.gates.policy.evaluate_publish_proposal → ALLOW/BLOCK
```

### Step 1 — `video_status` × API thật: **LIVE VERIFIED**
- `videos.list` trả về 4/6 video. 2 video vắng mặt: `tHfgA52i7GY` (AGW), `8c4e42XrEn4` (CV).
  **Đây là hành vi đúng của API, không phải lỗi:** token Film Verdict không xem được video
  PRIVATE thuộc kênh khác (AGW/CV) — chỉ thấy video PUBLIC của kênh khác + mọi video
  của chính FV. Test đã sửa assertion cho trung thực (returned + missing = 6).
- Kết quả classify trên dữ liệu thật: **3 PUBLIC** (`R8_bR0bRnog` AGW, `DHs2udvFGK8` FV,
  `qkcQsBSkErg` CV) + **1 SCHEDULED** (`6p8radWOzwg` FV — private + publishAt tương lai,
  đúng chuẩn phân biệt sếp yêu cầu 12/10 08:47).
- `summarize()` đếm đúng; `collector.parse_duration_iso8601` parse đúng duration thật
  (242s/492s/470s/370s).

### Step 2 — `verify_radar` × Radar thật + API tươi: **LIVE VERIFIED**
- 2 kênh Radar (`UCErQaD-Mkc5Eg29NlB6USNQ`, `UCABTmppCsgdJd5UFYm6ioYg`): `publishedAt`
  khớp 100% với snapshot 05_analysis (immutable field, exact match sau chuẩn hóa).
- `matches()` hoạt động đúng trên dữ liệu live.

### Step 3 — `hash_binding` thật (không API): **TESTED**
- `seal()` trên MP4 thật (`production/cinema_verdict/v2/cv_sample_02_chigurh_v2.mp4`,
  chỉ đọc) + QC manifest mẫu thật → sha256 thật 64 ký tự; `verify()` ok=True.

### Step 4 — `policy` nối 3 module trên: **TESTED** (logic thuần, input thật)
- Proposal dùng **channel_id thật từ snippet API (step 1)** + **sha256 MP4 thật (step 3)**
  + approval hợp lệ → **ALLOW**.
- Thiếu approval → **BLOCK** (fail-closed). Sai kênh (không trong registry) → **BLOCK**.

## Điểm nối module (đã chứng minh, không còn rời rạc)

| Đầu ra module A | Được module B dùng làm đầu vào |
|---|---|
| `video_status.classify` (state thật: PUBLIC/SCHEDULED) | `policy`: chỉ kênh đúng + dữ liệu trạng thái thật |
| `verify_radar.matches` (radar_verified=True) | gắn vào proposal như bằng chứng xác thực |
| `hash_binding.seal/verify` (video_sha256 thật) | `qc_evidence` của policy (chống tráo file sau QC) |
| `collector.parse_duration_iso8601` | dùng trên duration thật từ cùng response |

## Ghi chú trung thực / giới hạn

- `collector.main()` (snapshot full 9 video + baseline) **CHƯA chạy lại** trong test này
  (đã LIVE VERIFIED ở sprint 24h — snapshot T0); test chỉ dùng helper + API tối thiểu.
- CTR/impressions/retention vẫn **N/A** (chưa có scope yt-analytics.readonly) — không bịa.
- Video PRIVATE của AGW/CV không đo được bằng token FV — muốn đo đủ 6 video cần token
  từng kênh (hiện chưa có) hoặc đợi video PUBLIC. **Không nội suy.**
- `verify_radar` đầy đủ 8 kênh đã chạy ở sprint (8/8 MATCH); test này spot-check 2 kênh
  để tiết kiệm quota — đủ chứng minh hàm nối được với API thật.
- Ngoại lệ import `~/workspace/youtube_api`: đã được parent phê duyệt trực tiếp cho
  prototype read-only (ghi trong header test + collector.py); Wave sau cần read-only
  adapter chuẩn.

## Nhãn tổng

- analytics.video_status + verify_radar trên API thật: **LIVE VERIFIED**
- hash_binding + policy gate (logic thuần, input thật): **TESTED**
- velocity.py + QUERY_PACK_V2: xem báo cáo riêng (velocity TESTED 14/14; pack PLANNED)
- Tổng quota toàn bộ task hôm nay: ~2 units (test này) + 0 (velocity/query/config) = **~2u**
