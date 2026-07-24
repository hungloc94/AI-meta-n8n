# CASE-025: Migration Verification Gap — Bỏ sót node khi migrate

## Vấn đề
Sáng 2026-06-25: Health Check báo FAILED, không có data Sheet lúc 07:30, không có Yesterday Report lúc 08:13. Lỗi: `EAUTH — authorization grant invalid, expired, revoked`.

## Nguyên nhân
Khi migrate Google OAuth → Service Account (2026-06-24), chỉ migrate node Append Row và Update Row. Bỏ sót:
- Node "Đọc toàn bộ Sheet" (HTTP Request) — Daily Sheet Update
- Node "Đọc Google Sheet" (HTTP Request) — Yesterday Report
- Node đọc trong Health Check
- Message Telegram Health Check vẫn hướng dẫn reconnect OAuth cũ

**Process root cause:** Không lập inventory trước migration. Kết luận hoàn tất quá sớm sau khi workflow TEST PASS.

## Cách xử lý
- Migrate toàn bộ node còn sót sang Service Account
- Cập nhật Health Check message cho đúng kiến trúc mới
- Verify từng workflow production
- Chờ scheduler ngày tiếp theo chạy đủ chu kỳ

## Bài học
- **Inventory trước migration là bắt buộc** — nếu có inventory từ đầu, không thể bỏ sót.
- Workflow TEST PASS ≠ Production đã migrate — đây là 2 workflow khác nhau.
- Monitoring phải đồng bộ kiến trúc production — Health Check báo lỗi credential cũ = false alarm.
- Lifecycle chuẩn: Inventory → Migration → Production Execute → Scheduler Verify → Documentation → Archive.

## Status
Resolved — 2026-06-25.
