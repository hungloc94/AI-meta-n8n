# AI Start Here

## Role
File này là onboarding entrypoint chính cho AI agents làm việc với `n8n-meta-ads`.

Đây là memory router, không phải knowledge dump. Đọc file này trước, sau đó theo authority hierarchy.

## Project Overview
`n8n-meta-ads` là human-supervised AI-assisted Marketing Intelligence Operating System. Hệ thống dùng n8n, Telegram, Google Sheets, và Meta Ads data để hỗ trợ reporting, recovery, forensic debugging, và future AI-assisted automation.

Human vẫn là final authority.

## Current Operational Phase
Đang ở phase recovery và production-readiness governance cho MSP build.

Current focus:
- workflow vẫn inactive trong recovery
- supervised runtime polling continuity validation đã completed
- schema normalization sau Google Sheets ingestion
- production promotion readiness review
- credential và migration continuity

## Required Reading Order
1. `CURRENT_STATUS.md`
2. `GOVERNANCE_RULES.md`
3. `DECISIONS.md`
4. `INCIDENTS_AND_RECOVERY.md`
5. `TEST_PLAYBOOK.md`
6. `MIGRATION_RUNBOOK.md`
7. `ENVIRONMENT_REGISTRY.md`
8. Các file khác khi cần

## Authoritative Files
- `CURRENT_STATUS.md`: operational truth cao nhất
- `GOVERNANCE_RULES.md`: non-overridable safety constraints
- `DECISIONS.md`: architecture reasoning
- `INCIDENTS_AND_RECOVERY.md`: forensic history
- `LESSONS_LEARNED.md`: reusable engineering knowledge

## AI Agents Must Never
- Patch production trực tiếp.
- Activate workflows trong forensic recovery.
- Modify credentials tùy tiện.
- Rewrite polling architecture nếu chưa review.
- Touch forensic-only webhook reference files nếu chưa được yêu cầu rõ.
- Tin silent success nếu chưa verify output data.
- Store raw chat noise as memory.
- Invent incidents, deployment details, hoặc architecture decisions.
- Add secrets, tokens, cookies, encryption keys, hoặc sensitive values vào vault.

## Update Protocol After Important Changes
Sau mỗi important change, AI phải xác định:
- memory files nào cần update
- vì sao cần update
- importance: critical, high, medium, low
- change có ảnh hưởng governance, recovery, migration, hoặc active operational state không

Không lưu raw conversations. Chỉ lưu distilled operational knowledge.

## Governance Summary
- Continuity before optimization.
- Observability before patching.
- TEST workflow before production workflow.
- Canonical schemas before downstream KPI logic.
- Execute Step testing để isolate lỗi, sau đó real runtime activation để verify polling continuity.

## Memory Scope
Project operational memory self-contained trong folder:
`D:\AI_Project\Obdisian_2\My Vault\Project\AI_Meta_n8n_autoamation`

Không dùng `D:\AI_Project\Obdisian_2\My Vault\AI_MEMORY` cho project này.

## Operational Philosophy
Hệ thống phải recoverable cho future AI agent trên máy mới với ambiguity thấp nhất. Mọi operational lesson quan trọng nên trở thành playbook, không nằm rải rác trong chat.
