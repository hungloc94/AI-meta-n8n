# Plan — Task 07: Functional Testing

Test theo đúng thứ tự dependency: 07:00 → 07:05 → 07:30 → 08:13 → Today Report → Meta Report VERIFIED

| Bước | Mô tả | Kết quả mong đợi |
|------|-------|-------------------|
| 1 | Test thủ công Google Sheets Health Check (07:00) | PASS, không có alert Telegram |
| 2 | Test thủ công Meta API Health Check (07:05) | PASS, không có alert Telegram |
| 3 | Test thủ công Meta Ads Daily Sheet Update (07:30) | Sheet cập nhật đúng dữ liệu, không ghi đè Nhóm B |
| 4 | Test thủ công Yesterday Report (08:13) | Telegram nhận báo cáo, KPI khớp Sheet |
| 5 | Test thủ công Today Report (11:31 / 16:31 / 21:13) | Telegram nhận báo cáo, KPI khớp Meta API trực tiếp |
| 6 | Xác nhận Meta Report VERIFIED vẫn lỗi — không fix, không activate, ghi vào WORKLOG | Trạng thái lỗi được ghi nhận rõ ràng, không activate |
