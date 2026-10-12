# QUERY PACK V2 — Breakout Radar (mở rộng đa ngôn ngữ)

**Ngày:** 12/10/2026 | **Trạng thái:** PLANNED — **CHƯA CHẠY, chờ sếp phê duyệt trước khi chạy.**
**Cơ sở quota:** `radar.py` 12/10 — `search.list` = **100 units/call** (maxResults=25, order=date, type=video);
downstream trung bình ~2.5 units/query (channels.list batch 1u + playlistItems 1u + videos 1u).
**Quy tắc ước tính mỗi query mới: 100u (search) + ~3u (downstream) ≈ 103u.**

## Pack A — Anime tiếng Nhật (kênh 0–30 ngày, niche 考察/解説)

| # | Query | Mục đích | Ước tính |
|---|-------|----------|----------|
| A1 | `アニメ 考察` | Kênh phân tích anime JA (khớp niche AGW dài) | 103u |
| A2 | `アニメ 解説` | Kênh giải thích/giới thiệu anime JA | 103u |

## Pack B — Anime tiếng Trung

| # | Query | Mục đích | Ước tính |
|---|-------|----------|----------|
| B1 | `动漫 解析` | Kênh phân tích donghua/anime ZH (giản thể) | 103u |
| B2 | `动画 解说` | Kênh 解说 (thuyết minh/phân tích) ZH | 103u |

## Pack C — Movie analysis đa ngôn ngữ (khớp niche Film Verdict)

| # | Query | Ngôn ngữ | Ước tính |
|---|-------|----------|----------|
| C1 | `movie analysis` | EN | 103u |
| C2 | `análisis de películas` | ES (Mỹ Latinh + TBN, RPM trung bình) | 103u |
| C3 | `análise de filmes` | PT (Brazil — niche gospel đã chứng minh BR mạnh) | 103u |
| C4 | `analyse de films` | FR | 103u |
| C5 | `Filmanalyse` | DE | 103u |
| C6 | `映画 解説` | JA | 103u |
| C7 | `영화 분석` | KO | 103u |
| C8 | `电影 解说` | ZH (giản thể) | 103u |

## Pack D — Documentary đa ngôn ngữ (khớp niche Cinema Verdict + true crime/history đã bắt được kênh)

| # | Query | Ngôn ngữ | Ước tính |
|---|-------|----------|----------|
| D1 | `documentary explained` | EN | 103u |
| D2 | `documental explicado` | ES | 103u |
| D3 | `documentário explicado` | PT (Brazil) | 103u |
| D4 | `Dokumentation erklärt` | DE | 103u |
| D5 | `ドキュメンタリー 解説` | JA | 103u |
| D6 | `纪录片 解说` | ZH (giản thể) | 103u |

## Tổng hợp quota

| Gói | Số query | search.list (100u) | Downstream (~3u/q) | Tổng ước tính |
|-----|----------|--------------------|--------------------|---------------|
| A (anime JA) | 2 | 200 | ~6 | **~206u** |
| B (anime ZH) | 2 | 200 | ~6 | **~206u** |
| C (movie đa ngôn ngữ) | 8 | 800 | ~24 | **~824u** |
| D (documentary đa ngôn ngữ) | 6 | 600 | ~18 | **~618u** |
| **Tất cả (A+B+C+D)** | **18** | **1.800** | **~54** | **~1.854u** |

## Cảnh báo quota (quyết định sếp 09:11 + ràng buộc sprint)

- Đợt scan 12/10 dùng **824u** cho 8 query; giới hạn sếp duyệt cho cron Radar là **<1.500 units**.
- Chạy hết 18 query một lần (~1.854u) **VƯỢT** trần 1.500u → **đề xuất chia 3 đợt**:
  - Đợt 1: A+B (4 query, ~412u) — anime JA/ZH, trực tiếp phục vụ AGW.
  - Đợt 2: C (8 query, ~824u) — movie, phục vụ Film Verdict.
  - Đợt 3: D (6 query, ~618u) — documentary, phục vụ Cinema Verdict.
- Mỗi đợt cách nhau ≥24h để không dồn quota/ngày (quota ngày 10.000u — tổng 3 đợt ~1.854u vẫn an toàn).
- **LƯU Ý:** search.list còn giới hạn bucket riêng (radar.py ghi "100 calls/day"); 18 query = 18 search calls, trong giới hạn.

## Ghi chú triển khai (khi được duyệt)

- Reuse `radar.py` nguyên xi (đã LIVE VERIFIED), chỉ thêm query vào `SEARCH_QUERIES`,
  lưu snapshot vào `state/snapshots/breakout_radar_YYYYMMDD_pack{A,B,C,D}/`.
- Chạy `velocity.py` sau đợt 1: snap 12/10 vs snap đợt mới → velocity thật kênh cũ.
- **CHƯA CHẠY — báo parent/sếp trước khi chạy.**
