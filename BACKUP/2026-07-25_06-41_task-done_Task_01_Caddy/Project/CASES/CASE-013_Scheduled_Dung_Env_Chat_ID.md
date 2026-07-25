# CASE-013: Scheduled Workflow dùng $env.TELEGRAM_CHAT_ID

## Vấn đề
Scheduled workflow fail silently hoặc gửi sai chat vì không có Telegram message context.

## Nguyên nhân
Cron trigger không có user message → `$json.chat_id` = undefined. Node Send Telegram sẽ fail hoặc không biết gửi cho ai.

## Cách xử lý
- Bot-triggered workflow (user nhắn tin) → dùng `$json.chat_id` từ message context.
- Scheduled workflow (cron) → bắt buộc dùng `$env.TELEGRAM_CHAT_ID`.
- Giá trị `TELEGRAM_CHAT_ID` phải có trong `.env` container.

## Bài học
- Hardcode chat_id trong env — nếu muốn gửi nhiều chat phải thêm logic.
- Luôn phân biệt rõ context: có user trigger hay không.
- Verify bằng cách check node Send Telegram dùng biến nào.

## Status
Accepted — xác minh 2026-06-08.
