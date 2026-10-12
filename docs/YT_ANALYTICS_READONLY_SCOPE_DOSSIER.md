# DOSSIER XIN SCOPE `yt-analytics.readonly` — YouTube Content OS V2.1

**Ngày lập:** 12/10/2026 · **Căn cứ:** Quyết định #3 của sếp (09:11 12/10/2026): "Đồng ý chuẩn bị xin quyền yt-analytics.readonly." · **Tác giả:** subagent tài liệu (không chạm OAuth/token — tài liệu chỉ đọc)

---

## 1. Scope cần xin (chính xác, copy nguyên văn)

```
https://www.googleapis.com/auth/yt-analytics.readonly
```

Tên hiển thị trong Google Cloud Console Data Access: **YouTube Analytics API** → `.../auth/yt-analytics.readonly`.

Đây là scope *đọc-only*, cho phép đọc các báo cáo YouTube Analytics (views, watch time, impressions, CTR, retention...). Không cho phép đăng, sửa, xóa, hay thao tác gì lên kênh.

---

## 2. Tại sao cần

Module **Analytics + Learning** (V2.1) hiện đã có `collector.py` và snapshot T0 chạy thật bằng YouTube Data API v3 — nhưng Data API chỉ cho biết con số công khai (views, likes, comments, privacyStatus) và tình trạng video (PRIVATE/SCHEDULED/PUBLIC/FAILED qua `video_status.py`).

**4 chỉ số tăng trưởng quan trọng nhất đều mù (= N/A) vì thiếu scope:**

| Chỉ số | Dùng để làm gì | Trạng thái hiện tại |
|---|---|---|
| CTR (click-through rate) | Đánh giá thumbnail + tiêu đề, rút bài học vào `channel_learnings.md` | N/A |
| Impressions | Biết YouTube đẩy video ra bao nhiêu lần, tính CTR thật | N/A |
| Retention (audience retention) | Biết đoạn nào khán giả bỏ xem → cắt dựng cải tiến | N/A |
| Watch time (giờ xem) | Điều kiện YPP (4.000 giờ), tiêu chí sức khỏe kênh | N/A |

Hệ quả: mọi báo cáo Analytics hiện tại đều ghi "CTR/retention N/A trung thực" (xem `modules/analytics/ANALYTICS_SNAPSHOT_20261012.md`), và Learning Engine không thể tự rút bài học CTR/thumbnail/retention — đúng mục mà sếp đặt làm ưu tiên số 3 trong chỉ đạo sprint V2.1.

**Scope này mở khóa đúng 4 chỉ số trên, không thêm quyền ghi nào.**

---

## 3. Trạng thái hiện tại

- **Chưa được cấp.** Token hiện tại của các kênh (Film Verdict, AGW, Cinema Verdict trên tài khoản `[EMAIL-REDACTED]`) chỉ có scope: `readonly` + `upload` (theo Data Access đã khai báo, xác nhận 9–10/10).
- Scope `youtube.force-ssl` (để mở khóa comment-reply API 403) cũng **chưa được cấp** — đang chờ sếp duyệt chung một đợt re-consent.
- Scope `yt-analytics.readonly` **đã được tách ra khỏi vòng consent PUBLIC hiện tại** — tức vòng consent vừa rồi (mở quyền thumbnail cho sếp) KHÔNG bao gồm scope này, tránh trì hoãn việc mở kênh đang cần gấp.

---

## 4. Vòng consent nào sẽ xin

**Vòng tiếp theo** — dự kiến gộp chung một đợt với `youtube.force-ssl` (nếu sếp duyệt), để sếp chỉ cần duyệt OAuth **một lần duy nhất** cho cả 2 scope bổ sung:

1. `https://www.googleapis.com/auth/youtube.force-ssl` (comment-reply + publishAt removal)
2. `https://www.googleapis.com/auth/yt-analytics.readonly` (CTR/impressions/retention/watch time)

Chưa có lịch cụ thể — cần sếp quyết định thời điểm. Không thực hiện bất kỳ thao tác OAuth nào trước khi sếp duyệt.

---

## 5. Checklist từng bước sếp cần làm (trong Google Cloud Console)

> Sếp tự thao tác bằng tay. Mọi bước đều là đọc/cấu hình — không chạm token hiện tại, token cũ vẫn sống bình thường.

1. Mở [Google Cloud Console](https://console.cloud.google.com/) → chọn đúng project **"YouTube Content OS"** (project duy nhất dùng cho 1 OAuth client Content OS).
2. Vào **APIs & Services → OAuth consent screen**.
3. Tab **Data Access** → bấm **Add or remove scopes**.
4. Gõ/tích chọn scope: `https://www.googleapis.com/auth/yt-analytics.readonly`
   - (Cùng đợt, nếu duyệt) tích thêm `https://www.googleapis.com/auth/youtube.force-ssl`.
5. **Save** (lưu). Không cần submit cho Google verification ở bước này (mục 6 bên dưới).
6. Bấm **Test users / Publish** nếu app đang ở trạng thái cần — giữ nguyên trạng thái hiện tại nếu không chắc, báo lại cho em.
7. **Re-consent:** sếp đăng nhập lại flow OAuth một lần (giống lần xin scope upload) — màn hình Google sẽ hiện thêm quyền "Xem báo cáo YouTube Analytics của bạn". Bấm **Continue/Allow** → token mới có đủ scope → em đổi token (đã có backup token cũ).
8. **Verify:** sau khi re-consent xong, báo em để em chạy 1 query Analytics test (chỉ đọc) và xác nhận CTR/impressions/retention trả về — xong mới coi là hoàn tất.

---

## 6. Lưu ý rủi ro (đọc trước khi quyết định)

1. **Scope sensitive:** `yt-analytics.readonly` thuộc nhóm *sensitive scope* của Google. Nếu app xin scope này cho **người dùng ngoài (external)**, Google có thể yêu cầu **verification** (xác minh app: video demo, privacy policy, homepage). Trường hợp của mình — scope chỉ dùng cho kênh của chính sếp, test users giới hạn — verification thường không bắt buộc ở giai đoạn internal. Nhưng nếu sau này scale lên 50 kênh/người dùng ngoài, cần chuẩn bị hồ sơ verification.
2. **Re-consent bắt buộc:** scope mới chỉ có hiệu lực sau khi sếp re-consent; token cũ không tự động có scope mới. Không có cách "cộng thêm scope" mà không qua màn hình duyệt của Google.
3. **Không ảnh hưởng V1:** việc xin scope không đụng chạm video, playlist, cron hay pipeline production hiện tại. Nếu re-consent thất bại, token cũ vẫn nguyên (backup sẵn), Analytics tiếp tục báo N/A như hiện nay.
4. **Quota:** YouTube Analytics API có quota riêng (không ăn chung quota Data API 10.000 units/ngày của Radar). Mức dùng dự kiến: vài chục query/ngày cho 6 video — không đáng kể.

---

## 7. Không làm gì trong dossier này

- Không mở Cloud Console, không bấm nút nào, không sửa OAuth client.
- Không chạm token hiện tại (không đọc, không refresh, không revoke).
- Không gửi auth code/token qua chat.
