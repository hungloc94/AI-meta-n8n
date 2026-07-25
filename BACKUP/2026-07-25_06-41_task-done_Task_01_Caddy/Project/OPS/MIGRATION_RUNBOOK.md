# Migration Runbook

## Role
Critical deployment clone và recovery guide.

Sai migration order có thể phá credential continuity.

## New-Machine Deployment Order
1. Install compatible Docker Desktop.
2. Verify WSL2 backend operational.
3. Create project directory.
4. Restore `.env` trước khi tạo credentials.
5. Confirm `N8N_ENCRYPTION_KEY` đúng.
6. Create `docker-compose.yml`.
7. Start n8n locally only.
8. Create owner account.
9. Verify persistence after restart.
10. Recreate credentials theo safe order.
11. Import workflow as TEST branch only.
12. Verify workflow `active=false`.
13. Run controlled node-by-node tests.
14. Run supervised runtime activation để verify polling continuity.
15. Approve activation only after routing và duplicate-processing review.

## Environment Variable Restoration Order
1. `N8N_ENCRYPTION_KEY`
2. n8n host/protocol/URL variables
3. timezone variables
4. Telegram/Meta/Google/Cloudflare variables only when their phase is approved

## N8N_ENCRYPTION_KEY Continuity
- Không generate new key tùy tiện.
- Credentials được encrypted bằng active key tại thời điểm creation.
- n8n không auto-migrate credentials khi key thay đổi.
- Nếu key changes, existing credentials có thể undecryptable.

## Credential Recreation Order
1. Telegram credential
2. Meta Graph credential
3. Google Sheets OAuth2 re-authorization
4. Cloudflare token only if public exposure is approved

## Workflow Import Order
1. Import TEST workflow.
2. Confirm inactive state.
3. Verify credentials không auto-trigger runtime.
4. Test node-by-node.
5. Verify runtime polling continuity dưới supervised activation.
6. Import production only after TEST passes.

## Docker/PowerShell Startup Strategy
- Dùng Docker Compose từ project directory.
- Giữ bind local-only trong recovery: `127.0.0.1:5678:5678`.
- Không enable tunnel trong local recovery.
- Prefer explicit verification commands sau mỗi restart.

## Recovery Verification Checklist
- Docker daemon running
- n8n container running
- localhost UI returns 200
- volume exists
- database exists and is non-empty
- owner setup persists after restart
- credentials decrypt after restart
- workflow remains inactive
- no public exposure
- no stale replay after runtime restart

## Rollback Procedures
Before risky changes:
- export workflow
- backup `.env`
- backup compose file
- snapshot volume/database if needed

Rollback options:
- restore previous workflow export
- restore previous `.env`
- restore previous volume snapshot
- keep workflow inactive until verified

## Backup Restoration Process
1. Stop n8n.
2. Restore environment files.
3. Restore volume/database snapshot.
4. Start n8n locally.
5. Verify decrypt/login/workflow state.
6. Do not activate until tests pass.

## Controlled Activation Procedure
1. TEST workflow passes node-by-node.
2. Duplicate-processing review passes.
3. Credentials verified.
4. Telegram send behavior verified.
5. Polling continuity verified under supervised runtime activation.
6. Human approval granted.
7. Activate only one intended runtime workflow.
