# CASE-020: KPI Reports Always Returning Zero

## Vấn đề
Telegram report trả KPI = 0 toàn bộ (0đ chi tiêu, 0 kết quả, 0 CPA) cho mọi lệnh báo cáo.

## Nguyên nhân
6 lỗi chồng lên nhau cùng lúc:
1. Google Sheets API trả raw array rows — downstream expect object schema
2. Parser thiếu DD/MM support — `báo cáo 24/03` bị classify thành unknown
3. `Parse & Route` thiếu DD/MM parser
4. Polling staticData bị forced reset → duplicate-processing risk
5. `Send Report Telegram` vẫn còn debug placeholder `[NODE=REPORT]` thay vì `$json.message`
6. Downstream nodes depend vào raw third-party response shape

## Cách xử lý
- Thêm `Normalize Sheet Data` node ngay sau Google Sheet read
- Thêm DD/MM parser + report regex support
- Remove forced staticData reset
- Patch Send Report Telegram: `[NODE=REPORT]` → `$json.message`
- Giữ workflow inactive trong recovery

## Bài học
- Silent logic bugs nguy hiểm hơn crashes — 6 lỗi chồng nhau mà không có crash nào.
- Business-zero KPI (campaign thật sự có 0 leads) ≠ pipeline failure — phải phân biệt.
- Recovery baselines nên export ngay sau mỗi verified milestone.
- Debug placeholders phải remove trước import testing.

## Status
Resolved — 2026-05-27. Recovery baseline: `META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json`.
