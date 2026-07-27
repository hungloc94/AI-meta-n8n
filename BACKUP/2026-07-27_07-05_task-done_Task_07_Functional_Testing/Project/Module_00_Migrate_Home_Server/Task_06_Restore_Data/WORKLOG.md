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
- [x] Credential "Header Auth account" (`httpHeaderAuth`) đã tồn tại trên n8n — xác nhận 2026-07-26 qua `n8n export:credentials --all` (id `HAOqERh15wR0pB4r`), không đọc giá trị secret
- Ghi chú: được tạo giữa lúc Bước 4 dừng và lúc verify lại — không rõ ai/khi nào tạo, không có log; nếu cần audit trail chính xác, hỏi Human/Grok

### Bước 5 — Google Sheets Service Account
- [x] Credential "Google Service Account account" (`googleApi`) đã tồn tại (id `kGB4h5UyGGcK5ibl`)
- [x] Node "HTTP Request Google Sheets" (trong Google Sheets Health Check 07:00) dùng `nodeCredentialType: googleApi` — xác nhận Service Account là credential đang active, không phải OAuth2 cũ

### Bước 6 — Verify credentials decrypt đúng
- [x] **2026-07-26 — Claude Code test thật:**
  - Backup workflow "Google Sheets Health Check 07:00" ra JSON trước khi sửa
  - Thêm tạm 1 node `n8n-nodes-base.executeWorkflowTrigger` nối vào "HTTP Request Google Sheets" (để CLI `n8n execute` có entry point chạy được, vì workflow gốc chỉ có Schedule Trigger)
  - Chạy `docker exec n8n n8n execute --id=EWvw3tX4oV11xXbs`
  - Kết quả: node "HTTP Request Google Sheets" chạy **thành công, không lỗi**, đi nhánh Success; node "Telegram Alert" **không chạy** → không gửi tin nhắn thật
  - Xoá node tạm, PUT lại workflow gốc, đối chiếu với bản backup: `nodes`, `connections`, `settings`, `active` khớp 100%
  - **Kết luận: Google Sheets Service Account credential decrypt và hoạt động đúng trên Home Server**
- [ ] Chưa test riêng Header Auth (Meta) và không cần thiết — sẽ được verify tự nhiên ở Task 07 Bước 3 (Meta Ads Daily Sheet Update dùng credential này)

### Bước 7 — Đối chiếu với inventory Task 01
- [ ] **BLOCKER cấu trúc** — Task 01 (Audit Windows n8n) vẫn "0% chưa bắt đầu" theo `Task_01/STATUS.md` → chưa có inventory chính thức để đối chiếu
- Ghi chú: `Project/OPS/CREDENTIAL_INVENTORY.md` đã tồn tại từ 2026-06-16 (trước đợt migrate này), có thể dùng tạm làm tham chiếu nếu Human đồng ý, nhưng đây không phải output chính thức của Task 01 trong Module 00 hiện tại
