# Task 04: Design Docker Compose

## Mục tiêu
Thiết kế file docker-compose.yml và cấu trúc .env cho n8n chạy trên Home Server,
tuân theo MIGRATION_RUNBOOK.md — chưa deploy thật (Task 05).

## Lý do
MIGRATION_RUNBOOK.md — New-Machine Deployment Order yêu cầu: tạo project directory,
restore .env trước khi tạo credentials, confirm N8N_ENCRYPTION_KEY đúng trước khi
tạo docker-compose.yml. N8N_ENCRYPTION_KEY Continuity: không generate key mới —
credentials cũ sẽ không decrypt được nếu key thay đổi.

## Phạm vi
- Tạo project directory trên Home Server
- Thiết kế docker-compose.yml: image n8n, volume mapping cho persistence,
  bind local-only (127.0.0.1:5678:5678) theo Runbook — chưa public qua tunnel
- Thiết kế cấu trúc .env (tên biến, không điền giá trị thật vào file trong repo)
- Xác nhận N8N_ENCRYPTION_KEY sẽ dùng đúng key từ Task 01 audit (không tạo mới)
- Timezone: GENERIC_TIMEZONE=Asia/Ho_Chi_Minh, TZ=Asia/Ho_Chi_Minh

## Ngoài phạm vi
- Chạy `docker compose up` thật (Task 05)
- Restore dữ liệu vào volume (Task 06)
- Cấu hình Cloudflare Public Hostname / Caddyfile domain (Task 05)

## Kết quả mong đợi
docker-compose.yml + .env template sẵn sàng, đã review theo Runbook,
chưa deploy — sẵn sàng cho Task 05.
