# KIỂM TOÁN ĐỘC LẬP — YOUTUBE CONTENT OS V1
**Ngày đánh giá:** 10/10/2026  | **Đối tượng:** gói `YOUTUBE_CONTENT_OS_FULL_BACKUP_20261010.zip` (snapshot 10:54, giờ VN)  
**Phạm vi:** chỉ YouTube Content OS, trọng tâm Film Verdict, Anime Galaxy World và Cinema Verdict. Không kiểm toán/đụng các dự án audio, Bitcoin, outreach.  
**Hình thức:** kiểm tra trực tiếp archive, đối chiếu manifest, đọc source code, kiểm thử độc lập một phần ở môi trường tách biệt, đối chiếu tài liệu yêu cầu gốc và bản PDF UPDATED. **KHÔNG** có truy cập Google Cloud/YouTube với token thật, không tác động production và không tự động upload/public.  
**Quan trọng:** Những thay đổi Muse làm **sau 10:54 ngày 10/10**, nhất là `auto_publish/`, video PUBLIC và Google Branding mới, **không nằm trong snapshot V1**; không coi chúng là đã nghiệm thu thông qua việc kiểm toán ZIP.

## A. Kết luận điều hành

**V1 đạt yêu cầu làm nền tảng dữ liệu và dashboard READ-ONLY cho 3 kênh, nhưng CHƯA đạt điều kiện vận hành tự động A–Z không giám sát, chưa đủ bằng chứng mở rộng 50 kênh.** Không cần xây lại toàn bộ. Ưu tiên sửa an toàn xuất bản, chất lượng sản xuất/QC và vòng dữ liệu học từ hiệu quả thực trước khi tăng số lượng video.

Ba ưu thế thực tế: (1) có phân tách các chức năng tương đối rõ; (2) lớp YouTube API có Channel Identity Gate `channels.list(mine=true)`, checkpoint, file lock, `pending_verification`; (3) có dữ liệu snapshot, dashboard và công cụ nghiên cứu/SEO/QC/learning ở mức code.  
Ba thiếu hụt lớn nhất: (1) **QC hiện chưa chứng minh quyền sử dụng footage và có đường đi PASS khi thiếu bằng chứng**; (2) **legacy scheduler có đường xuất bản chưa ràng buộc kế hoạch/phê duyệt từng video**, và nhánh upload bất định có nguy cơ retry sai; (3) **mô hình sản xuất chuyển động còn dựa nhiều vào ảnh Ken Burns/motion text**, đối lập tiêu chuẩn nội dung người dùng đã chốt.

### Mức độ bằng chứng

- **Xác minh độc lập từ ZIP:** 815.622.883 byte, SHA-256 `53096b6255b58dc15a050537d2b90f187aeda760238527b173baed0c72cbf71a`; 1.108 mục ZIP; gồm **957 file + 151 thư mục**. ZIP kiểm tra CRC PASS. `MANIFEST.md` liệt kê **956/956 file** không tính manifest, tất cả SHA-256 **khớp**, không file thiếu/thừa.
- **Phân tích cú pháp:** 90 file `.py` thuộc các nguồn được trích xuất trong gói (bao gồm một số bản giao trùng), ~20.199 dòng; 90/90 parse Python AST PASS. Điều này chỉ chứng minh cú pháp, không chứng minh tất cả module chạy đúng.
- **Rebuild offline:** đã tạo lại HTML Health Monitor, Competitor Intelligence, AI Recommendations, Multi-channel từ bản `v1_handover/` và dữ liệu backup, **không dùng OAuth / API live**. Một bước `analyzer.py` ban đầu thiếu `health_data.json` trong thư mục staging tạm, sau khi bổ sung bản dữ liệu trong ZIP thì chạy PASS. Kết luận: **pipeline HTML có thể tái tạo trong môi trường dựng sẵn đường dẫn**, chưa xác nhận khôi phục toàn hệ thống ở máy mới.
- **Kiểm thử QC tối thiểu:** gọi `_check_footage()` khi không có manifest trailer/footage: kết quả **`footage_policy: PASS` với thông điệp “no trailer footage used”**. Đây là điểm yếu logic cần sửa; không chứng minh video có quyền sử dụng tư liệu.
- **Chưa kiểm thử:** OAuth thật, scope, status YouTube, API quota live, upload/private/public trực tiếp, Cron Muse đang chạy, thử tải quy mô 50 kênh, restore đủ secrets, chất lượng 3 MP4 sau backup, lớp `auto_publish/` mới Muse báo 64/64 test.

