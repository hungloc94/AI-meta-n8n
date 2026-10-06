# INCIDENT: Thiếu dữ liệu quảng cáo — 2026-10-02

> Incident log theo chuẩn AI_OS. Điều tra: 2026-10-06, Claude Code.
> Task fix: `Project/Module_02_Stabilize/Task_04_Fix_Missing_Ads_Pagination/`.

## Executive Summary

Workflow `Meta_Ads_Daily_Sheet_Update` (`rz3Wya5lFay7ShVL`) ghi thiếu ad vào Google Sheet.
Root cause: node `Lấy dữ liệu Meta` gọi Meta Insights API **không xử lý pagination** →
chỉ đọc page 1, các ad còn lại mất im lặng (không error, không cảnh báo).

## Timeline

| Thời điểm | Sự kiện |
|-----------|---------|
| 2026-10-02 | Anh Lộc phát hiện: một số ad ID không xuất hiện trong báo cáo; cập nhật thủ công cũng không lấy được |
| 2026-10-06 | Điều tra: phân tích workflow JSON + execution log → xác định root cause pagination |
| 2026-10-06 | Tạo Task_04 + PLAN + incident file (chưa sửa) |

## Triệu chứng

Một số ad ID không xuất hiện trong báo cáo. Chạy cập nhật thủ công cũng không lấy được
các ad đó → loại bỏ khả năng "chỉ là run bị lỡ".

## Điểm phát sinh

**Node:** `Lấy dữ liệu Meta` (`1537644d-fc50-4273-a8e5-85a772519976`)

- Input: 1 item/ngày (7 item cho scheduled run, `numDays=07`)
- Output: 1 item/ngày chứa `response.data` **của page 1 mà thôi**
- Mất tại đây: toàn bộ ad ở page 2 trở đi của `/insights`

**Bằng chứng code (workflow JSON, node `Lấy dữ liệu Meta`):**

```javascript
const url = `https://graph.facebook.com/v19.0/act_303484773737553/insights`
  + `?time_range=${timeRange}&level=ad&fields=...`;
//  ^ KHÔNG có &limit=...
//  ^ KHÔNG có vòng lặp paging.next
...
responseData = response;   // lấy nguyên object, KHÔNG gộp page
```

## Root Cause

**Failing behavior:** ad bị mất khỏi Sheet mà không có bất kỳ dấu hiệu lỗi nào.

**Component chịu trách nhiệm:** node Code `Lấy dữ liệu Meta` trong `rz3Wya5lFay7ShVL`.

**Điều kiện kích hoạt:** account có nhiều ad (trong 1 ngày) hơn page size của Meta Insights API
(mặc định 25, vì node không set `limit`) → Meta trả `paging.next` → code bỏ qua.

**Evidence:**
- Code node không có `limit` và không đọc `response.paging`.
- Node `Lấy cấu hình Meta` (`/ads`) **có** `limit=1000` nhưng cũng không loop → số dòng Sheet
  do node `/insights` quyết định, không phải node này.

**Why other causes ruled out:**
- Không có node IF/Filter nào trong flow → không lọc bỏ.
- Key `${date_start}_${ad_id}` unique → không collision.
- `level=ad` đúng; date range đúng (`numDays=7`); không lọc status/spend khi fetch.

**Confidence:** CAO — đã có **bằng chứng định lượng** (đo 2026-10-06, xem mục Phạm vi ảnh hưởng).

## Data Flow Trace

```
Meta Insights API (page 1: 25 ad + paging.next)
   → Node "Lấy dữ liệu Meta"   [page 1 only — MẤT page 2..N tại đây]
   → Node "Xử lý dữ liệu"      [1 item/ad-insight, giữ nguyên]
   → Node "Ghi vào Sheet"      [appendOrUpdate by Key]
   → Sheet
