# Incidents And Recovery

## Incident: KPI Reports Always Returning Zero

### Symptoms
Telegram reports hiển thị:
```text
0đ
0 CPA
0 results
```

### Root Cause
1. Google Sheets API trả về array rows thay vì object schema.
2. Downstream nodes đang expect fields như `spend` và `leads`.
3. Telegram parser classify `báo cáo 24/03` thành unknown.
4. `Parse & Route` chưa có DD/MM parser support.
5. Polling state reset tạo duplicate-processing risk.
6. `Send Report Telegram` vẫn còn debug placeholder `[NODE=REPORT]` thay vì `$json.message`.

### Recovery Steps
- Thêm `Normalize Sheet Data` ngay sau `Đọc Google Sheet`.
- Convert array rows thành normalized object schema.
- Thêm safe numeric normalization cho strings, decimal commas, empty cells, và `#VALUE!`.
- Thêm DD/MM parser support.
- Thêm report regex support.
- Remove forced staticData reset.
- Patch `Send Report Telegram`: `[NODE=REPORT]` -> `$json.message`.
- Giữ workflow inactive trong recovery.

### Result
Workflow chain đã reach:
```text
Parse & Route -> dates: ["2026-03-24"]
```

KPI recovery chain đã được verify end-to-end sau đó.

### Lessons Learned
- Silent logic bugs nguy hiểm hơn crash.
- Normalize layer nên nằm ngay sau ingestion.
- Polling systems phải giữ state continuity.
- Human-friendly inputs nên normalize một lần sang canonical format.
- Debug placeholders phải remove trước import testing.

## Recovery Continuation: Routing + Parser Chain Restored

### Recovery Milestone
Routing + parser chain restored.

### Verified Chain
```text
Telegram API getUpdates
→ Process Updates
→ Main Router Code
→ IF Report
→ Parse & Route
```

### Verified Output
```json
{
  "dates": ["2026-03-24"]
}
```

### Root Cause Details
- `command_type` parser thiếu DD/MM support.
- `Parse & Route` thiếu DD/MM parser.
- staticData forced reset gây polling replay risk.

### Final Patches
- Added DD/MM regex support.
- Removed forced staticData reset.
- Added canonical ISO date conversion.
- Preserved workflow inactive state during recovery.

### Current Remaining Verification
- Runtime polling continuity đã verify.
- Restart no-stale-replay check vẫn cần làm trước production promotion.

## Final Verified Recovery Result

### Recovery Status
RESOLVED.

### Verified Result
- Successful Telegram delivery.
- Successful KPI aggregation.
- Successful DD/MM command routing.
- Successful schema normalization.
- Successful end-to-end recovery.
- Markdown formatting verified.
- Divide-by-zero protection verified.

### Operational Insight
Breakthrough chính là tách bạch:
- pipeline corruption
- legitimate business-zero KPI data

Ban đầu hệ thống nhìn giống bị hỏng vì report trả KPI zero. Forensic verification chứng minh:
- routing đã fix
- aggregation đã fix
- normalization đã fix
- Telegram delivery đã fix
- campaign date được chọn thật sự có zero valid leads

### Final E2E Verified Output
```text
📊 BÁO CÁO META ADS
📅 Ngày: 2026-03-24
💰 Tổng chi tiêu: 22.587đ
📩 Tổng kết quả: 0
🎯 CPA: 0đ
```

### Baseline
`META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json` là first full end-to-end verified TEST recovery baseline.

## Runtime Validation: Polling Continuity Verified

### Validation Status
PASSED.

### Validation Method
Workflow runtime activation được verify bằng real background execution, không chỉ Execute Step.

### Temporary Test Control
- Temporary hardcoded Telegram offset dùng để drain backlog: `81218650`.
- Backlog replay prevention hoạt động đúng sau draining.
- Governance warning: không bao giờ để hardcoded Telegram offset values trong active production workflows sau testing.

### Verified Runtime Command
```text
báo cáo hôm qua
```

### Verified Runtime Behavior
- no stale replay
- no duplicate reports
- no replay of old 2026-03-24 report
- exactly one Telegram response generated

