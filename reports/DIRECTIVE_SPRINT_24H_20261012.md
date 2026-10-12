# CHỈ ĐẠO SPRINT 24H — V2.1 (bổ sung 08:40 12/10/2026, nguyên văn ý sếp)

Sếp đã xem phản hồi của em. Đồng ý cho V1 tiếp tục hoạt động bình thường và V2.1 phát triển song song.

**YÊU CẦU MỚI: TẬP TRUNG NGUỒN LỰC, CHỌN LỌC NHỮNG GIẢI PHÁP TỐT NHẤT VÀ ĐẨY NHANH V2.1 TRONG VÒNG 24 GIỜ KỂ TỪ KHI NHẬN CHỈ ĐẠO NÀY.**
Deadline: 08:40 13/10/2026.

## I. V1 TIẾP TỤC HOẠT ĐỘNG
- Giữ mục tiêu 2 video/ngày/kênh, tổng 6 video/ngày, theo chỉ đạo mới nhất.
- Chất lượng quan trọng hơn số lượng. Không được đăng video chưa đạt QC.
- Không để quá trình phát triển V2.1 ảnh hưởng V1.
- Tuân thủ quy trình duyệt PUBLIC và giới hạn quyền OAuth hiện tại.
- Báo cáo ngắn gọn kết quả sản xuất thực tế, phân biệt video hoàn thành, PRIVATE, SCHEDULED và PUBLIC.

## II. V2.1 — TẬP TRUNG TINH HOA
Không xây nhiều module hình thức hoặc tính năng trùng lặp. Tận dụng tối đa những gì V1 đã có.
Trong 24 giờ, ưu tiên hoàn thiện một bản V2.1 có thể chạy thử và chứng minh giá trị thực tế.

Ưu tiên theo thứ tự:
1. **NEW CHANNEL BREAKOUT RADAR — QUAN TRỌNG NHẤT**: tìm kênh mới thành lập trong 30 ngày gần nhất nhưng tăng trưởng cực mạnh, không giới hạn KEY/chủ đề/quốc gia/ngôn ngữ. Phân tích video bùng nổ, tốc độ tăng trưởng, KEY, format, nguyên nhân thành công. Phải có kết quả thực từ YouTube: Channel ID, link, ngày tạo, dữ liệu có nguồn và độ tin cậy. Không dùng dữ liệu giả lập để tuyên bố Radar hoạt động thực tế.
2. **MARKET + COMPETITOR INTELLIGENCE**: kết hợp dữ liệu Radar + đối thủ + xu hướng → KEY mới, chủ đề tăng tốc, cơ hội ít cạnh tranh. Bảng xếp hạng cơ hội, giải thích logic chấm điểm, đề xuất hành động.
3. **CONTENT INTELLIGENCE + VIDEO QUALITY**: nâng Research Brief, kịch bản, hình ảnh, footage, dựng, thumbnail, QC. Video hấp dẫn, tự nhiên, nguyên bản, giá trị thực chất, tránh máy móc.
4. **ANALYTICS + LEARNING**: vòng phản hồi từ hiệu suất thật để cải thiện chọn KEY, nội dung, cách sản xuất.
5. **INDEPENDENT OVERSIGHT**: giám sát độc lập từ ngày đầu — kiểm tra dữ liệu, quyết định AI, chất lượng video, tính đúng kênh, API, lịch đăng, mọi thao tác rủi ro. Không tự sản xuất → tự đánh giá → tự kết luận thành công thiếu kiểm chứng.

## III. TIÊU CHUẨN KỸ THUẬT
- Logic khoa học, có dữ liệu và bằng chứng.
- Module rõ ràng, dễ thay thế và mở rộng.
- Tự động hóa tối đa trong phạm vi đã được cấp quyền.
- Không spam, bot tăng tương tác, sao chép nội dung hoặc lách thuật toán, Content ID.
- Không phỏng đoán hoặc bịa dữ liệu.
- Kiểm thử nghiêm ngặt, có rollback.
- Không phát sinh chi phí mới khi chưa được duyệt.
- V2.1 chạy độc lập; chưa chuyển production, chưa bật cron hoặc auto-PUBLIC mới.

## IV. BÀN GIAO TRONG 24 GIỜ
1. Bản đối chiếu kiểm toán V1 với code hiện tại.
2. Kiến trúc V2.1 và sơ đồ Independent Oversight.
3. Radar kênh mới phiên bản chạy thử bằng dữ liệu thực, nếu khả thi với API/quota hiện có.
4. Bản nâng cấp chạy thử của các module ưu tiên đã hoàn thành.
5. Kết quả test, số liệu và bằng chứng.
6. Danh sách chức năng đã chạy thực tế, mới qua mock test và chưa hoàn thiện.
7. Những việc còn lại để đạt bản V2.1 hoàn chỉnh cùng thời gian dự kiến.
8. Những quyết định bắt buộc sếp phê duyệt.

Yêu cầu cao nhất là CHẤT LƯỢNG VÀ HIỆU QUẢ THỰC CHẤT.
Nếu trong 24 giờ không thể hoàn thành toàn bộ V2.1 đúng tiêu chuẩn, bàn giao bản tốt nhất đã kiểm thử, báo rõ phần chưa đạt và kế hoạch hoàn tất. Tuyệt đối không hạ tiêu chuẩn chất lượng hoặc báo cáo hoàn thành khi chưa nghiệm thu.
