# Điều tra "lỗi config bị mất và trạng thái kênh không cập nhật"

**Ngày:** 12/10/2026 | **Phạm vi:** CHỈ V2.1 staging (`v2_1/`) — đọc file, không sửa V1.
**Trạng thái nhãn:** TESTED (điều tra bằng đọc file/grep, không đoán).

## Tóm tắt kết luận

| Triệu chứng sếp báo | Nguyên nhân thật (đã verify) | Thuộc |
|---|---|---|
| "config bị mất" | **Config V2.1 chưa bao giờ tồn tại** — `v2_1/config/` được README.md (12/10) quy hoạch "sẽ tạo ở Wave 0" nhưng sprint 24h chỉ build 5 module ưu tiên; thư mục chưa được tạo, không có file nào để "mất". Không có dòng code V2.1 nào load config (grep `config` trong `*.py` → 0 kết quả). | V2.1 — Wave 0 chưa hoàn thành |
| "trạng thái kênh không cập nhật" | V2.1 **không có** nơi lưu trạng thái kênh: `modules/channel_registry/` là thư mục **rỗng** (scaffold tạo 12/10 01:34, chưa có code), `state/` chỉ chứa snapshot JSON + `readonly_mirror/` (mới có README, chưa có dữ liệu mirror V1). Nơi duy nhất lưu trạng thái kênh trong toàn hệ thống là V1: `youtube-os/channels/channel_status.json`. | V1 — **chỉ ghi nhận, KHÔNG sửa** |

## Chi tiết

### 1. Config load từ đâu? → Không có gì để load.

- `v2_1/config/`: **không tồn tại** (kiểm tra 12/10 09:2x).
- Grep toàn bộ `v2_1/**/*.py` tìm từ `config` (không tính __pycache__): **0 kết quả** — không module nào đọc config.
- `modules/channel_registry/`, `modules/content_intelligence/`, `modules/distribution/`,
  `modules/learning_engine/`, `oversight/alerts/`: đều là thư mục **rỗng** (scaffold chưa có code).
- → "Lỗi config bị mất" ở V2.1 không phải file bị xóa/mất: là **hạng mục Wave 0 chưa được làm**.
  Sprint 24h ưu tiên 5 mục (Radar, Quality, Analytics, Market Intel, Oversight) nên config +
  channel_registry chưa tới lượt.

### 2. Trạng thái kênh lưu ở đâu? → Chỉ có ở V1.

- V2.1: không có status store. `state/snapshots/*/` chứa snapshot JSON một lần (không phải bảng trạng thái sống).
- V1: `~/workspace/youtube-os/channels/channel_status.json`, ghi bằng `set_status()`
  trong `channels/channel_status.py` (ghi atomic qua file .tmp → os.replace, code đúng).

### 3. Tại sao trạng thái không tự cập nhật? → Không có ai gọi set_status() tự động.

- Grep toàn bộ `youtube-os/**/*.py` (trừ backup/, __pycache__): `set_status(` **không được gọi ở đâu**
  ngoài chính module `channel_status.py`. Các file như `scheduler/publish_queue.py`,
  `health_monitor/collector.py`, `refresh_dashboard_data.py` chỉ **đọc** (`get_status`,
  `api_calls_allowed`) — không bao giờ ghi.
- → Registry chỉ thay đổi khi **người hoặc agent gọi tay**. Sau sự kiện OAuth lại (9/10),
  upload/thumbnail thành công (8–12/10), không có hook nào tự flip trạng thái → hiển thị
  cũ cho tới khi ai đó cập nhật thủ công. Đây chính là cơ chế gây "trạng thái không cập nhật".

### 4. Kiểm tra thực tế file V1 hiện tại (đọc, không sửa)

`channels/channel_status.json` lúc điều tra (12/10): Film Verdict = `API_READY`
("OAuth OK, scheduler đang chạy", updated 2026-10-09T08:25:38Z); AGW + CV cũng `API_READY`;
Blockbuster Verdict / Everfable / TOP MUSICAS GOSPEL = `OAUTH_PENDING` (đúng thực tế —
chưa OAuth). → **Hiện tại không còn STALE** (audit 10/10 ghi FV = OAUTH_PENDING đã được
ai đó sửa tay sau đó), nhưng vì không có updater tự động, drift sẽ tái diễn.

## Cách khắc phục đề xuất (cần sếp duyệt trước khi làm)

**V2.1 (staging — làm được ngay, không động V1):**
1. Tạo `v2_1/config/` + schema config kênh (channel_id, tên, niche, slot đăng, trạng thái kỳ vọng)
   — Wave 0 còn nợ; điền từ dữ liệu đã biết (3 kênh Content OS), ghi rõ nguồn.
2. Xây `modules/channel_registry/` thật: profile/DNA theo Channel ID (ground truth),
   đọc **read-only** từ snapshot/API, không ghi ngược V1.
3. Thêm **status reconciler** trong V2.1: mỗi lần đo analytics, so trạng thái quan sát được
   (upload/thumbnail thành công qua API) với registry → báo lệch cho Oversight
   (không tự ghi V1 — chờ sếp duyệt cơ chế ghi).

**V1 (CHỈ ĐỀ XUẤT — không sửa khi chưa duyệt):**
4. Thêm hook `set_status()` vào các điểm sự kiện: OAuth verify thành công → `API_READY`;
   upload/thumbnail thành công → refresh `updated_at` + note; token revoke → `TOKEN_EXPIRED`.
5. Hoặc: cron đối chiếu 1 lần/ngày (verify token + 1 API call read-only) tự cập nhật
   `api_status` — rẻ (~4 units/ngày) và loại bỏ drift thủ công.

## Nhãn

- CONFIG_BUG_INVESTIGATION: **TESTED** (đọc file + grep thật, không suy đoán).
- Vấn đề gốc của "trạng thái kênh": **V1 — GHI NHẬN, không sửa** (theo chỉ đạo).
