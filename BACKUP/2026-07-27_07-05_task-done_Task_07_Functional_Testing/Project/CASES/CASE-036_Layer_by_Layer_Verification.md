# CASE-036: Layer-by-Layer Runtime Verification

## Vấn đề
Tích hợp nhiều layer cùng lúc (Data Source + Normalize + Engine + Presentation) → khi lỗi xảy ra, không biết lỗi ở layer nào.

## Nguyên nhân
Không verify từng layer độc lập trước khi ghép. Skip layer → lỗi propagate xuống downstream → forensic debugging tốn thời gian.

## Cách xử lý
Verify theo thứ tự layer, mỗi layer phải PASS trước khi sang tiếp:

```
Data Source (Google Sheet)
↓ PASS
Normalize Layer
↓ PASS
Engine Layer (Date Range Engine)
↓ PASS
Presentation Layer (Render Telegram)
↓ PASS
Production
```

Không tích hợp sang layer tiếp theo khi layer hiện tại chưa PASS.

## Bài học
- Áp dụng cho mọi component mới: Engine, Parser, Report, Dashboard, API...
- Verify bằng runtime execution thật, không chỉ đọc code.
- Đối chiếu với dữ liệu thật ở mỗi layer (filteredCount, spend, impressions...).
- Giúp cô lập lỗi nhanh, giảm rủi ro, tránh ảnh hưởng Production.

## Status
Lesson learned — 2026-07-03.
