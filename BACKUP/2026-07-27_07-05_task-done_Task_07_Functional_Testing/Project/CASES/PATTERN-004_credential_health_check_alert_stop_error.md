# PATTERN-004: Credential Health Check → Alert → Stop And Error

## Mục đích
Kiểm tra credential và API trước khi workflow phụ thuộc chạy.
Phát hiện lỗi sớm, tránh silent failure.

## Cấu trúc node
Schedule Trigger (trước workflow phụ thuộc)
→ HTTP Request test credential
→ IF PASS → End
→ IF FAIL → Telegram Alert → Stop And Error

## Code mẫu

### Node Telegram Alert khi FAIL
const time = new Date().toLocaleString('vi-VN',
  {timeZone: 'Asia/Ho_Chi_Minh'});
const msg = `⚠️ Health Check FAIL\n`
  + `Thời gian: ${time}\n`
  + `Lỗi: ${$json.error || 'Không kết nối được'}\n`
  + `Hành động: Kiểm tra credential trong n8n`;
return [{json: {message: msg}}];

## Lưu ý quan trọng
- Chạy trước workflow phụ thuộc
  GS Health 07:00 → Meta Health 07:05 → Sync 07:30
- Test runtime thật, không tin UI credential
- Fail path bắt buộc: Alert → Stop And Error
  (execution phải đỏ, không được PASS khi thực tế FAIL)
- Không reconnect Google OAuth nếu đã dùng Service Account
- Stop And Error giúp phát hiện ngay trên dashboard n8n

## CASE liên quan
CASE-022, CASE-026, CASE-030, CASE-034, CASE-039, CASE-040