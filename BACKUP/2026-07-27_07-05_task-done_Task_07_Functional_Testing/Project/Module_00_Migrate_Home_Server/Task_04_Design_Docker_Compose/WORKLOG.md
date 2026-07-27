# WORKLOG — Task 04: Design Docker Compose

## 2026-07-25

### Bước 1 — Tạo project directory
- [x] Tạo `/home/buivu/n8n-docker/`
- [x] Verify: thư mục tồn tại, owner `buivu`
- Ghi chú: `/home/buivu/n8n-data` chưa tạo (deploy/restore — Task 05/06)
- Scout: Docker 29.1.3, Compose 2.40.3, port 5678 free, TZ Asia/Ho_Chi_Minh, cloudflared active

### Bước 2 — Thiết kế docker-compose.yml
- [x] Đề xuất v1 → Human feedback (secrets .env, pin 1.103.2, .env.example git)
- [x] Đề xuất v2 → Human OK nguyên văn + Docker Hub `n8nio/n8n:1.103.2` + versioned copy trong Task_04
- [x] Ghi runtime: `/home/buivu/n8n-docker/docker-compose.yml`
- [x] Ghi versioned: `Task_04_Design_Docker_Compose/docker-compose.yml`
- Chưa chạy `docker compose up` (ngoài phạm vi Task 04)

### Bước 3 — Thiết kế .env template
- [x] Ghi `.env.example` (không giá trị secret) tại:
  - Runtime: `/home/buivu/n8n-docker/.env.example`
  - Versioned: `Task_04_Design_Docker_Compose/.env.example`
- [x] Tạo `.env` runtime từ example (placeholder trống, chmod 600) — **chưa điền secret thật**
- [x] `.gitignore` repo root: ignore `.env`, keep `.env.example`
- [x] `/home/buivu/n8n-docker/.gitignore` ignore `.env`

### Bước 4 — N8N_ENCRYPTION_KEY continuity
- [x] Ghi chú bắt buộc:
  - **Nguồn key:** giá trị đang dùng trên Windows n8n (Task 01 Audit) — export từ `.env` / config Windows
  - **Không** `openssl rand` / generate key mới khi restore credentials cũ
  - Task 01 STATUS hiện ⏳ chưa xong → `.env` runtime vẫn trống `N8N_ENCRYPTION_KEY=`
  - **Gate trước Task 05/06:** Human dán đúng key cũ vào `~/n8n-docker/.env` trước `docker compose up` và trước restore volume
  - Rủi ro nếu sai key: credentials trong DB undecryptable (MIGRATION_RUNBOOK — N8N_ENCRYPTION_KEY Continuity)

### Bước 5 — Review Runbook (New-Machine Deployment Order bước 1–6)
| # | Runbook step | Kết quả thiết kế |
|---|--------------|------------------|
| 1 | Install compatible Docker | ✅ Docker 29.1.3 + Compose 2.40.3 trên Home Server (không cần Docker Desktop/WSL) |
| 2 | Verify WSL2 backend | ⏭️ N/A — Ubuntu bare metal |
| 3 | Create project directory | ✅ `~/n8n-docker/` |
| 4 | Restore `.env` trước credentials | ✅ Cấu trúc sẵn; **chưa** restore giá trị thật (chờ Task 01 + Human) |
| 5 | Confirm `N8N_ENCRYPTION_KEY` đúng | ✅ Quy tắc continuity đã ghi; **chưa** confirm giá trị (chờ Task 01) |
| 6 | Create `docker-compose.yml` | ✅ Runtime + versioned; pin `n8nio/n8n:1.103.2`; bind `127.0.0.1:5678:5678`; volume `/home/buivu/n8n-data`; healthcheck |

**Review verdict:** PASS thiết kế (file + quy tắc).  
**Chưa PASS runtime:** thiếu giá trị `.env` thật + `n8n-data` + Task 01 key — handover Task 05.

### Handover Task 05
- Copy đã versioned trong Git path Task_04
- Runtime: `cd ~/n8n-docker &&` điền `.env` → `mkdir -p ~/n8n-data` → `docker compose up -d` (khi Human approve)
- Không public tunnel cho đến khi local verify xong
