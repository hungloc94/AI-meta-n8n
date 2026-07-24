# PATTERN-006: Schedule Dependency Chain 07:00 → 07:05 → 07:30 → 08:13

## Mục đích
Dùng để sắp xếp các workflow scheduled có dependency theo thứ tự an toàn: health check trước, data sync sau, report sau cùng.

## Cấu trúc node
```text
07:00 Google Sheets Health Check
07:05 Meta API Health Check
07:30 Meta Ads Daily Sheet Update
08:13 Yesterday Report đọc Google Sheet
11:31 / 16:31 / 21:13 Today Report gọi Meta API trực tiếp
```

## Code mẫu
Trích từ 5 workflow production/backup đã phân tích.

### Google Sheets Health Check 07:00
```json
{ "field": "cronExpression", "expression": "0 7 * * *" }
```

### Meta API Health Check 07:05
```json
{ "field": "cronExpression", "expression": "5 7 * * *" }
```

### Daily Sheet Update 07:30
```json
{ "field": "cronExpression", "expression": "30 7 * * *" }
```

### Yesterday Report 08:13
```json
{ "field": "cronExpression", "expression": "13 8 * * *" }
```

### Today Report trong ngày
```json
[
  { "field": "cronExpression", "expression": "31 11 * * *" },
  { "field": "cronExpression", "expression": "31 16 * * *" },
  { "field": "cronExpression", "expression": "31 21 * * *" }
]
```

## Lưu ý quan trọng
- Health check phải chạy trước workflow phụ thuộc credential/API.
- Daily Sync phải chạy trước Yesterday Report để Sheet có dữ liệu mới nhất.
- Khoảng cách giữa sync và report nên đủ để xử lý retry, lỗi API và ghi Sheet.
- Today Report gọi Meta API trực tiếp nên phụ thuộc Meta health check, không phụ thuộc Google Sheet sync.
- Khi thay đổi lịch, cập nhật toàn bộ dependency chain, không đổi từng workflow riêng lẻ.

## CASE liên quan
- CASE-012: Kiến trúc Yesterday và Today
- CASE-017: Today Report trực tiếp Meta API
- CASE-039: Scheduled health check trước workflow phụ thuộc
- CASE-040: Artifact active không bằng runtime active
