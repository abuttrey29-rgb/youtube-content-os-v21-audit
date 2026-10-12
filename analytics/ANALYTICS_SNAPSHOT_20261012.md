# ANALYTICS SNAPSHOT — Đợt đo đầu tiên (T0)

**Ngày đo:** 12/10/2026 ~08:50 (giờ VN) · **Vòng đo:** 24h / 72h / 7d · **Điểm:** 1/4 (T0)
**Snapshot thô:** `state/snapshots/analytics_20261012/snapshot_20261012_T0.json`
**Collector:** `modules/analytics/collector.py`
**Phương thức:** YouTube Data API v3 `videos.list` (statistics+snippet+status), token Film Verdict chế độ READ-ONLY tuyệt đối (sha256 file trước = sau, đã verify).

---

## 1. Bảng số liệu thật (chỉ video đã PUBLIC)

| Kênh | Video | Đợt | Tuổi (giờ) | Views | Likes | Comments | Views/giờ |
|---|---|---|---|---|---|---|---|
| Film Verdict | `deEOxCRVIjU` — Clayface (2026): Can Horror Save DC's Boldest Bet? | 11/10 | 11.2 | 1 | 0 | 0 | 0.09 |
| Anime Galaxy World | `BwjKllEAngI` — BTTH EP216: Soul Clan Strikes the Gu Clan's Jade | 11/10 | 11.0 | 7 | 0 | 0 | 0.64 |
| Cinema Verdict | `xcCHCHU67GM` — Anton Chigurh: The Villain Who Doesn't Need a Reason | 11/10 | 10.7 | 2 | 0 | 0 | 0.19 |
| Anime Galaxy World | `R8_bR0bRnog` — BTTH EP217: 5 Predictions the Jade War MUST Answer | 12/10 | 2.2 | 2 | 0 | 0 | 0.91 |
| Film Verdict | `DHs2udvFGK8` — Unabomber (2026): The True Story Behind Netflix's 40M | 12/10 | 0.2 | 0 | 0 | 0 | 0.0 |
| Cinema Verdict | `qkcQsBSkErg` — Digger: Why Tom Cruise Can't Save the Day — Ending Explained | 12/10 | 0.2 | 1 | 0 | 0 | 5.0* |

\* *0.2 giờ = 12 phút — con số 5.0 views/giờ ở tuổi này vô nghĩa thống kê, chỉ ghi lại để có điểm T0.*

**Không đo được ở điểm này:**
| Video | Lý do (đã verify) |
|---|---|
| `6p8radWOzwg` (FV, 12/10) | PRIVATE trên chính kênh FV (slot 20:00 VN chưa tới) → sẽ đo khi đã public |
| `tHfgA52i7GY` (AGW, 12/10) | PRIVATE trên kênh AGW — token kênh FV không có quyền thấy (đã verify PRIVATE lúc 06:27 12/10) |
| `8c4e42XrEn4` (CV, 12/10) | PRIVATE trên kênh CV — token kênh FV không có quyền thấy |

**N/A (tuyệt đối không bịa):** CTR, impressions, retention, watch time — chưa được cấp scope `yt-analytics.readonly` (dự kiến xin ở wave sau). Likes/comments = 0 trên cả 6 video → chưa có tín hiệu engagement nào.

---

## 2. So sánh với baseline kênh (views/giờ, tham khảo thô)

| Kênh | Baseline (video long-form PUBLIC gần nhất, ≥180s, loại Shorts) | Trung bình views/giờ | Video mới vs baseline |
|---|---|---|---|
| Film Verdict | n=2: `XEaUOMehxO8` 30v/83.2h (0.36), `aaodTd1ORN8` 1v/38.9h (0.03) | **0.20** | Clayface 0.09 → dưới baseline |
| Anime Galaxy World | n=5: 0.85 / 0.27 / 0.34 / 0.15 / 0.11 | **0.34** | BTTH EP216 0.64 → ~1.9× baseline; BTTH EP217 0.91 → trên baseline (nhưng mới 2.2h) |
| Cinema Verdict | n=0 — kênh mới, uploads playlist chỉ có 2 video (cả 2 đều thuộc đợt đo) | **không có** | chưa có mốc so sánh |

