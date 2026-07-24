# Current Status

## Authority
File này là operational truth cao nhất cho trạng thái hiện tại của hệ thống.

## Current Workflow State
- Workflow đang được test trong TEST branch/export.
- Workflow phải giữ `active=false` trong recovery.
- Chưa có phê duyệt production activation.
- `Execute Step` dùng để isolate lỗi node, nhưng polling continuity đã được verify bằng runtime activation thật.
- Polling continuity verified under supervised runtime activation.

## Current Stable Test Branch
Workflow lineage đang được test:
- `POLLING_IFCHAIN_FALLBACK_v20260513.json.json`
- Chỉ dùng cho TEST import.
- Khi import phải giữ inactive state.

## Current Verified Chain
Đã verify healthy:
- Telegram polling hoạt động.
- Main Router hoạt động.
- IF Report hoạt động.
- Google Sheet fetch hoạt động.
- Telegram send hoạt động.
- Markdown report formatting hoạt động.
- Main Router Code verified healthy.
- Process Updates parser đã patch.

## Current Parser Behavior
Known-good command:
- `báo cáo 24/03`

Expected parser output:
```json
{
  "dates": ["2026-03-24"]
}
```

Parser recovery status:
- Đã thêm DD/MM parser support.
- Đã thêm report regex support.
- Parser convert input thân thiện với người dùng sang ISO date nội bộ.

## Current Routing Behavior
Expected chain cho report testing:
```text
Process Updates
→ Main Router Code
→ IF Report
→ Parse & Route
→ Switch Route
→ Đọc Google Sheet
→ Normalize Sheet Data
→ Lọc Dữ Liệu Thiếu
→ Tạo Báo Cáo
→ Send Report Telegram
```

## Current KPI Recovery State
- KPI recovery chain đã được verify end-to-end.
- `Normalize Sheet Data` node đã thêm thành công.
- Google Sheet headers đã convert sang snake_case ASCII.
- Google Sheets rows được normalize từ array rows sang canonical objects.
- Downstream KPI logic dùng normalized fields như `chi_tieu` và `khach_hop_le`.

## Current Recovery State
Đã recover/stabilize:
- `Normalize Sheet Data` node added.
- `Send Report Telegram` đã patch từ `[NODE=REPORT]` sang `$json.message`.
- Đã remove forced staticData reset.
- Đã identify polling duplicate-processing risk.
- Workflow vẫn inactive.

## Current Unresolved Risks
- Polling systems vẫn nhạy với offset/staticData continuity.
- Future credential encryption key rotation cần full credential recreation plan.
- Webhook/Cloudflare public exposure vẫn deferred.
- Cần restore offset expressions sau testing trước mọi production review.
- Cần confirm không stale replay sau restart trước production promotion.

## Current Known-Good Commands
- `menu`
- `báo cáo hôm nay`
- `báo cáo hôm qua`
- `báo cáo 24/03`
- `báo cáo từ 2026-03-24 đến 2026-03-24`

TODO verify thêm:
- `báo cáo 7 ngày`
- custom date ranges

## Do Not Change Without Approval
- workflow active state
- Telegram polling architecture
- credentials
- webhook architecture
- forensic webhook reference files
- hardcoded Telegram offset expression/runtime logic

## Current Verified State: 2026-05-27 Recovery Baseline

PASS:
- Telegram polling working.
- Process Updates parser fixed.
- Main Router routing fixed.
- IF Report branch verified.
- Parse & Route outputs `dates: ["2026-03-24"]` cho `báo cáo 24/03`.
- DD/MM parsing support added.
- staticData forced reset removed.
- Normalize Sheet Data layer working.
- Google Sheet schema normalized to snake_case ASCII.
- Send Report Telegram payload patched to use `$json.message`.
- Recovery baseline export exists: `META_REPORT_TEST_RECOVERY_BASELINE_2026-05-27.json`.

## Current Recovery Progress
- Routing + parser chain restored.
- KPI schema normalization layer added.
- TEST recovery baseline exported manually by user.
- Workflow phải inactive trong import và recovery testing.

## Current Known Limitations
- Year rollover ambiguity chưa giải quyết.
- DD/MM/YYYY explicit format chưa support.
- Production activation chưa được approve.
- Public webhook/Cloudflare exposure vẫn deferred.

## Current Open Risks
- Production activation vẫn cần governance approval.
- **Blocker duy nhất còn lại trước production promotion:** chạy lại workflow sau khi restart và xác minh không có stale replay.
- Additional date formats và range behavior cần regression testing.
- Credential/key rotation high-risk và cần plan riêng.

## Current Verified State: First Full E2E Recovery Baseline

PASS:
- Telegram polling verified.
- Process Updates verified.
- Main Router Code verified.
- IF Report routing verified.
- Parse & Route verified.
- DD/MM parsing verified.
- Google Sheet read verified.
- Normalize Sheet Data verified.
- Lọc Dữ Liệu Thiếu verified.
- KPI aggregation verified.
- Telegram message delivery verified.
- Markdown formatting verified.
- Divide-by-zero protection verified.
- Send Report Telegram verified.

## Current Verified Chain
```text
Telegram
→ Polling
→ Routing
→ Date parsing
→ Google Sheet
→ Normalize
→ Filtering
→ KPI aggregation
→ Telegram delivery
```

Detailed operational chain:
```text
Telegram Input
→ Telegram API getUpdates
→ Process Updates
→ Main Router Code
→ IF Report
→ Parse & Route
→ Đọc Google Sheet
→ Normalize Sheet Data
→ Lọc Dữ Liệu Thiếu
→ Tạo Báo Cáo
→ Send Report Telegram
```

## Current Verified KPI Status
Verified Telegram output:
```text
📊 BÁO CÁO META ADS
📅 Ngày: 2026-03-24
💰 Tổng chi tiêu: 22.587đ
📩 Tổng kết quả: 0
🎯 CPA: 0đ
```