## B. Đánh giá module theo mục tiêu 9 khối

| Khối / module | Đã thấy trong source | Điểm mạnh | Hạn chế / thiếu bằng chứng | Đánh giá |
|---|---|---|---|---|
| **0. Channel Control Plane** (`channels/channel_status.py`, `youtube_api/config.py`, registry) | Có registry, `api_status`, chính sách chặn kênh không sẵn sàng, cấu hình theo kênh | Phân biệt 3 kênh bằng ID, fail-closed trong một số luồng | Chưa có profile/DNA hoàn chỉnh theo `channel_id` và cơ chế nhất quán toàn pipeline; còn 6 kênh lịch sử trong registry | **Một phần** |
| **1a. Competitor Intelligence** (`competitors/competitors.py`, builder V1) | API scan, snapshot, title/topic classification, so sánh nhóm | Dùng dữ liệu bên ngoài, có chấm điểm/nhật ký | Đối thủ/V1 dashboard AGW chủ yếu Roblox Shorts, không đúng định hướng AGW long-form anime đã chốt; score dễ sai khi lịch sử ít | **Có code, cần hiệu chỉnh** |
| **1b. Key Trends** (`key_trends/key_trends.py`) | Trending theo khu vực, đề xuất, pytrends best-effort, snapshots | Có logic phát hiện tín hiệu mới | Trending rộng chưa tương đương nhu cầu niche; Google Trends có thể lỗi; xác minh tín hiệu cạnh tranh/nhu cầu vẫn hạn chế | **Có code, chưa nghiệm thu dự báo** |
| **1c. Content Calendar** (`calendar/content_calendar.py`) | Pool, score, preserve slot non-idea, weights, 14 ngày | Có pipeline tính điểm chứ không chỉ văn bản | Logic `FilmCandidate`, film-specific; chưa chứng minh mở rộng AGW/CV; thiếu kiểm chứng lịch tự sinh tạo view tốt | **Có code, chưa nghiệm thu 3 kênh** |
| **2. Script/Writer/Critic** (`tools/script_writer`, research/scripts) | Scaffold, validator, kịch bản và nghiên cứu mẫu | Có khung cấu trúc | Chưa thấy workflow tự động thực sự bắt buộc Writer→Critic→fact-check→scene JSON ở mọi kênh | **Một phần / Muse thủ công** |
| **3. Video production** (`pipeline/fv_build/*`, scripts per-video) | TTS, ffmpeg, subtitles, shots, overlay, thumbnail | Có khả năng tạo MP4 và theo dõi bước dựng | `shots.py` dựa nhiều vào ảnh + Ken Burns; footage chỉ tối đa tỷ lệ theo policy cũ; chưa chứng minh tự dựng cinematic video dài cho cả 3 kênh | **Một phần; rủi ro chất lượng cao** |
| **4. Technical QC** (`qc/technical_qc.py`, `media_qc.py`) | ffprobe/ffmpeg, silence/black/frozen/scene/static, thumbnail/subs | Có kiểm tra kỹ thuật thực tế; tự phát hiện nhiều video lỗi | Phát hiện ảnh tĩnh dựa thay đổi pixel 1fps; ảnh zoom/đồ họa động đơn giản có thể qua; nguồn quyền không được chứng thực | **Có code, cần hardening** |
| **4b. Quality Gate** (`qc/gate.py`, `performance_score.py`, `originality.py`) | Gồm gate kỹ thuật, thumbnail, originality, metadata, điểm 9 chiều | Ràng buộc FAIL ở nhiều lớp | Missing video -> `SKIP`, không mặc định FAIL; missing script có cảnh báo; PASS có thể bị hiểu nhầm là mọi quyền đã được kiểm chứng; điểm chủ quan/manual override | **Chưa đủ chuẩn xuất bản** |
| **5. SEO/Packaging** (`seo/seo.py`, `seo_cli.py`) | Patterns, title score, descriptions, tags, thumbnail suggestions | Có khuôn phục vụ sản xuất | Không chứng minh thuật toán dự đoán CTR/retention thật; cần làm rõ đã chạy ở video nào; metadata chuẩn thống nhất chưa đầy đủ | **Một phần** |
| **6. YouTube API/OAuth** (`youtube_api/auth.py`, `_gate.py`, `upload.py`) | Token riêng kênh, file mode 0600, verify `mine=true`, upload private | Kiểm soát sai kênh đúng hướng | OAuth scope trong source khai báo 4 quyền, không khớp app Cloud production thời điểm 10/10 (chỉ 2 quyền); video status update thiếu quyền; audit API chưa rõ | **Upload PRIVATE có tiền lệ; PUBLIC API BLOCKED** |
| **6b. Queue/Scheduler** (`youtube_api/scheduler.py`, `scheduler/publish_queue.py`) | `fcntl` lock, atomic queue save, checkpoint, pending_verification, test_mode | Thiết kế chống upload trùng hơn bản script đơn lẻ | Legacy `test_mode=False` tự tính `publishAt`; chưa có kiểm tra `approved_plan` ở `publish_entry`; nhánh `needs_verification` có thể kết luận absent và cho upload lại dựa search không hoàn chỉnh | **Không được bật tự động** |
| **7. Daily Analytics** (`analytics/daily_report.py`, `health_monitor/collector.py`) | Read-only reporting, compare window, lags, YouTube counters | Dữ liệu thật cho kênh có lịch sử; ghi N/A khi thiếu | CTR/impressions API 400 chưa được root-cause xác minh; FV/CV thiếu mẫu; chưa đủ dữ liệu long-form AGW; phụ thuộc token | **Có, giới hạn dữ liệu** |
| **7b. Monitoring** (`monitor/checks.py`, `errors.py`, `quota.py`) | OAuth, video health, queue, quota, alerts/digest | Cơ chế phát hiện hỏng | Phát hiện != tự chữa; quota ledger chỉ ghi ước tính, Pacific được cố định UTC-8 không xử lý DST, hạ tầng log/file-based | **Có, chưa chứng minh self-healing** |
| **7c. Comments** (`scheduler/reply_comments.py`, API comments) | Có phân loại/dự thảo trả lời | Có nền tảng hỗ trợ vận hành | Scope thiếu; automation tương tác chưa có phép nghiệm thu; không nên ưu tiên trước công cụ nội dung | **BLOCKED/không ưu tiên** |
| **8. Learning** (`learn/learn.py`, `seo.record_performance`, `calendar` weights) | **Có code** thu thập, group winner/loser, điều chỉnh trọng số khi đủ dữ liệu | Tốt hơn nhận định "chưa có learning gì" | Không có bằng chứng vòng dữ liệu→quyết định→video mới→đo lường đã chạy khép kín trên 3 kênh; mẫu quá ít, dễ overfit | **Có code, chưa chứng minh tác dụng** |
| **A/B testing** (`abtest/abtest.py`) | Thử nghiệm tuần tự title/thumbnail, lifecycle, học kết quả | Có thiết kế kiểm tra có kiểm soát | Chưa có dữ liệu chứng minh cải thiện thực; API write status/scope kiểm thử chưa xong | **Có code / chưa nghiệm thu** |
| **Policy/Versioning** (`policy/`, `versioning/`) | Registry policy, lịch review, snapshot/rollback file | Có audit trail cơ bản | Không tự phát hiện thay đổi YouTube; rollback file ≠ rollback video đã PUBLIC; cần kiểm soát thao tác live | **Có, chưa đủ kiểm soát sản xuất** |
| **Backup & DR** (`backup/`, docs/DR) | Manifest/hash, state+artifact backups, exclusion secrets | Bộ disaster backup hiện tại toàn vẹn | Lưu token riêng cần tái cấp; RPO 1h/RTO 4–8h là **mục tiêu**, không phải SLA đã chứng minh trên máy mới; dependencies chưa đủ | **Backup PASS / DR end-to-end CHƯA** |
| **4 dashboard read-only V1** (`v1_handover`, `health_monitor`) | HTML + JSON builders | Có thể dựng lại offline | Snapshot tĩnh, không cập nhật tự động; AI Recommendations dựa nhiều vào mảng đề xuất cố định; dashboard competitor AGW có gợi ý Shorts cũ trái định hướng | **PASS ở mức báo cáo V1, không phải OS vận hành A–Z** |

