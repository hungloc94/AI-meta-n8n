# Task 01: Audit Windows n8n

## Mục tiêu
Lập inventory đầy đủ hệ thống n8n đang chạy trên máy Windows công ty (localhost:5678)
trước khi migrate sang Home Server.

## Lý do
CASE-025 (Migration Verification Gap): migrate trước đây từng bỏ sót node khi
không lập inventory đầy đủ, gây incident sáng hôm sau.
CASE-032 (Migration Definition of Done): Bước 1 bắt buộc của mọi migration là Inventory.

## Phạm vi
- Liệt kê toàn bộ workflow (active/inactive), tên, ID
- Liệt kê toàn bộ credential đang dùng — đối chiếu CREDENTIAL_INVENTORY.md
- Liệt kê tên biến môi trường trong .env (không ghi giá trị)
- Xác định version n8n, Docker Desktop, Node.js, OS Windows đang chạy

## Ngoài phạm vi
- Backup dữ liệu thực tế (Task 02)
- Sửa bất kỳ workflow/credential nào trên máy Windows — chỉ đọc, không sửa

## Kết quả mong đợi
Inventory đầy đủ, chính xác — làm căn cứ đối chiếu cho Restore Data (Task 06)
và Functional Testing (Task 07), tránh lặp lại CASE-025.
