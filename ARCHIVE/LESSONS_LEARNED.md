# Lessons Learned

## Reusable Engineering Lessons
- Tránh dùng Vietnamese accented operational schema keys.
- Dùng snake_case ASCII schema naming cho canonical data contracts.
- Normalize data ngay sau ingestion.
- Dùng ISO dates internally.
- Parser layer chịu trách nhiệm normalize human input.
- Execute Step testing tốt để isolate routing/node failures.
- Queue/polling systems phải luôn validate dưới real runtime activation, không chỉ manual execution.
- Polling systems phải preserve update offsets.
- AI operational memory giá trị hơn raw chat history.
- Silent failures nguy hiểm hơn crashes.
- Forensic recovery cần reproducible playbooks.
- Downstream nodes không nên depend vào raw third-party response shape.
- Không tin report chỉ vì Telegram đã gửi message; phải verify computed payload.

## Additional Recovery Lessons: 2026-05-27

- Không bao giờ reset polling staticData trong runtime execution.
- Silent routing failures khó detect hơn crashes.
- Normalize external schemas ngay sau ingestion.
- Human-friendly input nên normalize một lần sang canonical ISO dates.
- Execute Step testing an toàn để isolate lỗi trong recovery, nhưng không đủ cho polling continuity.
- AI operational memory giúp tránh lặp lại forensic debugging cycles.
- Downstream systems không nên depend vào raw external array structures.

## End-to-End Recovery Lessons: 2026-05-27

- Business-zero KPIs không tự động đồng nghĩa pipeline failure.
- End-to-end verification là bắt buộc trước khi declare recovery success.
- Silent data corruption nguy hiểm hơn crashes.
- Canonical schema normalization giảm downstream complexity rất nhiều.
- Recovery baselines nên export ngay sau verified milestones.
- AI operational memory tăng tốc forensic debugging.
- Execute Step verification critical để isolate routing failures.

## Runtime Polling Lessons: 2026-05-27

- Execute Step testing alone không validate runtime polling continuity vì staticData persistence behave khác dưới real workflow activation.
- Backlog draining có thể cần temporary hardcoded offset trong supervised testing.
- Temporary hardcoded Telegram offsets phải remove hoặc restore về expressions trước production activation review.
- Một Telegram response sạch dưới runtime activation là evidence mạnh hơn manual node success alone.

## Stable Runtime Lessons: 2026-05-29

### Lesson 1
One Telegram queue should only have one active polling workflow consumer.

### Lesson 2
Runtime UI workflow state can drift away from exported verified artifacts.

### Lesson 3
Topology bugs in event-driven systems are often harder than code bugs.

### Lesson 4
Business-intent dedupe is different from transport replay dedupe.

### Lesson 5
Operational stability is more important than premature optimization.


## Data Ownership Lessons: 2026-05-29

- Google Sheet is the operational Source of Truth for this reporting system.
- Workflow must adapt to the Sheet schema; the Sheet should not be changed only to satisfy workflow convenience.
- Not every Sheet column belongs to Meta Ads data sync.
- Formula columns, manual sales columns, and customer-quality fields require preservation rules before any append/update workflow is considered safe.
- Broad row replacement can silently destroy business knowledge even when Meta API logic is technically correct.
- Future sync design must separate Meta-owned fields from business-owned fields.

## Scheduled Workflow Lessons: 2026-06-08

- `$env.VAR` trong n8n editor hiển thị `[ERROR: access to env vars denied]` — đây chỉ là UI preview, không phải runtime error. Biến vẫn hoạt động bình thường nếu đã có trong `.env` container.
- Scheduled (cron) workflow không có Telegram message context → `$json.chat_id` không tồn tại. Phải dùng `$env.TELEGRAM_CHAT_ID`.
- Thiếu env var (`TELEGRAM_CHAT_ID`) là nguyên nhân silent failure phổ biến trong scheduled workflow — kiểm tra `.env` trước khi debug node.
- Khi Nhóm B (khach_hop_le, sdt, khach_chot) = 0, hiển thị "Chưa cập nhật" thay vì số 0 → tránh gây hiểu lầm rằng không có khách. Đây là business display rule, không phải data rule.

