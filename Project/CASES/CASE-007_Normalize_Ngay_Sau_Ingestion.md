# CASE-007: Normalize ngay sau External Ingestion

## Vấn đề
KPI corruption risk và silent aggregation failures khi dùng raw data từ external sources.

## Nguyên nhân
Array-based spreadsheet structures không có type safety. Field names có thể thay đổi, thiếu, hoặc sai thứ tự mà không báo lỗi.

## Cách xử lý
Normalize thành canonical schema objects ngay sau ingestion — trước khi bất kỳ downstream logic nào chạm vào data.

## Bài học
- Giảm mạnh downstream reliability risk.
- Forensic debugging time giảm vì lỗi được isolate tại normalize layer.
- Áp dụng cho mọi external data source, không chỉ Google Sheets.

## Status
Accepted and verified in end-to-end recovery.
