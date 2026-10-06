# WORKLOG — Task 04: Fix Missing Ads Data (Pagination)

---

## Session 2026-10-06 (Claude Code / CLI) — ĐIỀU TRA + PLAN

### Đã làm
- **PHASE 1** — Phân tích `Project/WORKFLOWS/Meta_Ads_Daily_Sheet_Update.json` (canonical, `rz3Wya5lFay7ShVL`).
  Kiểm 5 điểm nghi ngờ: chỉ **1 FAIL = pagination** ở node `Lấy dữ liệu Meta`; 4 điểm còn lại PASS
  (không filter node, key không collision, date range đúng, `level=ad` đúng).
- **PHASE 2** — Đọc 5 execution gần nhất từ SQLite (`file:...database.sqlite?mode=ro`, systemd-run 2G).
  3 run gần nhất (`589`, `582`, `575`) status = `error` → anh Lộc xác nhận **do mạng home server giật, KHÔNG phải lỗi workflow** → không thuộc scope Task_04.
- **PHASE 3** — Tạo `Task_04_Fix_Missing_Ads_Pagination/`:
  - `PLAN.md` — root cause, fix đề xuất (thêm `limit` + pagination loop + `MAX_PAGES=10`), risk, test plan, DoD.
  - `CURRENT_STATUS.md` — phase gate + facts + phần "CHƯA xác minh được".
  - `README.md`.
  - Incident log: `AI_OS/incidents/2026-10-02_missing-ads-data.md` (mới).
  - Index: `TASK_INDEX.md` (+Task 04), `AI_OS/SKILLS.md` (+ mục Incident Log).
  - Bài học: `Task_03.../LESSONS_LEARNED.md` + **Bài học 4 — Luôn kiểm tra pagination khi gọi API bên ngoài**.

### Chưa làm (chờ CONFIRM)
- Chưa sửa bất kỳ node/workflow nào. Chưa backup (chỉ backup ngay trước khi sửa — PHASE 4).

### Giới hạn của session
- **Không có `META_ACCESS_TOKEN`** trong env; đọc credential từ n8n bị chặn (Credential Materialization).
  → **Chưa đếm được** số ad thật từ Meta API, chưa có diff `ad_id` Meta vs Sheet.
  Root cause suy ra từ **bằng chứng code**, không từ diff dữ liệu. Bước verify định lượng nằm trong Test plan.

### Trạng thái
- **Task_04: IN_PROGRESS** — điều tra + verify TEST DONE; fix chưa lên production (chờ CONFIRM).

---

## Session 2026-10-06 (tiếp) — PHASE 4: IMPLEMENT + VERIFY (anh Lộc CONFIRM)

### Đã làm
- **Bước 1 — Backup (2 lớp)** trong `BACKUP/task-fix-pagination/`:
  - `Meta_Ads_Daily_Sheet_Update.REPO.bak_2026-10-06T23-19-13.json`
  - `Meta_Ads_Daily_Sheet_Update.PROD_LIVE.bak_2026-10-06T23-19-33.json`
  - Đối chiếu repo vs LIVE: 9 node, params **giống hệt**, versionId `af3bc19e` → không có drift.
- **Bước 2 — Sửa node `Lấy dữ liệu Meta`**: thêm `&limit=1000` + pagination loop (`MAX_PAGES=10`,
  đọc `paging.cursors.after`, gộp `data[]`, log số page). Giữ shape `{json:{data:[]}}`.
  Code nguồn: `BACKUP/task-fix-pagination/new_insights_node.js`.
- **Bước 3 — Test trên `TEST_DEUP`**: workflow `TEST Task04 — Pagination Fix` = id `nILRkiamvwXexV0o`, `active=false`.
  - Run 1: `appendCount=249, updateCount=0`
  - Run 2: `appendCount=0, updateCount=249` → idempotent
  - Diff Meta API vs Sheet (8 ngày): 249 vs 249, **lệch = 0**
  - Nhóm B: 9/9 cột có, 1829/1829 dòng có dữ liệu; 0 Key trùng
- **Bằng chứng định lượng** (`measure_pagination.js`, chạy trong container n8n):
  code cũ = **25 ad/ngày** (chốt cứng), code mới = 30–33; **7 ngày thiếu 45 ad**.

### Git
- ✅ Commit `26fdbe3` (6 file: Task_04/ + AI_OS/incidents/ + AI_OS/SKILLS.md).
- 🔴 **Push CHƯA thực hiện** — bị auto-mode classifier chặn ("Create Public Surface"). Cần anh Lộc push tay
  hoặc cấp quyền: `git push origin main` (branch hiện `ahead 1`).
- ⚠️ Canonical JSON KHÔNG nằm trong commit (BACKUP/ bị `.gitignore`).

### Dọn dẹp
- ✅ Xoá workflow đọc tạm `x7hjxy6WQh3u0H41` (ZZ Task04 READ TEST_DEUP).
- ⏳ Workflow TEST `nILRkiamvwXexV0o` **giữ lại** cho tới khi production verify xong (PHASE 5).

### Trạng thái
- **Task_04: TEST_DEUP PASS** — dừng tại gate trước production, chờ anh Lộc CONFIRM.

---

## Session 2026-10-06 (tiếp) — PHASE 5: LÊN PRODUCTION (anh Lộc CONFIRM "Lên production")

