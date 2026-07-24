# PROJECT BRAIN — n8n Meta Ads
_Đọc file này đầu tiên mỗi session. Tối đa 500 tokens._

---

### 1. Trạng thái hiện tại

- `Google Sheets Health Check`: ✅ ACTIVE 07:00 — PASS/FAIL path verified, Telegram alert hoạt động.
- `Meta API Health Check`: ✅ ACTIVE 07:05 — PASS/FAIL path verified, Telegram alert hoạt động.
- `Meta Ads Daily Sheet Update`: ✅ ACTIVE 07:30 — sync Meta → Sheet (idempotent verified, numDays=7).
- `Meta Report Yesterday Scheduled`: ✅ ACTIVE 08:13 — đọc Sheet → Telegram báo cáo hôm qua.
- `Meta Report Today Scheduled`: ✅ ACTIVE 11:31 / 16:31 / 21:13 — gọi Meta API trực tiếp, không phụ thuộc Google Sheet.
- `Meta Report VERIFIED`: ✅ ACTIVE — Telegram bot polling, **Source Controlled** (`META_REPORT_VERIFIED_EXPORT_2026-06-16.json`). versionId audit: không có drift.

**🟢 STABLE PRODUCTION STATE CONFIRMED — 2026-06-16**
Tất cả 7 workflow đã vận hành thành công trong production. Không cần can thiệp thủ công.

**📦 META REPORT VERIFIED — SOURCE CONTROLLED — 2026-06-16**
Export: `META_REPORT_VERIFIED_EXPORT_2026-06-16.json` trong `docs/workflows/`.
Audit: versionId `4c765861` khớp baseline 2026-05-29 — không có drift cấu trúc.
Trạng thái: không còn "runtime only" — đã vào source control và CREDENTIAL_INVENTORY.md.

**Lịch chạy production đầy đủ — 2026-06-15:**
```
07:00  Google Sheets Health Check
07:05  Meta API Health Check
07:30  Meta Ads Daily Sheet Update
08:13  Yesterday Report
11:31  Today Report
16:31  Today Report
21:13  Today Report
```

**Project Status: Production Ready — Pending 24-48h Observation**
(Chưa quan sát đủ một chu kỳ tự động đầy đủ)

**Đã hoàn tất 2026-06-15:**
- Google Sheets Health Check ACTIVE, PASS + FAIL path verified.
- Meta API Health Check ACTIVE, PASS + FAIL path verified.
- Monitoring layer hoàn chỉnh — alert Telegram khi credential/API fail.

**Đã hoàn tất 2026-06-11:**
- KPI Mess_Comment đã sửa đúng (exact match action_type).
- Lịch sử 24/03 → 10/06 đã backfill thành công (appendCount=0).
- Duplicate Key trong Sheet đã dọn sạch (COUNTIF=1).
- numDays giảm 30 → 7, verify production OK.
- Google OAuth reconnected, production chain hoạt động bình thường.
- Today Report deployed, KPI 100% khớp Ads Manager.
- CPC metric thêm vào báo cáo Telegram.
- **STABLE PRODUCTION STATE đạt được.**

**Đã hoàn tất 2026-06-25:**
- Phát hiện và khắc phục Migration Verification Gap: node đọc Sheet (HTTP Request) trong 2 workflow bị bỏ sót khi migrate 2026-06-24. Đã đổi toàn bộ node đọc + ghi + Health Check sang Service Account. Đã cập nhật Health Check message cho đúng kiến trúc mới.
- Rút ra 4 bài học quy trình quan trọng (LL-008 đến LL-011): Inventory trước migration, Monitoring đồng bộ production, TEST PASS ≠ Production PASS, Definition of Done cho mọi migration.

**Đã hoàn tất 2026-06-24:**
- Service Account Migration: FUNCTIONALLY COMPLETE. Pending: Scheduled Run Verification (scheduler 07:30 tự động).
  Workflow "Meta Ads Daily Sheet Update 7:30 2" đã migrate 3 node Google sang Service Account.
  Manual Execute PASS. Execute Workflow PASS. Chờ scheduler 07:30 tự động xác nhận lần cuối.
- Phát hiện: một số node dùng `$env.META_ACCESS_TOKEN` thay vì n8n credential UI.
  Khi đổi token phải cập nhật cả hai nguồn + restart Docker.

**Đã hoàn tất 2026-06-22:**
- Meta Access Token incident: resolved — thêm `Bearer ` vào Authorization header trong Header Auth credential. Token hiện tại hoạt động bình thường.
- Cần xác minh loại token (Short-lived / Long-lived / System User) bằng Token Debugger. Root cause gốc: đang dùng User Token thay vì System User Token. Hướng dài hạn: Business Manager → System User Token (chưa thực hiện, thêm vào backlog).

**Đã hoàn tất 2026-06-17:**
- ECONNRESET Telegram incident: resolved — Retry On Fail (3 lần, 2000ms) đã bật trong `Send Report Telegram` của Today Report.

---

### 2. Rules tuyệt đối không vi phạm

- Không activate workflow nếu chưa đọc `GOVERNANCE_RULES.md` trong session đó.
- Không patch production trực tiếp — luôn dùng TEST workflow trước.
- Không rotate `N8N_ENCRYPTION_KEY` nếu chưa có full credential recreation plan.
- Không reset staticData/Telegram offset tùy tiện.
- Không ghi đè cột Nhóm B trong Sheet (xem rule 3).
- Không lưu secrets, tokens, keys vào bất kỳ memory file nào.
- Một Telegram queue chỉ được có đúng một active polling consumer.
- Không tin silent success — luôn verify output data thực tế.

---

### 3. Quyết định đã chốt