**Lưu ý phương pháp (quan trọng):** baseline views/giờ tính trên toàn bộ tuổi video (gồm cả đuôi chậm sau nhiều ngày), trong khi video mới đang ở cửa sổ giờ đầu — so sánh chỉ là mốc tham khảo thô, không phải apple-to-apple. Baseline AGW loại Shorts theo đúng chỉ đạo (dữ liệu Shorts lịch sử không dùng cho long-form).

---

## 3. Nhận xét trung thực (mẫu nhỏ — không kết luận mạnh)

1. **Tín hiệu duy nhất tương đối ổn định là đợt 11/10 ở tuổi ~11h:** AGW BTTH EP216 (7 views, 0.64 v/h) chạy trên baseline kênh (0.34). FV Clayface (1 view) và CV Chigurh (2 views) còn rất thấp.
2. **Đợt 12/10 quá non để đọc:** 2 video mới public 12 phút (0 và 1 view) — mọi con số lúc này là nhiễu. BTTH EP217 (2.2h, 2 views) tạm trên baseline nhưng 2 giờ tuổi chưa nói được gì.
3. **Engagement = 0 tuyệt đối** (0 like, 0 comment trên 6/6 video): chưa có bằng chứng khán giả tương tác; cần theo dõi ở điểm 24h.
4. **Không suy diễn nguyên nhân.** Không thể biết từ dữ liệu này là do chủ đề, tiêu đề, thumbnail, giờ đăng hay thuật toán — mẫu n=6, thiếu CTR/retention.

---

## 4. Thiết kế vòng learning (chống overfit)

**Lịch chạy:** điểm 2 = +24h (**13/10/2026** ~08:50), điểm 3 = +72h (**15/10/2026**), điểm 4 = +7d (**19/10/2026**). Mỗi điểm chạy `collector.py` → file snapshot mới, không ghi đè. Chưa tạo cron (cấm trong v2_1 khi chưa duyệt) — chạy tay hoặc giao khi có phê duyệt.

**Ngưỡng dữ liệu tối thiểu trước khi điều chỉnh trọng số chọn KEY** (module learning_engine):
1. Không điều chỉnh trọng số nào trước khi một cohort hoàn thành điểm 7d.
2. n ≥ 10 video/cohort (kênh × format) mới được điều chỉnh định hướng; n ≥ 30 mới được thay đổi trọng số >10%.
3. Chênh lệch vs baseline phải vượt dải nhiễu (±2σ của cohort baseline) ở **2 điểm đo liên tiếp**.
4. Giữ 20% video mới làm nhóm control (không áp dụng thay đổi trọng số) để đối chiếu.
5. Mọi giả thuyết phải đăng ký trước vào experiment registry (learning_engine) với tiêu chí thành công viết sẵn — cấm "nhìn số rồi bịa lý do".

**Learning chỉ tác động:** trọng số chọn KEY trong content_intelligence (chọn niche/chủ đề nào). Không dùng để ra quyết định production (lịch đăng, format) khi chưa đủ ngưỡng.

**Dữ liệu còn thiếu cho learning thật:** CTR, retention, watch time, traffic source — cần scope `yt-analytics.readonly` (wave sau). Khi có, bổ sung vào snapshot schema v2.

---

## 5. Quota & toàn vẹn

- API calls: **9** (~9 units: 1 batch videos.list + 3 kênh × (channels.list + playlistItems.list + videos.list) + 1 diagnostic). Token file sha256 trước = sau → không bị đụng.
- Giới hạn đo: token FV chỉ thấy video PUBLIC của kênh khác; để đo đầy đủ điểm sau cần token read-only riêng của AGW và CV (hoặc chờ video public).
