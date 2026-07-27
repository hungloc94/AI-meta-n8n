# Environment Registry

## Role
Registry cho critical environment variables.

Không store secret values ở đây. Chỉ lưu purpose, risk, và dependency mapping.

## Variables
| Variable | Purpose | Criticality | Dependencies | Rotation Risk | Recovery Importance |
| --- | --- | --- | --- | --- | --- |
| `N8N_ENCRYPTION_KEY` | Encrypt/decrypt n8n credentials | Critical | All stored credentials | Very high | Essential |
| `TELEGRAM_BOT_TOKEN` | Telegram API access | High | Telegram polling/send nodes | Medium-high | Required for Telegram runtime |
| `TELEGRAM_CHAT_ID` | Default chat target/fallback | Medium | Telegram send/fallback logic | Medium | Useful for controlled tests |
| `GENERIC_TIMEZONE` | n8n timezone | Medium | schedule/date behavior | Low | Required for consistency |
| `TZ` | container timezone | Medium | runtime clock behavior | Low | Required for consistency |
| `N8N_HOST` | n8n host setting | Medium | editor/webhook URL behavior | Medium | Required for local/public mode |
| `N8N_PROTOCOL` | http/https mode | Medium | URL generation/cookies | Medium | Required for migration |
| `N8N_EDITOR_BASE_URL` | editor base URL | Medium | OAuth/callback behavior | Medium | Required for OAuth correctness |
| `WEBHOOK_URL` | webhook URL generation | High | webhook/public integrations | High | Critical if public mode enabled |
| Google Sheets spreadsheet ID | identifies source sheet | High | report data ingestion | Medium | Required for KPI data |
| Google Sheets range/tab name | identifies data range | High | report data ingestion | Medium | Required for KPI data |
| Meta App ID | Meta API app identity | High | Meta Graph credential | Medium | Required for Meta API recovery |
| Meta App Secret | Meta API secret | Critical | Meta Graph credential | High | Required for token refresh/app auth |
| Meta long-lived token | Meta Ads API access | Critical | Meta reporting | High | Required unless regenerated |
| Cloudflare token | tunnel/public exposure | High | public webhook/tunnel | High | Deferred |

## Rotation Rules
- Không rotate `N8N_ENCRYPTION_KEY` nếu chưa có full credential recreation plan.
- Chỉ rotate provider tokens khi có documented recovery source và rollback path.
- Không expose public endpoint variables trong local recovery.

## TODO Verification Items
- Confirm exact Google Sheets spreadsheet và range identifiers.
- Confirm Telegram token source ownership.
- Confirm Meta app ownership và token refresh path.
- Confirm Cloudflare token scope trước public exposure.