## Google OAuth và Credential Reliability: 2026-06-10

- **Google OAuth có thể expire đột ngột** dù UI n8n hiện "Account connected". Đây là silent failure điển hình — không có crash, không có alert, chỉ biết khi đi kiểm tra thủ công.
- **Không tin credential status trên UI.** Nguồn sự thật duy nhất là execution logs — xem node output thực tế.
- **Shared credential = shared risk.** Nếu một credential expire, tất cả workflow dùng nó đều fail cùng lúc mà không có cảnh báo.
- **Error Alert là bắt buộc cho production system.** Không có alert → phát hiện lỗi chậm → data stale tích lũy → khó recover.
- **Google Service Account không expire**, không cần reconnect định kỳ → là lựa chọn đáng tin hơn OAuth cho production workflow chạy tự động.
- **Khi data trong Sheet sai đột ngột**, ưu tiên kiểm tra credential và execution logs trước khi nghi ngờ code/logic.

## Idempotent Sync và safeCount Lessons: 2026-06-09

- Verify idempotent trước khi activate scheduled sync: chạy 2 lần liên tiếp, lần 2 phải Append=0. Nếu lần 2 vẫn có Append → có bug tạo trùng.
- Node Code gọi `$('NodeName')` sẽ throw nếu node đó chưa execute trong execution context hiện tại (ví dụ: Summary node sau hai nhánh IF). Dùng `safeCount` helper với try/catch để tránh false failure:
  ```js
  function safeCount(nodeName) {
    try { return $items(nodeName).length; } catch(e) { return 0; }
  }
  ```
- Chỉ catch lỗi `hasn't been executed` — không dùng catch trống để tránh che lỗi thật.
- Khi workflow có hai nhánh song song (Append + Update) hội tụ vào Summary, không phải lúc nào cả hai nhánh cũng execute. Summary node phải defensive với cả hai khả năng.

## KPI action_type và Data Integrity: 2026-06-11

- **Không bao giờ dùng `includes()` để match action_type từ Meta API.** Meta trả về nhiều action_type khác nhau cùng chứa keyword tương tự (`message`, `conversation`...). Fuzzy match gom nhầm, inflate KPI im lặng — rất nguy hiểm vì trông có vẻ "có data".
- **Luôn dùng exact match `===`** và kiểm tra Ads Manager để xác định tên chính xác của action_type trước khi viết logic.
- **Silent data inflation khó phát hiện hơn silent zero.** KPI = 0 trông rõ ràng là sai; KPI = 18 khi thực tế = 1 trông như "dữ liệu bình thường".
- **Sau mỗi patch workflow, chạy COUNTIF(Key) để detect duplicate ngay** — đặc biệt khi workflow có nhánh append.
- **Backfill lịch sử chỉ được gọi là thành công khi appendCount=0 và updateCount>0** cho toàn bộ phạm vi — không có append = không tạo trùng.
- **Spot check nhiều ngày khác nhau** sau backfill, không chỉ ngày đầu tiên, để đảm bảo logic đúng nhất quán.

## Today Report và Tách Nguồn Dữ Liệu: 2026-06-11