Important clarification:
KPI=0 cho test date này là business reality, không phải pipeline failure. Campaign date được chọn thật sự có zero valid leads.

## Current Baseline Qualification
`META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json` là FIRST FULL END-TO-END VERIFIED TEST RECOVERY BASELINE.

## Current Governance State
- Staging governance system initialized: `STAGING_TEST_MATRIX.md`.
- Production promotion governance initialized: `PRODUCTION_PROMOTION_CHECKLIST.md`.
- Production activation is NOT approved.
- Next work phải validate staging behavior trước mọi production promotion decision.

## Current Verified State: Polling Continuity Runtime Validation

PASS:
- Đã verify polling continuity thành công dưới supervised runtime activation thật.
- Workflow runtime activation được verify bằng real background execution, không chỉ Execute Step.
- Telegram backlog replay prevention hoạt động đúng sau temporary hardcoded offset draining.
- Temporary testing offset used: `81218650`.
- Runtime command validated: `báo cáo hôm qua`.
- Runtime behavior verified:
  - no stale replay
  - no duplicate reports
  - no replay of old 2026-03-24 report
  - exactly one Telegram response generated

Verified runtime Telegram output:
```text
📊 BÁO CÁO META ADS
📅 Ngày: 2026-05-26
💰 Tổng chi tiêu: 0đ
📩 Tổng kết quả: 0
🎯 CPA: 0đ
```

Important clarification:
KPI=0 cho 2026-05-26 là expected business reality do missing/no sheet data, không phải pipeline failure.

Current system status:
**Polling continuity verified under supervised runtime activation.**

## Production Promotion Readiness Checklist

* [x] rollback export completed
* [x] workflow imported inactive first
* [x] credential binding verified
* [x] Telegram polling continuity verified
* [x] duplicate-processing protection verified
* [x] first supervised runtime activation completed
* [x] restore offset expressions after testing — ĐÃ XÁC MINH 2026-06-02
* [x] confirm no stale replay after restart — ĐÃ XÁC MINH 2026-06-02 (Docker restart test)

Production activation: Phase A hoàn tất. Đang chờ Phase B (24h observation).

## Stable Single-Consumer Operational Baseline: 2026-05-29

### Stable Workflow Finalized
Current operationally stable workflow in n8n runtime:

```text
Meta Report VERIFIED
```

The workflow name was manually finalized by the user in n8n UI. Do not rename it from automation or database tooling.

### Single-Consumer State Verified
Verified active workflow state:
- `Meta Report VERIFIED`: active
- `Meta Report TEST new`: inactive/deprecated
- `Telegram Bot & Reports`: inactive/deprecated
- `My workflow`: inactive/non-Telegram test artifact

Only one active Telegram polling consumer remains. This resolves the previous double-consumer chaos.

### Current Verified Features
PASS:
- Telegram polling verified
- DD/MM parser verified
- KPI aggregation verified
- Google Sheet integration verified
- Telegram report delivery verified
- routing logic verified
- single-consumer workflow state verified
- multiple command routing verified
- no more `[NODE=UNKNOWN]` spam on stable workflow

### Current Verified Commands
- `báo cáo 24/03`
- `báo cáo hôm qua`
- `báo cáo hôm nay`

### Current Recovery Progress
Stable baseline frozen as:

```text
META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json
```

This is now the LAST KNOWN GOOD VERIFIED STABLE BASELINE.

### Current Known Limitation
Business-intent dedupe for same user + same command spam within a short interval is not production-verified.

Current decision:
- Treat business-intent dedupe as deferred enhancement.
- It is not a blocker for stable operational baseline.

Reason:
- Core functionality is operational.
- Dedupe behavior may require future architecture redesign.
- Business priority shifts toward data freshness and Meta realtime sync.

### Current Focus
Next strategic focus:
- Meta realtime sync architecture for current-day reporting.
- Google Sheet historical cache for previous-day reporting.


## Google Sheet Data Ownership Discovery: 2026-05-29

Current Meta Ads report Google Sheet is the operational Source of Truth.

Important discovery:
- The Sheet contains both Meta-managed fields and non-Meta business fields.
- Several columns contain Google Sheet formulas, manual sales updates, customer quality review data, or post-lead business evaluation.
- Future Meta Sync workflows must adapt to the Sheet structure.
- Do not modify the Sheet only to satisfy workflow convenience.

### Meta API Managed Columns
These columns may be updated by Meta API sync workflows:

- Ma_quang_cao
- Ngay
- Chi_tieu
- Nguoi_tiep_can
- Ngan_sach
- click
- Trang_Thai
- Key
- Thoi_diem_cap_nhat

### Business / Formula / Manual Columns
These columns are not owned by Meta API sync workflows and must not be overwritten or reset unless explicitly approved:

- Chien_dich
- Ten_quang_cao
- Mess_Comment
- Khach_sai_tep
- Khach_hop_le
- SDT
- Khach_chot
- Chi_phi_tren_khach_hop_le
- Chi_phi_khach_tren_SDT
- Chi_phi_khach_chot
- ghi_chu

Operational rule:
- Workflow patches should prefer schema compatibility and selective updates over broad row replacement.
- Manual/business/formula fields must be preserved during future Meta sync design.

## Xác Minh Telegram Offset: 2026-06-02

Đã đọc trực tiếp từ SQLite database bên trong Docker container `n8n-msp`.

### Kết quả

**Node: `Telegram API getUpdates`**
```
offset = {{$getWorkflowStaticData('global').last_update_id || undefined}}
```
→ Đây là dynamic expression — đọc từ staticData runtime. KHÔNG hardcoded.

