# MARKET + COMPETITOR INTELLIGENCE — Bảng xếp hạng cơ hội (12/10/2026)

**Sprint 24h V2.1 · Deadline 13/10 08:40 · Bàn giao cho parent orchestrator**
**Phạm vi:** 7 ngày tới (13–19/10/2026), 3 kênh AGW / FV / CV — 2 video/ngày/kênh

---

## 0. Nguồn dữ liệu (tất cả DỮ LIỆU THẬT)

| Nguồn | Thời gian | Nội dung |
|---|---|---|
| Breakout Radar | 12/10 08:43–08:44 VN | 8 search + 179 kênh sàng lọc → 8 kênh 0–30 ngày; 4 giữ lại sau tiêu chí CHỈ video dài: **Crime Lore, Badge and Bloodshed, stuffedcice, theworldhistory** |
| Search bổ sung (mới) | 12/10 ~08:47 VN | **6 search.list**: anime analysis / video essay film / donghua explained / character analysis / ending explained / movie documentary → 138 kênh → 10 kênh 0–30 ngày |
| Đối thủ V1 (chỉ đọc) | 09/10 | `competitors/snapshots/2026-10-09.json`: 250 video mới nhất của 10 kênh film-review (The Critical Drinker, New Rockstars, Jeremy Jahns, Screen Junkies, RedLetterMedia, Mr Sunday Movies, Chris Stuckmann, CinemaSins, YMS, Beyond The Trailer) |

**Quota:** Radar 824 + bổ sung 623 = **1.447 / 10.000 units**; search bucket **14/100** (yêu cầu giữ <20 ✓).
**Token:** read-only tuyệt đối — `sha256 bff8e326…` trước = sau, size 793→793, mtime không đổi. Raw JSON: `state/snapshots/market_intel_20261012/` (30 file).

---

## 1. Kết quả search bổ sung (tóm tắt trung thực)

| Kênh | Tuổi | Subs/ngày | Video bùng nổ | Dài | Nhận xét |
|---|---|---|---|---|---|
| Bangladesh Rewind (BD) | 16d | 28,3 | VCR-nostalgia 53.585 views | Shorts 24s | Mạnh nhưng Shorts + nước RPM thấp → **loại theo tiêu chí sếp**; giữ lại insight format "nostalgia-documentary" |
| entropy.enigma | 16d | 14,9 | "Why did JD Vance give up his black eyeliner" 1.012 | 1m09s | Format **câu hỏi tò mò nhân vật** — chuyển được sang CV/AGW |
| No Signal After Four (ES) | 21d | 3,4 | Sopranos micro-essay 30.141 | Shorts 33s | **Video essay về phim bùng nổ trên Shorts** — chứng minh demand, nhưng format Shorts không áp dụng cho 3 kênh |
| CineRush (US) | 24d | 0,8 | "The Joker's Pencil Trick Was Smarter Than You Think 🤯" 1.398 | Shorts 11s | Title pattern tò mò — tham khảo cách đặt title |
| Watch This Week (IN) | 4d | 1,5 | Netflix release-date news 1.851 | 1m20s | Format tin tức ngày phát hành — FV có thể làm bản essay sâu hơn |
| Ossan Anime Explainer (IN) | 22d | 2,1 | hashtag farm 231 | Shorts | Loại — farm |
| Còn lại | — | <1 | hashtag farm / mỏng | — | Loại |

**Kết luận trung thực:** query anime/donghua VẪN không bắt được kênh breakout long-form nào trong 30 ngày. Đây là **giới hạn mẫu query + độ trễ của kênh mới**, không phải bằng chứng niche anime chết. Đề xuất cho AGW vì vậy dựa chủ yếu vào (a) logic mùa vụ + (b) format-transfer từ kênh breakout + (c) curiosity-signal mỏng — **độ tin cậy đánh thấp/trung bình và ghi rõ lý do ở từng mục**.

---

## 2. Logic chấm điểm (MINH BẠCH — không blackbox)

Mỗi cơ hội chấm 5 yếu tố, thang 0–10, rồi tính:

