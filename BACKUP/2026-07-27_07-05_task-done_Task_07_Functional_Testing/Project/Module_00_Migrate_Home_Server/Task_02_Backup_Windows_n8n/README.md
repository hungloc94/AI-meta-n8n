# Task 02: Backup Windows n8n

## Mục tiêu
Backup toàn bộ dữ liệu n8n trên máy Windows trước khi migrate — đảm bảo
có thể rollback về trạng thái hiện tại nếu migration thất bại.

## Lý do
MIGRATION_RUNBOOK.md — Rollback Procedures: trước mọi thay đổi rủi ro phải
export workflow, backup .env, backup compose file, snapshot volume/database.
Migration sang Home Server là thay đổi rủi ro cao nhất trong Project.

## Phạm vi
- Export toàn bộ workflow (JSON) — cả active và inactive
- Backup file `.env` (D:\AI_Project\n8n-meta-ads\.env)
- Backup `docker-compose.yml`
- Snapshot Docker volume (database n8n)
- Đối chiếu với inventory đã lập ở Task 01 — đảm bảo không thiếu item

## Ngoài phạm vi
- Restore dữ liệu vào Home Server (Task 06)
- Tắt hoặc gỡ n8n khỏi máy Windows (chỉ thực hiện ở Task 08 sau khi verify PASS)

## Kết quả mong đợi
Bộ backup đầy đủ, có thể restore lại 100% trạng thái hiện tại của n8n Windows
nếu cần rollback.
