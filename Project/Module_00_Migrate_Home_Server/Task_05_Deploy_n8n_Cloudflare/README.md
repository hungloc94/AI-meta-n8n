# Task 05: Deploy n8n + Cloudflare

## Mục tiêu
Deploy n8n bằng Docker Compose (thiết kế ở Task 04) trên Home Server, và hoàn thiện
kết nối public qua Cloudflare Tunnel + Caddy — tunnel đã Healthy nhưng chưa có
Public Hostname.

## Phạm vi
- Chạy `docker compose up -d` theo thiết kế Task 04
- Tạo owner account cho n8n, verify persistence sau restart
- Cấu hình Caddyfile: thêm site block cho domain, reverse_proxy vào localhost:5678
- Cấu hình Cloudflare Tunnel Public Hostname trỏ domain → Caddy
- Cập nhật N8N_HOST, N8N_PROTOCOL, N8N_EDITOR_BASE_URL, WEBHOOK_URL cho đúng domain public
- Verify truy cập n8n UI qua domain public

## Ngoài phạm vi
- Restore dữ liệu workflow/credential thật (Task 06)
- Functional Testing đầy đủ (Task 07)
- Tắt hoặc chuyển production khỏi Windows (Task 08)

## Lưu ý
- Cần domain đã add vào Cloudflare account — chưa xác nhận trong Project docs,
  cần Human/Grok xác nhận trước khi cấu hình Public Hostname.
- N8N_ENCRYPTION_KEY phải dùng đúng key từ Task 01 — không tạo mới (continuity).

## Kết quả mong đợi
n8n chạy trên Home Server, truy cập được qua domain public qua Cloudflare Tunnel + Caddy.
Workflow/credential thật chưa restore (Task 06 riêng).