- **Today Report phải lấy từ Meta API trực tiếp**, không qua Google Sheet — vì Sheet chỉ sync lúc 07:30, dữ liệu hôm nay chưa có trong Sheet tại 11:31/16:31/21:13.
- **Tách nguồn dữ liệu = giảm single point of failure.** Today Report không phụ thuộc Google OAuth, nên khi OAuth expire, Today Report vẫn chạy bình thường.
- **3 cron triggers trong 1 workflow là hợp lệ trong n8n** — không cần tách thành 3 workflow riêng.
- **Verify bằng Ads Manager trước khi declare done**, không chỉ verify bằng "workflow không lỗi". Silent logic bugs thường không crash workflow.
- **CPC cần bảo vệ divide-by-zero** (`totalClick > 0 ? ... : 0`) — trường hợp 0 click hoàn toàn có thể xảy ra trong production.
- **Google Sheet used range có thể gây hiểu nhầm là duplicate.** Khi append ghi vào dòng xa hơn dữ liệu thực, đây là behavior của Sheet (nhớ last used row), không phải duplicate — kiểm tra COUNTIF(Key) để phân biệt.

## n8n Debug Strategy — Authentication First, Full Workflow Third: 2026-06-22

### Rule
> "Authentication first. Permission second. Full workflow execution third. Node-level debugging only when necessary."

### Bối cảnh
Phát hiện trong quá trình migrate Google OAuth → Service Account cho `Meta Ads Daily Sheet Update`. Đã mất nhiều vòng execute riêng lẻ từng node (Đọc Sheet, Lấy cấu hình Meta, IF Update?, Update Row, Append Row...) trong khi workflow hoàn toàn có thể chạy end-to-end và dừng đúng tại node lỗi.

### Khi nào nên dừng Execute từng node

Sau khi tất cả điều kiện sau đã được xác nhận:
- ✅ Credential / Access Token hợp lệ
- ✅ Permission đúng (Service Account, OAuth scope, Header Auth format)
- ✅ Spreadsheet / Sheet TEST hoạt động và được share
- ✅ Node đã load được schema (không còn lỗi auth/permission khi mở node)

→ **Dừng execute riêng lẻ. Chạy toàn bộ workflow TEST.**

### Tại sao Full Workflow tốt hơn

n8n mặc định: node lỗi → workflow dừng → hiển thị node đỏ → hiển thị stack trace.

| Phương pháp | Phát hiện node lỗi | Kiểm tra luồng thực | Phát hiện lỗi mapping/field propagation | Tốc độ |
|-------------|-------------------|--------------------|-----------------------------------------|--------|
| Execute từng node | ✅ | ❌ | ❌ | Chậm |
| Full workflow TEST | ✅ | ✅ | ✅ | Nhanh |

Execute từng node tạo dữ liệu test không phản ánh luồng thực → dễ sinh kết luận sai (node pass riêng lẻ nhưng fail khi chạy chuỗi).

### Quy trình chuẩn sau khi xác thực Credential + Permission

1. Chạy toàn bộ workflow TEST.
2. Nếu lỗi: xác định node đỏ đầu tiên → sửa đúng node đó.
3. Chạy lại workflow TEST.
4. Chỉ Execute riêng node khi cần forensic/debug chuyên sâu (ví dụ: cần xem intermediate output của một node cụ thể).

### Sai lầm điển hình cần tránh

- Execute riêng Append Row trước khi biết Update Row đã pass full workflow chưa.
- Execute riêng HTTP Request để kiểm tra token trong khi có thể biết ngay bằng cách chạy workflow và xem node đỏ.
- Kết luận "node pass" từ Execute Step riêng lẻ, sau đó ngạc nhiên khi workflow fail toàn bộ.

## Service Account Migration — Credential Type Verification: 2026-06-22

### Rule
> KHÔNG BAO GIỜ kết luận Service Account migration thành công chỉ vì workflow chạy thành công.

### Root Cause của Sai Lầm
Đã đánh đồng hai việc không tương đương:
- ❌ Workflow chạy thành công = Service Account đang được dùng
- ✅ Chỉ mở bút chì trong node mới xác nhận được Credential Type thực tế

Node có thể execute thành công, dữ liệu ghi đúng — nhưng thực tế vẫn đang dùng OAuth credential cũ. Mất nhiều vòng debug vì kiểm tra hành vi workflow trước khi xác minh Credential Type.

### Verification Order — Bắt buộc

