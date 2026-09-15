# WORKLOG — Task 03: Fix Duplicate Row (Daily Sheet Update)

---

## Session 2026-09-13 (Claude Code / CLI) — ĐÓNG TASK

### Đã làm
- **MỤC 1** — Yesterday Report (`9l8RHgYuRZYz0TVr`) node "Send Report Telegram": `timeout=30000ms`, **giữ `retryOnFail=true`** (chống tin trùng mà không mất báo cáo). Backup: `Meta_Report_Yesterday.bak_2026-09-13T10-15-07.json`.
- **MỤC 2** — commit `5886be1` + push `main` (canonical upsert + Task_03 docs + AI_OS skills). Đã quét không có secret.
- **MỤC 3** — `LESSONS_LEARNED.md` (3 bài học) + index `CASE_INDEX.md`.
- **MỤC 4** — dọn backup tạm: giữ PROD_LIVE gốc + Yesterday backup, xoá 13 file.
- **PHASE 5b** — tool `TOOL — Highlight Duplicate Rows` (`QWowp4oT9PA1iJTO`) + doc `AI_OS/skills/highlight_duplicate_rows/`.

### Dọn workflow n8n (anh Lộc thao tác UI = archive) — HOÀN TẤT 2026-09-13
- ✅ Đã archive TẤT CẢ 7 workflow rác: `T0x1qzedADrmj1LA` (clone lỗi gốc), `Task03TestDedup1`, `5MUMksN5LrdulcLi`, `1eFjFdCjXZRCRV0x`, `ORFWlIrApPASnALp`, `baIwZg6ZSu5sWfDS`, `WyLKW6wrPEagQvNa`.
- ✅ GIỮ (không archive): `QWowp4oT9PA1iJTO` = TOOL — Highlight Duplicate Rows.
- ✅ Xác nhận: **chỉ còn duy nhất `rz3Wya5lFay7ShVL` (upsert)** active ghi Sheet production.

### Trạng thái
- **Task_03: CLOSED (kỹ thuật).** Fix verify ĐẠT trên production (0 dup, upsert idempotent, Mess_Comment phương án 3, báo cáo hết nhân đôi), nguồn đẻ trùng đã tắt, workflow rác đã dọn. Còn lại: **quan sát chu kỳ tự động 2026-09-14** để đóng hoàn toàn.

---

## Sự cố báo cáo nhân đôi — điều tra 2026-09-14 (RIÊNG khỏi lỗi Sheet duplicate)
- Hiện tượng: sáng 09-14 vẫn nhận **2 báo cáo hôm qua** (~08:15).
- Điều tra DB: Yesterday Report chạy **1 lần**, log **1 message_id** — nhưng node **"Send Report Telegram" mất 189s** (giải mã runData eid=455: đọc Sheet 1.9s, send 189s). Today Report send chỉ 3.5s (cùng bot).
- **Root cause:** send 08:13 chập/chậm (~61s/lần) → `retryOnFail` retry 3 lần (≈189s); các lần "false-fail" VẪN gửi tới Telegram = nhận 2-3 tin; n8n chỉ lưu lần cuối thành công (nên log tưởng 1).
- **Phát hiện quan trọng:** fix timeout=30000 (session 09-13) **chưa từng hiệu lực** vì n8n **chưa restart** → scheduler vẫn chạy bản cũ (eid=455 không có timeout).
- **Fix (đúng Lesson 2 — giữ retry):** set node send **timeout=120000ms** để lần gửi đầu (chậm nhưng thành công) kịp hoàn tất trong 1 attempt → không retry → 1 tin. Backup: `Meta_Report_Yesterday.bak_2026-09-13T10-15-07.json`.
- **⚠️ CẦN restart n8n** mới hiệu lực. Nếu (sau restart) vẫn trùng → send fail do lỗi mạng (không phải timeout) → chuyển Option B (giảm maxTries=1) hoặc điều tra kết nối Telegram từ server lúc 01:13 UTC.

### Trạng thái fix timeout (cập nhật 2026-09-14)
- ✅ DB đã set: `9l8RHgYuRZYz0TVr` node "Send Report Telegram" → `timeout=120000`, `retryOnFail=true`, active=true.
- 🔴 **RESTART VẪN CHƯA THỰC HIỆN** — `docker ps` cho thấy n8n **"Up 2 days"** (chưa restart kể từ 09-11). ⇒ fix **CHƯA có hiệu lực**; run 08:13 hôm nay (09-14) vẫn dùng bản cũ nên vẫn trùng.
- 👉 Việc cần làm: `docker restart n8n` → verify sáng 09-15: node send < 120s (1 attempt), nhận đúng 1 tin báo cáo.