### Verified Runtime Telegram Output
```text
📊 BÁO CÁO META ADS
📅 Ngày: 2026-05-26
💰 Tổng chi tiêu: 0đ
📩 Tổng kết quả: 0
🎯 CPA: 0đ
```

### Operational Finding
KPI=0 cho 2026-05-26 đã verify là expected business reality do missing/no sheet data, không phải pipeline failure.

### Critical Lesson
Execute Step testing alone không validate đủ runtime polling continuity vì staticData persistence behave khác khi workflow chạy bằng real runtime activation.

## Incident: [NODE=UNKNOWN] Routing Spam

### Recovery Status
RESOLVED.

### Symptoms
- Duplicate reports.
- `[NODE=UNKNOWN]` messages.
- Valid report and unknown branch appearing for the same command.
- Inconsistent routing behavior.
- Misleading topology symptoms.

### Root Causes
- Multiple active Telegram polling consumers.
- Runtime topology drift from exported artifact lineage.
- Mixed workflow lineage confusion during forensic testing.

### Critical Discovery
Two workflows were previously active simultaneously and consumed the same Telegram update queue:
- `Meta Report lỗi`
- `Meta Report TEST new`

This created double consumer chaos. Later investigation confirmed the major chaos source was multiple active polling workflows, not parser failure.

### Resolution
- Stable runtime workflow isolated as `Meta Report VERIFIED`.
- Deprecated workflows confirmed inactive.
- Single-consumer state verified.
- No more `[NODE=UNKNOWN]` spam on stable workflow.

### Lesson
When Telegram polling is used, one queue must have exactly one active consumer. Topology symptoms can be misleading when multiple runtime workflows consume the same event source.

## Incident: Google OAuth Credential Hết Hạn Đột Ngột — 2026-06-10

### Trạng thái
RESOLVED.

### Triệu chứng
- 07:30: Meta Ads Daily Sheet Update không chạy được — không update Sheet.
- 08:13: Meta Report Yesterday không gửi Telegram.
- Dữ liệu ngày 09/06 trong Sheet sai (~71.000đ thay vì ~636.020đ theo Ads Manager).
- n8n UI vẫn hiện "Account connected" cho Google Sheets credential.

### Root Cause — ĐÃ XÁC MINH
Google Sheets OAuth refresh token bị invalid/revoked đột ngột. n8n UI không phát hiện được trạng thái lỗi này ở credential level — chỉ lộ ra lúc runtime thực sự cần gọi API. Cả 2 workflow dùng chung một credential → cùng chết.

Pattern nguy hiểm: **UI xanh, runtime chết, không có alert.**

### Điều tra ban đầu (trước khi tìm ra root cause)
Đã kiểm tra và loại trừ:
- Không thiếu campaign, không thiếu ad_id, không thiếu records.
- Không phải lỗi pagination hay mất data từ Meta API.
- Nghi ngờ ban đầu là timezone/date range lệch — **sai**.

### Resolution
1. Vào n8n UI → Credentials → Google Sheets account.
2. Click "Sign in with Google" → xác thực lại OAuth.
3. Chạy lại cả 2 workflow thủ công.
4. Kết quả: Sheet cập nhật đúng, data 09/06 khớp ~636k, Telegram gửi bình thường.

### Rủi ro còn lại
Google OAuth là single point of failure. Có thể expire lại bất kỳ lúc nào mà không báo trước. Giải pháp dài hạn: chuyển sang Google Service Account (không expire).

### Lessons
- UI credential status không đáng tin — phải dựa vào execution logs.
- Cần Telegram Error Alert tự động để phát hiện workflow fail ngay, không chờ phát hiện thủ công.
- Shared credential là shared risk — một credential chết, tất cả workflow dùng nó đều chết cùng lúc.

---

## Incident: KPI Mess_Comment Sai Toàn Bộ Lịch Sử — 2026-06-09 → 2026-06-11

### Trạng thái
RESOLVED.