| Bước | Hành động | Ghi chú |
|------|-----------|---------|
| 1 | Mở node → mở bút chì Credential → xác nhận **Credential Type thực tế** | Phải làm TRƯỚC |
| 2 | Node Execute thành công | |
| 3 | Dữ liệu thực tế thay đổi trong Google Sheet | |

Chỉ khi cả 3 bước đều PASS mới được kết luận migration hoàn tất.

### PASS / KHÔNG PASS

| Credential Type thấy trong node | Kết luận |
|----------------------------------|---------|
| `Google Service Account API` | ✅ PASS — đang dùng Service Account |
| `Google Sheets account (OAuth)` | ❌ KHÔNG PASS — vẫn dùng OAuth cũ |

### Anti-Pattern

- **Sai:** "Update Row chạy thành công nên Service Account đã hoạt động."
- **Đúng:** "Update Row chạy thành công. Xác minh Credential Type = `Google Service Account API`. Sau đó mới kết luận migration PASS."

## Lessons Learned — Service Account Migration (2026-06-19 đến 2026-06-24)

### LL-001: Không kết luận Service Account đang chạy chỉ vì tên credential hiển thị

- **Symptom:** Node hiển thị tên "Google Sheets Service Account" → người dùng cho rằng migration hoàn tất.
- **Reality:** Tên hiển thị có thể không phản ánh đúng Credential Type bên trong. Node vẫn đang dùng OAuth2.
- **Rule:** Luôn mở bút chì (edit icon) cạnh Credential → xác nhận Credential Type thật là `Google Service Account API`. Không tin vào tên hiển thị.

### LL-002: "No output data returned" không phải lỗi credential

- **Symptom:** Update Row execute xong, không báo lỗi, OUTPUT hiện "No output data returned" → người dùng nghi ngờ Service Account không hoạt động.
- **Reality:** Node tìm được dòng theo Key thành công nhưng không có field nào được cấu hình để update. Mapping mode bị reset sau khi đổi credential.
- **Rule:** Kiểm tra "Values to Update" trước khi điều tra credential hoặc permission. `No output data` ≠ lỗi xác thực.

### LL-003: Đổi credential Google Sheets có thể reset cấu hình mapping

- **Symptom:** Update Row fail sau khi đổi credential sang Service Account. Lỗi: "Column to Match On parameter is required".
- **Reality:** n8n reset các field sau khi đổi credential: Column to match on → trống; Mapping mode → Map Each Column Manually; Values to Update / Values to Send → trống.
- **Rule:** Sau mỗi lần đổi credential Google Sheets, kiểm tra lại:
  - ☐ Column to match on = `Key`
  - ☐ Mapping mode đúng
  - ☐ Values to Update / Values to Send không trống
  - ☐ Sheet target đúng (production vs TEST)

### LL-004: Cách verify Update Row đúng chuẩn

- **Symptom:** Execute thành công nhưng không biết node có thực sự ghi dữ liệu không. Dữ liệu cũ = dữ liệu mới nên không thấy thay đổi.
- **Rule:** Sửa tay 1 giá trị trong Sheet (ví dụ `Chi_tieu` = 999999) → Execute workflow → Xác nhận giá trị quay về số gốc → Kiểm tra Version History Google Sheets để có bằng chứng rõ ràng.

### LL-005: Execute từng node PASS không đảm bảo full workflow PASS

- **Symptom:** Từng node Google Sheets PASS với Service Account. Full workflow execute sau đó phát sinh lỗi Meta token 401.
- **Reality:** Google Service Account hoạt động hoàn hảo nhưng Meta Access Token đã hết hạn ở node khác. Node-by-node test không phát hiện được lỗi cross-node.
- **Rule:** Sau khi verify từng node, luôn chạy full workflow end-to-end. Xác nhận toàn bộ chuỗi: Meta API → Google Sheets → không lỗi.

### LL-006: Một số workflow đang sử dụng 2 nguồn token Meta khác nhau