- **Option B**: Workflow Yesterday (đọc Sheet) + Workflow Today (gọi Meta API) — đã build.
- **Google Sheet là nguồn sự thật duy nhất** — workflow phải thích nghi với Sheet, không ngược lại.
- **Nhóm A** (Meta được ghi): `Ma_quang_cao, Ngay, Chien_dich, Ten_quang_cao, Chi_tieu, Nguoi_tiep_can, Ngan_sach, click, Trang_Thai, Key, Thoi_diem_cap_nhat`.
- **Nhóm B** (PROTECTED): `Mess_Comment, Khach_sai_tep, Khach_hop_le, SDT, Khach_chot, Chi_phi_*, ghi_chu`.
- Schema: snake_case ASCII. Dates: ISO `YYYY-MM-DD`. Timezone: Asia/Ho_Chi_Minh.
- Polling: dùng dynamic offset expression, không hardcode.

---

### 4. Việc còn lại theo thứ tự ưu tiên

1. ✅ ~~Giảm `numDays` 30 → 7 trong Meta Sync.~~ — DONE 2026-06-11
2. ✅ ~~Test + Activate `Meta Report Today Scheduled`.~~ — DONE, ACTIVE 11:31/16:31/21:13
3. ✅ ~~Thêm CPC vào báo cáo Telegram.~~ — DONE 2026-06-11
4. ✅ ~~Thêm Telegram Error Alert (Monitoring Layer).~~ — DONE 2026-06-15 (Health Check 07:00 + 07:05)
5. ✅ ~~Quan sát 24-48h hệ thống tự chạy.~~ — CONFIRMED 2026-06-16
6. ✅ ~~Credential Inventory.~~ — DONE 2026-06-16 (`CREDENTIAL_INVENTORY.md`)
7. ✅ ~~Export `META_REPORT_TODAY_SCHEDULED_2026-06-17.json` (có Retry On Fail).~~ — DONE 2026-06-17
8. ✅ ~~Chuyển Google OAuth → Service Account.~~ — Implementation Done 2026-06-25 (pending scheduler 2026-06-26)
9. Xác minh loại Meta token hiện tại (Short-lived / Long-lived / System User)
   → `https://developers.facebook.com/tools/debug/accesstoken/`
   → Ghi kết quả vào `CREDENTIAL_INVENTORY.md`
10. ✅ ~~Date Range Engine V1: Runtime Verified~~ — DONE 2026-07-03 (xem CURRENT_STATUS.md)
11. Tích hợp Date Range Engine vào Yesterday Report
    - Presentation Layer (Render Telegram) sử dụng output của Date Range Engine
    - Chỉ loại bỏ "Lọc & Tính KPI Hôm Qua" sau khi Production verify PASS
12. Verify Yesterday Report với Engine mới (full workflow)
13. Tái sử dụng Engine cho 7 ngày, 30 ngày, khoảng ngày
14. Patch format báo cáo ngắn hơn
15. ⚠️ Revoke Telegram Bot Token cũ (đã lộ trong chat)
16. Chuyển Meta từ User Token → System User Token (Business Manager ID: `187499196519080`)
17. Telegram Alert phân loại severity levels
18. Dashboard tổng quan KPI

---

### 5. Khi nào đọc file nào thêm

| Tình huống | File cần đọc |
|---|---|
| Lỗi lạ chưa từng gặp | `INCIDENTS_AND_RECOVERY.md` |
| Đụng schema / cột Sheet | `DATA_SCHEMA_RULES.md` |
| Thay đổi kiến trúc | `DECISIONS.md` |
| Trước khi activate workflow | `GOVERNANCE_RULES.md` |
| Không chắc rule nào áp dụng | `GOVERNANCE_RULES.md` |
| Onboard AI mới hoàn toàn | Đọc hết tất cả file |

---

### 6. Baseline files quan trọng nhất

- `META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json` — last known good production baseline.
- `META_ADS_DAILY_SHEET_UPDATE_N8N_2_20_6_SCHEMA_PATCH_2026-05-29.json` — base file cho Daily Sheet Update.
- `META_ADS_DAILY_SHEET_UPDATE_SCHEDULED_SUMMARY_SAFE_2026-06-09.json` — Daily Sync ACTIVE.
- `META_REPORT_YESTERDAY_SCHEDULED_2026-06-07.json` — Workflow Yesterday ACTIVE.
- `META_REPORT_TODAY_SCHEDULED_2026-06-07.json` — Workflow Today (ACTIVE, production verified 2026-06-11).

Tất cả tại: `D:\AI_Project\n8n-meta-ads\docs\backups\` hoặc `docs\workflows\`.
**Đã hoàn tất 2026-06-17:**
- OAuth reconnect incident: resolved bằng reconnect thủ công
  Google Sheets OAuth2 API. Quan sát: OAuth cần reconnect định kỳ.
  Rủi ro mở: Service Account migration chưa thực hiện.
- Telegram Gateway Timeout (Yesterday Report): Retry On Fail
  (3 lần, 2000ms) đã áp dụng cho Today Report và Yesterday Report.
  Verified: Telegram nhận báo cáo hôm qua chính xác.
- Export hoàn tất:
  META_REPORT_YESTERDAY_SCHEDULED_2026-06-17.json
  META_REPORT_TODAY_SCHEDULED_2026-06-17.json

**Lesson Learned 2026-06-17:**
- Mọi node gửi Telegram trong production PHẢI bật Retry On Fail
  (3 lần, 2000ms).
- Khi fix một workflow, phải kiểm tra ngay các workflow tương tự
  có cùng pattern — tránh bỏ sót như Today/Yesterday Report.

**⚠️ Priority Override:**
Mặc dù nằm ở mục #10, Service Account Migration hiện là
ưu tiên cao nhất. OAuth sẽ expire lại trong vài ngày.