### Triệu chứng
- Ads Manager hiển thị Kết quả = 1 cho một số ngày.
- Telegram và Google Sheet hiển thị Mess_Comment = 5, 8, 12, 18... cho cùng ngày đó.
- Dữ liệu không khớp thực tế suốt toàn bộ lịch sử từ 24/03 → 10/06.

### Root Cause — ĐÃ XÁC MINH
Node KPI trong các workflow dùng logic fuzzy matching:

```javascript
a.action_type.includes('message') || a.action_type.includes('conversation')
```

Logic này vô tình gom nhiều action_type không mong muốn:
- `messaging_conversation_replied_7d`
- `messaging_welcome_message_view`
- `total_messaging_connection`
- `messaging_first_reply`
- và các action khác liên quan messaging

Trong khi Ads Manager chỉ hiển thị **"Lượt bắt đầu cuộc trò chuyện"** (`onsite_conversion.messaging_conversation_started_7d`).

### Resolution
Patch toàn bộ 3 workflow liên quan:
- `META_REPORT_TODAY_SCHEDULED`
- `META_ADS_DAILY_SHEET_UPDATE`
- `META_ADS_BACKFILL`

Thay toàn bộ fuzzy matching bằng exact match:

```javascript
a.action_type === 'onsite_conversion.messaging_conversation_started_7d'
```

### Verify
- Telegram Today Report khớp Ads Manager.
- Dữ liệu mới ghi vào Sheet khớp Ads Manager.
- Spot check nhiều ngày xác nhận đúng.

### Lessons
- Fuzzy string matching trên action_type từ Meta API rất nguy hiểm — Meta trả về nhiều action type khác nhau cùng chứa keyword tương tự.
- Luôn dùng exact match cho action_type. Kiểm tra Ads Manager để xác định metric đúng trước khi viết logic.
- Silent data inflation (số quá cao) khó phát hiện hơn silent zero vì trông có vẻ "có data".

---

## Incident: ECONNRESET — Today Report Telegram Send — 2026-06-17

### Trạng thái
RESOLVED.

### Triệu chứng
- `Meta Report Today Scheduled` fail tại node `Send Report Telegram`.
- Lỗi: `ECONNRESET` — kết nối bị ngắt đột ngột khi n8n gửi request đến Telegram API.
- Báo cáo không được gửi ở một trong các slot 11:31 / 16:31 / 21:13.

### Điều tra và loại trừ
- TEST TELEGRAM thủ công: **thành công** → xác nhận token hợp lệ, bot hoạt động bình thường.
- Slot 11:31 hôm nay: **chạy bình thường** → xác nhận scheduler và cron không lỗi.
- Kết luận: ECONNRESET là lỗi mạng tạm thời giữa n8n container và Telegram API, không phải lỗi hệ thống.

### Root Cause — ĐÃ XÁC MINH
ECONNRESET tạm thời khi n8n kết nối Telegram API. Không liên quan đến:
- Scheduler / cron
- TELEGRAM_BOT_TOKEN
- Logic workflow
- Meta API

### Resolution
Bật **Retry On Fail** trong node `Send Report Telegram` của `Meta Report Today Scheduled`:
- `Retry On Fail`: `true`
- `Max Attempts`: `3`
- `Wait Between Attempts`: `2000ms`

### Việc còn lại sau incident
1. Export `META_REPORT_TODAY_SCHEDULED_2026-06-17.json` (có retry config).
2. Patch format báo cáo ngắn hơn cho Today và Yesterday.
3. Thêm `impressions` vào API call để tính CPM và CTR.
4. Sau khi ổn định: Google Service Account Migration.

### Lessons
- ECONNRESET với external API là lỗi mạng tạm thời, không phải lỗi workflow — retry là giải pháp đúng.
- Mọi node gửi request đến external API trong production nên có Retry On Fail.
- Phân biệt: TEST thủ công PASS + slot tiếp theo PASS = lỗi transient, không phải lỗi hệ thống.

---

## Incident: Duplicate Key Trong Google Sheet — 2026-06-09 → 2026-06-11

### Trạng thái
RESOLVED.