- **Reality:** Node "Lấy cấu hình Meta" dùng n8n credential (UI). Node "Lấy dữ liệu Meta" dùng `$env.META_ACCESS_TOKEN` (file `.env` Docker). Hai nguồn này độc lập nhau.
- **Rule:** Khi đổi Meta token, kiểm tra workflow đang dùng nguồn nào. Nếu dùng cả hai: (1) cập nhật n8n credential trong UI, (2) cập nhật `META_ACCESS_TOKEN` trong file `.env`, (3) restart Docker container sau khi sửa `.env`.
- **Warning:** Không giả định toàn bộ workflow dùng cùng một nguồn token. Luôn kiểm tra trực tiếp từng node hoặc code node.

### LL-007: Migration chỉ được xem là hoàn tất khi đủ 5 điều kiện

- **Symptom:** Node PASS → kết luận migration xong → bỏ qua các bước verify tiếp theo.
- **Reality:** Node PASS chỉ chứng minh node đó hoạt động, không chứng minh toàn bộ workflow, không chứng minh scheduler thật hoạt động.
- **Rule:** Migration chỉ được xem là COMPLETED khi đủ cả 5 điều kiện:
  - ☐ Node-level PASS
  - ☐ Execute Workflow PASS
  - ☐ Dữ liệu ghi đúng vào Sheet (có bằng chứng thực tế)
  - ☐ Chạy lại sau thời gian dài vẫn PASS
  - ☐ Scheduler thực tế chạy đúng lịch PASS

## Lessons Learned — Migration Verification Gap (2026-06-25)

### LL-008: Mọi migration phải bắt đầu bằng inventory toàn bộ điểm sử dụng

- **Ngày phát hiện:** 2026-06-25
- **Symptom:** Đã migrate node Append Row và Update Row sang Service Account. Kết luận migration hoàn tất. Sáng hôm sau scheduler fail vì node đọc Sheet vẫn dùng OAuth.
- **Root Cause:** Không có inventory trước khi bắt đầu migration. Không biết có bao nhiêu node đang dùng credential cần migrate.
- **Reality:** Nếu có inventory từ đầu, không thể bỏ sót bất kỳ node nào. Áp dụng cho mọi loại migration: credential, endpoint, schema...
- **Rule:** Trước khi bắt đầu bất kỳ migration nào, lập inventory đầy đủ. Checklist inventory:
  - ☐ HTTP Request node
  - ☐ Native node (Google Sheets, Gmail, Slack...)
  - ☐ Code node đọc `$env` hoặc `$credentials`
  - ☐ Health Check workflow
  - ☐ Monitoring workflow
  - ☐ Scheduled workflow
  - ☐ Alert message content

  Chỉ khi inventory đầy đủ mới bắt đầu migrate.

**Ví dụ inventory Google Sheets credential:**
```
Google Sheets credential
├── Workflow A: Meta Ads Daily Sheet Update
│   ├── Node: Đọc toàn bộ Sheet (HTTP Request)
│   ├── Node: Cập nhật vào Google Sheet (Update Row)
│   └── Node: Ghi mới vào Google Sheet (Append Row)
├── Workflow B: Yesterday Report
│   └── Node: Đọc Google Sheet (HTTP Request)
├── Workflow C: Health Check
│   └── Node: Đọc Sheet (HTTP Request)
└── Monitoring: Health Check message Telegram
```

### LL-009: Monitoring phải luôn đồng bộ với kiến trúc production