> **SCORE = 10 × (0,35·D + 0,15·R + 0,20·G + 0,15·F + 0,15·E)** → thang 0–100

| Yếu tố | Trọng số | Định nghĩa & cách chấm | Nguồn số liệu |
|---|---|---|---|
| **D — Demand** | 35% | Cầu thật đã đo: views/ngày (V1) hoặc breakout multiplier (views video bùng nổ ÷ median kênh). ≥100k/ngày hoặc ≥20× → 9–10; 30–100k → 7–8; 10–30k → 6; 1–10k hoặc 3–5× → 4–5; mỏng/không có → 2–3 | insights.json 09/10; Radar + search bổ sung |
| **R — Recency** | 15% | Độ tươi của trend. Xuất hiện ≤3 ngày → 9–10; ≤7 ngày → 7; ≤14 ngày → 5; >14 ngày → 3 | first_seen_days_ago (V1); publishedAt (Radar) |
| **G — Competition gap** | 20% | Khoảng trống cho KÊNH CỦA SẾP: ít kênh cùng góc khai thác → cao; bão hòa → thấp. Kênh breakout 0–30 ngày và kênh triệu-sub là 2 tầng khác nhau — FV/CV/AGW cạnh tranh ở tầng commentary/essay, không đối đầu trực tiếp | phân tích thủ công, ghi rõ |
| **F — Format fit** | 15% | Mức độ "native" với chỉ đạo: AGW long-form tuyệt đối (không Shorts), FV essay/bình luận chuyên sâu, CV nhân vật/chủ đề/ending explained; transformative, chống quét AI | chỉ đạo 10–11/10 |
| **E — Execution** | 15% | Làm được trong nhịp 2 video/ngày: footage hợp lệ sẵn (trailer/tin tức/official), rủi ro bản quyền kiểm soát được, đạt chuẩn chất lượng | đánh giá thủ công |

Điểm ≥75 = ưu tiên cao; 65–74 = tốt; <65 = thử nghiệm có kiểm soát. **Điểm KHÔNG phải dự báo view** — chỉ là thứ tự ưu tiên sản xuất.

---

## 3. Bảng xếp hạng cơ hội (13–19/10)

### 3.1. Film Verdict — bình luận/đánh giá phim chuyên sâu

| # | Cơ hội (KEY) | D | R | G | F | E | **SCORE** | Độ tin cậy | Vì sao |
|---|---|---|---|---|---|---|---|---|---|
| FV-1 | **Star Wars: sequel trilogy discourse** ("ANOTHER Sequel Trilogy!?") | 9 (378k views/ngày, Critical Drinker) | 6 (6,3d) | 7 (1 kênh, góc commentary mở) | 9 | 9 (footage tin tức dồi dào) | **82** | **Cao** — 250-video dataset V1, pattern controversy_split 40k/ngày |
| FV-2 | **Avengers Doomsday — bình luận/dự đoán** (không làm breakdown frame-by-frame như New Rockstars) | 9 (389k/ngày) | 9 (1,2d) | 5 (New Rockstars thống trị breakdown; FV phân biệt bằng essay dự đoán) | 8 | 8 | **79** | **Cao** — emerging mạnh nhất V1; rủi ro: cạnh tranh trực tiếp ở mảng tin nhanh |
| FV-3 | **NYCC 2026 Marvel panel — recap + bình luận** | 8 (262k/ngày) | 10 (0,5d) | 6 | 7 | 8 | **78** | **Cao** — tin 12 giờ tuổi, tốc độ là lợi thế; hết hạn sau 48–72h |
| FV-4 | **Horror mùa Halloween: Texas Chain Saw Massacre / Jeepers Creepers** (re:View / review) | 8 (RLM 1,06M views; Stuckmann 19,1× median) | 6 (7–9d) | 7 | 9 | 8 | **77** | **Cao** — tailwind Halloween (31/10); FV đã làm Flanagan/Exorcist 12/10 nên chọn phim KHÁC để tránh trùng |
| FV-5 | **Mike Flanagan's Carrie (series mới)** | 7 (28,3k/ngày) | 7 | 8 | 9 | 8 | **77** | **Trung bình-cao** — ⚠️ FV vừa đăng Flanagan/Exorcist hôm nay: cách nhau ≥3 ngày hoặc gộp thành series "Flanagan universe" để tránh mệt mỏi khán giả |
| FV-6 | **Resident Evil — reboot Zach Cregger (essay/bình luận)** | 8 (30,9k/ngày, 6 kênh = strong key) | 5 (16,4d) | 6 (bão hòa ở tầng review tiếng Anh; khoảng trống ở essay sâu tiếng Việt + góc phân tích) | 9 | 8 | **73** | **Cao** — key mạnh duy nhất được 6/10 kênh cùng làm; R thấp vì đã 16 ngày nhưng reboot còn nóng đến khi ra mắt |

