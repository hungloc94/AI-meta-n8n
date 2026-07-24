# CASE-041: Legacy AutoMapInputData Risk

## Ngày phát hiện
2026-07-22

## Vấn đề
Một số workflow JSON đã verified trước đây vẫn có node Google Sheets dùng `mappingMode = autoMapInputData`. Nếu rebuild hoặc migrate theo artifact này mà không kiểm tra lại, workflow có thể ghi sai cột, ghi thừa cột, hoặc đụng vào vùng dữ liệu business được bảo vệ.

## Nguyên nhân
Artifact cũ phản ánh cấu hình đã từng chạy được tại một thời điểm, nhưng không nhất thiết khớp với rule vận hành hiện tại. Sau credential migration hoặc sheet schema change, n8n có thể reset mapping hoặc auto-map theo schema hiện tại, tạo rủi ro silent write sai.

## Cách xử lý
Khi rebuild/migrate workflow có Google Sheets write node:
- Kiểm tra `mappingMode`.
- Chuyển sang `Map Each Column Manually` theo rule hiện tại.
- Với Update Row, bắt buộc match bằng `Key`.
- Chỉ map các cột thuộc Nhóm A / Meta owns.
- Không map các cột business protected như dữ liệu Nhóm B.
- Verify bằng thay đổi dữ liệu thật hoặc Version History, không chỉ nhìn node PASS.

## Bài học
- Workflow “verified” cũ không tự động an toàn cho rebuild hiện tại.
- Auto mapping tiện nhưng nguy hiểm với Sheet có dữ liệu business nhập tay.
- Sau mỗi credential/schema change phải kiểm tra lại mapping.
- Mapping thủ công là chi phí nhỏ để tránh mất dữ liệu business.
