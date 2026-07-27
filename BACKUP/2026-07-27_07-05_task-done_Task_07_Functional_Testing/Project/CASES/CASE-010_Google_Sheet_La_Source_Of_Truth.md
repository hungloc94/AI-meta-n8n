# CASE-010: Google Sheet là Nguồn Sự Thật Duy Nhất

## Vấn đề
Risk mất dữ liệu business khi Meta sync workflow ghi đè toàn bộ hàng.

## Nguyên nhân
Google Sheet chứa cả dữ liệu từ Meta Ads API lẫn dữ liệu kinh doanh do người vận hành nhập tay (chất lượng khách, ghi chú, công thức tính chi phí). Ghi đè toàn bộ hàng = xóa dữ liệu kinh doanh vĩnh viễn.

## Cách xử lý
**Workflow phải thích nghi với cấu trúc Sheet — không phải ngược lại.** Selective update chỉ ghi vào cột thuộc về Meta API.

## Bài học
- Cần logic selective update, phức tạp hơn ghi đè toàn hàng nhưng an toàn.
- Sheet structure do business quyết định, không phải engineering convenience.
- Documented trong `DATA_SCHEMA_RULES.md`.

## Status
Accepted — 2026-05-29.
