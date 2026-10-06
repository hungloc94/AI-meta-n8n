# CURRENT_STATUS — Task 04: Fix Missing Ads Data (Pagination)

> **Canonical execution state** của AIOS cho Task này. Master plan: `PLAN.md`.
> Case gốc: incident `AI_OS/incidents/2026-10-02_missing-ads-data.md`.
> Không duplicate state sang file khác.

**Cập nhật lần cuối:** 2026-10-06 — Claude Code

## Trạng thái

- **Task status:** 🟢 **DONE** — fix đã verify PASS trên production (2026-10-06 23:12 UTC / 2026-10-07 06:12 VN).
  **Đã push remote (2026-10-07)** — commit `26fdbe3`, `8ca892a`. Không còn blocker.
- **Phase hiện tại:** PHASE 6 — đóng task.

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
| PHASE 4 | Backup + implement fix trên bản TEST | ✅ **DONE** (2026-10-06) | Backup 2 lớp `BACKUP/task-fix-pagination/`; TEST workflow `nILRkiamvwXexV0o` (active=false, tab `TEST_DEUP`) |
| PHASE 4b | Verify trên `TEST_DEUP` | ✅ **PASS** (2026-10-06) | Run1 `appendCount=249`; Run2 `appendCount=0, updateCount=249`; diff Meta vs Sheet = **0**; Nhóm B nguyên vẹn |
| PHASE 5 | Canonical JSON + lên production | ✅ **DONE** (2026-10-06 23:xx UTC) | Canonical cập nhật từ PROD_LIVE backup + code fix (base đúng, không lẫn tab TEST); PUT update workflow `rz3Wya5lFay7ShVL` |
| PHASE 5b | Verify production thật | ✅ **PASS** (2026-10-06 23:12 UTC) | `docker exec n8n n8n execute --id=rz3Wya5lFay7ShVL`: EXIT=0, error=None, `appendCount=141, updateCount=101, total=242`; 8/8 ngày `pages=1, truncated=false` |
| PHASE 6 | Commit + dọn dẹp | ✅ DONE | Commit `8ca892a` (canonical); xoá workflow TEST `nILRkiamvwXexV0o` |

## PHASE 4 — Kết quả verify (2026-10-06)

### Bằng chứng định lượng — root cause xác nhận

Đo trực tiếp trong container n8n (`BACKUP/task-fix-pagination/measure_pagination.js`), 7 ngày:

| Ngày | Code cũ (page 1, không limit) | Code mới (limit + pagination) | **Thiếu** |
|---|---|---|---|
| 2026-09-29 | 25 | 32 | **7** |
| 2026-09-30 | 25 | 33 | **8** |
| 2026-10-01 | 25 | 32 | **7** |
| 2026-10-02 | 25 | 33 | **8** |
| 2026-10-03 | 25 | 30 | **5** |
| 2026-10-04 | 25 | 30 | **5** |
| 2026-10-05 | 25 | 30 | **5** |
| **Tổng** | **175** | **220** | **45** |

→ Code cũ chốt cứng ở **25 ad/ngày** (default page size Meta); `paging.next` luôn tồn tại mọi ngày.

### Verify trên `TEST_DEUP` (workflow `nILRkiamvwXexV0o`)

| Test | Kết quả | Đánh giá |
|---|---|---|
| Run 1 | `appendCount=249, updateCount=0` | ✅ ghi đủ |
| Run 2 | `appendCount=0, updateCount=249` | ✅ idempotent |
| Diff Meta vs Sheet (8 ngày) | 249 vs 249, lệch từng ngày = **0** | ✅ **không thiếu ad nào** |
| Nhóm B | đủ 9 cột; 1829/1829 dòng có dữ liệu; 0 cột rỗng bất thường | ✅ nguyên vẹn |
| Key trùng | `distinctKeys=1826`, `duplicateKeyCount=0` | ✅ |
| Page/ngày | 1 page (do `limit=1000` phủ hết); `truncated=false` | ✅ guard không kích hoạt |

Số row theo ngày trên Sheet sau fix khớp **chính xác** số từ Meta API: 32/33/32/33/30/30/30/29.
Sau fix, 7 ngày = **220 rows** (so với 175 nếu chạy code cũ).

## PHASE 5b — Verify production thật (2026-10-06 23:12 UTC)

```
docker exec n8n n8n execute --id=rz3Wya5lFay7ShVL
EXIT=0
```

| Tiêu chí | Kết quả |
|---|---|
| `resultData.error` | None |
| Node `Ghi vào Sheet` | `executionStatus: success` |
| `appendCount` | 141 |
| `updateCount` | 101 |
| **Total** | **242** |
| Pages/ngày | 1 page × 8 ngày; `truncated=false` mọi ngày |
| `Lấy dữ liệu Meta` → `Xử lý dữ liệu` → `Ghi vào Sheet` | 242 → 242 → 242 (không mất qua pipeline) |

Số ad/ngày (window đã trượt sang 09-30 → 10-07 do scheduled run 00:30 UTC chạy trước lúc verify):
`33, 32, 33, 30, 30, 30, 29, 25`.

## Sự cố nhỏ trong quá trình implement (đã xử lý)

Hai lệnh tự động hóa (python ghép JSON, `cp` copy bản TEST) bị **auto-mode classifier chặn**
("Credential Materialization", "Out-of-Place Publication"). Anh Lộc tự chạy lệnh ghép JSON
(Cách A) để mở khoá — không có workaround nào được thực hiện để lách lớp chặn.
Kết quả sau khi anh chạy: canonical đúng, base = PROD_LIVE backup (không lẫn tab TEST_DEUP).



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