### Triệu chứng
- Một số Key xuất hiện 2 lần trong Sheet.
- Dòng cũ có timestamp: 09/06/2026.
- Dòng mới có timestamp: 11/06/2026.
- COUNTIF(Key) > 1 trên nhiều hàng.

### Root Cause — ĐÃ XÁC MINH
Trong giai đoạn sửa workflow (patch KPI Mess_Comment), workflow đã thực hiện **append thay vì update** với dữ liệu đã được patch đúng. Kết quả:
- Dòng cũ (09/06) chứa KPI Mess_Comment sai.
- Dòng mới (11/06) chứa KPI Mess_Comment đúng.

### Resolution
1. Dọn duplicate thủ công trong Sheet.
2. Giữ bản ghi mới (timestamp 11/06 — KPI đúng).
3. Loại bỏ bản ghi cũ (timestamp 09/06 — KPI sai).

### Verify
- COUNTIF toàn bộ cột Key = 1.
- Không còn duplicate.

### Lessons
- Sau bất kỳ patch workflow nào liên quan đến Sheet, phải chạy COUNTIF(Key) để detect duplicate trước khi declare done.
- Append vs Update là hai hành vi khác nhau hoàn toàn — phải verify workflow đang dùng đúng mode.

---

## Incident: Migration Verification Gap — Missed OAuth Nodes — 2026-06-25

### Tóm tắt sự cố
Sáng 2026-06-25, hệ thống báo lỗi qua Telegram:
- GOOGLE SHEETS HEALTH CHECK FAILED
- Không có dữ liệu cập nhật vào Sheet lúc 07:30
- Không có báo cáo Yesterday Report lúc 08:13

### Detection
Incident được phát hiện bởi:
- Telegram Health Check alert lúc 07:00 ngày 2026-06-25
- Kiểm tra thủ công: thiếu dữ liệu Google Sheet sau 07:30
- Kiểm tra thủ công: thiếu Yesterday Report lúc 08:13

### Triệu chứng
- Telegram nhận alert: GOOGLE SHEETS HEALTH CHECK FAILED
- Workflow "Meta Ads Daily Sheet Update 7:30 2" dừng tại node "Đọc toàn bộ Sheet"
- Workflow "Meta Report Yesterday Scheduled 1" dừng tại node "Đọc Google Sheet"
- Lỗi: `EAUTH — The provided authorization grant is invalid, expired, revoked`

### Technical Root Cause
Trong quá trình migration Google Sheets OAuth → Service Account (2026-06-24),
các workflow TEST đã PASS nhưng production còn bỏ sót một số node.

**Node đã migrate:**
- ✅ Append Row → Service Account
- ✅ Update Row → Service Account

**Node bị bỏ sót:**
- ❌ Node "Đọc toàn bộ Sheet" (HTTP Request) — workflow Daily Sheet Update
- ❌ Node "Đọc Google Sheet" (HTTP Request) — workflow Yesterday Report
- ❌ Node đọc trong Google Sheets Health Check
- ❌ Message Telegram Health Check vẫn hướng dẫn reconnect OAuth cũ

### Process Root Cause (nguyên nhân quy trình — quan trọng hơn)
- Không lập inventory toàn bộ node trước khi bắt đầu migration
- Không có migration checklist chuẩn
- Kết luận migration hoàn tất quá sớm sau khi workflow TEST PASS
- Không verify toàn bộ workflow production trước khi đóng migration
- Không chờ scheduler thật chạy trước khi đánh dấu COMPLETED

**Nếu có inventory từ đầu, việc bỏ sót node không thể xảy ra.**

### Hành động khắc phục

**Workflow "Meta Ads Daily Sheet Update 7:30 2":**
- Đổi node "Đọc toàn bộ Sheet" → Google Sheets Service Account
- Execute workflow → PASS

**Workflow "Meta Report Yesterday Scheduled 1":**
- Đổi node "Đọc Google Sheet" → Google Sheets Service Account
- Execute workflow → PASS

**Workflow "Meta Report Today Scheduled 11:31 16:31 21:13":**
- Xác nhận không có node đọc Google Sheet
- Không cần migrate

