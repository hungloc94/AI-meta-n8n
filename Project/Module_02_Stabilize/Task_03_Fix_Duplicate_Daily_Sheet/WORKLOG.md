# WORKLOG — Task 03: Fix Duplicate Row (Daily Sheet Update)

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
