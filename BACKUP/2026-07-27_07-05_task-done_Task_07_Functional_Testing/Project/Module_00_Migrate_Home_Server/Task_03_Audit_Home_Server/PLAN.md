# Plan — Task 03: Audit Home Server

| Bước | Mô tả | Kết quả mong đợi |
|------|-------|-------------------|
| 1 | Kiểm tra Docker Engine + Docker Compose plugin (`docker --version`, `docker compose version`) | Xác định rõ đã cài hay chưa |
| 2 | Kiểm tra dung lượng ổ đĩa còn trống (`df -h`) | Đủ dung lượng cho volume n8n |
| 3 | Kiểm tra port 5678 chưa bị chiếm dụng (`ss -tlnp`) | Port free, sẵn sàng bind cho n8n |
| 4 | Xác nhận trạng thái Caddy + cloudflared hiện tại (`systemctl status caddy cloudflared`) | Caddy active/enabled, cloudflared tunnel Healthy — ghi nhận thiếu Public Hostname |
| 5 | Kiểm tra timezone server (`timedatectl`) | Timezone = Asia/Ho_Chi_Minh hoặc ghi nhận cần đổi |
| 6 | Tổng hợp báo cáo readiness | GO/NO-GO rõ ràng cho Task 04 |
