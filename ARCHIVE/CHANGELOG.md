# Changelog

## Notes
Changelog này chỉ ghi distilled operational changes. Không lưu raw debug chat.

## 2026-05-27
- Added `Normalize Sheet Data` node vào workflow export.
- Converted Google Sheet array rows thành canonical snake_case ASCII object schema.
- Added numeric normalization cho KPI fields.
- Added DD/MM parser support.
- Fixed report routing cho `báo cáo 24/03`.
- Patched `command_type` classification trong parser flow.
- Replaced `[NODE=REPORT]` bằng `$json.message` trong `Send Report Telegram`.
- Converted Google Sheet headers sang snake_case ASCII.
- Removed dangerous staticData reset.
- Added canonical ISO date conversion.

## 2026-05-27 Recovery Baseline Finalization

- Added DD/MM parser support.
- Added report command regex support.
- Removed polling staticData forced reset.
- Confirmed recovery baseline export exists: `META_REPORT_TEST_RECOVERY_BASELINE_2026-05-27.json`.
- Initialized project-scoped operational memory system trong Obsidian project folder.
- Updated backup inventory với LAST KNOWN GOOD TEST RECOVERY BASELINE.
- Updated current operational status, test playbook, lessons learned, và incident recovery notes.

## 2026-05-27 End-to-End Recovery Verification

- First successful end-to-end Telegram KPI delivery verified.
- KPI aggregation verified operational.
- DD/MM command parsing verified operational.
- Telegram Markdown formatting verified.
- End-to-end recovery baseline exported: `META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json`.
- Recovery status upgraded to RESOLVED.

## 2026-05-27 Governance Initialization

- Staging governance system initialized với `STAGING_TEST_MATRIX.md`.
- Production promotion governance initialized với `PRODUCTION_PROMOTION_CHECKLIST.md`.
- Production activation vẫn chưa approved, pending staging validation và GO / NO-GO review.

## 2026-05-27 Polling Continuity Runtime Validation

- Polling continuity runtime validation passed dưới supervised real workflow activation.
- Verified `báo cáo hôm qua` generated exactly one Telegram response.
- Confirmed no stale replay, no duplicate reports, và no replay of old 2026-03-24 report.
- Confirmed KPI=0 cho 2026-05-26 là expected business reality do missing/no sheet data, không phải pipeline failure.
- Recorded temporary hardcoded offset used for backlog draining: `81218650`.
- Added governance warning để restore offset expressions sau testing và không bao giờ để hardcoded offsets trong active production workflows.
- Updated project memory scope: operational memory là project-scoped và self-contained trong `Project\AI_Meta_n8n_autoamation`.

## 2026-06-15 Monitoring Layer Complete

- Deploy `Google Sheets Health Check` — ACTIVE 07:00.
  - PASS path verified: credential hoạt động → không gửi alert.
  - FAIL path verified: credential fail → Telegram alert gửi đúng.
  - PASS after restore verified: credential khôi phục → workflow trở lại bình thường.
- Deploy `Meta API Health Check` — ACTIVE 07:05.
  - PASS path verified: API hoạt động → không gửi alert.
  - FAIL path verified: API fail → Telegram alert gửi đúng.
  - PASS after restore verified: API khôi phục → workflow trở lại bình thường.
- Lịch chạy production đầy đủ: 07:00 / 07:05 / 07:30 / 08:13 / 11:31 / 16:31 / 21:13.
- Project Status: **Production Ready — Pending 24-48h Observation**.

## 2026-06-11 Today Report Production Deploy + CPC + Stable Production State

- Deploy `Meta Report Today Scheduled` lên production — ACTIVE 11:31 / 16:31 / 21:13.
- Kiến trúc: Schedule → Tính Ngày Hôm Nay → Lấy Insights Meta → Tính KPI → Send Telegram.
- Nguồn dữ liệu: Meta Marketing API trực tiếp — không đọc Google Sheet, không phụ thuộc Google OAuth.
- Verify KPI Today Report: Chi tiêu / Tiếp cận / Tin nhắn / Clicks / CPC — 100% khớp Ads Manager.
- Thêm CPC metric (`totalSpend / totalClick`, divide-by-zero protected).
- Xác nhận 3 cron rules trong 1 workflow: `31 11 * * *`, `31 16 * * *`, `13 21 * * *`.
- `onsite_conversion.messaging_conversation_started_7d` xác nhận là metric chính xác cho Tin nhắn.
- **STABLE PRODUCTION STATE đạt được**: tất cả 4 workflow active, data integrity verified.

## 2026-06-11 KPI Mess_Comment Fix + Backfill Hoàn Tất

- Phát hiện root cause KPI Mess_Comment sai toàn bộ lịch sử: fuzzy `includes('message')` matching gom nhầm nhiều action_type.
- Patch toàn bộ 3 workflow (`META_REPORT_TODAY_SCHEDULED`, `META_ADS_DAILY_SHEET_UPDATE`, `META_ADS_BACKFILL`) sang exact match `onsite_conversion.messaging_conversation_started_7d`.
- Dọn duplicate Key trong Sheet (giữ bản ghi mới timestamp 11/06, loại bỏ bản ghi cũ timestamp 09/06).
- Verify COUNTIF(Key) = 1 toàn cột — không còn duplicate.
- Backfill lịch sử 24/03 → 10/06: appendCount=0, updateCount>0 toàn bộ giai đoạn.
- Spot check 01/04 xác nhận: trước=18, sau=6, Ads Manager=6 — khớp.
- Giảm `numDays` từ 30 xuống 7 (Meta adjust data chậm chủ yếu trong vòng 7 ngày).
- Verify production thành công sau khi đổi numDays=7.

## 2026-06-09 → 2026-06-10 Google OAuth Hết Hạn + Production Chain Activate

- Phát hiện Google Sheets OAuth refresh token expired đột ngột — UI vẫn hiện "connected" nhưng runtime fail.
- Impact: 07:30 Sheet Update không chạy, 08:13 Yesterday Report không gửi Telegram.
- Resolution: Reconnect Google Sheets credential (Sign in with Google lại).
- Verify: Sheet cập nhật đúng, data 09/06 khớp ~636k, Telegram gửi bình thường.
- Production chain đã được activate: `Meta Ads Daily Sheet Update` (07:30) + `Meta Report Yesterday Scheduled` (08:13).
- Verify idempotent: Lần 1 Append=21/Update=0, Lần 2 Append=0/Update=319 — không tạo trùng.

## 2026-05-29 Stable Single-Consumer Baseline Freeze

- Investigated double consumer behavior in Telegram polling workflows.
- Discovered `Meta Report lỗi` and `Meta Report TEST new` had previously consumed the same Telegram queue simultaneously.
- Confirmed runtime UI workflow state had drifted from exported verified artifact state.
- Completed routing/topology forensic investigation across IF chains, false branch behavior, topology leaks, and ACK merge assumptions.
- Identified major chaos source as multiple active Telegram polling workflows, not parser failure.
- Isolated stable workflow as `Meta Report VERIFIED`.
- Verified only one active Telegram polling consumer remains.
- Deprecated workflows confirmed inactive.
- Froze stable baseline export: `META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json`.
- Finalized current operational stabilization state.

