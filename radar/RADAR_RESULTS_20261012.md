# Breakout Radar — Kết quả chạy DỮ LIỆU THẬT (12/10/2026)

**Prototype:** `~/workspace/youtube-os/v2_1/modules/breakout_radar/radar.py`
**Chạy lúc:** 2026-10-12 01:43:53 UTC – 01:44:03 UTC (08:43–08:44 giờ VN)
**Bằng chứng:** mọi API response thô lưu tại `~/workspace/youtube-os/v2_1/state/snapshots/breakout_radar_20261012/` (28 file JSON, có timestamp)

---

## 1. Phương pháp (thực hiện thật)

1. **8 search.list** — `order=date`, `publishedAfter=2026-09-12T00:00:00Z`, `type=video`, `maxResults=25`,
   query: anime donghua review / movie video essay analysis / true crime documentary / easy cooking recipe /
   tech review 2026 / home workout fitness / history documentary / personal finance investing
2. Thu **179 channelId** (dedupe từ 200 kết quả search) → **channels.list** (snippet.publishedAt + statistics), batch 50 id/call → 4 calls.
3. **Lọc tạo kênh 0–30 ngày:** `snippet.publishedAt >= 2026-09-12` → **8 kênh** thỏa mãn.
4. Xếp hạng theo **subs/ngày** = subscriberCount / số ngày tuổi (tuổi tính từ ngày tạo kênh đến 12/10/2026).
5. Top 8: lấy uploads playlist → **playlistItems.list** (15 video mới nhất) → **videos.list** (statistics + duration) để tìm video bùng nổ và tính views/video trung bình.

**Chế độ token:** read-only tuyệt đối. File token chỉ đọc 1 lần lúc khởi động; `get_authenticated_service()` KHÔNG nhận `token_path` → không có auto-persist; refresh (nếu có) chỉ trong memory. Token truyền vào script duy nhất qua env var `BR_TOKEN_PATH` — không copy token vào v2_1. Verify hash trước/sau (mục 4).

---

## 2. Top 8 kênh thật (tạo trong 0–30 ngày)

> **QUY TẮC NGHIỆM THU (sếp chốt 12/10 08:47):** Scoring tách 2 trục — (a) **quy mô thành tích tuyệt đối** (subs, views, video bùng nổ) và (b) **velocity (tốc độ tăng trưởng thực)**. Chỉ có 1 snapshot → velocity = **CHƯA ĐỦ DỮ LIỆU**, cấm nội suy/suy diễn tăng 24h. Cột "subs/ngày" dưới đây là **proxy quy mô** (tổng subs ÷ tuổi kênh), KHÔNG phải velocity đo được.

| # | Kênh (link) | Ngày tạo | Subs | Views tổng | Video | Subs ÷ tuổi kênh (proxy quy mô) | Velocity thực | Niche (query bắt được) | Video bùng nổ + views | Dài/Shorts | Độ tin cậy |

