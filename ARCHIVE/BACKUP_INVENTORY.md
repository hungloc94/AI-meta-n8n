# Backup Inventory

## Role
Inventory cho backup và forensic artifacts.

## Workflow Exports
| Artifact | Location | Purpose | Status |
| --- | --- | --- | --- |
| `POLLING_IFCHAIN_FALLBACK_v20260513.json.json` | `IMPORTS/` | Current TEST workflow lineage | Active recovery artifact |
| `workflow-clean-post-rotation-2026-05-24.json` | `docs/backups/` | Backup/simple test export | Reference |

## Forensic Only Files
| Artifact | Rule |
| --- | --- |
| `WEBHOOK_CLOUDFLARE_REFERENCE_20260508.json` | FORENSIC ONLY; không modify nếu chưa explicit instruction |

## Credential Backup Status
- Telegram token source: TODO verify secure vault/source.
- Meta long-lived token source: recovered artifact, TODO verify current validity.
- Meta App ID/App Secret source: recovered artifact, TODO verify ownership.
- Google OAuth: re-authorize; không rely vào old encrypted credential.
- Cloudflare token: recovered artifact, deferred tới public exposure phase.

## Database Backup Status
- n8n runtime database nằm trong Docker volume.
- TODO establish recurring volume/database backup procedure.

## Snapshot Dates
TODO maintain dated snapshots cho:
- workflow exports
- `.env` không expose secrets
- compose file
- n8n database/volume backups

## Recovery Artifacts
Recovered artifacts phải track bằng location và sensitivity. Không paste raw secrets vào file này.

## Recovery Baseline Export: 2026-05-27

| Field | Value |
| --- | --- |
| Filename | `META_REPORT_TEST_RECOVERY_BASELINE_2026-05-27.json` |
| Location | `D:\AI_Project\n8n-meta-ads\docs\backups\META_REPORT_TEST_RECOVERY_BASELINE_2026-05-27.json` |
| Export date | 2026-05-27 |
| Purpose | Recovery baseline cho current TEST workflow sau forensic parser/routing/schema fixes |
| Workflow status | TEST recovery workflow export; khi import for testing phải giữ `active=false` |
| Recovery milestone | First known-good TEST recovery baseline sau Normalize Sheet Data, DD/MM parser support, report routing fix, staticData reset removal, và Send Report Telegram payload patch |
| Associated incidents fixed | KPI reports returning zero; DD/MM report command routed as unknown; polling replay risk từ forced staticData reset; debug placeholder trong Telegram send payload |
| active=false verification | Required trước mọi import test; baseline đại diện inactive TEST recovery state |
| Credentials embedded or excluded | Workflow có thể reference credential IDs/names; secret credential values không lưu trong inventory và không được paste vào memory |

**LAST KNOWN GOOD TEST RECOVERY BASELINE.**

Baseline này dùng làm rollback reference cho future import testing. Không overwrite tùy tiện.

## E2E Verified Recovery Baseline Export: 2026-05-27

| Field | Value |
| --- | --- |
| Filename | `META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json` |
| Location | `D:\AI_Project\n8n-meta-ads\docs\backups\META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json` |
| Export date | 2026-05-27 |
| Purpose | First full end-to-end verified TEST recovery baseline sau successful Telegram KPI delivery |
| Verification level | Full end-to-end TEST verification: Telegram input through polling, routing, date parsing, Google Sheet ingestion, normalization, filtering, KPI aggregation, Markdown formatting, và Telegram delivery |
| active=false verification | Required cho import/testing; baseline này là TEST baseline và không imply production activation |
| Recovery milestone | First full end-to-end verified recovery state |
| Associated incidents resolved | KPI reports returning zero do suspected pipeline failure; DD/MM command routing failure; schema mismatch từ Google Sheets array rows; debug placeholder trong Telegram send payload; polling staticData reset risk |
| Telegram delivery verified | Yes |
| KPI aggregation verified | Yes |
| Credentials embedded or excluded | Secret credential values excluded; workflow có thể reference credential IDs/names only |

STATUS: **FIRST FULL END-TO-END VERIFIED TEST BASELINE**

File này supersede earlier recovery baseline làm primary rollback reference cho TEST import verification.

## Production Promotion Readiness Artifacts: 2026-05-27

| Artifact | Status | Notes |
| --- | --- | --- |
| Rollback export | Completed | `META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json` vẫn là primary rollback reference cho TEST import verification |
| Workflow imported inactive first | Verified | Import/testing governance yêu cầu `active=false` |
| Credential binding | Verified | Không store credential values trong memory files |
| Telegram polling continuity | Verified | Supervised runtime activation passed |
| Duplicate-processing protection | Verified | Không stale replay, duplicate reports, hoặc old 2026-03-24 replay observed |

Offset testing note:
Temporary testing offset `81218650` đã dùng cho supervised backlog draining. Restore offset expressions sau testing; không để hardcoded offsets trong active production workflows.

## Stable Baseline Export: 2026-05-29

| Field | Value |
| --- | --- |
| Filename | `META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json` |
| Location | `D:\AI_Project\n8n-meta-ads\docs\backups\META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json` |
| Export date | 2026-05-29 |
| Source workflow | `Meta Report VERIFIED` |
| Purpose | Stable single-consumer operational baseline after routing/polling forensic investigation |
| Verification level | Runtime stable, Telegram polling verified, routing verified, KPI aggregation verified, Telegram delivery verified, single-consumer state verified |
| Recovery milestone | Stable workflow isolation and baseline freeze |
| Associated incidents resolved | `[NODE=UNKNOWN]` routing spam, duplicate consumer chaos, runtime drift confusion, topology false-positive investigation |
| Active state in runtime export | `active=true` for stable operational workflow |
| Deprecated workflows | `Meta Report TEST new`, `Telegram Bot & Reports`, and old error/test lineages remain inactive/deprecated |
| Credentials embedded or excluded | Secret credential values are not recorded in memory; workflow export may reference credential IDs/names only |

STATUS: **LAST KNOWN GOOD VERIFIED STABLE BASELINE**

A full timestamped runtime workflow backup was created before this export.

