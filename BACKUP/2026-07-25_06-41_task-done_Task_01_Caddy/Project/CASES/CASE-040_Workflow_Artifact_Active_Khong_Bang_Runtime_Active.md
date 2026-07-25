# CASE-040: Workflow Artifact active Không Bằng Runtime Active

## Ngày phát hiện
2026-07-22

## Vấn đề
Khi đọc workflow JSON backup/export, field `active=true` hoặc `active=false` chỉ phản ánh trạng thái trong artifact tại thời điểm export. Nếu dùng field này để kết luận workflow production hiện đang active/inactive, AI có thể hiểu sai trạng thái runtime thực tế.

## Nguyên nhân
Workflow JSON là artifact lưu trữ, không phải nguồn sự thật runtime. Sau khi export, workflow có thể đã được import lại, đổi trạng thái, thay credential, bị disable, hoặc có bản khác đang chạy trong n8n.

## Cách xử lý
- Dùng JSON artifact để hiểu cấu trúc node, logic, lịch chạy và credential dependency.
- Không dùng `active` trong artifact làm bằng chứng runtime cuối cùng.
- Khi cần xác nhận trạng thái thật, phải kiểm tra runtime n8n bằng danh sách workflow, execution logs, hoặc execute/activation thực tế nếu được Human duyệt.
- Khi báo cáo, ghi rõ: “Trạng thái trong JSON artifact” nếu chưa verify runtime.

## Bài học
- Artifact state ≠ runtime state.
- Source of truth cho production là runtime n8n và execution logs, không phải file backup.
- Mọi migration/rebuild phải có bước verify runtime sau import.
- Nếu chưa verify runtime thì đánh dấu là `TODO verification`, không kết luận chắc chắn.
