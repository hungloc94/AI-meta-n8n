# CASE-018: CPC là Metric bắt buộc trong Báo cáo Telegram

## Vấn đề
Cần đánh giá nhanh campaign efficiency qua Telegram mà không cần mở Ads Manager.

## Nguyên nhân
CPC (Cost Per Click) là metric cơ bản để đánh giá hiệu quả chi phí quảng cáo theo lượt click. Thiếu CPC = phải mở Ads Manager thủ công mỗi lần.

## Cách xử lý
- CPC = `totalSpend / totalClick`.
- Nếu `totalClick = 0` → CPC = 0 (bảo vệ divide-by-zero).
- Hiển thị trong tất cả báo cáo Telegram (Today và Yesterday).

## Bài học
- Luôn bảo vệ divide-by-zero cho mọi derived metric.
- Metric mới thêm vào phải verify khớp Ads Manager trước khi deploy.

## Status
Accepted — 2026-06-11.
