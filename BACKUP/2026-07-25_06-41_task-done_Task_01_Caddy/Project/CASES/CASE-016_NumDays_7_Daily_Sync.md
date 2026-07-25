# CASE-016: numDays = 7 cho Meta Daily Sync

## Vấn đề
API calls không cần thiết và workflow chạy chậm khi dùng numDays=30.

## Nguyên nhân
Meta điều chỉnh dữ liệu chậm (attribution window) chủ yếu trong vòng 7 ngày. Dùng 30 ngày tạo thêm 23 ngày API calls mà không mang lại lợi ích thực tế.

## Cách xử lý
- Production: `numDays = 7`.
- Nếu cần backfill dài hơn → dùng workflow backfill riêng, không thay đổi numDays production.

## Bài học
- Case hiếm attribution >7 ngày sẽ bỏ sót — accepted trade-off vì rất hiếm xảy ra.
- Đã verify production thành công với numDays=7.
- Workflow chạy nhanh hơn đáng kể.

## Status
Accepted — 2026-06-11.