### 3.2. Cinema Verdict — nhân vật / chủ đề / ending explained

| # | Cơ hội (KEY) | D | R | G | F | E | **SCORE** | Độ tin cậy | Vì sao |
|---|---|---|---|---|---|---|---|---|---|
| CV-1 | **Lanterns (HBO) — ending explained / breakdown tập** | 8 (192k/ngày, 3 video) | 9 (2,2d) | 6 (New Rockstars làm tiếng Anh; khoảng trống bản essay sâu tiếng Việt) | 10 (native CV) | 8 | **81** | **Cao** — đúng format CV nhất trong mọi key đang nóng |
| CV-2 | **VisionQuest / Vision MCU — ending explained** | 7 (149k/ngày) | 8 (3,1d) | 7 | 10 | 8 | **78** | **Cao** — series sắp chiếu, làm sớm = đón sóng |
| CV-3 | **Format "fascinating disaster": giải phẫu bom xịt qua nhân vật/chủ đề** (học từ "Digger - A Fascinating Disaster" 491k/ngày của Critical Drinker; CV đã làm Digger ending → làm phim KHÁC) | 7 (35,8k/ngày "tom cruise digger flop" + hiệu ứng Drinker) | 7 | 7 | 8 | 8 | **73** | **Trung bình** — format đã được chứng minh demand; cần chọn đúng "nạn nhân" tiếp theo, không làm lại Digger |
| CV-4 | **Character analysis dạng câu hỏi tò mò** ("Why did [nhân vật]...?" — format từ entropy.enigma) áp cho nhân vật phim đang hot | 3 (kênh mẫu nhỏ) | 5 | 9 (gần như chưa ai làm deep character analysis tiếng Việt) | 10 | 8 | **63** | **Trung bình-thấp** — D thấp vì dữ liệu mỏng; điểm tựa là G+F: khoảng trống thật + format native |

### 3.3. Anime Galaxy World — phân tích anime/donghua LONG-FORM (không Shorts)

| # | Cơ hội (KEY) | D | R | G | F | E | **SCORE** | Độ tin cậy | Vì sao |
|---|---|---|---|---|---|---|---|---|---|
| AGW-1 | **Demon Slayer: Infinity Castle — phân tích sâu** (film 2026 đang chiếu/nóng) | 4 (không có số liệu trực tiếp; suy từ độ nóng franchise) | 8 (mùa phim) | 8 | 9 | 7 (⚠️ bản quyền: chỉ transformative, không reupload — học từ baseline Yuewen) | **66** | **Trung bình** — không có breakout data trực tiếp; điểm tựa là mùa vụ + G |
| AGW-2 | **Anime character question-hook** ("Vì sao [nhân vật]...?") — format từ entropy.enigma ("Why did Robin's eyes turn gold in live-action One Piece") làm bản long-form sâu | 3 (tín hiệu mỏng, kênh mẫu nhỏ) | 7 | 8 | 8 | 7 | **60** | **Trung bình-thấp** — format-transfer, chưa có bằng chứng long-form |
| AGW-3 | **Mùa anime Thu 2026 — phân tích/giới thiệu có chiều sâu** | 2 (không dữ liệu) | 8 | 8 | 9 | 7 | **59** | **Thấp** — logic mùa vụ thuần túy; cần validate bằng số liệu tuần sau |