- **Ngày phát hiện:** 2026-06-25
- **Symptom:** Production đã migrate sang Service Account. Health Check vẫn kiểm tra OAuth credential. OAuth expire → Health Check báo FAILED. Người vận hành tưởng production hỏng, mất thời gian điều tra sai hướng.
- **Reality:** Health Check FAILED ≠ Production FAILED. Monitoring đang báo lỗi cho credential production không còn dùng. Đây là false alarm gây hiểu nhầm nghiêm trọng.
- **Rule:** Sau mỗi lần thay đổi kiến trúc production:
  - ☐ Cập nhật ngay tất cả Health Check liên quan
  - ☐ Cập nhật message alert cho đúng hành động cần làm
  - ☐ Không để monitoring báo lỗi cho thành phần không còn dùng trong production
  - ☐ Message FAIL phải nêu tiêu chí thành công rõ ràng
  - ☐ Không hardcode ngày tháng vào message

  Áp dụng cho mọi loại monitoring: Google Sheets, Meta API, Telegram, Docker...

### LL-010: Workflow TEST PASS không đồng nghĩa Production đã được migrate

- **Ngày phát hiện:** 2026-06-25
- **Symptom:** Workflow TEST "7:30 3 - SA TEST" đã PASS với Service Account. Kết luận migration hoàn tất. Workflow production "7:30 2" vẫn còn node dùng OAuth. Sáng hôm sau production fail.
- **Reality:** TEST workflow và Production workflow là 2 workflow khác nhau. PASS trên TEST không tự động migrate Production. Người vận hành có thể đổi credential ở một số node production nhưng bỏ sót node khác (đặc biệt HTTP Request vs native node).
- **Lifecycle chuẩn của một migration:**
  ```
  Inventory → Migration → Production Execute → Scheduler Verify → Documentation → Archive
  ```
  Không được bỏ qua bất kỳ bước nào.
- **Production Migration Checklist:**
  - ☐ Mở từng workflow Production (không phải TEST)
  - ☐ Rà soát tất cả node có dùng credential vừa migrate (theo inventory)
  - ☐ Mở bút chì xác nhận Credential Type thật của từng node
  - ☐ Đổi credential cho tất cả node còn thiếu
  - ☐ Execute manual toàn bộ workflow Production
  - ☐ Verify dữ liệu thực tế (không chỉ xem n8n không báo lỗi)
  - ☐ Cập nhật Health Check và Monitoring
  - ☐ Chờ ít nhất 1 lần scheduler thật chạy thành công
  - ☐ Chỉ khi đủ tất cả điều kiện trên mới đánh dấu COMPLETED

### LL-011: Mỗi migration phải có Definition of Done (DoD)

- **Ngày phát hiện:** 2026-06-25
- **Vấn đề:** Migration thường bị đánh dấu "xong" dựa trên cảm tính — thấy không lỗi là kết luận xong. Không có tiêu chí rõ ràng dẫn đến bỏ sót bước quan trọng và phát sinh incident sau đó.
- **Rule:** Trước khi bắt đầu bất kỳ migration nào, phải xác định rõ "xong" nghĩa là gì.

**DoD chuẩn cho dự án này (REQUIRED — bắt buộc):**
```
☐ Bước 1: Inventory
  Đã liệt kê đầy đủ toàn bộ nơi cần migrate
  (node, workflow, health check, monitoring, env variable...)
  Không bắt đầu migrate khi chưa có inventory.

☐ Bước 2: Migration
  Đã thực hiện migration cho tất cả item trong inventory.
  Không được bỏ sót bất kỳ item nào.

☐ Bước 3: Production Verify
  Đã execute manual toàn bộ workflow production liên quan.
  Đã xác nhận dữ liệu thực tế đúng (không chỉ "n8n không báo lỗi").

☐ Bước 4: Monitoring Update
  Đã cập nhật Health Check theo kiến trúc mới.
  Đã cập nhật message alert — hướng dẫn đúng hành động,
  không còn nhắc đến credential cũ.

☐ Bước 5: Scheduler Verify
  Đã chờ ít nhất 1 lần scheduler thật chạy thành công.
  Đã xác nhận toàn bộ chuỗi tự động hoạt động đúng lịch.

☐ Bước 6: Documentation
  Đã cập nhật CURRENT_STATUS.md.
  Đã cập nhật PROJECT_BRAIN.md.
  Đã ghi Lesson Learned nếu phát sinh bài học mới.
```

