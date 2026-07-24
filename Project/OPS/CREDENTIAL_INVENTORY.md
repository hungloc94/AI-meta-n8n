# CREDENTIAL INVENTORY — n8n Meta Ads
_Tạo: 2026-06-16. Nguồn: quét JSON trong `docs/workflows/` và `docs/backfills/`._
_Không ghi giá trị thực của bất kỳ token/secret nào._

---

## 1. Bảng Credential → Workflow Sử Dụng

| Credential | Loại (n8n) | Workflow sử dụng | Node cụ thể |
|------------|-----------|------------------|-------------|
| **Google Sheets account** | `googleSheetsOAuth2Api` | Meta Report Yesterday Scheduled | `Đọc Google Sheet` |
| **Google Sheets account** | `googleSheetsOAuth2Api` | Meta Ads Daily Sheet Update (tất cả phiên bản) | `Đọc toàn bộ Sheet`, `Ghi mới vào Google Sheet`, `Cập nhật vào Google Sheet` |
| **Google Sheets account** | `googleSheetsOAuth2Api` | Google Sheets Health Check 07:00 | `HTTP Request Google Sheets` |
| **Google Sheets account** | `googleSheetsOAuth2Api` | Meta Ads Backfill ONE TIME (cả 2 phiên bản) | `Đọc toàn bộ Sheet`, `Ghi mới vào Google Sheet`, `Cập nhật vào Google Sheet` |
| **Google Sheets account** | `googleSheetsOAuth2Api` | Meta Ads Daily Sheet Update Backfill TAO_NGAY_ONLY | `Đọc toàn bộ Sheet`, `Ghi mới vào Google Sheet`, `Cập nhật vào Google Sheet` |
| **Header Auth account** | `httpHeaderAuth` | Meta Ads Daily Sheet Update (tất cả phiên bản) | `Lấy cấu hình Meta` |
| **Header Auth account** | `httpHeaderAuth` | Meta Ads Backfill ONE TIME (cả 2 phiên bản) | `Lấy cấu hình Meta` |
| **Header Auth account** | `httpHeaderAuth` | Meta Ads Daily Sheet Update Backfill TAO_NGAY_ONLY | `Lấy cấu hình Meta` |

> **Ghi chú credential ID (nội bộ n8n — không phải secret):**
> - Google Sheets account: `iGuF5SVznNPI7ihl`
> - Header Auth account: `5lVOKqN2KFAC4E0t`
>
> **Ghi chú Header Auth account:** Đây là credential lưu trữ Meta Access Token dưới dạng HTTP header.
> **⚠️ Format bắt buộc:** `Authorization: Bearer <token>` — KHÔNG phải `Authorization: <token>` (thiếu `Bearer ` sẽ gây lỗi 403 từ Meta API, lỗi misleading là `(#200) Provide valid app ID`).
> **Khi rotate token:** luôn kiểm tra prefix `Bearer ` còn nguyên sau khi thay giá trị.
> Dùng để gọi Meta Ads API lấy cấu hình ad (status, budget).
>
> **Workflow `Meta Report VERIFIED` — Runtime Export 2026-06-16:**
> Export từ runtime xác nhận workflow dùng:
> - `Google Sheets account` (`iGuF5SVznNPI7ihl`) → node `Đọc Google Sheet`
> - `$env.TELEGRAM_BOT_TOKEN` → nodes `Telegram API getUpdates`, `ACK Telegram Update`, `Send Menu`, `Send Report Telegram`, `Send Unknown Notice`, `Send Range Guide`
> - Không dùng `$env.META_ACCESS_TOKEN` (workflow này chỉ đọc Sheet, không gọi Meta API trực tiếp)
> - Không dùng `Header Auth account`

---

## 2. Bảng .env Variables → Workflow Sử Dụng

_Chỉ ghi tên biến. Không ghi giá trị._

### META_ACCESS_TOKEN

