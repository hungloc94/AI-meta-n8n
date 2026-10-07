# PLAN — Task 05: Backfill Missing Ads Data

> Phạm vi: backfill dữ liệu quảng cáo từ `2026-08-01` đến `2026-10-06`.
> Workflow đích: `Meta Ads Daily Sheet Update 7:30 2` (`rz3Wya5lFay7ShVL`).
> Trạng thái thực thi: `CURRENT_STATUS.md`.
> Tiền đề: Task_04 đã triển khai pagination cho Meta Insights API.

## Mục tiêu

Bổ sung các dòng ad/ngày bị thiếu trước khi Task_04 được triển khai, không xóa dòng nào
và không làm thay đổi dữ liệu business do người dùng quản lý trong Nhóm B.

## Phạm vi

- **Từ:** `2026-08-01`
- **Đến:** `2026-10-06` (inclusive)
- **Tổng:** **67 ngày**
- API dùng `time_range={since: date, until: date}` từng ngày; không dùng `date_preset`.
- Tháng 8 lấy được vì `time_range` cho phép chỉ định ngày trực tiếp; không phụ thuộc giới hạn
  cửa sổ của `date_preset`.

## Thiết kế workflow backfill

Tên: `Task05 — Backfill 2026-08-01 to 2026-10-06`

- `active=false`, chỉ có manual trigger.
- Node tạo date range trả đúng 67 item `{json:{date:"YYYY-MM-DD"}}`.
- Node Meta dùng pagination của Task_04: `limit=1000`, đọc `paging.cursors.after`,
  `MAX_PAGES=10`, gộp `data[]`, giữ shape `{json:{data:[]}}`.
- Node `Xử lý dữ liệu` giữ nguyên code production.
- Node `Ghi vào Sheet` dùng `appendOrUpdate`, matching `Key`, tab production
  `BÁO_CÁO_QUẢNG_CÁO`, chỉ map Nhóm A.
- Không xóa dòng nào trong Sheet.

## An toàn dữ liệu

- Key = `{date_start}_{ad_id}`; ad đã có sẽ update Nhóm A, ad thiếu sẽ append dòng mới.
- Nhóm B (`Mess_Comment`, `Khach_sai_tep`, `Khach_hop_le`, `SDT`, `Khach_chot`,
  `Chi_phi_*`, `Doanh_thu`, `ghi_chu`) không được map vào input của node ghi.
- `appendOrUpdate` là cơ chế idempotent; chạy lại không tạo Key mới cho dòng đã có.
- Không activate workflow backfill.

## Rate limit và thời gian chạy

- Ước tính tối thiểu 67 call Meta (một call/ngày khi mỗi ngày <=1000 record).
- Pagination có thể làm số call tăng; `MAX_PAGES=10` giới hạn vòng lặp.
- Giữ delay giữa các ngày để giảm burst; cần kiểm tra execution time thực tế.
- Không chạy đồng thời với scheduled production sync nếu có thể tránh được.

## Test/verify sau khi chạy

1. `EXIT=0`, không có node error.
2. Ghi lại `appendCount`, `updateCount`, `total`.
3. Trace item count: date range (67) → Meta → transform → Sheet.
4. Đếm ad/ngày trong Sheet từ 08-01 đến 10-06; ngày không còn bị chốt ở 25,
   trừ ngày thực sự có ít ad.
5. `COUNTIF(Key)>1 = 0`.
6. Lấy mẫu ít nhất 5 dòng đã có Nhóm B trước backfill và xác nhận Nhóm B còn nguyên.
7. Sau khi verify PASS mới xóa workflow backfill và đóng task.

## Definition of Done

- [x] Backfill 67 ngày chạy thành công, không error.
- [x] `appendCount=66`, `updateCount=1133`, total=1199 được ghi nhận.
- [x] Không còn ngày bị chốt ở 25 record trong phạm vi.
- [x] Không có duplicate Key.
- [x] Nhóm B của 5 dòng có dữ liệu thực không thay đổi so với snapshot trước.
- [x] Workflow backfill đã xóa sau verify.
- [x] Bài học 5 được ghi vào AI_OS.
- [ ] Commit Task_05 + AI_OS tạo và báo anh Lộc push.

## Trạng thái cuối

- **DONE** — backfill và verify hoàn tất ngày 2026-10-07.
- 67 ngày đã xử lý; 1199 records; 66 append + 1133 update.
- Sheet sau backfill: 2379 dòng; duplicate Key = 0; không ngày nào đúng 25 ad.
- 5 mẫu Nhóm B đối chiếu snapshot trước/sau: **không thay đổi**.
- Workflow backfill `udyHkTWSU2c3J54v` đã xóa sau verify PASS.