**Workflow "Google Sheets Health Check 07:00":**
- Đổi node đọc → Google Sheets Service Account
- Cập nhật message Telegram FAIL cho đúng kiến trúc mới

### Trạng thái sau khắc phục
- ✅ Các workflow production hiện có đã migrate hoàn toàn sang Service Account
- ✅ Cả node đọc và node ghi đều dùng Service Account
- ✅ Health Check message đã cập nhật đúng kiến trúc mới
- ⏳ Chờ scheduler ngày 2026-06-26 chạy đủ chu kỳ để xác nhận

### Bài học
Xem LL-008, LL-009, LL-010, LL-011 trong `LESSONS_LEARNED.md`

---

## Incident: Meta API 403 — Bearer Prefix Missing — 2026-06-22

### Trạng thái
RESOLVED.

### Timeline
- **21/06/2026 ~17:19 VN**: Meta Access Token hết hạn. Các workflow gọi Meta API bắt đầu fail với lỗi 403.
- **22/06/2026**: Renew token mới nhưng vẫn lỗi 403.
- **Phát hiện:** Header Auth credential thiếu prefix `Bearer ` trước token.
- **Fix:** Thêm `Bearer ` vào giá trị Authorization header → tất cả workflow hoạt động bình thường.

### Triệu chứng
- Workflow gọi Meta API (Daily Sheet Update, Today Report, Health Check) fail 403.
- Lỗi: `(#200) Provide valid app ID` — **misleading**: không phải lỗi App ID mà là lỗi Authorization header format.

### Root Cause — ĐÃ XÁC MINH
Header Auth credential trong n8n được cấu hình với giá trị:
```
Authorization: EAASxxxx...
```
Thiếu prefix `Bearer `. Format đúng bắt buộc:
```
Authorization: Bearer EAASxxxx...
```

Khi renew token, chỉ thay giá trị token nhưng không kiểm tra lại format header → lỗi tái diễn.

### Workflows bị ảnh hưởng
- `Meta Ads Daily Sheet Update` (07:30) — node `Lấy cấu hình Meta`
- `Meta Report Today Scheduled` (11:31 / 16:31 / 21:13) — nếu dùng Header Auth
- `Meta API Health Check` (07:05) — nếu dùng Header Auth

### Resolution
1. Vào n8n UI → Credentials → Header Auth account.
2. Sửa giá trị Authorization header: thêm `Bearer ` trước token.
3. Verify các workflow gọi Meta API hoạt động bình thường.

### Lessons
- Lỗi `(#200) Provide valid app ID` từ Meta API **không** phải lỗi App ID — có thể là lỗi Authorization header format.
- Header Auth credential format bắt buộc: `Bearer <token>` — không phải `<token>` thuần.
- Khi renew Meta token: luôn kiểm tra format `Bearer ` prefix trong Header Auth credential, không chỉ giá trị token.
- Token Meta thường hết hạn sau 60-90 ngày — cần lịch rotate định kỳ.

---

## Incident: Meta Access Token Expired & Auth Header Issue — 2026-06-22

### Context
Phát hiện trong quá trình test Service Account Migration
(workflow Meta Ads Daily Sheet Update 7:30 3 test), khi Execute
Previous Nodes từ node Append Row.
Xem tiến độ migration tại: CURRENT_STATUS.md

### Tóm tắt sự cố

