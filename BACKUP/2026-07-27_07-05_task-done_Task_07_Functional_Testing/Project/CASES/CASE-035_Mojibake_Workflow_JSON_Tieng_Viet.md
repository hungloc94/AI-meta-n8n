# CASE-035: Mojibake khi xử lý Workflow JSON tiếng Việt

## Vấn đề
Sau quy trình export → chỉnh sửa → import workflow JSON:
- "Tính Ngày Hôm Qua" → "TÃ­nh NgÃ y HÃ´m Qua"
- "BÁO_CÁO_QUẢNG_CÁO" → "BÃO_CÃO_QUẢNG_CÁO"
- Node đọc Sheet báo lỗi: "Unable to parse range"

## Nguyên nhân
Root cause **chưa xác định**. Mojibake xuất hiện sau quy trình export → chỉnh sửa → import. Không kết luận nguyên nhân cụ thể khi chưa có bằng chứng.

## Cách xử lý
Sửa thủ công các chuỗi bị mojibake trong JSON trước khi import.

Checklist sau khi export/import workflow JSON:
- ☐ Kiểm tra tất cả node Display Name có tiếng Việt
- ☐ Kiểm tra tất cả URL parameter có tiếng Việt (đặc biệt Sheet name)
- ☐ Kiểm tra tất cả expressions có chuỗi tiếng Việt
- ☐ Execute thử node có tiếng Việt trong URL trước khi activate Production

## Bài học
- Tiếng Việt trong workflow JSON là điểm rủi ro khi export/import.
- Luôn verify chuỗi tiếng Việt sau mỗi lần xử lý JSON workflow.
- Khi gặp lỗi "Unable to parse range" → kiểm tra Sheet name trong URL có bị mojibake không.

## Status
Lesson learned — 2026-07-03. Root cause chưa xác định.