| Workflow | Node sử dụng | Mục đích |
|----------|-------------|---------|
| Meta Report Today Scheduled (cả 2 phiên bản) | `Lấy Insights Meta Hôm Nay` | Gọi Meta Insights API lấy dữ liệu ngày hôm nay |
| Meta Ads Daily Sheet Update (tất cả phiên bản) | `Lấy dữ liệu Meta` | Gọi Meta Insights API theo từng ngày |
| Meta Ads Backfill ONE TIME (cả 2 phiên bản) | `Lấy dữ liệu Meta` | Gọi Meta Insights API cho backfill |
| Meta API Health Check 07:05 | `Code Node` | Kiểm tra Meta API còn hoạt động không |

> **Cơ chế:** Các node dùng `$env.META_ACCESS_TOKEN` trực tiếp trong Code node. Khác với `Header Auth account` (credential n8n) — đây là biến môi trường trong `.env` của Docker container.

### TELEGRAM_BOT_TOKEN

| Workflow | Node sử dụng | Mục đích |
|----------|-------------|---------|
| Meta Report Today Scheduled (cả 2 phiên bản) | `Send Report Telegram` | Gửi báo cáo hôm nay |
| Meta Report Yesterday Scheduled | `Send Report Telegram` | Gửi báo cáo hôm qua |
| Google Sheets Health Check 07:00 | `Telegram Alert` | Gửi alert khi Google Sheets credential fail |
| Meta API Health Check 07:05 | `Telegram Alert` | Gửi alert khi Meta API fail |

### TELEGRAM_CHAT_ID

| Workflow | Node sử dụng | Mục đích |
|----------|-------------|---------|
| Meta Report Today Scheduled (cả 2 phiên bản) | `Send Report Telegram` | Chat ID nhận báo cáo |
| Meta Report Yesterday Scheduled | `Send Report Telegram` | Chat ID nhận báo cáo |
| Google Sheets Health Check 07:00 | `Telegram Alert` | Chat ID nhận alert |
| Meta API Health Check 07:05 | `Telegram Alert` | Chat ID nhận alert |

> **Lưu ý:** Các scheduled workflow PHẢI dùng `$env.TELEGRAM_CHAT_ID` (không có user trigger context). Workflow bot polling (`Meta Report VERIFIED`) dùng `$json.chat_id` từ message context của người dùng.

### N8N_BLOCK_ENV_ACCESS_IN_NODE