**Node: `ACK Telegram Update`**
```
offset = {{ $node["Process Updates"].json.update_id + 1 }}
```
→ Đây là dynamic expression — tính từ update_id runtime. KHÔNG hardcoded.

**Tìm kiếm giá trị `81218650` trong toàn bộ node:**
→ KHÔNG TÌM THẤY. Đã xóa hoàn toàn.

### Kết luận
Blocker "restore offset expressions" đã được giải quyết trước phiên 2026-06-02.
Hardcoded offset `81218650` không còn tồn tại trong workflow đang chạy.
Checklist item này đã tick.

---

## Trạng Thái Workflows Trong n8n: 2026-06-02

Đã xác minh từ SQLite database.

| Workflow | Trạng thái |
|----------|-----------|
| **Meta Report VERIFIED** | ✅ ACTIVE — workflow duy nhất đang chạy |
| Meta Report TEST new | ❌ inactive |
| Telegram Bot & Reports | ❌ inactive |
| My workflow | ❌ inactive |
| Meta Ads Daily Sheet Update | ❌ inactive |
| Meta Ads Daily Sheet Update 1 | ❌ inactive |
| Meta Ads Daily Sheet Update 2 | ❌ inactive |

Ghi chú: Có 3 phiên bản "Meta Ads Daily Sheet Update" đang tồn tại trong n8n nhưng tất cả đều inactive. Đây là các workflow sync Meta → Sheet chưa hoàn thiện, chưa được review và chưa được approve để chạy.

---

## Bước Tiếp Theo Để Production Promotion

Chỉ còn **một blocker duy nhất**:

> **Restart workflow `Meta Report VERIFIED`, sau đó gửi command Telegram và xác minh không có stale replay, không có báo cáo cũ bị gửi lại.**

Quy trình cụ thể:
1. Deactivate `Meta Report VERIFIED` trong n8n UI.
2. Activate lại.
3. Gửi `báo cáo hôm qua` qua Telegram.
4. Quan sát: chỉ được nhận đúng 1 response, không có report cũ bị replay.
5. Nếu pass → tick checklist → xem xét GO/NO-GO với người có thẩm quyền.

Sau khi blocker này pass → bắt đầu Phase B (24h sustained observation) theo `PRODUCTION_PROMOTION_CHECKLIST.md`.

---

## Trạng Thái Meta Ads Daily Sheet Update: 2026-06-02

### Kết quả đã xác minh

| Hạng mục | Trạng thái |
|----------|-----------|
| Workflow hoạt động | ✅ OPERATIONAL |
| Dữ liệu 27/04/2026 → 01/06/2026 | ✅ ĐẦY ĐỦ — đã xác minh |
| Backfill 27/04 và 28/04 | ✅ HOÀN THÀNH — đã xác minh |
| Protected columns (Nhóm B) không bị ghi đè | ✅ VERIFIED thực tế |
| Column to match on | ✅ `Key` — đã cấu hình |
| Schema migration sang snake_case | ✅ HOÀN THÀNH |

### Nguyên tắc hoạt động

- Workflow chỉ ghi vào **Nhóm A** (cột Meta API owns).
- Dùng cột `Key` để match hàng — tránh tạo trùng.
- Các cột Nhóm B (khách hàng, chất lượng lead, công thức) được giữ nguyên.
- Chi tiết phân loại cột → xem `DATA_SCHEMA_RULES.md`.

### Việc còn lại

| Việc | Ghi chú |
|------|---------|
| Build **Workflow Yesterday** | ✅ DONE — test PASS, đã ACTIVATE 2026-06-09 |
| Build **Workflow Today** | ✅ DONE — chưa test |
| Meta Ads Daily Sheet Update cron | ✅ ACTIVATE 07:30 hằng ngày 2026-06-09 |
| Test restart persistence cho `Meta Report VERIFIED` | Blocker cuối của production promotion — xem section trên |

### Trạng thái Production Chain — ĐÃ XÁC MINH 2026-06-09

Chuỗi đang chạy production:
```
07:30  Meta Ads Daily Sheet Update   → Sheet cập nhật dữ liệu Meta
08:13  Meta Report Yesterday         → Telegram báo cáo hôm qua
```

| Workflow | Trạng thái | Ghi chú |
|----------|-----------|---------|
| `Meta Ads Daily Sheet Update` | ✅ ACTIVE 07:30 | File: `META_ADS_DAILY_SHEET_UPDATE_SCHEDULED_SUMMARY_SAFE_2026-06-09.json` |
| `Meta Report Yesterday Scheduled` | ✅ ACTIVE 08:13 | KPI khớp Sheet, Telegram đúng format |
| `Meta Report Today Scheduled` | ⏳ CHƯA TEST | Chưa activate |
| `Meta Report VERIFIED` | ✅ ACTIVE | Telegram bot polling — giữ nguyên |

**Verify idempotent Meta Ads Sync (ĐÃ XÁC MINH):**
- Lần 1: Append=21, Update=0 → ghi hàng mới
- Lần 2: Append=0, Update=319 → không ghi trùng, chỉ update
- Kết luận: an toàn khi chạy nhiều lần cùng ngày

**Fix Summary node:** Dùng `safeCount` helper với try/catch — chỉ bắt lỗi `hasn't been executed`, không che lỗi thật. Root cause: gọi `$('IF: Update?')` khi node chưa execute trong context hiện tại.

**Patch "Chưa cập nhật":** Nếu `khach_hop_le = 0` VÀ `sdt = 0` VÀ `khach_chot = 0` → hiển thị "Chưa cập nhật". Nếu bất kỳ giá trị nào > 0 → hiển thị số thật và tính CPA bình thường.

### Incident 2026-06-10 — Google OAuth Hết Hạn — ĐÃ XÁC MINH RESOLVED

**Root cause:** Google Sheets OAuth refresh token bị invalid/revoked đột ngột. UI n8n vẫn hiện "Account connected" nhưng runtime không refresh được token.