## Theo dõi 2026-09-15 — KẾT QUẢ (chu kỳ đầu SAU restart, timeout 120s đã live)

| Mốc | Kỳ vọng | Kết quả thực tế | |
|-----|---------|-----------------|--|
| 07:30 Daily Sheet Update (eid=461) | success, appendCount = ad ngày mới | success, `appendCount=23` (ad ngày 09-15), `updateCount=135` | ✅ |
| COUNTIF(Key) sau 07:30 | 0 trùng | **0 Key trùng** | ✅ |
| 08:13 Yesterday Report (eid=462) | đúng 1 tin, không nhân đôi | **1 message_id**, duration **57.6s** (giảm từ 189-264s → 1 attempt, không retry) | ✅ |
| Anh Lộc xác nhận | 1 báo cáo | "sáng nay chỉ có 1 báo cáo duy nhất" | ✅ |

→ **CẢ 2 LỖI ĐÃ ĐÓNG:**
1. **Sheet duplicate** (2 workflow ghi) — fixed: upsert + tắt clone + dọn dup → 0 trùng.
2. **Báo cáo nhân đôi** (retry gửi Telegram chậm) — fixed: timeout=120s → send hoàn tất 1 lần (57.6s) → 1 tin.

**Ghi chú:** send vẫn chậm ~57s (Today Report chỉ 3.5s) — kết nối Telegram từ server lúc 01:13 UTC chậm bất thường. KHÔNG còn gây trùng (đã trong ngưỡng timeout), nhưng nếu sau này chậm >120s sẽ retry lại → cân nhắc điều tra kết nối/route Telegram (infra) như cải tiến tương lai.

## TASK_03 — PASS TẠM THỜI / MONITORING MODE (2026-09-15)
Cả 2 lỗi hết biểu hiện qua chu kỳ tự động thật 09-15. Đánh dấu **PASS tạm thời**, KHÔNG đóng cứng — giữ hồ sơ này để tái nghiên cứu nếu tái phát.

### [MONITORING] Telegram send của Yesterday Report còn chậm — theo dõi, chưa cần fix thêm
> Ghi cho phiên AI sau: đây là điểm còn treo, đã chấp nhận tạm thời.
- **Bối cảnh:** node "Send Report Telegram" (workflow `9l8RHgYuRZYz0TVr`) mất ~57s ngày 09-15 (baseline bất thường; Today Report cùng bot chỉ ~3.5s). Nguyên nhân sâu: kết nối tới api.telegram.org từ server chậm lúc ~01:13 UTC (nghi throttle/route vùng).
- **Vì sao hiện OK:** 57s < `timeout=120000ms` → send hoàn tất trong 1 attempt → `retryOnFail` KHÔNG kích hoạt → gửi đúng 1 tin. (Trước đây send fail ~61s/lần → retry 3 lần ~189-264s → nhận 2-3 tin.)
- **NGƯỠNG KÍCH HOẠT XỬ LÝ (trigger để mở lại việc):** nếu (a) lại nhận **>1 báo cáo/sáng**, HOẶC (b) duration node send / execution `9l8RHgYuRZYz0TVr` **> ~10 phút** (send chậm vượt xa 120s → sẽ bị retry lại).
- **KHI TRIGGER → HÀNH ĐỘNG (theo thứ tự):**
  1. Đọc lại hồ sơ này + `CASES/CASE-051` + `LESSONS_LEARNED.md`.
  2. Kiểm chứng: `docker cp n8n:/home/node/.n8n/database.sqlite` → query `execution_entity`/`execution_data` của `9l8RHgYuRZYz0TVr`; so duration node send với baseline ~57s. LƯU Ý: các lần retry-fail KHÔNG được lưu trong runData (n8n chỉ giữ lần cuối thành công) → dùng **duration** làm chỉ dấu retry, đừng chỉ đếm message_id.
  3. Cân nhắc **Option B**: giảm `maxTries=1` (tắt retry riêng node send) → chắc chắn không trùng, đổi lại hiếm khi mất 1 báo cáo. (Bằng chứng cho thấy send vẫn tự thành công nên retry lợi ít hại nhiều — cân lại Lesson 2 khi đó.)
  4. Hoặc **root fix (infra):** điều tra/route lại kết nối Telegram (proxy/DNS) từ server để send về ~vài giây.
- **Trạng thái deploy hiện tại:** `9l8RHgYuRZYz0TVr` node send `timeout=120000`, `retryOnFail=true`, active=true (đã live sau restart 09-14).

