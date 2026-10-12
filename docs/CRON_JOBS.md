# V2.1 CRON JOBS — Read-only (sếp duyệt 12/10/2026)

## Danh sách job

| Job ID | Lịch | Việc | Quota cap | Trạng thái |
|---|---|---|---|---|
| `v21-radar-rescan-1910` | 1 lần: 19/10/2026 08:00 +07 | Re-scan Radar (snapshot 2) + tính velocity thật vs snapshot 1 (12/10) | 1.500 units, ≤15 search.list | Đã duyệt, đang chờ chạy |
| `v21-analytics-daily` | Hằng ngày 08:00 +07 | Đo video PUBLIC: mốc 24h/72h/7d; phân loại PRIVATE/SCHEDULED/PUBLIC/FAILED | <200 units/lần | Đã duyệt, chạy từ 13/10 |

## Quy tắc chung (ghi trong body từng job)
- CHỈ ĐỌC YouTube Data API. Không upload, không đổi trạng thái video, không động V1.
- CTR/impressions/retention: ghi N/A cho đến khi có scope yt-analytics.readonly. Không bịa số liệu.
- Mọi con số phải có API response làm bằng chứng (lưu raw JSON).
- Log số API calls + units đã dùng vào cuối mỗi báo cáo.

## KILL-SWITCH KHẨN CẤP

Tắt job ngay lập tức (không xóa định nghĩa):

```
cron.update id=v21-radar-rescan-1910 enabled=false
cron.update id=v21-analytics-daily enabled=false
```

Xóa hẳn job:

```
cron.remove id=<job-id>
```

## Nhật ký tắt/mở

| Ngày | Job | Hành động | Lý do | Người thực hiện |
|---|---|---|---|---|
| 12/10/2026 | cả 2 | Tạo + duyệt | Sếp duyệt quyết định #1 | coordinator |
| | | | | |

## Dry-run verification

- `v21-analytics-daily`: đã chạy thử bằng cron.run ngày 12/10 (xem run history).
- `v21-radar-rescan-1910`: KHÔNG dry-run full bằng API (tiết kiệm ~800 quota units + tránh snapshot nhiễu); đã verify radar.py compile OK + snapshot 1 đầy đủ 31 file. Job sẽ chạy thật lần đầu vào 19/10.