```

**Mất tại:** `Lấy dữ liệu Meta` — không gộp các page.

**Số liệu trước fix (đo 2026-10-06, 7 ngày):** 175 rows lấy được / 220 rows thực tế → **mất 45 rows**.
Chốt cứng ở 25 ad/ngày (default page size Meta), `paging.next` tồn tại **mọi ngày**.

## Phạm vi ảnh hưởng

**Trước fix:** mất **5–8 ad mỗi ngày** trên mọi ngày có >25 ad. Đo 2026-09-29 → 10-05:
tổng 45 rows bị thiếu trên 7 ngày (175/220).

| Ngày | Trước fix | Sau fix | Thiếu |
|---|---|---|---|
| 2026-09-29 | 25 | 32 | 7 |
| 2026-09-30 | 25 | 33 | 8 |
| 2026-10-01 | 25 | 32 | 7 |
| 2026-10-02 | 25 | 33 | 8 |
| 2026-10-03 | 25 | 30 | 5 |
| 2026-10-04 | 25 | 30 | 5 |
| 2026-10-05 | 25 | 30 | 5 |

Ảnh hưởng **cột Nhóm A** (dữ liệu Meta ghi). Nhóm B không bị ảnh hưởng trực tiếp, nhưng
báo cáo downstream đọc Nhóm A sẽ thiếu. Phạm vi thời gian: **toàn bộ vòng đời workflow**
(lỗi có từ khi node được viết — không phải sự cố mới phát sinh).

## Trạng thái

- Phát hiện : 2026-10-02
- Fix        : **DONE** (2026-10-06) — lên production, verify PASS
- Verified   : **DONE** (2026-10-06 23:12 UTC) — xem kết quả bên dưới

## Fix đã áp dụng

Node `Lấy dữ liệu Meta` (workflow `rz3Wya5lFay7ShVL`):
1. Thêm `&limit=1000` vào URL `/insights`.
2. Thêm pagination loop: đọc `paging.cursors.after`, gộp `data[]` các page,
   guard `MAX_PAGES=10`, log số page/ngày.

**Verify trên TEST_DEUP** (workflow `nILRkiamvwXexV0o`, active=false, đã xoá sau khi production PASS):
- Run 1 `appendCount=249`; Run 2 `appendCount=0, updateCount=249` → idempotent.
- Diff Meta API vs Sheet (8 ngày): 249 vs 249 → **lệch = 0**.
- Nhóm B nguyên vẹn (9/9 cột; 1829/1829 dòng có dữ liệu); 0 Key trùng.

**Verify trên production thật** (`rz3Wya5lFay7ShVL`, 2026-10-06 23:12 UTC):
```
docker exec n8n n8n execute --id=rz3Wya5lFay7ShVL
EXIT=0, resultData.error=None, Ghi vào Sheet: success
appendCount=141, updateCount=101, total=242
8/8 ngày: pages=1, truncated=false
```

Code nguồn: `BACKUP/task-fix-pagination/new_insights_node.js`.
Canonical: `Project/WORKFLOWS/Meta_Ads_Daily_Sheet_Update.json` (commit `8ca892a`, đã push remote 2026-10-07).
Chi tiết: `Project/Module_02_Stabilize/Task_04_Fix_Missing_Ads_Pagination/`.

## Bài học tổng quát (đã tách sang cấp AI_OS)

Bài học **"luôn kiểm tra pagination khi gọi API bên ngoài"** áp dụng cho MỌI tích hợp API
tương lai, không riêng Meta Ads — đã nâng cấp thành checklist dùng lại:
`AI_OS/templates/ops/API_INTEGRATION_CHECKLIST.md`. Đọc file đó **trước khi** viết bất kỳ
node/script gọi API ngoài nào mới.

## Bài học

**Luôn kiểm tra pagination khi gọi API bên ngoài.**
Meta API (và hầu hết REST API) mặc định trả về tối đa N records/page. Không xử lý pagination
= âm thầm mất dữ liệu, không có error. Dấu hiệu nhận biết: số records tròn (đúng 1000, đúng 25…).
Xem chi tiết tại `Project/Module_02_Stabilize/Task_04_Fix_Missing_Ads_Pagination/PLAN.md`.

## Còn cần thu thập

1. Số ad thực tế từ Meta API trong cửa sổ ngày (cần `META_ACCESS_TOKEN`).
2. Danh sách `ad_id` từ Meta API vs từ Sheet → diff = danh sách ad bị thiếu.
3. Đặc điểm chung của ad bị thiếu (cùng campaign? cùng status? thứ tự trong page?).
