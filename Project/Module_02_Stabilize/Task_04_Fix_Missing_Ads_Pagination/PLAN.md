# PLAN — Task 04: Fix Missing Ads Data (Pagination)

> Master plan. Trạng thái execution: `CURRENT_STATUS.md` (Task_04).
> Case gốc: incident 2026-10-02. Task_03 (duplicate) là task tiền nhiệm — cùng workflow `rz3Wya5lFay7ShVL`.
> Điều tra: 2026-10-06, Claude Code. **Chưa sửa gì.**

---

## 1. Root cause

Node **"Lấy dữ liệu Meta"** (`1537644d-fc50-4273-a8e5-85a772519976`) gọi Meta Insights API
nhưng **không xử lý pagination**.

Cụ thể trong code node (line 111 workflow JSON):

```javascript
const url = `https://graph.facebook.com/v19.0/act_303484773737553/insights`
  + `?time_range=${timeRange}&level=ad&fields=ad_id,ad_name,...`;
//  ^ KHÔNG có &limit=...
//  ^ KHÔNG có vòng lặp đọc response.paging.next
```

Sau khi nhận response, code chỉ làm:

```javascript
responseData = response;   // lấy nguyên object, KHÔNG gộp page
success = true;
```

→ **Chỉ dùng `response.data` của page 1.** Nếu account có nhiều ad hơn page size,
các ad ở page 2, 3, … **bị mất hoàn toàn, không có error, không có cảnh báo.**

### Điểm mấu chốt — có HAI node, chỉ một node có `limit`, và node đó không phải node bị lỗi

| Node | Endpoint | Có `limit`? | Có pagination? | Dùng để làm gì |
|---|---|---|---|---|
| `Lấy cấu hình Meta` | `/ads` | ✅ `limit=1000` | ❌ không | Lấy `effective_status` + `adset.daily_budget` để map trạng thái/ngân sách |
| `Lấy dữ liệu Meta` | `/insights` | ❌ **không có** | ❌ không | Lấy metrics thực tế (spend, reach, click…) — **đây là nguồn sinh 1 dòng/ad/ngày** |

`limit=1000` ở node `/ads` là **ngụy trang** — nó khiến ta tưởng đã xử lý page size.
Nhưng số dòng trong Sheet do node `/insights` quyết định, và node này không set limit.

### Vì sao các giả thuyết khác bị loại bỏ

| Giả thuyết | Kết luận | Bằng chứng |
|---|---|---|
| Filter/IF node loại bỏ ad | ❌ Loại | Không có node IF/Filter nào trong flow. Connections: `Lấy dữ liệu Meta → Xử lý dữ liệu → Ghi vào Sheet → Summary` |
| Key collision (2 ad cùng Key) | ❌ Loại | Key = `${row.date_start}_${row.ad_id}`; `ad_id` unique theo ad → 1 ad/1 ngày = 1 Key |
| Date range sai | ❌ Loại | `numDays=07` cho scheduled; `time_range={since,until}` cùng ngày; `slice(-2)` chỉ áp cho manual run |
| API level sai | ❌ Loại | `level=ad` — đúng, trả về từng ad riêng lẻ |
| Status filter (PAUSED/ARCHIVED) | ❌ Loại | Không lọc status ở tầng fetch; `effective_status` chỉ dùng để map cột `Trang_Thai` = Bật/Tắt |
| Lỗi ghi Sheet | ❌ Loại | Dùng `appendOrUpdate` matching `Key`; nếu ghi lỗi sẽ có exception, không im lặng |

→ Pagination là **nguyên nhân duy nhất** giải thích được kiểu mất dữ liệu "im lặng".

---

## 2. Fix đề xuất

Thêm **pagination loop** vào node `Lấy dữ liệu Meta`, và set `limit` tường minh cho `/insights`.

### Thay đổi 1 — thêm `&limit=1000` vào URL của `/insights`
Loại bỏ sự phụ thuộc vào default page size của Meta (25).

### Thay đổi 2 — vòng lặp lấy hết page, gộp `data[]`

Pseudo:

```
let allRows = [];
let nextUrl = buildUrl(date);          // page 1
let page = 0;
const MAX_PAGES = 10;                  // guard chống infinite loop
while (nextUrl && page < MAX_PAGES) {
  const response = await httpRequest(nextUrl);
  allRows.push(...(response.data || []));
  nextUrl = response.paging?.next || null;
  page++;
}
results.push({ json: { data: allRows } });   // giữ nguyên shape cũ
```

Điểm cần giữ:
- Output phải vẫn là `{ json: { data: [...] } }` — node `Xử lý dữ liệu` đọc `insightItem.json.data`,
  nếu đổi shape sẽ vỡ toàn bộ transform phía sau.
- Retry/rate-limit hiện có phải áp cho **từng page**, không chỉ page 1.
- Delay 2000ms giữa các ngày; thêm delay nhỏ giữa các page nếu cần.

### Lưu ý khi implement
- **Rate limit Meta:** ~200 calls/giờ/ad account. 7 ngày × 3 page = 21 calls/run → an toàn,
  nhưng phải đếm để không vượt khi account lớn lên.
- **Page size:** mặc định 25; với `limit=1000` account lớn có thể cần 2-3 page/ngày.
- **Timeout node:** loop nhiều page làm node chạy lâu hơn — cần kiểm tra `timeout` của Code node
  và của execution, tránh bị cắt giữa chừng.
- **Cooldown 60s** (`staticData.lastMetaCall`) đã có — không đụng.

---

## 3. Risk assessment

| Mức | Rủi ro | Kiểm soát |
|---|---|---|
| **Thấp** | Thêm loop fetch — không thay đổi logic ghi Sheet, không đổi Key, không đụng Nhóm B | Giữ nguyên `Xử lý dữ liệu` + `Ghi vào Sheet` |
| **Trung bình** | Loop không có điều kiện dừng → **infinite loop**, treo workflow | `MAX_PAGES = 10` guard bắt buộc |
| **Trung bình** | Đổi shape output → vỡ transform | Giữ nguyên `{ json: { data: [] } }`; test trước |
| **Thấp** | Tăng số call Meta → chạm rate limit | Đếm call/run; giữ delay; đo trước khi lên production |
| **Thấp** | Node chạy lâu hơn → timeout | Đo thời gian run thực tế ở TEST trước |

**Rollback:** revert node `Lấy dữ liệu Meta` về JSON backup. Không có thay đổi schema/dữ liệu → rollback sạch.

---

## 4. Test plan

1. Backup workflow JSON hiện tại (`BACKUP/task-fix-pagination/`) trước khi sửa.
2. Sửa node trên bản TEST (không sửa production trực tiếp).
3. Chạy trên tab `TEST_DEUP`.
4. **So sánh số ad lấy được với số ad thật từ Meta API** — đếm độc lập qua `GET /insights?level=ad`
   có pagination thủ công (curl/script), đối chiếu từng `ad_id`.
5. Verify **không ad nào bị thiếu**: diff danh sách `ad_id` từ Meta API vs danh sách trong Sheet → diff = 0.
6. Chạy lần 2 → `appendCount=0` (vẫn idempotent), `updateCount` ổn định.
7. Kiểm tra Nhóm B không đổi.

---

## 5. Definition of Done

- [ ] Số ad trong Sheet = số ad từ Meta API (diff = 0), verify bằng danh sách `ad_id`
- [ ] Workflow chạy thành công không error
- [ ] Idempotent: chạy 2 lần không sinh dòng mới (`appendCount=0`)
- [ ] Không ảnh hưởng Nhóm B (Mess_Comment, Khach_*, SDT, Chi_phi_*, Doanh_thu, ghi_chu)
- [ ] Không vượt rate limit Meta
- [ ] Rollback path đã verify (backup còn nguyên tới khi task đóng)
- [ ] Canonical JSON trong `Project/WORKFLOWS/` đã cập nhật + commit

---

## 6. Trạng thái

- **Điều tra:** DONE (2026-10-06) — Phase 1 (phân tích JSON) + Phase 2 (execution log).
- **Fix:** PENDING — chờ anh Lộc CONFIRM.
- **Ghi chú Phase 2:** 3 execution gần nhất (`589`, `582`, `575`) status = `error`.
  Anh Lộc xác nhận **do mạng home server giật, KHÔNG phải lỗi workflow** → không thuộc scope Task_04.
