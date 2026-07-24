# CASE-017: Today Report lấy trực tiếp từ Meta API

## Vấn đề
Today Report không có data trong Sheet vì Sheet chỉ sync lúc 07:30.

## Nguyên nhân
Dữ liệu hôm nay chưa tồn tại trong Sheet tại thời điểm báo cáo (11:31, 16:31, 21:13). Nếu đọc Sheet sẽ lấy dữ liệu cũ hoặc không có gì.

## Cách xử lý
- `Meta Report Today Scheduled` gọi Meta Marketing API trực tiếp.
- Không đọc Google Sheet.
- Không phụ thuộc Google Sheets credential.

## Bài học
- Tách khỏi Google OAuth dependency — Today Report sống độc lập, không bị ảnh hưởng khi OAuth expire.
- Cần quản lý Meta API token riêng.
- Nếu Meta API thay đổi response format, chỉ Today Report bị ảnh hưởng.

## Status
Accepted — deploy và verify 2026-06-11.
