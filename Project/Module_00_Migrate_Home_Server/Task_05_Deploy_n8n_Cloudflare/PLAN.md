# Plan — Task 05: Deploy n8n + Cloudflare

| Bước | Mô tả | Kết quả mong đợi |
|------|-------|-------------------|
| 1 | Chạy `docker compose up -d` theo thiết kế Task 04 | n8n container running |
| 2 | Tạo owner account, verify UI truy cập qua localhost:5678 | Đăng nhập thành công |
| 3 | Restart container, verify persistence (owner account + config còn nguyên) | Persistence PASS |
| 4 | Cấu hình Caddyfile — site block cho domain, reverse_proxy `localhost:5678` | Caddy proxy đúng |
| 5 | Cấu hình Cloudflare Tunnel Public Hostname trỏ domain → Caddy | Tunnel route domain thành công |
| 6 | Cập nhật N8N_HOST/N8N_PROTOCOL/N8N_EDITOR_BASE_URL/WEBHOOK_URL theo domain public, restart n8n | Env vars khớp domain |
| 7 | Verify truy cập n8n UI qua domain từ internet (máy khác, không phải server) | n8n UI load được qua domain public |
