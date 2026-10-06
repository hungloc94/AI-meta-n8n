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

**Confidence:** CAO (bằng chứng code xác định) — nhưng **chưa có verify định lượng**
(số ad thiếu thực tế chưa đếm được do session không có quyền truy cập Meta token).

## Data Flow Trace

```
Meta Insights API (page 1: 25 ad + paging.next)
   → Node "Lấy dữ liệu Meta"   [page 1 only — MẤT page 2..N tại đây]
   → Node "Xử lý dữ liệu"      [1 item/ad-insight, giữ nguyên]
   → Node "Ghi vào Sheet"      [appendOrUpdate by Key]
   → Sheet
```

**Mất tại:** `Lấy dữ liệu Meta` — không gộp các page.

## Phạm vi ảnh hưởng

- **Chưa xác định định lượng** (chưa có token để đếm). Giả thuyết: mọi ngày có >25 ad
  (page size mặc định) đều bị thiếu phần đuôi.
- Ảnh hưởng **cột Nhóm A** (dữ liệu Meta ghi). Nhóm B không bị ảnh hưởng trực tiếp, nhưng
  báo cáo downstream đọc Nhóm A sẽ thiếu.

## Trạng thái

- Phát hiện : 2026-10-02
- Fix        : **PENDING** — Task_04 (`Project/Module_02_Stabilize/Task_04_Fix_Missing_Ads_Pagination/`)
- Verified   : PENDING

## Fix đã áp dụng

_(điền sau khi sửa xong)_

## Bài học

**Luôn kiểm tra pagination khi gọi API bên ngoài.**
Meta API (và hầu hết REST API) mặc định trả về tối đa N records/page. Không xử lý pagination
= âm thầm mất dữ liệu, không có error. Dấu hiệu nhận biết: số records tròn (đúng 1000, đúng 25…).
Xem chi tiết tại `Project/Module_02_Stabilize/Task_04_Fix_Missing_Ads_Pagination/PLAN.md`.

## Còn cần thu thập

1. Số ad thực tế từ Meta API trong cửa sổ ngày (cần `META_ACCESS_TOKEN`).
2. Danh sách `ad_id` từ Meta API vs từ Sheet → diff = danh sách ad bị thiếu.
3. Đặc điểm chung của ad bị thiếu (cùng campaign? cùng status? thứ tự trong page?).