**Impact:** Cả 2 workflow chết cùng lúc vì dùng chung credential:
- 07:30 — Sheet Update không đọc được Sheet → không update data
- 08:13 — Yesterday Report không đọc được Sheet → không gửi Telegram
- Dữ liệu 09/06 sai (~71k thay vì ~636k)

**Resolution:** Reconnect Google Sheets credential (Sign in with Google lại) → cả 2 workflow chạy lại bình thường. Data 09/06 đã khớp ~636k.

**Risk còn lại:** Google OAuth là single point of failure. Cần chuyển sang Service Account để loại bỏ nguy cơ expire.

---

### Việc Còn Lại Theo Thứ Tự — VIỆC CẦN LÀM 2026-06-10

**Giai đoạn 1 — Làm ngay:**
1. Giảm `numDays` từ 30 → 7 trong Meta Sync (Meta adjust data chậm 1-7 ngày, không cần 30).
2. Thêm **Telegram Error Alert** khi workflow fail — Global Error Handler dùng chung.
3. Thêm **CPC** vào báo cáo Telegram (Chi tiêu / Click).

**Giai đoạn 2 — Sau khi ổn định 1 tuần:**
4. Chuyển Google OAuth → **Google Service Account** (không expire, loại bỏ single point of failure).
5. Test và Activate `Meta Report Today Scheduled`.
6. Tối ưu báo cáo (CTR, CPM nếu cần).

**Giai đoạn 3 — Chưa cần làm ngay:**
7. **⚠️ Revoke Telegram Bot Token cũ** (token đã lộ trong chat).
8. Health Check workflow định kỳ riêng biệt.

### Ghi chú kiến trúc

- **Yesterday**: đọc từ Sheet → an toàn, không cần gọi API.
- **Today**: gọi Meta API trực tiếp → cần xử lý token và rate limit.
- Scheduled workflow **không dùng `$json.chat_id`** — phải dùng `$env.TELEGRAM_CHAT_ID` vì không có user trigger context.
- Thứ tự activate quan trọng: Sync trước (07:30), Report sau (08:13).

---

---

## META REPORT VERIFIED SOURCE CONTROLLED — 2026-06-16

| Hạng mục | Chi tiết |
|----------|---------|
| File export | `docs/workflows/META_REPORT_VERIFIED_EXPORT_2026-06-16.json` |
| Runtime ID | `tD0zGW33435s4Y8y` |
| versionId | `4c765861-8d02-47b5-b578-cb96acad6c4b` |
| Audit vs baseline 2026-05-29 | ✅ Khớp — không có drift cấu trúc |
| Node count | 15 nodes — không thay đổi |
| Credentials | Google Sheets account `iGuF5SVznNPI7ihl` — không thay đổi |
| Trigger | Cron Polling 30s — không thay đổi |
| Routing | IF Menu → IF Report → IF Range — không thay đổi |
| KPI logic | chi_tieu + khach_hop_le + CPA — không thay đổi |
| CREDENTIAL_INVENTORY.md | ✅ Đã cập nhật (Section 1 + Section 6) |
| Trạng thái mới | **Source Controlled Production Workflow** |

---

## STABLE PRODUCTION STATE CONFIRMED — 2026-06-16

### Project Status
🟢 **STABLE PRODUCTION STATE**
Tất cả 7 workflow đã vận hành thành công trong production. Hệ thống tự chạy đúng lịch, không cần can thiệp thủ công.

### Production Verification — 2026-06-16

| Thời gian | Workflow | Trạng thái |
|-----------|----------|-----------|
| 07:00 | Google Sheets Health Check | ✅ CONFIRMED |
| 07:05 | Meta API Health Check | ✅ CONFIRMED |
| 07:30 | Meta Ads Daily Sheet Update | ✅ CONFIRMED |
| 08:13 | Yesterday Report | ✅ CONFIRMED |
| 11:31 | Today Report | ✅ CONFIRMED |
| 16:31 | Today Report | ✅ CONFIRMED |
| 21:13 | Today Report | ✅ CONFIRMED |

### Major Incidents Resolved

| Incident | Trạng thái |
|----------|-----------|
| Google OAuth credential failure | ✅ RESOLVED |
| KPI Mess_Comment inflation (fuzzy action matching) | ✅ RESOLVED |
| Historical data inconsistency 24/03 → 10/06 | ✅ RESOLVED |
| Duplicate Key records in Google Sheet | ✅ RESOLVED |

### Roadmap Tiếp Theo

| # | Việc | Priority |
|---|------|----------|
| 1 | Credential Inventory (mapping đầy đủ) | HIGH |
| 2 | Google Service Account Migration | MEDIUM |
| 3 | Telegram Alert phân loại mức độ lỗi | LOW |
| 4 | KPI Dashboard | LOW |

---

## Service Account Migration — Google Sheets (2026-06-19, đang tiếp tục)

### Mục tiêu
Thay thế Google OAuth2 (expire định kỳ, cần reconnect thủ công) bằng
Google Service Account (không expire, không cần can thiệp thủ công).

### Tiến độ

| Bước | Trạng thái | Ghi chú |
|------|-----------|---------|
| Tạo Service Account | ✅ DONE | `n8n-meta-ads-sa@handy-tiger-494410-m3.iam.gserviceaccount.com` |
| Tạo JSON Key | ✅ DONE | |
| Share Google Sheet quyền Editor | ✅ DONE | |
| Tạo credential "Google Sheets Service Account" trong n8n | ✅ DONE | |
| HTTP Request Read (`Đọc toàn bộ Sheet`) | ✅ PASS | Cần bật "Set up for use in HTTP Request node" + scope `https://www.googleapis.com/auth/spreadsheets` |
| Update Row (`Cập nhật vào Google Sheet`) | ✅ PASS | Đã xác minh bằng dữ liệu thật (xóa Chi_tieu → chạy → khôi phục đúng) |
| Append Row (`Ghi mới vào Google Sheet`) | ⏳ CHƯA TEST | Bị gián đoạn do Meta token incident 2026-06-22 |
| Full workflow test trên tab TEST | ⏳ PENDING | |
| So sánh output vs production | ⏳ PENDING | |
| Go-live production | ⏳ PENDING | |