## C. Phát hiện kỹ thuật ưu tiên (có dẫn file)

**P0 — Rủi ro trước khi giao quyền tự PUBLIC**

**P0.1 — Thiếu "video approval gate" gắn mật mã hoặc trạng thái kế hoạch vào lệnh ghi.** Ở `source/youtube_api/youtube_api/scheduler.py`, `publish_entry()` khoảng dòng 480–519 xác minh `qc_report.json` có `result=PASS`, sau đó nếu không ở test mode sẽ tính `publishAt` (khoảng 572–590). Không thấy chặn bắt buộc dựa trên `approved_plan_id` và duyệt cụ thể từ chủ kênh tại bước này. **Hậu quả tiềm tàng:** nếu bật lại legacy scheduler, video chỉ cần QC PASS + pending sẽ được đưa vào luồng lên lịch PUBLIC. **Hành động:** giữ legacy scheduler disabled; thiết kế approval-gated publish tách riêng, ràng buộc channel ID, video ID, hash MP4, batch ID, thời gian, chính sách quyền dùng, chữ ký người duyệt. Không tự động lại khi chưa có live test.

**P0.2 — QC có nhánh "thiếu hồ sơ thì PASS".** `source/youtube-os/qc/technical_qc.py`, `_check_footage()` khoảng 382–387: nếu không thấy `trailers.json` và `footage_usage.json`, mã trả `footage_policy=PASS` với lý do "no trailer footage used". Đã kiểm thử độc lập đúng kết quả này; thực chất **không thể kết luận không dùng trailer từ việc không có manifest**. Ngoài ra điều kiện tối đa `7s/clip` hoặc `30% footage` không phải giấy phép bản quyền hay "safe harbor". **Hành động:** thiếu manifest nguồn/giấy phép = REVIEW/BLOCKED, không PASS; chứng cứ theo shot (tác giả, URL, giấy phép, bằng chứng tải về, timecode) và khả năng dùng thực tế; kiểm duyệt riêng footage có bản quyền.

