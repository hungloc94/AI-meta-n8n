# Task 03: Audit Home Server

## Mục tiêu
Xác nhận Home Server (Ubuntu 24.04, RAM 16GB, SSD 256GB) đã sẵn sàng về hạ tầng
để cài Docker + n8n, trước khi thiết kế docker-compose.yml (Task 04).

## Phạm vi
- Kiểm tra Docker Engine + Docker Compose plugin đã cài chưa
- Kiểm tra dung lượng ổ đĩa còn trống
- Kiểm tra port 5678 chưa bị chiếm dụng
- Xác nhận trạng thái Caddy (Task 01 DONE) và cloudflared (đã cài, tunnel Healthy,
  chưa có Public Hostname)
- Kiểm tra timezone server khớp Asia/Ho_Chi_Minh (Project/RULES.md)

## Ngoài phạm vi
- Cài Docker nếu chưa có — nếu phát sinh, báo Human trước khi cài (ngoài phạm vi audit thuần túy)
- Thiết kế docker-compose.yml (Task 04)
- Cấu hình Public Hostname cho cloudflare tunnel (Task 05)

## Kết quả mong đợi
Báo cáo readiness đầy đủ — GO/NO-GO cho Task 04 Design Docker Compose.