### Lesson Learned

- Khi đổi Google Sheets OAuth → Service Account, n8n **reset cấu hình node** → KHÔNG coi migration PASS chỉ vì node không báo lỗi → Phải xác minh bằng dữ liệu thật.
- Khi đổi credential, n8n reset: Column to match on + Mapping mode + Values to Update → phải cấu hình lại thủ công.
- **Mapping mode:** `Map Each Column Manually` (không dùng `Map Automatically`).
- **Chỉ map Nhóm A:** `Ma_quang_cao, Ngay, Chi_tieu, Nguoi_tiep_can, Ngan_sach, click, Trang_Thai, Thoi_diem_cap_nhat, Mess_Comment`
- **Để trống Nhóm B:** `Khach_sai_tep, Khach_hop_le, SDT, Khach_chot, Chi_phi_tren_khach_hop_le, Chi_phi_khach_tren_SDT, Chi_phi_khach_chot, ghi_chu`
- Tên credential hiển thị không đáng tin → phải mở bút chì xem Credential Type thật.
- **DATA_SCHEMA_RULES.md đang phân loại sai:** `Mess_Comment` thuộc Nhóm A (Meta tự động), không phải Nhóm B → cần sửa sau migration.

### Việc Cần Làm Tiếp (theo thứ tự)

1. Test Append Row (xóa vài dòng trong tab `BÁO_CÁO_QUẢNG_CÁO_TEST` để tạo điều kiện nhánh Append chạy → Execute Previous Nodes)
2. Bổ sung field `Mess_Comment` vào mapping Update Row (đã xác nhận là Nhóm A)
3. Quyết định `Chien_dich` + `Ten_quang_cao` thuộc nhóm nào
4. Full workflow test trên tab TEST
5. So sánh output `BÁO_CÁO_QUẢNG_CÁO` vs `BÁO_CÁO_QUẢNG_CÁO_TEST`
6. Go-live production
7. Sửa `DATA_SCHEMA_RULES.md` cho đúng thực tế nghiệp vụ

---

## INCIDENT RESOLVED: Meta API 403 Bearer Prefix Missing — 2026-06-22

| Hạng mục | Chi tiết |
|----------|---------|
| Thời gian | 21/06/2026 ~17:19 VN (token expire) → 22/06/2026 (resolved) |
| Workflows bị ảnh hưởng | Daily Sheet Update, Today Report, Meta API Health Check |
| Lỗi gốc | `(#200) Provide valid app ID` — 403 từ Meta API |
| Root cause | Header Auth credential thiếu prefix `Bearer ` trong Authorization header |
| Sai | `Authorization: EAASxxxx` |
| Đúng | `Authorization: Bearer EAASxxxx` |
| Resolution | Thêm `Bearer ` prefix vào Header Auth credential → tất cả workflow PASS |
| Trạng thái | ✅ RESOLVED |

**Lesson quan trọng:** Lỗi `(#200) Provide valid app ID` không phải lỗi App ID — kiểm tra Authorization header format trước.
**Khi renew token:** luôn verify `Bearer ` prefix còn nguyên, không chỉ thay giá trị token thuần.

---

## INCIDENT RESOLVED: ECONNRESET Today Report — 2026-06-17

| Hạng mục | Chi tiết |
|----------|---------|
| Workflow bị ảnh hưởng | Meta Report Today Scheduled |
| Node lỗi | `Send Report Telegram` |
| Lỗi | `ECONNRESET` — mạng tạm thời |
| Root cause | Kết nối tạm thời bị ngắt giữa n8n container và Telegram API |
| Loại trừ | Token OK (TEST PASS), cron OK (slot 11:31 PASS), workflow logic OK |
| Resolution | Retry On Fail = true, Max Attempts = 3, Wait = 2000ms |
| Trạng thái | ✅ RESOLVED |

**Việc còn lại sau incident:**
1. Export `META_REPORT_TODAY_SCHEDULED_2026-06-17.json`
2. Patch format báo cáo ngắn hơn (Today + Yesterday)
3. Thêm `impressions` → CPM và CTR
4. Google Service Account Migration

---

## PRODUCTION BASELINE — 2026-06-16
_Dùng làm rollback reference cho Service Account Migration và các thay đổi infrastructure tiếp theo._

### Workflows Đang ACTIVE

| Workflow | Schedule | Ghi chú |
|----------|----------|---------|
| Google Sheets Health Check | 07:00 | Alert Telegram nếu credential fail |
| Meta API Health Check | 07:05 | Alert Telegram nếu token fail |
| Meta Ads Daily Sheet Update | 07:30 | numDays=7, KPI exact match |
| Meta Report Yesterday Scheduled | 08:13 | Đọc Sheet → Telegram |
| Meta Report Today Scheduled | 11:31 / 16:31 / 21:13 | Meta API trực tiếp, CPC included |
| Meta Report VERIFIED | Realtime polling | Telegram bot, Source Controlled |

### Credentials Đang Dùng

| Credential | Loại | Dùng bởi |
|------------|------|---------|
| Google Sheets account (OAuth) | n8n credential | Daily Sheet Update, Yesterday Report, GS Health Check, Backfills |
| Header Auth account (Meta API) | n8n credential | Daily Sheet Update, Backfills |
| META_ACCESS_TOKEN | .env | Today Report, Daily Sheet Update, Meta API Health Check |
| TELEGRAM_BOT_TOKEN | .env | Today Report, Yesterday Report, GS Health Check, Meta API Health Check, Meta Report VERIFIED |
| TELEGRAM_CHAT_ID | .env | Today Report, Yesterday Report, GS Health Check, Meta API Health Check |

