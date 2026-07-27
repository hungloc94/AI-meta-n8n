# CASE-044: Mojibake trong Node Code sau khi import Workflow Windows → Ubuntu

- **Ngày phát hiện:** 2026-07-26
- **Ngày xác minh:** 2026-07-26

## Vấn đề
Sau khi import workflow JSON từ Windows sang Ubuntu, string tiếng Việt và emoji trong node Code bị mojibake.

Ví dụ: `"📊 BÁO CÁO"` → `"ðŸ"Š BÃO CÃO"`

## Nguyên nhân
Windows lưu file JSON với encoding khác. Khi import vào n8n Ubuntu, file bị đọc sai encoding, làm vỡ các ký tự UTF-8 nhiều byte (tiếng Việt có dấu, emoji).

## Cách xử lý
1. Sau mỗi lần import workflow từ Windows
2. Mở từng node Code có string tiếng Việt hoặc emoji
3. Kiểm tra xem có bị vỡ không
4. Nếu bị → sửa lại thủ công bằng text đúng UTF-8
5. Save workflow và test send message

## Bài học
Luôn test send message ngay sau import để phát hiện mojibake sớm, trước khi activate workflow.

Liên quan: [[CASE-035]] — mojibake cũng từng xảy ra ở workflow JSON export → edit → import (chưa xác định root cause). CASE-044 xác định rõ hơn: nguyên nhân là encoding khác biệt giữa Windows và Ubuntu khi import.

## Status
Lesson learned — 2026-07-26.
