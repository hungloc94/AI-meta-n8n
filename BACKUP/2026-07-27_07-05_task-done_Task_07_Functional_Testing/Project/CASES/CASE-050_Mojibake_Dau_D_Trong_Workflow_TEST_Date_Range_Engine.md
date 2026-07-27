# CASE-050: Mojibake Ký Tự "đ" Trong Tin Nhắn Telegram Của Workflow TEST Date Range Engine

- **Ngày phát hiện:** 2026-07-27
- **Ngày xác minh:** 2026-07-27

## Vấn đề
Test workflow "Meta Report Yesterday Scheduled 1 TEST - Date Range Engine V1" — tin nhắn Telegram gửi ra bị lỗi hiển thị ký tự "đ": xuất hiện thành "Ä‘" ở một số chỗ (`154.446Ä‘`, `165.107Ä‘`, `15.227Ä‘`), trong khi chỗ khác cùng tin nhắn lại hiện đúng (`1.081.123đ`). Workflow production tương đương ("Meta Report Yesterday Scheduled 1") **không** bị lỗi này khi test cùng ngày.

## Nguyên nhân
Chưa xác định chính xác gốc rễ — nhưng khác biệt với [[CASE-044]] ở chỗ: CASE-044 mô tả mojibake làm vỡ **toàn bộ** chuỗi (emoji + tiếng Việt), còn đây chỉ **một ký tự cụ thể** ("đ") bị double-encode ở một số vị trí trong cùng 1 chuỗi, các ký tự có dấu khác (á, ị, ầ...) và emoji vẫn hiển thị đúng. Nhiều khả năng: chuỗi text trong node Code của bản TEST này được chỉnh sửa thủ công sau khi workflow gốc đã bị ảnh hưởng một phần bởi encoding lúc tạo bản TEST (export/edit/import), chỉ 1 số literal "đ" trong source code bị lưu sai encoding, không phải toàn bộ file.

## Cách xử lý
Chưa sửa — ngoài phạm vi Task 07 (Task 07 chỉ test hệ thống hiện tại, không sửa workflow TEST đang phát triển thuộc Module 01). Trước khi Module 01 dùng workflow này thay thế logic cũ, cần:
1. Mở node Code liên quan (nhiều khả năng "Lọc & Tính KPI Hôm Qua" — node tạo text báo cáo) trong n8n UI.
2. Tìm và thay toàn bộ ký tự "đ" bị sai bằng ký tự đúng (gõ lại tay, không copy-paste từ nguồn có thể nhiễm encoding).
3. Test gửi lại, xác nhận hiển thị đúng toàn bộ chuỗi.

## Bài học
- Đối chiếu bản TEST với bản production **cùng logic, cùng ngày** giúp phát hiện lỗi hiển thị mà một mình bản TEST không tự lộ ra (nếu không có bản đối chứng, dễ tưởng đây là "vấn đề chung" thay vì lỗi riêng của bản TEST).
- Mojibake có thể xảy ra **một phần** trong chuỗi (chỉ 1 ký tự cụ thể), không nhất thiết làm vỡ toàn bộ text — cần đọc kỹ toàn bộ nội dung gửi ra, không chỉ nhìn lướt xem "có gửi được không".
- Liên quan: [[CASE-044]], [[CASE-035]] (mojibake do export/edit/import workflow JSON).

## Status
Phát hiện, chưa fix — cần xử lý trước khi Module 01 (Date Range Engine Integration) dùng workflow này thay production.