### Rollback References

| File | Mục đích |
|------|---------|
| `docs/workflows/META_REPORT_VERIFIED_EXPORT_2026-06-16.json` | Runtime export hiện tại — versionId verified |
| `docs/backups/META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json` | Last known good baseline |
| `CREDENTIAL_INVENTORY.md` Section 4 | Thứ tự migration an toàn: Google OAuth → Service Account |

### Trạng Thái
**Known Good State** — Tất cả workflow verified, data integrity confirmed, source control updated.

### Giai Đoạn Tiếp Theo
**Infrastructure Hardening Phase:**
1. Google Service Account Migration (loại bỏ OAuth single point of failure)
2. Telegram Alert phân loại mức độ lỗi
3. KPI Dashboard

## MONITORING LAYER COMPLETE — 2026-06-15

### Project Status
**Production Ready — Pending 24-48h Observation**
Chưa quan sát đủ một chu kỳ tự động đầy đủ. Hệ thống đang chạy tự động, không cần can thiệp thủ công.

### Lịch Chạy Production Đầy Đủ

| Thời gian | Workflow | Mô tả |
|-----------|----------|-------|
| 07:00 | Google Sheets Health Check | Kiểm tra credential Google Sheets — alert nếu fail |
| 07:05 | Meta API Health Check | Kiểm tra Meta API token — alert nếu fail |
| 07:30 | Meta Ads Daily Sheet Update | Sync Meta → Sheet (numDays=7) |
| 08:13 | Meta Report Yesterday Scheduled | Báo cáo hôm qua → Telegram |
| 11:31 | Meta Report Today Scheduled | Báo cáo hôm nay → Telegram |
| 16:31 | Meta Report Today Scheduled | Báo cáo hôm nay → Telegram |
| 21:13 | Meta Report Today Scheduled | Báo cáo hôm nay → Telegram |

### Monitoring Layer — Verification State

| Workflow | PASS path | FAIL path | PASS after restore |
|----------|-----------|-----------|-------------------|
| Google Sheets Health Check | ✅ verified | ✅ verified (Telegram alert) | ✅ verified |
| Meta API Health Check | ✅ verified | ✅ verified (Telegram alert) | ✅ verified |

### Roadmap Tiếp Theo

| Việc | Priority |
|------|----------|
| Quan sát 24-48h hệ thống tự chạy | **ĐANG THỰC HIỆN** |
| Google Service Account migration | MEDIUM |
| Telegram Alert phân loại mức độ lỗi | LOW |
| Dashboard tổng quan KPI | LOW |
| Credential Inventory (mapping đầy đủ) | LOW |
| ⚠️ Revoke Telegram Bot Token cũ | MEDIUM |

---

## STABLE PRODUCTION STATE — 2026-06-11

### Tất Cả Workflow Production

| Workflow | Schedule | Trạng thái | Ghi chú |
|----------|----------|-----------|---------|
| `Meta Report VERIFIED` | realtime polling | ✅ ACTIVE | Telegram bot, single consumer |
| `Meta Ads Daily Sheet Update` | 07:30 | ✅ ACTIVE | numDays=7, exact match KPI |
| `Meta Report Yesterday Scheduled` | 08:13 | ✅ ACTIVE | Đọc Sheet → Telegram |
| `Meta Report Today Scheduled` | 11:31 / 16:31 / 21:13 | ✅ ACTIVE | Meta API trực tiếp, CPC included |

### Production Chain Verified

```
07:30  Meta Ads Daily Sheet Update   → Sheet cập nhật dữ liệu Meta (numDays=7)
08:13  Meta Report Yesterday         → Telegram báo cáo hôm qua (đọc Sheet)
11:31  Meta Report Today             → Telegram báo cáo hôm nay (Meta API trực tiếp)
16:31  Meta Report Today             → Telegram báo cáo hôm nay (Meta API trực tiếp)
21:13  Meta Report Today             → Telegram báo cáo hôm nay (Meta API trực tiếp)
```

### Today Report — Kiến Trúc Và KPI

Nguồn dữ liệu: **Meta Marketing API trực tiếp** — không đọc Google Sheet, không phụ thuộc Google OAuth.

Telegram output đã verify:
- Chi tiêu
- Tiếp cận
- Tin nhắn (`onsite_conversion.messaging_conversation_started_7d` — exact match)
- Clicks
- CPC (`totalSpend / totalClick`, bảo vệ divide-by-zero)

Kết quả verify: **100% khớp Ads Manager.**

### Data Integrity State

| Hạng mục | Trạng thái |
|----------|-----------|
| KPI Mess_Comment exact match | ✅ VERIFIED |
| Backfill 24/03 → 10/06 | ✅ appendCount=0, updateCount>0 |
| COUNTIF(Key) = 1 toàn cột | ✅ Không còn duplicate |
| Today Report khớp Ads Manager | ✅ 100% |
| Yesterday Report khớp Sheet | ✅ VERIFIED |

### Việc Còn Lại (Không Phải Blocker)

| Việc | Priority |
|------|----------|
| Telegram Error Alert (Global Error Handler) | HIGH |
| Google Service Account migration | MEDIUM (loại bỏ OAuth single point of failure) |
| ⚠️ Revoke Telegram Bot Token cũ | MEDIUM |
| Phase B 24h observation `Meta Report VERIFIED` | LOW (workflow đã stable) |

---

## Trạng Thái Sau Giai Đoạn 09/06 → 11/06/2026

### Đã Xác Minh và Hoàn Tất

