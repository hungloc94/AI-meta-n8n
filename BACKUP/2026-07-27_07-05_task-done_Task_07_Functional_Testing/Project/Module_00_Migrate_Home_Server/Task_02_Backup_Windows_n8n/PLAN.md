# Plan — Task 02: Backup Windows n8n

| Bước | Mô tả | Kết quả mong đợi |
|------|-------|-------------------|
| 1 | Export toàn bộ workflow JSON qua n8n UI/CLI, đối chiếu danh sách Task 01 | Đủ số workflow JSON, không thiếu workflow nào |
| 2 | Backup file `.env` (chỉ copy nguyên file, không đọc/ghi giá trị secret ra nơi khác) | File `.env` backup nguyên vẹn |
| 3 | Backup `docker-compose.yml` | File compose backup nguyên vẹn |
| 4 | Snapshot Docker volume/database n8n | Volume backup có thể restore |
| 5 | Đối chiếu backup với inventory Task 01 — xác nhận không thiếu item | Checklist khớp 100% |