**OPTIONAL (làm sau khi đủ Required, tùy tình huống):**
- Archive workflow TEST (có thể giữ lại nếu cần regression test)
- Revoke credential cũ (chờ vài ngày quan sát production ổn định trước khi revoke)

**Tóm tắt:** Migration chỉ được đánh dấu COMPLETED khi đủ cả 6 bước Required.

---

## LL-013: Sau khi thay đổi schema Google Sheet, luôn verify lại toàn bộ chuỗi đọc dữ liệu

Ngày phát hiện: 2026-06-30

Symptom:
- Vừa thêm cột mới (Click_All) vào Google Sheet
- Node Xử lý dữ liệu báo "THIẾU CỘT BẮT BUỘC: Click_All"
- Sau khi execute lại, lỗi tự hết

Quan trọng: Nguyên nhân chính xác CHƯA được xác minh bằng bằng chứng runtime đầy đủ.
Không kết luận là do cache hay timing nếu chưa có bằng chứng cụ thể.

Rule:
Sau khi thay đổi schema Google Sheet (thêm/xóa/đổi tên cột), luôn verify theo thứ tự:
1. Execute riêng node đọc Sheet → xem output có cột mới chưa
2. Execute riêng node parser → xem header row parser nhận được là gì
3. Kiểm tra requiredHeaders trong code có khớp tên cột thật trong Sheet không
4. Chỉ kết luận PASS/FAIL dựa trên kết quả thực tế của từng bước, không đoán nguyên nhân

Cách debug hiệu quả: thêm tạm dòng `throw new Error(JSON.stringify({...}))` ngay sau bước nghi ngờ để lấy bằng chứng runtime, thay vì suy luận từ các lần execute trước đó.

---

## LL-014: Mojibake khi xử lý workflow JSON có chuỗi tiếng Việt

Ngày phát hiện: 2026-07-03

Symptom:
- Sau quy trình export → chỉnh sửa → import workflow JSON, xuất hiện mojibake
- "Tính Ngày Hôm Qua" → "TÃ­nh NgÃ y HÃ´m Qua"
- "BÁO_CÁO_QUẢNG_CÁO" → "BÃO_CÃO_QUẢNG_CÁO"
- Node Đọc Google Sheet báo lỗi: "Unable to parse range: BÃO_CÃO_QUẢNG_CÁO"

Root Cause: CHƯA XÁC ĐỊNH
Fact hiện có: mojibake xuất hiện sau quy trình export → chỉnh sửa → import.
Không kết luận nguyên nhân cụ thể khi chưa có bằng chứng.

Rule:
Sau khi export/import workflow JSON qua bất kỳ công cụ nào:
□ Kiểm tra tất cả node Display Name có tiếng Việt
□ Kiểm tra tất cả URL parameter có tiếng Việt (đặc biệt Sheet name)
□ Kiểm tra tất cả expressions có chuỗi tiếng Việt
□ Execute thử node có tiếng Việt trong URL trước khi activate Production

---

## LL-015: Layer-by-Layer Runtime Verification
Ngày phát hiện: 2026-07-03

Bài học:
Luôn verify runtime theo từng layer độc lập trước khi tích hợp sang layer tiếp theo. Nguyên tắc này áp dụng cho mọi thành phần của hệ thống (Parser, Engine, Report, Dashboard, API...), giúp cô lập lỗi nhanh, giảm rủi ro và tránh ảnh hưởng Production.
Rule:
Luôn verify theo thứ tự layer trước khi tích hợp:
Data Source (Google Sheet)
↓ PASS
Normalize Layer
↓ PASS
Engine Layer (Date Range Engine)
↓ PASS
Presentation Layer (Render Telegram)
↓ PASS
Production

Không tích hợp sang layer tiếp theo khi layer hiện tại chưa PASS.
Áp dụng cho mọi component mới: Engine, Parser, Report, Dashboard...
