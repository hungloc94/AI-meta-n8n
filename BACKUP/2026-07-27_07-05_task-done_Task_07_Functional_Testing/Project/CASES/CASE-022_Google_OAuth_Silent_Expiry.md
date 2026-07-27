# CASE-022: Google OAuth Silent Expiry

## Vấn đề
Workflow chết im lặng — không crash, không alert. n8n UI vẫn hiện "Account connected". Dữ liệu ngày 09/06 sai (~71k thay vì ~636k). Cả 2 workflow (Daily Sheet Update + Yesterday Report) fail cùng lúc.

## Nguyên nhân
Google Sheets OAuth refresh token bị invalid/revoked đột ngột. n8n UI không phát hiện được trạng thái lỗi ở credential level — chỉ lộ ra lúc runtime thực sự gọi API.

Pattern nguy hiểm: **UI xanh, runtime chết, không có alert.**

Shared credential = shared risk: 1 credential expire → tất cả workflow dùng nó đều fail cùng lúc.

## Cách xử lý
1. Reconnect Google OAuth thủ công (Sign in with Google lại)
2. Chạy lại cả 2 workflow — data khớp lại bình thường
3. Dài hạn: migrate sang Google Service Account (không expire)

## Bài học
- UI credential status không đáng tin — nguồn sự thật duy nhất là execution logs.
- Cần Telegram Error Alert tự động — không chờ phát hiện thủ công.
- Google Service Account không expire, không cần reconnect → đáng tin hơn OAuth cho production.
- Khi data trong Sheet sai đột ngột → kiểm tra credential + execution logs trước khi nghi ngờ code.

## Status
Resolved — 2026-06-10. Service Account migration hoàn tất 2026-06-25.
