# CASE-006: TEST Workflow bắt buộc trước Production Changes

## Vấn đề
Hệ thống từng gặp runtime propagation uncertainty và duplicate-processing risk khi patch trực tiếp.

## Nguyên nhân
Production workflow có side effects thật (gửi Telegram, ghi Sheet). Patch sai = data corruption hoặc spam user.

## Cách xử lý
Bắt buộc dùng TEST workflow trước mọi production change. Verify đầy đủ rồi mới promote.

## Bài học
- Thêm import/testing step nhưng giảm risk đáng kể.
- TEST workflow nên dùng tab Sheet riêng (ví dụ: `BÁO_CÁO_QUẢNG_CÁO_TEST`).
- Mandatory rule — không có ngoại lệ.

## Status
Mandatory.
