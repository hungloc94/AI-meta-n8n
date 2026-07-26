# WORKLOG — Task 06: Restore Data

## 2026-07-25

### Bước 1 — Restore .env
- [x] `.env` tại `/home/buivu/n8n-docker/.env` (Human đã điền)
- [x] Đối chiếu key với `.env.example` (Task 04): **khớp đầy đủ tên biến**
- [x] Biến critical filled: `N8N_ENCRYPTION_KEY`, Basic Auth, Host/URL, TZ, Telegram, Meta
- [x] EMPTY (không chặn n8n core): `GOOGLE_SHEETS_SPREADSHEET_ID`, `GOOGLE_SHEETS_RANGE`
- [x] n8n healthy: `/healthz` → 200
- Nguồn: Task 02 backup **chưa có** — dùng `.env` runtime + `Project/WORKFLOWS/` theo Human

### Bước 2 — Import workflow JSON
- [x] Source: `~/AI-meta-n8n/Project/WORKFLOWS/` (7 file)
- [x] Preprocess: force `active=false` trước import (Runbook)
- [x] `n8n import:workflow --separate --input=/tmp/import-workflows`
- [x] **Successfully imported 7 workflows**
- [x] Verify DB `workflow_entity.active` = 0 cho cả 7:

| id | name | active |
|----|------|--------|
| EWvw3tX4oV11xXbs | Google Sheets Health Check 07:00 | false |
| sgzJXYTLQKk4hWp3 | Meta API Health Check 07:05 | false |
| rz3Wya5lFay7ShVL | Meta Ads Daily Sheet Update 7:30 2 | false |
| VKZzSlAvsb4GiyMs | Meta Report Today Scheduled 11:31 16:31 21:13 | false |
| VAmww3hiXjLvxIt4 | Meta Report VERIFIED | false |
| 9l8RHgYuRZYz0TVr | Meta Report Yesterday Scheduled 1 | false |
| EXLgZOPrMl8hK8TT | Meta Report Yesterday Scheduled 1 TEST - Date Range Engine V1 | false |

### Bước 3 — Recreate credential Telegram
- [x] Phân tích WORKFLOWS: **không có** n8n credential type Telegram
- [x] Node Telegram dùng `httpRequest` + `$env.TELEGRAM_BOT_TOKEN` / `$env.TELEGRAM_CHAT_ID`
- [x] Env vars đã filled ở Bước 1 → **không cần tạo Telegram credential trong n8n UI**
- Kết luận: Bước 3 = N/A (env-based), không blocker

### Bước 4 — Recreate credential Meta Graph (Header Auth)
- [ ] **BLOCKER** — cần thao tác trong n8n UI (hoặc API key) — xem báo cáo Human

### Bước 5–7
- [ ] Chưa làm (dừng tại Bước 4)