| Hạng mục | Trạng thái |
|----------|-----------|
| Google OAuth reconnected | ✅ XÁC MINH |
| Daily Sheet Update hoạt động | ✅ XÁC MINH |
| Yesterday Report gửi Telegram | ✅ XÁC MINH |
| KPI Mess_Comment exact match | ✅ PATCHED + VERIFIED |
| Backfill 24/03 → 10/06 | ✅ HOÀN TẤT (appendCount=0) |
| Duplicate Key đã dọn | ✅ COUNTIF(Key)=1 toàn cột |
| Dữ liệu Meta khớp Ads Manager | ✅ SPOT CHECK PASS |
| numDays = 7 | ✅ ĐÃ ÁP DỤNG + VERIFY |

### Workflow State — 2026-06-11

| Workflow | Trạng thái | Ghi chú |
|----------|-----------|---------|
| `Meta Report VERIFIED` | ✅ ACTIVE | Telegram bot polling |
| `Meta Ads Daily Sheet Update` | ✅ ACTIVE 07:30 | numDays=7, KPI fix applied |
| `Meta Report Yesterday Scheduled` | ✅ ACTIVE 08:13 | Đọc Sheet → Telegram |
| `Meta Report Today Scheduled` | ⏳ CHƯA TEST | Chưa activate |

### Việc Cần Làm Tiếp

1. Execute test production 1 lần cuối (verify appendCount=0, updateCount>0).
2. Publish workflow production.
3. Tiếp tục roadmap:
   - Thêm Telegram Error Alert (Global Error Handler).
   - Thêm CPC vào báo cáo Telegram.
   - Test + Activate Today Report (11:31 / 16:31 / 21:13).
   - Google Service Account migration (dài hạn).

### Root Cause Summary Giai Đoạn Này

| Incident | Root Cause | Resolution |
|----------|-----------|------------|
| Google OAuth hết hạn | Refresh token expired, UI không báo | Reconnect credential |
| KPI Mess_Comment sai | Fuzzy `includes()` match action_type | Exact `===` match |
| Duplicate Key | Append thay vì update khi patch | Dọn thủ công, giữ bản mới |
| Lịch sử sai 24/03→10/06 | Hệ quả của KPI bug | Backfill với logic đúng |

---

## 2026-06-25 — Service Account Migration: Implementation Done, Scheduler Pending

### Trạng thái Migration
- Implementation: **Done**
- Scheduler Verification: **Pending**

| Bước | Trạng thái |
|------|-----------|
| Configuration verified — toàn bộ node đã đổi sang Service Account | ✅ |
| Manual execution verified — execute workflow PASS, dữ liệu ghi đúng | ✅ |
| Scheduled execution — chờ scheduler 2026-06-26 chạy tự động | ⏳ |

### Chi tiết sau khắc phục incident sáng 2026-06-25

| Workflow | Node | Trạng thái |
|----------|------|-----------|
| Meta Ads Daily Sheet Update 7:30 2 | Đọc toàn bộ Sheet (HTTP Request) | ✅ → Service Account |
| Meta Ads Daily Sheet Update 7:30 2 | Cập nhật vào Google Sheet (Update Row) | ✅ → Service Account |
| Meta Ads Daily Sheet Update 7:30 2 | Ghi mới vào Google Sheet (Append Row) | ✅ → Service Account |
| Meta Report Yesterday Scheduled 1 | Đọc Google Sheet (HTTP Request) | ✅ → Service Account |
| Meta Report Today Scheduled | (không có node đọc Google Sheet) | ✅ Không bị ảnh hưởng |
| Google Sheets Health Check 07:00 | Node đọc | ✅ → Service Account |
| Google Sheets Health Check 07:00 | Message Telegram FAIL | ✅ Đã cập nhật đúng kiến trúc mới |

### Scheduler Verification Checklist (2026-06-26)
- ☐ 07:00 Google Sheets Health Check PASS
- ☐ 07:30 Meta Ads Daily Sheet Update PASS
- ☐ 08:13 Yesterday Report PASS
- ☐ 11:31 Today Report PASS

Sau khi đủ 4 điều kiện trên → đổi trạng thái thành **FULLY VERIFIED**

### Trạng thái credential
- Google Sheets: Service Account ✅ (toàn bộ các workflow production hiện có)
- Meta Access Token: đang hoạt động ⏳ (chưa xác minh loại token)
- Telegram Bot Token: ⚠️ Token cũ đã lộ trong chat, cần revoke

---

## 2026-06-24 — Service Account Migration COMPLETE - PENDING SCHEDULED RUN VERIFICATION

### Kết quả Migration
✅ Workflow production "Meta Ads Daily Sheet Update 7:30 2" đã migrate sang Google Service Account
✅ 3 node Google đã đổi credential:
   - Đọc toàn bộ Sheet (HTTP Request)
   - Cập nhật vào Google Sheet (Update Row)
   - Ghi mới vào Google Sheet (Append Row)
✅ Không còn OAuth credential trong các node Google Sheets của workflow production
✅ Manual Execute PASS
✅ Execute Workflow PASS
✅ Update Row ghi dữ liệu đúng vào Sheet production
✅ Append Row ghi dữ liệu đúng vào Sheet production
✅ Meta API token hoạt động bình thường

### Lưu ý kỹ thuật
- Node HTTP Request: cần bật "Set up for use in HTTP Request node" + set scope
- Node Update Row: sau khi đổi credential, kiểm tra lại Column to match on + Mapping mode
- Node Append Row: sau khi đổi credential, kiểm tra lại Sheet target + Mapping mode
- Workflow TEST song song: "Meta Ads Daily Sheet Update 7:30 3 - SA TEST" [ARCHIVED 2026-06-24]

