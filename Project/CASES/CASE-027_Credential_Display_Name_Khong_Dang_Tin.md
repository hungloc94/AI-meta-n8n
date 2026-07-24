# CASE-027: Credential Display Name Không Đáng Tin

## Vấn đề
Tưởng đã migrate sang Service Account vì tên credential hiển thị là "Google Sheets Service Account". Thực tế node vẫn dùng OAuth2.

## Nguyên nhân
Tên hiển thị (display name) của credential trong n8n có thể không phản ánh đúng Credential Type bên trong. Người dùng đặt tên gì cũng được — n8n không validate tên vs type.

## Cách xử lý
Luôn mở bút chì (edit icon) cạnh credential → xác nhận Credential Type thật:
- `Google Service Account API` → ✅ PASS
- `Google Sheets account (OAuth)` → ❌ vẫn dùng OAuth cũ

Chỉ khi cả 3 điều kiện đều PASS mới kết luận migration hoàn tất:
1. Credential Type đúng (mở bút chì xác nhận)
2. Node execute thành công
3. Dữ liệu thực tế thay đổi trong Google Sheet

## Bài học
- Không tin display name — phải verify Credential Type thật.
- "Workflow chạy thành công" ≠ "Service Account đang được dùng" — node có thể PASS với OAuth cũ.
- Anti-pattern: kết luận migration PASS từ hành vi workflow mà không kiểm tra credential type.

## Status
Lesson learned — 2026-06-22.
