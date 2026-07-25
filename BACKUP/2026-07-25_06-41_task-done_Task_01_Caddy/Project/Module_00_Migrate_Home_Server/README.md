# Module 00: Migrate lên Home Server

## Mục tiêu
Chuyển toàn bộ hệ thống n8n từ máy Windows công ty
sang Home Server MSI GE chạy Ubuntu 24.04 LTS.

## Lý do
- AI CLI SSH vào thẳng, không bị sandbox Windows
- Uptime 24/7, không phụ thuộc máy tính bật
- Ubuntu ổn định hơn cho Docker và n8n
- Không cần VPS — dùng Cloudflare Tunnel

## Phần cứng Home Server
- Model: MSI GE
- RAM: 16GB
- Storage: SSD 256GB
- Network: LAN trực tiếp (cắm dây)
- OS: Ubuntu Server 24.04 LTS
- Tailscale IP: 100.105.119.88

## Kết quả mong đợi
Toàn bộ 6 workflow chạy đúng trên server.
AI CLI truy cập trực tiếp qua SSH không cần hỏi quyền.