> **Trung thực về AGW:** cả 14 query (8 Radar + 6 bổ sung) KHÔNG cho ra kênh anime/donghua long-form breakout nào trong 30 ngày. 3 đề xuất trên là **suy luận từ mùa vụ + format-transfer**, không phải kết luận từ dữ liệu. Không nên đọc điểm số AGW như "dự báo" — đó chỉ là thứ tự thử nghiệm. Radar nên chạy lại với query tiếng Nhật/Trung + `order=viewCount` để kiểm chứng.

---

## 4. Format-transfer: bài học dùng được từ 4 kênh breakout (nguyên bản, không sao chép)

1. **Serialization "(Part 1)"** — Crime Lore: Jack the Ripper Part 1 (1.983 views, 1h24m) > Part 2 (991). *Áp dụng:* CV có thể tách ending-explained phim phức tạp thành 2 phần; AGW tách phân tích dài kỳ. Không copy title, chỉ mượn cấu trúc.
2. **Question-hook titles** — V1: pattern `question_hook` đạt 48k views/ngày trung bình (14 video, 6 kênh), cao nhất trong các pattern có chủ đích. Crime Lore: "Who Was the Zodiac Killer? The Evidence That Still Doesn't Add Up". *Áp dụng:* FV/CV đặt title dạng câu hỏi thật, không clickbait rỗng.
3. **Curiosity micro-detail** — CineRush: "The Joker's Pencil Trick Was Smarter Than You Think 🤯"; entropy.enigma: "Why did Robin's eyes turn gold in the live action One Piece". *Áp dụng:* 1 chi tiết nhỏ → 1 video essay, thay vì ôm cả bộ phim.
4. **Keyword-stacking có chủ đích trong title** — Crime Lore nhồi "True Crime Documentary" nhưng vẫn giữ title đọc như người viết. *Áp dụng có chừng mực:* 1 cụm niche-keyword ở cuối title, không phá vỡ nguyên tắc "như người thật làm".
5. **Tốc độ với tin nóng** — NYCC panel 0,5 ngày tuổi → 262k views/ngày. *Áp dụng:* FV giữ 1 slot/ngày linh hoạt cho tin nóng 24–48h (thay vì khóa cứng cả 14 slot từ đầu tuần).
6. **Không học:** hashtag-farm (BOLO W: 71 video/16 ngày, TB 136 views/video), Shorts farm, kênh RPM thấp (Bangladesh Rewind dù mạnh vẫn loại).

---

## 5. Đề xuất hành động 7 ngày (13–19/10) — CỤ THỂ từng kênh

> Đã trừ các chủ đề **đã sản xuất 12/10**: Digger (CV), Social Reckoning (CV), Flanagan/Exorcist (FV), Unabomber (FV), BTTH EP217 + Link Click S3 EP11 (AGW). Không lặp lại trong 7 ngày trừ khi là Part 2 có lý do.

### Film Verdict (slot 13:00 & 20:00)

| Ngày | 13:00 | 20:00 |
|---|---|---|
| 13/10 | **FV-6: Resident Evil reboot** — essay "Zach Cregger có cứu được Resident Evil?" (góc phân tích, không review cạn) | **FV-4: Texas Chain Saw Massacre** — re:View/horror essay đón Halloween |
| 14/10 | **FV-2: Avengers Doomsday** — video dự đoán/bình luận (tránh đấu breakdown với New Rockstars) | **FV-1: Star Wars sequel trilogy discourse** — opinion essay (controversy_split, học Critical Drinker nhưng giọng FV) |
| 15/10 | **FV-4: Jeepers Creepers** — review + phân tích | Slot linh hoạt: **tin nóng 24–48h** (nếu không có tin → FV-6 part 2: Resident Evil deep-dive) |
| 16/10 | **FV-3: NYCC Marvel panel** — chỉ làm nếu còn nóng (kiểm tra lại views trước khi sản xuất; tin nguội thì thay bằng FV-1/FV-2) | **FV-5: Mike Flanagan's Carrie** — series mới, tách ≥3 ngày khỏi video Exorcist 12/10 |
| 17–19/10 | Lặp chu kỳ theo số liệu 24–72h: key nào trong 4 ngày đầu có velocity tốt → làm video follow-up (part 2 / góc mới) | Slot linh hoạt tin nóng |

