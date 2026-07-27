# CASE-019: Google Sheet Used Range Cleanup

## Vấn đề
Ctrl+End nhảy tới dòng 1011 dù dữ liệu thực tế kết thúc sớm hơn.

## Nguyên nhân
Google Sheet giữ "last used row" lịch sử kể cả khi data đã xóa. Đây là hành vi mặc định của Sheet.

## Cách xử lý
Deferred — không ảnh hưởng tính đúng đắn dữ liệu, không gây duplicate, không ảnh hưởng workflow.

Khi cần xử lý:
- Delete các rows cuối dư thừa (không dùng Clear — Clear không reset used range).
- Verify Ctrl+End sau khi dọn.

## Bài học
- Không phải mọi "bất thường" đều cần fix ngay.
- Chỉ fix khi có impact thực tế lên workflow hoặc data integrity.

## Status
Deferred / Low Priority — 2026-06-11.
