# CASE-034: $env UI Error Không Phải Runtime Error

## Vấn đề
`$env.TELEGRAM_CHAT_ID` trong n8n editor hiển thị `[ERROR: access to env vars denied]`. Người dùng tưởng biến không hoạt động.

## Nguyên nhân
n8n editor không cho preview env vars ở UI level — đây chỉ là UI restriction, không phải runtime error. Biến vẫn hoạt động bình thường nếu đã có trong `.env` container.

## Cách xử lý
- Bỏ qua UI error — đây là hành vi bình thường của n8n editor.
- Verify bằng cách execute node thật, không tin UI preview.
- Nếu node vẫn fail → kiểm tra `.env` file trong Docker container có đúng biến không.

## Bài học
- Thiếu env var (`TELEGRAM_CHAT_ID`, `META_ACCESS_TOKEN`) là nguyên nhân silent failure phổ biến trong scheduled workflow.
- Khi scheduled workflow fail silently → kiểm tra `.env` trước khi debug node logic.
- Khi Nhóm B (khach_hop_le, sdt, khach_chot) = 0 → hiển thị "Chưa cập nhật" thay vì 0 (business display rule).

## Status
Lesson learned — 2026-06-08.