### [DEFERRED] Bài học 3 (auto-cảnh báo data trùng) — CHƯA cần
- Anh Lộc đã tự làm **conditional formatting** trên Sheet: tự bôi màu các dòng trùng Key (cột S) khi trùng nhau. → đã có cơ chế phát hiện trùng ở tầng Sheet, đủ với nhu cầu hiện tại. **Không cần** workflow auto-alert lúc này. (Nếu sau muốn cảnh báo Telegram chủ động thì mới làm.)

---

## Session 2026-09-12 (Claude Code / CLI) — PHASE 4 đóng + PHASE 5

### Điều tra "2 báo cáo hôm qua mỗi sáng"
- Tra execution log + DB n8n: Yesterday Report (`9l8RHgYuRZYz0TVr`) chạy **đúng 1 lần/ngày, gửi đúng 1 tin** (1 message_id) — **không phải 2 workflow gửi 2 tin**.
- Root cause: **1 tin nhưng nội dung nhân đôi** vì report đọc Sheet có dòng trùng. Bằng chứng `filteredCount`: 09-07/08/09 = **32/32/34** (gấp đôi ~16); sau khi tắt clone (09-09) → 09-10/11/12 = **22/22/22** (bình thường). ⇒ Đã hết nhân đôi.
- Ghi chú: node gửi Telegram của report bật `retryOnFail` (rủi ro gửi trùng khi timeout) — đề xuất tắt (chưa làm).

### 🔴 Sửa lỗi triển khai của chính mình
- Phát hiện: production đang chạy bản upsert CŨ (Mess_Comment ghi đè MỌI ngày) — **import nhầm candidate trước khi anh Lộc chọn phương án 3**.
- Đã re-import bản đúng: **phương án 3** (Mess_Comment chỉ ghi hôm nay/hôm qua) + fix phụ **`slice(0,2)→slice(-2)`** (manual lấy 2 ngày mới nhất). Set active=true. **CẦN restart n8n để scheduler nạp code mới.**
- Cập nhật **file canonical** `Project/WORKFLOWS/Meta_Ads_Daily_Sheet_Update.json` = bản LIVE cuối (9 node, upsert, phương án 3, slice(-2)).

### Trạng thái dedup Sheet
- Anh Lộc đã xoá tay tới chỉ còn 1 dup (08-11), rồi xoá nốt → **Sheet dự kiến 0 trùng** (chờ verify sau restart).

### Còn lại (PHASE 5)
- [ ] Anh restart n8n → verify: appendCount=0, COUNTIF=1, Mess_Comment phương án 3 đúng.
- [ ] Xoá hẳn trong UI: clone `T0x1qzedADrmj1LA` + workflow test (`Task03TestDedup1`, `baIwZg6ZSu5sWfDS`, `5MUMksN5LrdulcLi`, `QWowp4oT9PA1iJTO`, `1eFjFdCjXZRCRV0x`, `WyLKW6wrPEagQvNa`, `ORFWlIrApPASnALp`, `Task03 COLOR final`) — CLI không có delete:workflow.
- [ ] Commit canonical + docs (chờ anh duyệt).
- [ ] (Tuỳ chọn) tắt retryOnFail node gửi Telegram của Yesterday Report.
- [ ] Xoá backup tạm sau khi verify OK.

---

## Session 2026-09-09 (Claude Code / CLI) — PHASE 4 production (đang dở)

### 🔑 PHÁT HIỆN LỚN — root cause thật (2026-09-09)
Sau khi apply upsert mà vẫn thấy trùng ngày mới (anh Lộc báo "thêm loạt hàng 8/9"), quét TOÀN BỘ workflow n8n:
- **`T0x1qzedADrmj1LA` "TEST Daily Sheet Update Task09"** — clone TEST từ task09 (11/08), **để active=TRUE**, lịch **07:30**, ghi thẳng Sheet production bằng **node append cũ**.
- Bản fix `rz3Wya5lFay7ShVL` (upsert) khi đó **active=false**.
→ **2 workflow cùng append 07:30 = nguồn đẻ trùng hàng ngày** (root cause thật, không chỉ là "manual 22:00 + auto 07:30").
- **Xử lý:** `update:workflow` tắt clone + bật upsert → **restart container n8n** (đổi active chỉ hiệu lực sau restart) → verify: chỉ còn 1 workflow production active = upsert. ✅
- Đã cập nhật `CASES/CASE-051`.

