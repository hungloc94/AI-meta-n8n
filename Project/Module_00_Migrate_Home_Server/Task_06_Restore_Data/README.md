# Task 06: Restore Data

## Mục tiêu
Restore toàn bộ workflow, credential, và biến môi trường từ backup Windows (Task 02)
vào n8n trên Home Server (đã deploy ở Task 05) — theo đúng thứ tự an toàn trong
MIGRATION_RUNBOOK.md.

## Lý do
MIGRATION_RUNBOOK.md — Credential Recreation Order và Workflow Import Order quy định
thứ tự bắt buộc để tránh phá credential continuity. CASE-025 (Migration Verification Gap):
bỏ sót node khi migrate credential từng gây incident — phải đối chiếu inventory Task 01.

## Phạm vi
- Restore file .env vào project directory (dựa trên backup Task 02 + template Task 04)
- Import toàn bộ workflow JSON (từ backup Task 02) — giữ `active=false`
- Recreate credentials theo thứ tự: Telegram → Meta Graph → Google Sheets OAuth2
- Verify credentials decrypt đúng (N8N_ENCRYPTION_KEY liên tục từ Task 01/04)
- Đối chiếu workflow/credential đã restore với inventory Task 01 — không thiếu item

## Ngoài phạm vi
- Test chức năng / activate workflow thật (Task 07)
- Chuyển traffic production sang Home Server (Task 08)

## Kết quả mong đợi
Toàn bộ workflow (inactive) + credential đã restore đầy đủ trên Home Server,
đối chiếu khớp 100% với inventory Task 01, sẵn sàng cho Functional Testing (Task 07).