**P0.3 — Xử lý uncertain-upload chưa fail-closed đủ chặt.** `source/youtube_api/youtube_api/scheduler.py`, khoảng 530–555: khi `needs_verification` và `find_recent_upload` không thấy, cờ bị đặt False và logger coi "safe to re-upload". Tìm không thấy không chứng minh server chưa nhận upload (trễ indexing/metadata). Code `reconcile_pending_verification()` ở ~248–318 thực tế thận trọng hơn. **Hành động:** uncertain luôn chờ đối soát tin cậy/manual review; không tự upload lần hai chỉ dựa search âm tính; test API timeout ngay sau server committed.

**P0.4 — Thiếu xác minh OAuth/YouTube API cho PUBLIC thật.** Backup `source/youtube_api/youtube_api/auth.py` khai báo 4 scopes (`youtube.upload`, `youtube.force-ssl`, `youtube.readonly`, `yt-analytics.readonly`), nhưng Google Cloud thật được người dùng chụp lúc 10/10 chỉ có 2 scope khai báo; thiếu quyền đã gây 403. Branding mới/Konsin Media, Search Console VERIFIED là **sự kiện sau snapshot**, không thể coi đã sửa OAuth. Thêm scope và compliance audit là 2 vấn đề khác. **Hành động:** hoàn tất Google verification chính thức, kiểm tra scope tối thiểu với API hiện hành và kiểm thử PUBLIC 1 video thử đã duyệt; không dùng thủ thuật vượt block.

**P0.5 — Cổng QC và dữ liệu duyệt có thể lệch file thực tế.** `scheduler.publish_entry` tin một file JSON chứa `PASS`, không thấy ràng buộc SHA-256 của `qc_report.json` với byte MP4 ngay trước thao tác publish. Một file MP4 có thể đổi sau QC nhưng report vẫn PASS. `qc/gate.py` cho `technical_qc=SKIP` khi thiếu `video` (~95–99), chỉ xem `FAIL` là hard fail. **Hành động:** thiếu video/script/bằng chứng QC = FAIL mặc định, hash file/manifest và tái xác nhận hash sát bước write; chống sửa báo cáo QC bởi chính quá trình sản xuất mà không tái kiểm.

**P1 — Chất lượng nội dung và phát triển kênh**

