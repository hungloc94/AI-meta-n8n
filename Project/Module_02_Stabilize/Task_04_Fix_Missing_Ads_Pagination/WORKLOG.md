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
- **Task_04: IN_PROGRESS** — điều tra DONE, fix PENDING. Dừng tại **approval gate PHASE 4**, chờ anh Lộc CONFIRM `PLAN.md`.
