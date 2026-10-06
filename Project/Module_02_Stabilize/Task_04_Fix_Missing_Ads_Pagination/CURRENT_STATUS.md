# CURRENT_STATUS — Task 04: Fix Missing Ads Data (Pagination)

> **Canonical execution state** của AIOS cho Task này. Master plan: `PLAN.md`.
> Case gốc: incident `AI_OS/incidents/2026-10-02_missing-ads-data.md`.
> Không duplicate state sang file khác.

**Cập nhật lần cuối:** 2026-10-06 — Claude Code

## Trạng thái

- **Task status:** 🟡 IN_PROGRESS — điều tra DONE, fix PENDING (chờ CONFIRM).
- **Phase hiện tại:** PHASE 3 DONE (PLAN + docs) — **dừng tại approval gate**, chờ anh Lộc CONFIRM trước khi sửa node.
- **Blocker:** cần CONFIRM để implement (sửa node `Lấy dữ liệu Meta` trên bản TEST).

## Root cause (đã xác định)

Node **"Lấy dữ liệu Meta"** (`1537644d-fc50-4273-a8e5-85a772519976`) gọi Meta Insights API
`/insights?level=ad` **không set `limit`** và **không xử lý pagination** (`paging.next`).

→ Chỉ đọc `response.data` của page 1. Account có nhiều ad hơn page size → các ad còn lại
**mất hoàn toàn, im lặng**.

**Bẫy:** node `Lấy cấu hình Meta` (`/ads`) **có** `limit=1000` và không có pagination →
tạo cảm giác đã xử lý page size, nhưng đó không phải node quyết định số dòng trong Sheet.

## Phase gate

| Phase | Nội dung | Trạng thái | Evidence |
|-------|----------|-----------|----------|
| PHASE 1 | Phân tích workflow JSON (5 điểm nghi ngờ) | ✅ DONE (2026-10-06) | Chỉ 1 điểm fail: pagination node `/insights`; 4 điểm còn lại loại bỏ |
| PHASE 2 | Xác minh bằng execution log | ✅ DONE (2026-10-06) | 5 execution gần nhất đọc từ SQLite; 3 lỗi `error` là do mạng (anh Lộc xác nhận) |
| PHASE 3 | Viết PLAN + cập nhật vault | ✅ DONE (2026-10-06) | `PLAN.md`, file này, incident, TASK_INDEX, LESSONS_LEARNED |
| PHASE 4 | Backup + implement fix trên bản TEST | ⏸ **CHỜ CONFIRM** | — |
| PHASE 5 | Verify trên TEST_DEUP + lên production | ⏸ Chưa bắt đầu | — |

## 5 điểm nghi ngờ — kết quả Phase 1

| # | Điểm kiểm tra | Kết quả |
|---|---------------|---------|
| 1 | Pagination Meta API | ❌ **FAIL — root cause.** `/insights` không có `limit`, không loop `paging.next` |
| 2 | Filter nodes | ✅ PASS — không có node IF/Filter nào |
| 3 | Key collision | ✅ PASS — `{date_start}_{ad_id}`, `ad_id` unique |
| 4 | Date range logic | ✅ PASS — `numDays=7` scheduled, `slice(-2)` chỉ cho manual |
| 5 | API level | ✅ PASS — `level=ad` đúng |

## Facts

| Entity | Giá trị |
|--------|---------|
| Workflow id | `rz3Wya5lFay7ShVL` — "Meta Ads Daily Sheet Update 7:30 2", active |
| Node lỗi | `Lấy dữ liệu Meta` = `1537644d-fc50-4273-a8e5-85a772519976` |
| Endpoint lỗi | `GET /v19.0/act_303484773737553/insights?level=ad` |
| Node "ngụy trang" | `Lấy cấu hình Meta` = `bf2d19b2-1f46-4e23-8131-6dd17845af47` (`/ads`, có `limit=1000`) |
| Key format | `{date_start}_{ad_id}` — cột **S** |
| Sheet | `1EHpEws60xWjJUaBfd_qDFLoeikMYvhUc9f9fR-HcL_s` — tab `BÁO_CÁO_QUẢNG_CÁO`, tab test `TEST_DEUP` |
| Nhóm B (không đụng) | Mess_Comment, Khach_sai_tep, Khach_hop_le, SDT, Khach_chot, Chi_phi_*, Doanh_thu, ghi_chu |

## CHƯA xác minh được (cần token — giới hạn của session)

Số ad thực tế từ Meta API **chưa đếm được** trong session này (không có `META_ACCESS_TOKEN`,
và bị chặn đọc credential từ n8n). Do đó **chưa có bằng chứng định lượng**: chưa biết chính xác
thiếu bao nhiêu ad, ngày nào, `ad_id` nào.

Root cause được suy ra từ **bằng chứng code** (chắc chắn: không có pagination trong code),
không phải từ diff dữ liệu. Bước verify định lượng nằm trong Test plan của `PLAN.md`.

## Next action

→ Anh Lộc CONFIRM `PLAN.md` → tôi backup workflow JSON → implement fix pagination trên bản TEST (active=false)
→ verify trên `TEST_DEUP` → báo cáo kết quả diff trước khi lên production.

## Auto-fix attempt counter

| Root cause | Attempts | Trạng thái |
|-----------|----------|-----------|
| Missing pagination node `/insights` | 0 / 3 | Chưa sửa (chờ CONFIRM ở PHASE 4) |
