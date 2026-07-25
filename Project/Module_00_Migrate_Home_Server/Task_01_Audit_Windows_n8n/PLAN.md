# Plan — Task 01: Audit Windows n8n

| Bước | Mô tả | Kết quả mong đợi |
|------|-------|-------------------|
| 1 | Liệt kê toàn bộ workflow trên n8n Windows (active/inactive, tên, ID) | Danh sách đầy đủ, khớp Project/STATUS.md (6 workflow hiện có) |
| 2 | Liệt kê toàn bộ credential đang dùng, đối chiếu CREDENTIAL_INVENTORY.md | Bảng credential đầy đủ, không bỏ sót node nào |
| 3 | Liệt kê tên biến môi trường trong .env (không ghi giá trị) | Danh sách tên biến khớp ENVIRONMENT_REGISTRY.md |
| 4 | Xác định version n8n, Docker Desktop, Node.js, OS Windows | Bản ghi version đầy đủ để đối chiếu compatibility khi cài trên Ubuntu |
| 5 | Tổng hợp thành inventory hoàn chỉnh, đối chiếu OPS docs hiện có, ghi sai khác nếu có | Inventory PASS — sẵn sàng làm căn cứ cho Task 02-08 |