### Trạng thái hiện tại
- **PHASE 1–3: DONE trên TEST** (upsert idempotent + dọn dup PASS trên `TEST_DEUP`).
- **PHASE 4 (production): ĐANG DỞ** — đã apply bản upsert + chạy thử + phân tích dup, anh Lộc đã **tự xoá dup tay**, NHƯNG **chưa verify lại sau khi xoá** và **workflow đang `active=false` (chưa bật lại)**.

### Đã làm (production)
| Bước | Kết quả |
|------|---------|
| Backup LIVE prod workflow | ✅ `BACKUP/task-fix-duplicate/Meta_Ads_Daily_Sheet_Update.PROD_LIVE.bak_2026-09-08T11-08-15.json` (15 node, versionId khớp canonical — không drift) |
| Import bản upsert đè `rz3Wya5lFay7ShVL` | ✅ 9 node, trỏ tab thật `BÁO_CÁO_QUẢNG_CÁO`, giữ **`active=false`** |
| Chạy upsert 1 lần trên production | ✅ 8/8 node success, 0 error; Summary **appendCount=0, updateCount=133** → không sinh dòng mới |
| Phân tích dup production (từ `prod_run1.json`) | ✅ 1675 dòng, **94 Key trùng**: 90 auto + 4 duyệt tay. Báo cáo: `BACKUP/task-fix-duplicate/PROD_DUPLICATE_REPORT_2026-09-08.txt`, `PROD_manual_cases_2026-09-08.txt` |
| Bắt & sửa lỗi so-sánh timestamp (chuỗi `DD/MM/YYYY`) trong script phân tích | ✅ đổi sang parse datetime → kết quả đúng thứ tự thời gian |
| Phát hiện xung đột `Khach_sai_tep` (0 vs số thật) ở CA#1/#2/#4/DIV | ✅ cảnh báo, KHÔNG tự xoá |
| Anh Lộc tự xoá dup trên Sheet production | ✅ (anh xác nhận "tự xoá rồi") — **chưa verify lại bằng script** |

### Kết quả
- **Cơ chế upsert PASS trên production run thử**: không tạo dòng trùng mới (appendCount=0), không đụng Nhóm B.
- **Dedup**: anh Lộc đã xoá tay theo danh sách; số dòng/COUNTIF sau xoá **chưa được đo lại**.
- **Chưa đóng**: verification sau xoá + reactivate + monitor.

