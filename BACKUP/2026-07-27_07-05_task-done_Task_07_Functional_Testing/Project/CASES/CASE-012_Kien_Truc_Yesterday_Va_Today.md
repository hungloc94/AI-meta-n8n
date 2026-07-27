# CASE-012: Kiến trúc Option B — Workflow Yesterday + Workflow Today

## Vấn đề
Một workflow chung cho cả báo cáo hôm qua và hôm nay quá phức tạp và dễ gây lỗi im lặng.

## Nguyên nhân
Hai loại nhu cầu khác nhau:
1. **Báo cáo hôm qua** — dữ liệu đã ổn định, đã có trong Sheet → đọc Sheet là đủ.
2. **Báo cáo hôm nay** — dữ liệu chưa có trong Sheet (chưa sync) → phải gọi Meta API trực tiếp.

Nếu dùng chung, logic rẽ nhánh phức tạp và dễ fail silently khi dữ liệu hôm nay chưa vào Sheet.

## Cách xử lý
Tách thành 2 workflow riêng:
- **Yesterday**: đọc Google Sheet → normalize → KPI → Telegram
- **Today**: gọi Meta Marketing API → KPI → Telegram

## Bài học
- Maintain 2 workflow nhưng mỗi cái có trách nhiệm rõ ràng, dễ debug.
- Yesterday phụ thuộc Google OAuth. Today phụ thuộc Meta API token.
- Khi một credential fail, chỉ ảnh hưởng workflow tương ứng.

## Status
Accepted — implement 2026-06-07.
