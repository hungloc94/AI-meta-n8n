# CASE-024: Duplicate Key trong Google Sheet

## Vấn đề
Một số Key xuất hiện 2 lần trong Sheet. Dòng cũ (09/06) có KPI sai, dòng mới (11/06) có KPI đúng. COUNTIF(Key) > 1 trên nhiều hàng.

## Nguyên nhân
Trong giai đoạn patch workflow (sửa KPI Mess_Comment), workflow đã thực hiện **append thay vì update** với dữ liệu đã patch. Kết quả: dòng cũ giữ nguyên, dòng mới được thêm → duplicate.

## Cách xử lý
1. Dọn duplicate thủ công trong Sheet
2. Giữ bản ghi mới (timestamp 11/06 — KPI đúng)
3. Xóa bản ghi cũ (timestamp 09/06 — KPI sai)
4. Verify: COUNTIF toàn bộ cột Key = 1

## Bài học
- Sau bất kỳ patch workflow nào liên quan Sheet → chạy COUNTIF(Key) detect duplicate ngay.
- Append vs Update là 2 hành vi hoàn toàn khác — phải verify workflow đang dùng đúng mode.
- Google Sheet used range có thể nhảy xa hơn data thực — đây là Sheet behavior, không phải duplicate. Dùng COUNTIF(Key) để phân biệt.

## Status
Resolved — 2026-06-11.