### Cinema Verdict (slot 14:00 & 21:00)

| Ngày | 14:00 | 21:00 |
|---|---|---|
| 13/10 | **CV-1: Lanterns** — ending explained tập mới nhất (format native, demand 192k/ngày) | **CV-4: Character analysis câu hỏi tò mò** — nhân vật phim hot tuần ("Vì sao [X]...?") |
| 14/10 | **CV-2: VisionQuest** — ending explained / everything you need to know (đón sóng trước giờ chiếu) | **CV-3: "Fascinating disaster"** — giải phẫu 1 bom xịt (KHÔNG phải Digger — chọn phim khác, gợi ý: phim flop gần đây có nhân vật đáng mổ xẻ) |
| 15–19/10 | Xoay vòng CV-1/CV-2 theo tập phim mới ra; CV-4 mỗi 2–3 ngày 1 video (format mới cần test) | Slot linh hoạt: ending explained phim mới ra rạp/tin nóng |

### Anime Galaxy World (slot 12:00 & 19:00)

| Ngày | 12:00 | 19:00 |
|---|---|---|
| 13/10 | **AGW-1: Demon Slayer Infinity Castle** — phân tích sâu (transformative tuyệt đối, không reupload footage dài) | **AGW-2: Character question-hook** — "Vì sao [nhân vật anime]...?" bản long-form |
| 14–19/10 | Tiếp tục AGW-1 (series phân tích theo arc/cảnh) + **AGW-3: mùa anime Thu 2026** (1–2 video/tuần, test demand) | AGW-2 xen kẽ; giữ 1 slot linh hoạt cho tin anime nóng |

**Lưu ý bản quyền AGW:** video 12/10 lần đầu dùng footage Bilibili — theo dõi claim 48h trước khi lặp lại nguồn footage này. Mọi video AGW: phân tích/bình luận transformative, footage ngắn có bình luận đè.

---

## 6. Giới hạn & việc cần làm tiếp (trung thực)

1. **V1 competitor data đã 3 ngày tuổi (09/10)** — key nóng có thể đã nguội (đặc biệt NYCC). Trước khi sản xuất FV-3, re-check velocity bằng 1 search.list.
2. **Radar đo subs/ngày từ 1 snapshot** — ước tính, không phải tốc độ thật. Cần re-scan 24h/7d để đo thật (đã có script).
3. **Anime gap chưa lấp được** — đề xuất sprint tiếp: query tiếng Nhật/Trung ("アニメ 考察", "动漫 解析"), `order=viewCount` thay vì `order=date`, và quét kênh 30–90 ngày (không chỉ 0–30).
4. **Không có CTR/retention** — thiếu scope yt-analytics.readonly; mọi "demand" ở đây là views công khai, không phải sức hút thật.
5. **Điểm số là thứ tự ưu tiên, không phải dự báo view.** Đặc biệt với AGW: 59–66 điểm = "thử nghiệm có kiểm soát", không phải "cơ hội đã xác thực".

---

## 7. Bằng chứng & file

- Raw search bổ sung (6 query): `~/workspace/youtube-os/v2_1/state/snapshots/market_intel_20261012/01_search_*` (+ `02_channels_batch_*`, `03_playlistitems_*`, `04_videos_*`, `05_analysis_*`) — 30 file JSON
- Script chạy lại: `~/workspace/youtube-os/v2_1/modules/market_intel/supplement_search.py` (`--dry-run` để xem quota trước)
- Kết quả Radar: `~/workspace/youtube-os/v2_1/modules/breakout_radar/RADAR_RESULTS_20261012.md`
- Đối thủ V1: `~/workspace/youtube-os/competitors/snapshots/2026-10-09.json`, `insights.json`

*Báo cáo này xong lúc ~08:55 12/10/2026. Người nhận tiếp theo: parent orchestrator → coordinator sản xuất 6 video/ngày.*