### Blocker kỹ thuật — 2 lệnh tự động bị chặn
- `python3` ghép code fix vào bản PROD_LIVE backup → chặn ("Credential Materialization")
- `cp TEST_pagination_fix.json → canonical` → chặn ("Out-of-Place Publication")
- Cả 2 lần KHÔNG tìm cách lách qua tool/script khác — báo anh Lộc, chờ quyết định.
- Phát hiện sai sót trong plan gốc: "copy bản TEST vào canonical" sẽ làm production ghi
  nhầm vào tab `TEST_DEUP` (3 điểm khác: `name`, URL đọc, gid ghi). Đã báo và sửa: base phải là
  **PROD_LIVE backup**, chỉ thay `jsCode`.
- Anh Lộc chọn "Cách A" — tự chạy lệnh python ghép JSON (base=PROD_LIVE + code fix, giữ `active=false`).

### Bước 1 — Verify canonical sau khi anh Lộc chạy lệnh
- Đọc lại `jsCode` đầy đủ: xác nhận `limit=1000`, `paging.cursors.after`, `MAX_PAGES=10`,
  shape `{json:{data:allData,...}}` — đúng bản đã PASS trên TEST_DEUP.
- Tab đọc/ghi vẫn đúng production: `BÁO_CÁO_QUẢNG_CÁO` / gid `1598114539`.
- `active: false` trong file.

### Bước 2 — Import lên production
- `PUT /api/v1/workflows/rz3Wya5lFay7ShVL` với payload từ canonical.
- **Phát hiện quan trọng:** payload PUT không gửi field `active` → n8n giữ nguyên trạng thái
  cũ của workflow, mà workflow này **active=true từ trước** (chạy 07:30 hàng ngày, không phải do
  task này tắt). Kết quả: `active: True` sau PUT — Bước 2/3 trộn vào nhau ngoài dự kiến.
- Giờ PUT: 2026-10-06 23:07 UTC. Lần chạy scheduled kế tiếp: 00:30 UTC (~82 phút sau) → đã
  DỪNG và báo anh Lộc ngay, không tự ý để code chưa-manual-verify chạy tự động.
- Anh Lộc CONFIRM (1) — chạy verify ngay.

### Bước 3 — Verify production thật
```
docker exec n8n n8n execute --id=rz3Wya5lFay7ShVL > /tmp/prod_pagination_verify.json 2>&1; echo "EXIT=$?"
EXIT=0
```
- File output lẫn log CLI (permissions warning, deprecation notice) ở 8 dòng đầu — cắt bỏ
  (`tail -n +9`) trước khi parse JSON, không phải lỗi dữ liệu.
- `resultData.error`: None. Node `Ghi vào Sheet`: `executionStatus=success`.
- `appendCount=141, updateCount=101, total=242`.
- 8/8 ngày: `pages=1, truncated=false` — không bị cắt bởi `MAX_PAGES`.
- `Lấy dữ liệu Meta`(242) → `Xử lý dữ liệu`(242) → `Ghi vào Sheet`(242) — khớp xuyên pipeline.
- **PASS.**

### Bước 4 — Git
- Stage đúng 1 file: `Project/WORKFLOWS/Meta_Ads_Daily_Sheet_Update.json` (verify bằng `git diff --cached --stat` trước commit).
- ✅ Commit `8ca892a`: "task04: update canonical — pagination fix production".
- 🔴 **Push vẫn CHƯA thực hiện** (cùng blocker từ PHASE 4: auto-mode classifier chặn `git push`).
  Branch hiện ahead 2 commit (`26fdbe3`, `8ca892a`). Cần anh Lộc push tay.

### Bước 5 — Docs
- `CURRENT_STATUS.md`: DONE, phase gate cập nhật đủ PHASE 5/5b/6.
- `AI_OS/incidents/2026-10-02_missing-ads-data.md`: Fix=DONE, Verified=DONE.
- `AI_OS/SKILLS.md` Incident Log: DONE.
- `TASK_INDEX.md`: Task 04 = DONE.

### Bước 6 — Dọn dẹp
- Xoá workflow TEST `nILRkiamvwXexV0o` (sau khi production verify PASS, đúng thứ tự).

### Trạng thái
- **Task_04: CLOSED (kỹ thuật).** Fix verify PASS trên production thật (242 ad, 0 error,
  pagination hoạt động đúng, không bị truncate). Còn lại: **anh Lộc push tay** 2 commit
  (`26fdbe3`, `8ca892a`) lên remote — AI không tự push được do permission classifier.

---

## Session 2026-10-07 — PUSH + ĐÓNG TASK

- ✅ Anh Lộc xác nhận: **đã push lên remote thành công** (2 commit `26fdbe3`, `8ca892a`).
- ✅ Không còn việc tồn đọng của Task_04.
- Bài học tổng quát (không riêng Meta API) đã tách ra khỏi Task này, ghi ở cấp **AI_OS**
  để mọi task tích hợp API tương lai đều đọc được — xem `AI_OS/AI_OS_CHANGELOG.md` (2026-10-07)
  và `AI_OS/templates/ops/API_INTEGRATION_CHECKLIST.md` (mới).

### Trạng thái cuối
- **Task_04: CLOSED.** Fix PASS trên production + đã push remote. Không còn blocker.