| # | Kênh (link) | Ngày tạo | Subs | Views tổng | Video | Subs/ngày | Niche (query bắt được) | Video bùng nổ + views | Dài/Shorts | Độ tin cậy |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | gatsbygalaxykitty — youtube.com/channel/UCErQaD-Mkc5Eg29NlB6USNQ | 2026-09-29 | 121 | 28.447 | 22 | **10,1** | chưa đủ dữ liệu (1 snapshot) | Mèo/thú cưng (US) | "Send this to someone you'd absolutely bite if you were a cat" — **9.252** | Shorts 11s | Cao — breakout rõ (9.252 vs trung bình 2.026/video), nhưng là Shorts, niche không liên quan chiến lược |
| 2 | Crime Lore — youtube.com/channel/UCABTmppCsgdJd5UFYm6ioYg | 2026-09-19 | 67 | 6.571 | 21 | **3,0** | chưa đủ dữ liệu (1 snapshot) | True crime documentary (nước: không khai báo) | "Jack the Ripper: The Killer Who Was Never Caught | Victims, Evidence & Suspects (Part 1) | True Crime" — **1.983** | **DÀI 1h24m** | Cao — đúng format long-form crime doc; breakout 1.983 vs TB 346/video |
| 3 | Cronicas de Tinta y Libertad — youtube.com/channel/UCq05p1hrReWcl9oXb-7KOmw | 2026-10-06 | 15 | 150 | 2 | **3,0** | chưa đủ dữ liệu (1 snapshot) | History tiếng Tây Ban Nha (VE) | "El Picasso de Hace 36.000 años #documentary #history #spain" — **1.174** | Shorts 33s | Trung bình — mới 5 ngày, chỉ 2 video; VE = nước RPM thấp (tiêu chí sếp: bỏ qua) |
| 4 | theworldhistory — youtube.com/channel/UCFy6QIVt1kuQD2kEW2tsLQw | 2026-10-10 | 3 | 0 | 2 | **3,0** | chưa đủ dữ liệu (1 snapshot) | History (nước: không khai báo) | "England Chapter One" — **225** | Dài 1m35s | Thấp — mới 2 ngày, views tổng kênh = 0 (views chưa cập nhật), số liệu quá mỏng |
| 5 | BOLO W — youtube.com/channel/UCTsP5yOat1VjWZN_V6dY49A | 2026-09-25 | 44 | 23.793 | 71 | **2,8** | chưa đủ dữ liệu (1 snapshot) | History quân sự/WW2 (US) | video hashtag-only "#history #historydocumentary #militaryhistory…" — **425** | Shorts 53s | Trung bình-thấp — 71 video/16 ngày = Shorts farm, title nhồi hashtag, TB chỉ 136 views/video |
| 6 | stuffedcice — youtube.com/channel/UCvS0HuI3-c9PK0xBbqVrSog | 2026-09-12 | 72 | 12.675 | 26 | **2,5** | chưa đủ dữ liệu (1 snapshot) | Cooking (GB) | "Surf and turf meal 🦞🥩#Recipe #cooking #yummy #explore" — **1.465** | Dài 1m25s | Cao — breakout 1.465 vs TB 422/video, niche cooking rõ ràng |
| 7 | Badge and Bloodshed — youtube.com/channel/UCp3btVAXQQ1aFPBcJEjn25Q | 2026-09-19 | 46 | 7.610 | 16 | **2,1** | chưa đủ dữ liệu (1 snapshot) | True crime documentary (US) | "The Terrifying Truth Behind the Clown #truecrimedocumentary #crimedocumentary #7july2005" — **1.257** | Dài 1m3s | Cao — breakout 1.257 vs TB 504/video, đúng niche crime |
| 8 | Dark Chapters — youtube.com/channel/UCyNh1mOCVsQ90_oTffjfsLg | 2026-09-26 | 27 | 5.103 | 16 | **1,8** | chưa đủ dữ liệu (1 snapshot) | History documentary (nước: không khai báo) | "They Were Given 12 Days to Live | Churchill's Secret Army #history #documentary" — **1.843** | Shorts 39s | Trung bình — breakout tốt (1.843 vs TB 489) nhưng là Shorts |

**Theo tiêu chí sếp "CHỈ video dài, loại Shorts":** 4 kênh giữ lại là **Crime Lore, stuffedcice, Badge and Bloodshed, theworldhistory** (video bùng nổ ≥ 61s).

**Quan sát niche:** lần quét này chỉ bắt được kênh mới ở true crime, history, cooking. anime donghua review / movie video essay / tech review / fitness / personal finance KHÔNG cho ra kênh nào tạo trong 30 ngày — không có nghĩa các niche đó không có kênh mới, chỉ là 8 query này không bắt được (xem giới hạn).

**Số liệu video mẫu:** TB views/video tính trên ≤15 video mới nhất (playlistItems), riêng Cronicas/theworldhistory chỉ có 2 video nên mẫu = 2. views_total là `statistics.viewCount` của kênh (tổng mọi thời đại).

---

## 3. Quota đã dùng: **824 / 10.000 units**

| API | Calls | Units/call | Units |
|---|---|---|---|
| search.list | 8 | 100 | 800 |
| channels.list | 8 (4 lần chạy chính + 4 lần chạy lại để bổ sung raw snapshot — xem ghi chú) | 1 | 8 |
| playlistItems.list | 8 | 1 | 8 |
| videos.list | 8 | 1 | 8 |
| **Tổng** | 32 | | **824** |