### Signal — CẦN LÀM TIẾP (ưu tiên)
OPEN_TASKS:
- [ ] **VERIFY sau xoá** (chạy `docker exec n8n n8n execute --id=rz3Wya5lFay7ShVL > /tmp/prod_verify.json` — anh chạy bằng `!` vì AI bị chặn execute production) → kiểm: COUNTIF>1 = **0**, appendCount=0.
- [ ] **Xác nhận 4 ca xung đột `Khach_sai_tep`** (CA#1 `..767900202`/06-09, CA#2 `..206846150202`/06-09, CA#4 `..124262390202`/09-08, DIV `..767860202`/06-09) — anh đã điền đúng số thật trước khi xoá chưa? Nếu chưa, giá trị có thể đã mất.
- [ ] **BẬT LẠI workflow** (`active=true`) — hiện đang TẮT → 07:30 sẽ không tự chạy nếu quên bật.
- [ ] Monitor **2 chu kỳ 07:30** liên tiếp → appendCount=0, COUNTIF=1.
- [ ] PHASE 5: `slice(0,2)`→`slice(-2)`, kiểm timezone container, **commit bản canonical PASS** vào `Project/WORKFLOWS/`, dọn workflow test trong n8n (`baIwZg6ZSu5sWfDS`, `5MUMksN5LrdulcLi`, `Task03TestDedup1`), xoá backup tạm, update Module docs.

---

## Session 2026-09-08 (Claude Code / CLI)

### Tóm tắt
Điền PLAN sang PHASE 0–5 (Risk/Rollback/DoD), tạo `CURRENT_STATUS.md`, cập nhật Module STATUS/TASK_INDEX.
Sau đó **build bản fix upsert PHASE 1** trong scratchpad (chưa import production).

### Đã làm
| Bước | Kết quả |
|------|---------|
| Backup PLAN.md cũ | ✅ `BACKUP/task-fix-duplicate/PLAN.md.bak_2026-09-08T09-05-39` |
| Viết PLAN PHASE 0–5 + Risk/Rollback/DoD | ✅ |
| Tạo `CURRENT_STATUS.md` (Task_03) | ✅ |
| Cập nhật Module `STATUS.md` + `TASK_INDEX.md` | ✅ |
| Build bản fix upsert (9 node) | ✅ `BACKUP/task-fix-duplicate/Meta_Ads_Daily_Sheet_Update.FIX_upsert_candidate.json` |
| Validate: node count, connection, matchingColumns, không rò Nhóm B | ✅ PASS |

### Phát hiện cần CONFIRM
- **Mess_Comment**: doc ghi Nhóm B (protected) nhưng prod đang GHI (=totalMess Meta). Build hiện giữ nguyên hành vi prod → chờ anh Lộc chốt: giữ-ghi hay bảo vệ.
- **Summary** phải viết lại (2 node IF bị xóa) — đã derive append/update từ snapshot.

### Cập nhật cuối phiên (thực thi)
- CHỐT: bỏ PHASE 0; `Mess_Comment` = **phương án 3** (ghi khi date_start là hôm nay/hôm qua, đóng băng ngày cũ hơn).
- Import workflow TEST `baIwZg6ZSu5sWfDS` (active=false, trỏ `TEST_DEUP` gid 1579187793) — KHÔNG đụng production `rz3Wya5lFay7ShVL`.
- **PHASE 2 PASS:** T2/T3 `appendCount=0, updateCount=133`; Nhóm B thay đổi=0; Mess_Comment ghi đúng 09-07/09-08, đóng băng 09-01..09-06.
- **PHASE 3 analysis:** `TEST_DEUP` có **78 Key trùng** (copy từ production). Danh sách giữ/xóa: `BACKUP/task-fix-duplicate/TEST_DEUP_dedup_plan_2026-09-08.json` + `.txt`. 74 key auto-safe; **4 key (2026-09-06) có Nhóm B ở CẢ 2 dòng → cần anh Lộc quyết tay**.

### Signal
OPEN_TASKS:
- [ ] CONFIRM danh sách xóa 74 key auto-safe + quyết 4 key risky (PHASE 3) + chọn cơ chế xóa (n8n deleteRows / tay)
- [ ] PHASE 4: apply lên production Sheet + import workflow production, monitor 2 cycle 07:30
- [ ] PHASE 5: slice(-2), timezone, commit canonical, dọn backup, update Module docs

---

## Session 2026-09-07 (Claude Code / CLI) ⚠️

### Người thực hiện
- AI: Claude Code
- Human: anh Lộc

### Tóm tắt
Audit lỗi duplicate row của Daily Sheet Update. Ban đầu nghi append-retry; sau khi anh Lộc
cung cấp evidence cột T (22:00 chạy tay + 07:30 auto) → xác định lại root cause là **2 execution
cùng append 1 Key**. Chốt hướng fix **upsert (appendOrUpdate by Key)**. Backup 2 lớp.

### Đã làm
| Bước | Kết quả |
|------|---------|
| Đọc INIT/AI_OS + PROJECT_BRAIN + PATTERN-003 | ✅ |
| Audit workflow `rz3Wya5lFay7ShVL` (15 node) | ✅ Key=`{date_start}_{ad_id}` cột S |
| So sánh key/dedup/retry qua các bản backup + git | ✅ Check Dedup + retry append thêm ở commit c9aff03 (2026-08-11) |
| Bác bỏ giả thuyết retry nhờ evidence cột T | ✅ 22:00 (manual) + 07:30 (auto) = 2 run |
| Backup repo JSON + export LIVE từ n8n | ✅ `BACKUP/task-fix-duplicate/`, md5 khớp |
| Tạo Task_03 + README + PLAN + STATUS | ✅ |

### Signal
OPEN_TASKS:
- [ ] Dựng bản fix upsert ở scratchpad (Bước 2)
      → Phát hiện khi: chốt hướng với anh Lộc
- [ ] Xin anh Lộc tạo/duyệt tab TEST trên Sheet để test không đụng production (Bước 3)
      → Lý do: test idempotency cần ghi thử, không được ghi vào tab thật
- [ ] Rà `slice(0,2)` manual mode lấy 2 ngày cũ nhất (Bước 6) — đề xuất `slice(-2)`
      → Phát hiện khi: đọc node "Lấy dữ liệu Meta"

STALE_DOCS:
- [ ] `Project/Module_02_Stabilize/TASK_INDEX.md` → chưa có Task 03
- [ ] `Project/Module_02_Stabilize/STATUS.md` → chưa phản ánh Task 03 đang chạy

PROPOSAL:
- [ ] Thêm dòng Task 03 vào TASK_INDEX.md + Module STATUS
      → Lý do: task mới đã được anh Lộc duyệt triển khai
      → Tạo lúc: kết thúc phiên audit
