# CASE-023: ECONNRESET — Telegram Send Fail

## Vấn đề
`Meta Report Today Scheduled` fail tại node `Send Report Telegram`. Lỗi `ECONNRESET` — báo cáo không được gửi ở 1 trong 3 slot.

## Nguyên nhân
Kết nối tạm thời bị ngắt giữa n8n container và Telegram API. Đây là lỗi mạng transient, không phải lỗi hệ thống.

Đã loại trừ:
- Token OK (TEST PASS)
- Cron OK (slot 11:31 PASS)
- Workflow logic OK

## Cách xử lý
Bật Retry On Fail trong node `Send Report Telegram`:
- Retry On Fail: `true`
- Max Attempts: `3`
- Wait Between Attempts: `2000ms`

Sau đó patch tương tự cho Yesterday Report (cùng pattern).

## Bài học
- ECONNRESET với external API là lỗi mạng tạm thời — retry là giải pháp đúng.
- Mọi node gửi request đến external API trong production PHẢI có Retry On Fail.
- Khi fix 1 workflow, kiểm tra ngay các workflow tương tự có cùng pattern — tránh bỏ sót.
- Phân biệt: TEST thủ công PASS + slot tiếp theo PASS = transient, không phải systemic.

## Status
Resolved — 2026-06-17.