**Ghi chú trung thực:** lần chạy chính lưu thiếu 2/4 raw response `channels.list` do 2 batch chạy cùng 1 giây → tên file trùng (timestamp chính xác đến giây) → ghi đè. ĐÃ SỬA: `save_snapshot()` thêm sequence counter chống trùng, và chạy lại 4 batch channels (tốn thêm 4 units) để đủ bằng chứng. Không ảnh hưởng kết quả phân tích (dữ liệu trong memory đầy đủ từ lần đầu).

Search bucket riêng: 8/100 calls ngày. Còn dư ~9.180 units.

---

## 4. Bằng chứng token KHÔNG bị đụng (read-only tuyệt đối)

| | sha256 (8 ký tự đầu) | size | mtime UTC |
|---|---|---|---|
| Trước khi chạy | `bff8e326` | 793 bytes | 2026-10-09T12:47:02.798188+00:00 |
| Sau khi chạy | `bff8e326` | 793 bytes | 2026-10-09T12:47:02.798188+00:00 |

Identical — file không hề bị ghi. Token chỉ đọc qua `BR_TOKEN_PATH` env var, không copy vào v2_1.

---

## 5. Giới hạn & trung thực (những gì CHƯA đo được)

1. **Chỉ 1 snapshot thời điểm** — "subs/ngày" là ƯỚC TÍNH (subscriberCount hiện tại ÷ số ngày tuổi), KHÔNG phải tốc độ tăng trưởng đo từ 2 điểm thời gian. Kênh 2 ngày tuổi có 3 subs cũng ra 3,0/ngày — con số này nhiễu. Muốn đo thật phải re-scan sau 24h/7d.
2. **Tuổi kênh = ngày tạo kênh** (`snippet.publishedAt`), phân biệt rõ với ngày đăng video đầu — đúng yêu cầu. Nhưng không xác minh được kênh có từng đổi tên/xóa video cũ không.
3. **Subscriber count** hiển thị công khai cho cả 8 kênh (không kênh nào ẩn) — nếu ẩn, cột subs sẽ ghi "hidden".
4. **Bias của search:** `order=date` ưu tiên video mới nhất, không phải video phổ biến nhất; kết quả phụ thuộc query. 8 query là mẫu nhỏ, không phủ hết niche/ngôn ngữ.
5. **Breakout video** định nghĩa: video có views ≥ 3× trung vị của mẫu (hoặc cao nhất mẫu nếu không có). Ngưỡng này là heuristic của prototype, không phải chuẩn khoa học.
6. **Không đo CTR/retention/watch time** — cần scope `yt-analytics.readonly` chưa được cấp (đúng giới hạn đã biết của V1).
7. Tất cả con số truy được về file snapshot trong `state/snapshots/breakout_radar_20261012/` (01_search_* → 02_channels_batch → 03_playlistitems_* → 04_videos_* → 05_analysis).


## 6. Thiết kế velocity (áp dụng khi có ≥2 snapshots)

Theo yêu cầu nghiệm thu của sếp, velocity CHỈ được tính khi có ≥2 snapshots của cùng kênh, cách nhau đủ xa (tối thiểu 24h, khuyến nghị 7d):

- `velocity_subs = (subs_t2 - subs_t1) / (t2 - t1)` (subs/ngày, khoảng [t1, t2] ghi rõ)
- `velocity_views = (views_t2 - views_t1) / (t2 - t1)` (views/ngày)
- Độ tin cậy: khoảng cách càng xa, mẫu video càng nhiều → tin cậy càng cao. Ghi rõ khoảng thời gian và n (số điểm đo) kèm mọi con số velocity.
- Cấm: nội suy tuyến tính từ 1 điểm, suy diễn "tăng X views/24h" khi chưa đo.

Snapshot hiện tại (12/10 08:44 VN) là điểm T0. Điểm T1 đề xuất: re-scan sau 7 ngày (19/10) — lúc đó mới có velocity thực đầu tiên.

## 7. Cách chạy lại

```bash
~/workspace/youtube_api/venv/bin/python ~/workspace/youtube-os/v2_1/modules/breakout_radar/radar.py --dry-run   # in kế hoạch quota
~/workspace/youtube_api/venv/bin/python ~/workspace/youtube-os/v2_1/modules/breakout_radar/radar.py --top 8        # chạy thật
```
Token path qua env `BR_TOKEN_PATH` (mặc định `~/workspace/youtube_api/.secrets/tokens_film_verdict.json`).