### Monitoring
⏳ Chờ xác nhận scheduler 07:30 chạy tự động thành công ít nhất 1 lần
   → Chỉ sau khi scheduler PASS mới đổi trạng thái thành COMPLETED

### Trạng thái credential
- Google Sheets: Service Account ✅ (không expire)
- Meta Access Token: đang hoạt động ⏳ (chưa xác minh loại token - Short/Long/System User)
- Telegram Bot Token: ⚠️ Token cũ đã lộ trong chat, cần revoke

### Next Priorities (theo thứ tự thực tế)
1. Xác minh loại Meta token hiện tại qua Token Debugger → ghi vào CREDENTIAL_INVENTORY.md
2. Báo cáo Hôm nay
3. Báo cáo Hôm qua
4. Báo cáo 7 ngày
5. Báo cáo 30 ngày
6. Khoảng ngày tùy chọn
7. Patch format báo cáo ngắn hơn
8. Thêm Impressions → CPM / CTR
9. ⚠️ Revoke Telegram Bot Token cũ
10. Chuyển Meta từ User Token → System User Token (Business Manager ID: 187499196519080)
11. Telegram Alert severity levels
12. Dashboard KPI

---

## Đã Xác Minh — 2026-06-02

**Nguồn xác minh:** Docker container restart test + Telegram runtime test.

| Kết quả | Trạng thái |
|---------|-----------|
| `Meta Report VERIFIED` đang ACTIVE | ✅ XÁC MINH |
| Chỉ có 1 consumer Telegram duy nhất | ✅ XÁC MINH |
| Hardcoded offset `81218650` đã xoá | ✅ XÁC MINH |
| Offset động hoạt động bình thường sau container restart | ✅ XÁC MINH |
| Không xảy ra stale replay sau restart | ✅ XÁC MINH |
| Lệnh `báo cáo hôm qua` trả về đúng 1 phản hồi | ✅ XÁC MINH |

**Trạng thái production promotion:**
- Phase A (controlled activation): ✅ HOÀN TẤT
- Phase B (24h observation): ⏳ CHƯA BẮT ĐẦU — việc tiếp theo

---

## 2026-06-30 — Meta KPI Expansion COMPLETED

### Feature
Meta KPI Expansion (Impressions, CPM, CTR, bộ KPI Video)

### Status
COMPLETED
Verified: 2026-06-30
Production: PASS

### Production Baseline
Workflow: 2026-06-30_Meta_Ads_Daily_KPI_Video_VERIFIED.json

### Production Verification
Date: 2026-06-30
Verified by:
✓ Meta API response
✓ Parser
✓ Update Row
✓ Append Row
✓ Google Sheet (dữ liệu thực tế xác nhận đúng)
✓ End-to-end full workflow

### Schema mới
Đã mở rộng schema với nhóm Raw Metrics và Derived Metrics.
Chi tiết xem DATA_SCHEMA_RULES.md và META_FIELD_MAPPING.md

### Trạng thái Parser
FEATURE FREEZE — không sửa thêm node Xử lý dữ liệu trừ khi:
- Meta thay đổi cấu trúc API
- Cần bổ sung KPI hoàn toàn mới

---

## Date Range Engine V1: RUNTIME VERIFIED

### Trạng thái
Date Range Engine V1: RUNTIME VERIFIED
Architecture Freeze V1: PASS
Integration Flow V1: PASS
Production: KHÔNG thay đổi

### Những gì đã làm
- Thiết kế Date Range Engine V1 với kiến trúc Filter + Aggregate + KPI
- Implement trên workflow TEST (inactive, không ảnh hưởng Production)
- Verify từng node theo thứ tự: Đọc Google Sheet → Normalize Sheet Data → Build Engine Input → Date Range Engine
- Verified using runtime execution against real production data
- Đối chiếu với dữ liệu thật: filteredCount, spend, impressions, click, clickAll, CPM, CTR Link, CTR All, CPC — tất cả khớp

### Cấu trúc workflow hiện tại (Meta Report Yesterday Scheduled 1)

Current production logic:
Normalize Sheet Data → Lọc & Tính KPI Hôm Qua → Send Report Telegram

TEST branch (inactive):
Normalize Sheet Data → Build Engine Input → Date Range Engine

### Design Decisions đã chốt (Architecture Freeze V1)
- Date Range Engine V1 = Filter + Aggregate + KPI (1 node)
- AGGREGATION_CONFIG đặt ở vùng Configuration đầu node
- normalizeDate() giữ như Compatibility Layer (có TODO xóa sau khi dữ liệu migrate sang YYYY-MM-DD)
- Validate input ngay đầu Engine
- Chưa tách KPI Engine riêng — xem xét ở V2 khi có bằng chứng thực tế

### Known Issue: Mojibake trong workflow JSON
Sau quy trình export → chỉnh sửa → import workflow JSON, xuất hiện mojibake ở một số chuỗi tiếng Việt:
- Node Display Name: "Tính Ngày Hôm Qua" → "TÃ­nh NgÃ y HÃ´m Qua"
- Sheet name trong URL: "BÁO_CÁO_QUẢNG_CÁO" → "BÃO_CÃO_QUẢNG_CÁO"
Đã sửa thủ công. Root cause CHƯA XÁC ĐỊNH.
Xem LL-014 và LL-015 trong LESSONS_LEARNED.md.

### Việc tiếp theo
- Tích hợp Date Range Engine vào Yesterday Report
- Presentation Layer (Render Telegram) sử dụng output của Date Range Engine
- Chỉ loại bỏ logic cũ (Lọc & Tính KPI Hôm Qua) sau khi Production verify PASS
- Sau Yesterday PASS → tái sử dụng Engine cho 7 ngày, 30 ngày, khoảng ngày

### KPI Engine Status
Parser: FEATURE FREEZE
KPI Engine: ACTIVE DEVELOPMENT