**P1.1 — Dựng video vẫn có đường dùng ảnh Ken Burns để đạt chỉ số motion:** `source/youtube-os/pipeline/fv_build/shots.py` mô tả 12 mẫu zoom/pan, overlay chuyển động và mục tiêu làm QC chống slideshow PASS. Đây không thỏa yêu cầu chủ dự án **footage chuyển động có ý nghĩa**, kể cả khi pixel thay đổi. **Hành động:** thêm gate "visual relevance and motion authenticity" kiểm tra sampling + đánh giá người xem; không dùng pixel motion như proxy duy nhất; footage phù hợp chủ đề và có quyền sử dụng.

**P1.2 — Rủi ro nguồn trailer/cơ chế tải:** `source/youtube-os/pipeline/fv_build/trailer_fetch.py` có `yt_dlp`, `nocheckcertificate=True` và tùy chọn client Android được chú thích để vượt mạng bị bot-check. Việc công khai trailer không trao quyền tải/dựng lại; tắt kiểm tra chứng chỉ và bypass không phù hợp tiêu chuẩn tuân thủ. **Hành động:** loại khỏi pipeline production chức năng vượt giới hạn, kiểm tra SSL bình thường và ưu tiên footage từ nguồn được cấp phép; nội dung bình luận có trích đoạn phải được đánh giá riêng.

**P1.3 — Recommendations V1 có đề xuất AGW Shorts trái lệnh hiện tại.** `source/youtube-os/health_monitor/build_competitor_v2.py` và `build_recommendations.py` chứa hardcoded đề xuất Rumi/Roblox Shorts 4–9s và diễn giải "AGW 100% Shorts"; trong khi chủ dự án chốt AGW chỉ long-form anime/donghua. **Hành động:** phiên bản tiếp theo phải segment lịch sử Shorts vs long-form, không dùng hiệu suất Shorts để dự báo video dài; khóa channel DNA.

**P1.4 — Learning chưa chứng minh hiệu quả thực:** Code `learn/learn.py` có cơ chế nhóm winner/loser, trọng số và ngưỡng dữ liệu tối thiểu; tốt ở mức hệ thống quy tắc. Chưa có bằng chứng đủ để nói "AI tự học" hay tự tối ưu tăng view. **Hành động:** thêm event log/experiment registry, holdout, tracking theo video và từng kênh; tránh overfit mẫu nhỏ; cho GPT/Muse đưa đề xuất chứ không tự sửa sản xuất.

**P1.5 — Khả năng di chuyển môi trường chưa đủ:** `config/requirements.txt` chỉ pin lỏng Google API, PyYAML, PySocks; source thực còn dùng `numpy`, Pillow, `imagehash`, scikit-learn, `yt_dlp`, Whisper/ffmpeg... và nhiều `~/workspace/...` hardcoded (`health_monitor/build_dashboard.py`, `scheduler.py`). **Hành động:** dependency lock, cấu hình đường dẫn di động, test clean-container, các gói yêu cầu theo feature, fallback minh bạch.

**P1.6 — Giới hạn upload trực tiếp 64 MiB tại Muse** (`youtube_api/upload.py` MAX_UPLOAD_MB = 64, `scheduler.py` check `MAX_UPLOAD_BYTES` ~565). Với video 1080p dài, 64 MiB dễ vượt. **Hành động:** đo output thực, kế hoạch upload chính thức hỗ trợ resumable hợp lệ khi môi trường cho phép; không hạ video xuống chất lượng kém để vừa giới hạn.

**P1.7 — Chỉ số CTR API 400 chưa được chẩn đoán xong:** `health_monitor/collector.py` gọi `impressions,impressionClickThroughRate` và ghi N/A khi lỗi; tài liệu V1 kết luận "không phải code" nhưng chưa có chứng cứ đầy đủ. **Hành động:** xác minh tính khả dụng/nguồn metric, quyền Analytics, query và API response; không bịa CTR hay retention.

**P1.8 — Quota/giờ reset** (`monitor/quota.py` dòng 21) sử dụng offset UTC−8 cố định thay vì timezone Pacific có DST; ledger nội bộ không chứng minh quota chính thức trong Cloud. **Hành động:** dùng IANA `America/Los_Angeles`, tra usage/quota thật, cảnh báo theo kênh và project, dự trù 50 kênh cho search/report/retry, không chỉ upload.