**Triệu chứng:**
- Google Sheets Health Check báo META API HEALTH CHECK FAILED
- Daily Sheet Update trả dữ liệu sai (thực tế ~325k, trả về ~41k)
- Node `Lấy cấu hình Meta` lỗi 401 error_subcode 463
- Sau khi thay token mới tiếp tục lỗi 403 (#200) Provide valid app ID

### Root Cause #1
`META_ACCESS_TOKEN` đã hết hạn.
- Error: `Error validating access token / Session has expired / error_subcode 463`
- Nguyên nhân: đang dùng User Access Token, token đã hết hạn.

### Root Cause #2
Header Auth Credential cấu hình sai.
- Sai: `Authorization: EAASxxxx`
- Đúng: `Authorization: Bearer EAASxxxx`
- Sau khi thêm `Bearer`: toàn bộ workflow chạy thành công.

### Root Cause Sâu Hơn
Nguyên nhân gốc: đang dùng **User Access Token** trong production.

| Loại Token | Hạn sử dụng | Phù hợp |
|------------|------------|---------|
| Short-lived User Token | Vài giờ | ❌ KHÔNG dùng production |
| Long-lived User Token | ~60 ngày | ⚠️ Tạm chấp nhận |
| System User Token | Không expire | ✅ MỤC TIÊU |

Giải pháp dài hạn: Business Manager → System User → System User Token
(loại bỏ hoàn toàn việc đổi token thủ công)
Business Manager ID hiện tại: `187499196519080`

### Sai Lầm Trong Quá Trình Xử Lý
Đã lấy token mới bằng Graph API Explorer → Generate Access Token.
Token này chỉ là **Short-lived User Token**, chỉ sống vài giờ.
**Không được dùng loại token này cho production.**

### Checklist Xác Minh Token

1. Lấy token đang chạy thực tế trong n8n.
2. Mở `https://developers.facebook.com/tools/debug/accesstoken/`
3. Kiểm tra Loại / Hết hạn / Scopes:
   - Loại User + hết hạn vài giờ = Short-lived Token (**KHÔNG dùng production**)
   - Loại User + hết hạn ~60 ngày = Long-lived Token (tạm chấp nhận)
   - Loại System User = Token phù hợp production (**MỤC TIÊU**)

### Quy Trình Lấy Long-Lived Token (60 ngày)

1. Generate Short-lived Token với scopes: `ads_read`, `ads_management`, `business_management`
2. Lấy App ID + App Secret từ Meta App Dashboard
3. Exchange token:
```
GET https://graph.facebook.com/v25.0/oauth/access_token
  ?grant_type=fb_exchange_token
  &client_id=APP_ID
  &client_secret=APP_SECRET
  &fb_exchange_token=SHORT_LIVED_TOKEN
```
4. Dùng `access_token` trả về (`expires_in` ~5183944 giây = 60 ngày)

### Các Lỗi Meta Đã Gặp

| Lỗi | Ý nghĩa thực |
|-----|-------------|
| `401 error_subcode 463` | Token hết hạn |
| `403 (#200) Provide valid app ID` | Authorization header sai format (thiếu `Bearer `) |
| `ECONNRESET` | Lỗi mạng/TLS tạm thời — không phải lỗi token |

### Bài Học Vận Hành
Khi workflow báo lỗi: **không chờ cron**.
Quy trình chuẩn: Sửa → Execute Node → Execute Workflow → Verify dữ liệu → Sau đó mới chờ Scheduler.

### Trạng thái
RESOLVED.

---

## Incident: Backfill Lịch Sử KPI Mess_Comment — 2026-06-11

### Trạng thái
RESOLVED — COMPLETED.

### Mục tiêu
Sửa lại toàn bộ dữ liệu Mess_Comment từ 24/03 → 10/06 đã bị sai do fuzzy matching (xem Incident KPI Mess_Comment Sai).

### Phương pháp
Tạo workflow backfill riêng sử dụng exact match:

```javascript
a.action_type === 'onsite_conversion.messaging_conversation_started_7d'
```

### Phạm vi đã backfill
| Giai đoạn | Kết quả |
|-----------|---------|
| 24/03 → 31/03 | ✅ appendCount=0, updateCount>0 |
| 01/04 → 30/04 | ✅ appendCount=0, updateCount>0 |
| 01/05 → 31/05 | ✅ appendCount=0, updateCount>0 |
| 01/06 → 10/06 | ✅ appendCount=0, updateCount>0 |

### Spot Check Xác Minh
- 01/04: Trước backfill = 18 → Sau backfill = 6 → Ads Manager = 6 ✅
- 24/03: Sau backfill khớp Ads Manager ✅

### Kết luận
Toàn bộ dữ liệu Mess_Comment từ 24/03 → 10/06 đã được làm sạch và khớp Ads Manager.

