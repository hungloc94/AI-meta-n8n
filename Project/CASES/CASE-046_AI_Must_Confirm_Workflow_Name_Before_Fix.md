# CASE-046: AI Phải Confirm Tên Workflow Trước Khi Fix

- Ngày phát hiện: 2026-07-26
- Ngày xác minh: 2026-07-26

## Vấn đề
AI đang làm workflow A, Human báo lỗi mới →
AI tự assume lỗi thuộc workflow A → sửa nhầm workflow → tốn thời gian.

## Nguyên nhân
AI không hỏi rõ tên workflow trước khi tiến hành fix.

## Cách xử lý
Rollback thay đổi nhầm. Fix đúng workflow theo tên Human cung cấp.

## Bài học dành cho AI
Khi Human báo lỗi mới trong lúc đang làm việc:
1. DỪNG lại
2. Hỏi rõ: "Lỗi này thuộc workflow nào?"
3. Chờ Human xác nhận tên workflow
4. Mới tiến hành fix

Không tự assume. Xác nhận tên workflow trước — fix sau.