**P2 — Độ ổn định và quản trị**

- Snapshot tĩnh thiếu `data_as_of`, TTL cảnh báo, lịch làm mới đã nghiệm thu, nguồn theo từng chỉ số. Chặn hành động dựa dữ liệu cũ.
- Registry hiện lẫn channel lịch sử không thuộc 3 pilot và trạng thái cron snapshot cũ; cần 1 nguồn sự thật trạng thái runtime có tag project và `read_only`.
- File JSON + locks cục bộ thích hợp pilot; chưa chứng minh transaction/queue, giám sát, multi-worker 50 kênh. Nên dùng SQLite transaction/locking trước khi tính database lớn; thử tải trước khi đầu tư.
- Backup SHA chứng minh file giữ nguyên, **không** chứng minh thông tin bí mật chắc chắn sạch 100%. Secret scan cần có test lặp, không lưu secret trong log/backup; rotation theo kế hoạch có kiểm soát.
- Mục tiêu RPO 1h/RTO 4–8h trong docs vẫn là *mục tiêu*, chưa có kiểm thử khôi phục nguyên hệ thống và kết nối live trên máy mới.

## D. Đánh giá chiến lược 3 kênh

**Anime Galaxy World:** phải tối ưu cho **long-form anime/donghua** theo các video dài gần nhất. Hệ thống analytics phải có tập dữ liệu long-form riêng và so với nhóm đối thủ long-form tương ứng; không lấy 44,8 triệu view lịch sử Shorts làm baseline dự đoán. Video essay breakdown có góc nhìn gốc; clip trailer có bản quyền cần cân nhắc quyền sử dụng, không tính "≤7 giây/đoạn" hay "≤25%" là an toàn. Chấm điểm idea theo nhu cầu, giá trị phân tích, nguồn clip hợp lệ và năng lực sản xuất.

**Film Verdict:** ưu tiên timing phim mới và giả thuyết trước công chiếu được gắn nhãn rõ, phân biệt với review sau khi xem. Footage stock hợp pháp là nguồn hữu ích, nhưng cần liên quan trực tiếp đến luận điểm chứ không chỉ "có chuyển động". Tối ưu CTR bằng lời hứa đúng thực chất; nghiên cứu tiếng Anh/niche phim mới.

**Cinema Verdict:** định vị video essay dài có lập luận/chiều sâu về nhân vật, chủ đề, ending explained. Không được là Film Verdict phiên bản khác tiêu đề. Lỗi video Anton Chigurh (Muse báo 80% ít chuyển động) là ca nghiệm thu chống "text card" cần đưa vào bộ regression QC. Chưa coi kênh có pipeline sản xuất ổn định khi chưa có mẫu đầu tiên PASS độc lập.

**Tăng view có phương pháp:** theo dõi thumbnail/title → impressions/CTR (nếu có) → 30s retention → completion/watch time → audience return/subscribe → tác động video tiếp theo. 24h/72h/7d với mức tự tin, mùa vụ, cỡ mẫu. Chọn 1 video dài/ngày/kênh làm giả thuyết vận hành, **không** thành quota buộc đăng nếu QC fail. Tăng số lượng chỉ sau khi chứng minh video và kênh không bị suy giảm chất lượng/tương tác.

## E. Kiến trúc nâng cấp đề xuất, không viết lại từ đầu

**Lớp 1 — Ground truth/data contracts:** channel registry chuẩn, mỗi kênh profile/DNA, số liệu có nguồn/timestamp/quality/format video, API quota, lineage source, TTL.  
**Lớp 2 — Intelligence:** scan niche/đối thủ long-form, tính topic opportunity score có bằng chứng, lịch nội dung có human approval; không hardcode kết luận về trend.  
**Lớp 3 — Production:** scene manifest → licensed moving footage / motion graphics thực chất → TTS/subtitles → MP4/hash → tài liệu bản quyền theo từng asset.  
**Lớp 4 — QC & approval:** fail-closed rights/fact/policy/content/motion/audio/thumbnail, QC report ký theo artifact hash, duyệt một **lô có ID cụ thể** (không cần hỏi từng thao tác).  
**Lớp 5 — Distribution:** upload PRIVATE an toàn; after-approval publish worker riêng, quyền tối thiểu, idempotency, pending_verification, real API test, check post-publish. Không tương tác giả/né bot.  
**Lớp 6 — Learning & reliability:** thu 24h/72h/7d, phân tích theo format/niche; đề xuất điều chỉnh có giải thích; auto-retry hẹp cho lỗi kỹ thuật và rollback code an toàn, **không tự cấp OAuth, PUBLIC ngoài danh sách, thay chiến lược hoặc tự sửa production**.

