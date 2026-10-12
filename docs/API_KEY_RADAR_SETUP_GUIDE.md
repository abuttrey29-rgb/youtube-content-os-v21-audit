# HƯỚNG DẪN TẠO API KEY RIÊNG CHO BREAKOUT RADAR — Google Cloud Console

**Ngày lập:** 12/10/2026 · **Căn cứ:** Quyết định #5 của sếp (09:11 12/10/2026): "Đồng ý tạo API key riêng cho Radar, bảo đảm phân quyền và quản lý quota độc lập." · **Tác giả:** subagent tài liệu

## Tại sao cần API key riêng

Breakout Radar hiện mượn **token OAuth read-only của kênh Film Verdict** để gọi YouTube Data API v3. An toàn ở mức dùng thử (read-only tuyệt đối, token không bị đụng — đã kiểm chứng sha256/mtime trước/sau mỗi lần chạy), nhưng không phải thiết kế dài hạn: quota ăn chung với kênh, rủi ro nhầm token, và không tách bạch được ai dùng bao nhiêu. API key riêng → quota độc lập, không liên quan gì đến token OAuth của kênh.

**Quy tắc tuyệt đối:** em **KHÔNG tự tạo key** và **KHÔNG chạm key của sếp**. Toàn bộ dưới đây sếp tự làm bằng tay trong Cloud Console.

---

## Phần 1 — Tạo key (sếp thao tác, ~5 phút)

1. Mở [Google Cloud Console](https://console.cloud.google.com/) → đăng nhập tài khoản quản lý Content OS.
2. **Chọn đúng project:** bấm dropdown project trên thanh top → chọn **"YouTube Content OS"**
   (project duy nhất chứa 1 OAuth client Content OS — KHÔNG chọn project khác).
3. Vào menu ☰ → **APIs & Services → Credentials** (Thông tin xác thực).
4. Bấm **+ CREATE CREDENTIALS → API key**.
   - Google tạo ngay 1 key và hiện popup "API key created" — bấm **RESTRICT KEY** (hoặc Close rồi bấm vào tên key để sửa sau).
5. **Đặt tên gợi nhớ** (trường Name): ví dụ `radar-readonly-20261012` — để sau này phân biệt với các key khác.
6. **API restrictions** (quan trọng nhất — khóa key chỉ dùng được 1 API):
   - Chọn **Restrict key**.
   - Trong danh sách API, tích **chỉ** `YouTube Data API v3`.
   - **Không** tích thêm API nào khác. Không để "Don't restrict key".
7. **Application restrictions:** API key là *public credential* (đi kèm request từ server, không phải bí mật tuyệt đối như OAuth client secret). Để trống (None) cũng được vì API restriction ở bước 6 đã khóa phạm vi. Nếu sếp muốn chặt hơn, có thể đặt **IP addresses** = IP outbound cố định của VM — nhưng IP có thể đổi khi VM restart, dễ tự khóa mình, nên **khuyên để None** ở giai đoạn này.
8. Bấm **Save**.

## Phần 2 — Đặt quota limit cho key (~3 phút)

Mục đích: kể cả key có lọt ra ngoài, kẻ khác cũng không đốt hết quota project.

1. Trong cùng project → **APIs & Services → Quotas** (hoặc **Enabled APIs & services** → bấm **YouTube Data API v3** → tab **Quotas**).
2. Tìm metric **Queries per day** (tổng quota mặc định 10.000 units/ngày cho project).
3. Nếu cần giới hạn riêng cho key: hiện Google Cloud cho phép **quota override theo key** ở một số API; nếu không thấy, cách thực tế là theo dõi ở **APIs & Services → Dashboard** và đặt **cảnh báo** khi dùng vượt ngưỡng.
4. Mức dùng Radar thực tế sprint 24h: **~1.460 units/ngày** cho 1 lần scan đầy đủ (8 search + deep-scan). Đề xuất ngưỡng cảnh báo: **3.000 units/ngày** — dư gấp đôi cho re-scan + Market Intel bổ sung.

## Phần 3 — Lưu key vào Secure Vault (BẮT BUỘC)

- **TUYỆT ĐỐI KHÔNG** ghi key dạng plaintext vào: file bất kỳ (kể cả `~/workspace/`), chat, email, Google Sheet, ghi chú.
- Cách lưu đúng: mở **Secure Vault** của Muse (trang capture) → tạo entry mới tên `YOUTUBE_DATA_API_KEY_RADAR` → dán key vào → lưu.
- Sau khi lưu vào Vault: **xóa key khỏi clipboard**, đóng popup Console.
- Khi Radar cần dùng key: em lấy từ Vault qua flow chuẩn (không bao giờ in key ra log/chat).

## Phần 4 — Checklist xác minh sau khi tạo (sếp tự check hoặc báo em)

- [ ] Key hiện trong **APIs & Services → Credentials**, tên `radar-readonly-20261012`.
- [ ] Cột **Restrictions** của key ghi rõ: API restrictions = `YouTube Data API v3` (không phải "None").
- [ ] Key **không** xuất hiện ở bất kỳ file/chat/email nào (kiểm tra nhanh: không có).
- [ ] Key đã nằm trong **Secure Vault** dưới tên `YOUTUBE_DATA_API_KEY_RADAR`.
- [ ] Test sống: báo em — em chạy **1 lệnh duy nhất** `radar.py --dry-run` rồi 1 search thật (~100 units) bằng key từ Vault để xác nhận key hoạt động và trả đúng dữ liệu. Token OAuth các kênh không bị đụng trong bước này.

## Lưu ý rủi ro

1. **API key ≠ OAuth token:** key chỉ đọc được dữ liệu *công khai* của YouTube (search, channels, videos public). Key **không** đọc được video PRIVATE của kênh mình, không upload, không thao tác kênh — đó vẫn là việc của OAuth token. Radar chỉ cần dữ liệu công khai → key là đủ.
2. **Key lộ ≠ mất kênh:** vì key bị khóa chỉ dùng YouTube Data API v3 (bước 6), kẻ nhặt được key cũng chỉ search được video công khai — không vào được kênh, không tốn tiền (API miễn phí theo quota). Rủi ro chính là **đốt quota** → đã có quota limit (Phần 2) + có thể **xoá key và tạo lại trong 1 phút** nếu nghi lộ.
3. **Không dùng key này cho việc khác:** mỗi key một mục đích. Cần thêm tính năng → tạo key mới, không "tiện tay" mở rộng key Radar.

---

## Không làm gì trong tài liệu này

- Em không mở Cloud Console, không tạo/sửa/xóa key.
- Em không yêu cầu sếp gửi key qua chat — key chỉ đi thẳng vào Secure Vault.