> Biến hệ thống n8n — không tìm thấy trong workflow JSON. Nếu được set thành `true`, sẽ chặn toàn bộ `$env.*` trong Code node, làm hỏng `META_ACCESS_TOKEN`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`. Phải để `false` (hoặc không set) để hệ thống hoạt động.

### META_APP_ID / META_APP_SECRET

> Không tìm thấy trong bất kỳ workflow JSON nào trong `docs/workflows/` hoặc `docs/backfills/`. Có thể được dùng trong cấu hình Meta App nhưng không xuất hiện trong logic workflow.

---

## 3. Migration Impact Matrix

| Credential / Biến | Khi thay đổi ảnh hưởng workflow nào | Node nào cần đổi |
|--------------------|-------------------------------------|-----------------|
| **Google Sheets account** (OAuth) | Meta Report Yesterday, Daily Sheet Update, Google Sheets Health Check, tất cả Backfill | `Đọc Google Sheet`, `Đọc toàn bộ Sheet`, `HTTP Request Google Sheets`, `Ghi mới vào Google Sheet`, `Cập nhật vào Google Sheet` |
| **Header Auth account** (Meta token in n8n credential) | Daily Sheet Update, tất cả Backfill | `Lấy cấu hình Meta` |
| **META_ACCESS_TOKEN** (.env) | Today Report, Daily Sheet Update, tất cả Backfill, Meta API Health Check | `Lấy Insights Meta Hôm Nay`, `Lấy dữ liệu Meta`, `Code Node` |
| **TELEGRAM_BOT_TOKEN** (.env) | Today Report, Yesterday Report, Google Sheets Health Check, Meta API Health Check | `Send Report Telegram`, `Telegram Alert` |
| **TELEGRAM_CHAT_ID** (.env) | Today Report, Yesterday Report, Google Sheets Health Check, Meta API Health Check | `Send Report Telegram`, `Telegram Alert` |

> **Điểm rủi ro cao nhất:** `Google Sheets account` là single point of failure — nếu expire, đồng thời làm chết: Daily Sheet Update (07:30), Yesterday Report (08:13), và Google Sheets Health Check (07:00).

---

## 4. Ghi Chú Migration: Google OAuth → Service Account

### Workflow bị ảnh hưởng

| Workflow | Node cần đổi credential | Ghi chú |
|----------|------------------------|---------|
| **Google Sheets Health Check 07:00** | `HTTP Request Google Sheets` | Đổi `authentication` sang Service Account |
| **Meta Ads Daily Sheet Update** | `Đọc toàn bộ Sheet`, `Ghi mới vào Google Sheet`, `Cập nhật vào Google Sheet` | 3 nodes trong 1 workflow |
| **Meta Report Yesterday Scheduled** | `Đọc Google Sheet` | 1 node |
| **Meta Ads Backfill** (các phiên bản) | `Đọc toàn bộ Sheet`, `Ghi mới vào Google Sheet`, `Cập nhật vào Google Sheet` | Backfill one-time — ít urgent hơn |

### Thứ tự migration an toàn

1. Tạo Google Service Account mới trong Google Cloud Console.
2. Chia sẻ Google Sheet với Service Account email (quyền Editor).
3. Tạo credential mới trong n8n (`Google Service Account` type).
4. **Test trước với workflow ít rủi ro nhất:** Health Check 07:00 — nếu fail không ảnh hưởng data.
5. Sau khi Health Check PASS: migrate Daily Sheet Update (đổi cả 3 nodes).
6. Cuối cùng: migrate Yesterday Report.
7. Verify từng workflow sau migration.
8. Revoke OAuth credential cũ sau khi tất cả đã migrate và stable.

> **KHÔNG migrate tất cả cùng lúc.** Nếu Service Account bị lỗi permission, sẽ làm chết toàn bộ hệ thống cùng một lúc.

---

## 5. Credential Ownership

| Credential | Chủ sở hữu | Loại | Trạng thái | Ghi chú |
|------------|-----------|------|-----------|---------|
| **Google Sheets account** | Google account người dùng | OAuth 2.0 | ⚠️ CÓ THỂ EXPIRE | Single point of failure. Cần migrate sang Service Account. Từng expire 2026-06-10. |
| **Header Auth account** | Meta Business / người dùng | HTTP Header Auth | ⚠️ TOKEN CÓ HẠN | Lưu Meta Access Token trong n8n credential. Token Meta thường hết hạn sau 60-90 ngày. |
| **META_ACCESS_TOKEN** (.env) | Meta Business / người dùng | .env biến môi trường | ⚠️ TOKEN CÓ HẠN | Cùng loại token với Header Auth account nhưng dùng qua `$env` trong Code node. Cần đồng bộ khi rotate. |
| **TELEGRAM_BOT_TOKEN** (.env) | BotFather / người dùng | .env biến môi trường | ⚠️ TOKEN CŨ CẦN REVOKE | Token cũ đã lộ trong chat. Cần revoke và tạo token mới. |
| **TELEGRAM_CHAT_ID** (.env) | Telegram | .env biến môi trường | ✅ ổn định | Không expire. Giá trị: xem `.env` container. |

---

## 6. Phiên Bản Workflow Trong Inventory

### docs/workflows/ (8 files — PRODUCTION candidates)

| File | Tên Workflow | Phiên bản | Trạng thái |
|------|-------------|-----------|-----------|
| `META_REPORT_VERIFIED_EXPORT_2026-06-16.json` | Meta Report VERIFIED | Runtime export — Current Production | ✅ Current Production Runtime Export |
| `GOOGLE_SHEETS_HEALTH_CHECK_0700_2026-06-15.json` | Google Sheets Health Check 07:00 | Production | ✅ Current |
| `META_API_HEALTH_CHECK_0705_2026-06-15.json` | Meta API Health Check 07:05 | Production | ✅ Current |
| `META_ADS_DAILY_SHEET_UPDATE_SCHEDULED_SUMMARY_SAFE_KPI_MESS_PATCHED_2026-06-11.json` | Meta Ads Daily Sheet Update | KPI patched | ✅ Current production |
| `META_REPORT_TODAY_SCHEDULED_KPI_MESS_PATCHED_2026-06-11.json` | Meta Report Today Scheduled | KPI patched + CPC | ✅ Current production |
| `META_REPORT_YESTERDAY_SCHEDULED_2026-06-07.json` | Meta Report Yesterday Scheduled | — | ✅ Current production |
| `META_ADS_DAILY_SHEET_UPDATE_SCHEDULED_SUMMARY_SAFE_2026-06-09.json` | Meta Ads Daily Sheet Update | Pre-KPI-patch | ⚠️ Superseded |
| `META_ADS_DAILY_SHEET_UPDATE_SCHEDULED_2026-06-09.json` | Meta Ads Daily Sheet Update | Older | ⚠️ Superseded |
| `META_REPORT_TODAY_SCHEDULED_2026-06-07.json` | Meta Report Today Scheduled | Fuzzy match (sai) | ❌ Deprecated |

### docs/backfills/ (3 files — ONE TIME, đã chạy)

| File | Tên Workflow | Phiên bản | Trạng thái |
|------|-------------|-----------|-----------|
| `META_ADS_BACKFILL_2026-04-27_2026-04-28_ONE_TIME_KPI_MESS_PATCHED_2026-06-11.json` | Meta Ads Backfill ONE TIME | KPI patched | ✅ Đã chạy |
| `META_ADS_BACKFILL_2026-04-27_2026-04-28_ONE_TIME.json` | Meta Ads Backfill ONE TIME | Pre-patch | ⚠️ Superseded |
| `META_ADS_DAILY_SHEET_UPDATE_BACKFILL_2026-04-27_2026-04-28_TAO_NGAY_ONLY.json` | Meta Ads Daily Sheet Update | Partial (tạo ngày only) | Archive |

---

## Workflows Ngoài Phạm Vi Inventory

Các file JSON trong `docs/backups/` — **không đưa vào Credential Inventory chính** vì đây là backup/archive, không phải production workflow.

| File | Thư mục | Ghi chú |
|------|---------|---------|
| `workflow-clean-post-rotation-2026-05-24.json` | `docs/backups/` | Backup sau key rotation |
| `META_REPORT_TEST_RECOVERY_BASELINE_2026-05-27.json` | `docs/backups/` | TEST recovery baseline |
| `META_REPORT_TEST_E2E_DEDUPE_ROUTING_FIXED_2026-05-28.json` | `docs/backups/` | TEST E2E dedupe |
| `META_REPORT_TEST_E2E_VERIFIED_2026-05-27.backup-20260528-235050.json` | `docs/backups/` | Backup file |
| `META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json` | `docs/backups/` | TEST E2E verified baseline |
| `META_REPORT_TEST_POLLING_CONTINUITY_VERIFIED_2026-05-27.json` | `docs/backups/` | TEST polling continuity |
| `META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json` | `docs/backups/` | **Last known good** production baseline cho `Meta Report VERIFIED` |
| `META_ADS_DAILY_SHEET_UPDATE_RUNTIME_EXPORT_BEFORE_COMPAT_PATCH_2026-05-29.json` | `docs/backups/` | Pre-patch export |
| `META_ADS_DAILY_SHEET_UPDATE_N8N_2_20_6_COMPAT_2026-05-29.json` | `docs/backups/` | Compat patch version |
| `META_ADS_DAILY_SHEET_UPDATE_N8N_2_20_6_SCHEMA_PATCH_2026-05-29.json` | `docs/backups/` | Schema patch version |
| `runtime_workflows_backup_20260529-003510/coH2npQzxnroIK7r.json` | `docs/backups/` | Runtime backup |
| `runtime_workflows_backup_20260529-003510/DgZzP1IVTKFbwVlZ.json` | `docs/backups/` | Runtime backup |
| `runtime_workflows_backup_20260529-003510/tD0zGW33435s4Y8y.json` | `docs/backups/` | Runtime backup |
| `runtime_workflows_backup_20260529-003510/TiSKx1gKA99UnA4x.json` | `docs/backups/` | Runtime backup |

## Environment File
- Đường dẫn: D:\AI_Project\n8n-meta-ads\.env
- Chứa: API tokens, secrets, encryption keys
- Lưu ý: KHÔNG copy vào vault. KHÔNG để AI đọc.
  Chỉ truy cập trực tiếp khi cần thiết.