## F. Lộ trình ưu tiên sau khi chủ dự án phê duyệt

| Thứ tự | Giai đoạn | Tiêu chí nghiệm thu cụ thể | Đụng production? |
|---|---|---|---|
| 0 | Giữ V1 analytics READ-ONLY và tiến độ sản xuất thủ công | Dashboard đọc được; lịch đăng không phụ thuộc sửa OAuth | Không |
| 1 | P0 QC/approval/identity/retry an toàn | Test cố ý thay MP4 sau PASS bị BLOCK; thiếu source/rights bị BLOCK; nhầm kênh bị BLOCK; API timeout không reupload trùng | Chỉ sau phê duyệt |
| 2 | OAuth Branding/Scopes/Compliance theo quy trình Google | Thử PUBLIC một video disposable **được chủ kênh duyệt**, xác minh thật YouTube API; dừng khi warning/block | Có, giới hạn 1 video |
| 3 | Production quality cho 3 kênh | 3 mẫu video đạt đánh giá nội dung + chuyển động thực, từng asset có hồ sơ, không slideshow | Môi trường thử |
| 4 | Data-backed topic selection & learning | Dữ liệu 24/72/7d đầy đủ, có baseline long-form, đề xuất có căn cứ và thử nghiệm | Môi trường thử |
| 5 | Thử 1 video/ngày/kênh rồi stress 10→50 kênh | Chạy 7–14 ngày với số liệu runtime, quota, worker, lỗi và khả năng khôi phục; không vượt ngân sách/quyền đã duyệt | Duyệt riêng |

## G. Chốt trạng thái nghiệm thu

**PASS được xác minh:** hash toàn bộ ZIP và từng file manifest, cú pháp Python, chạy lại 4 dashboard trên dữ liệu offline sau setup, tồn tại code cho phần lớn module.  
**PASS có điều kiện / cần xác minh hơn:** hoạt động Analytics bằng API, một số luồng QC, SEO, Learning, video production, sự khác biệt 3 kênh.  
**CHƯA PASS:** auto-PUBLIC đầu-cuối, quyền OAuth, xác minh API private-lock, hard gate provenance, fail-safe uncertain upload, restore full production, vận hành tải 50 kênh, đạt mục tiêu tự nâng cấp an toàn.  
**Không kết luận "Google/YouTube phạt kênh":** evidence hiện tại chỉ cho thấy vấn đề quyền OAuth và xác minh ứng dụng; chưa chứng minh bị xử phạt tài khoản.

### Hướng dẫn giao việc cho Muse

**Chưa lập trình V2 và chưa sửa production V1.** Trong lúc kiểm toán hiện tại, Muse tiếp tục sản xuất 3 kênh, dùng upload PRIVATE và lịch duyệt như đã chốt; quyền PUBLIC tự động chưa bật. Nếu chủ dự án đồng ý bước sửa, Muse dùng **từng P0/P1** trong báo cáo này làm đầu bài kiểm thử có tiêu chí nghiệm thu, báo một lô thay vì hỏi từng việc. Bất cứ phát hiện nào không khớp bản backup 10:54 cần ghi rõ là **thay đổi sau snapshot** và kèm file/test mới, không sửa lịch sử V1.

---

**Nguồn kiểm toán chính:** mã nguồn trong ZIP; `docs/MASTER_REQUIREMENTS.md`; 8 tài liệu `architecture/`; `v1_handover/TEST_RESULTS.md`; `config/requirements.txt`; `docs/DISASTER_RECOVERY_GUIDE.md`; PDF yêu cầu chi tiết UPDATED ngày 10/10 do Muse gửi (dùng làm chuẩn mục tiêu, không thay thế bằng chứng thực thi). Đường dẫn file và vị trí code nêu trong từng phát hiện có thể tra lại trong bản ZIP gốc.
