# API Integration Checklist

> Bắt buộc đọc **TRƯỚC KHI** viết bất kỳ node/script mới gọi một API bên ngoài
> (Meta, Google, Telegram, hoặc bất kỳ REST API nào khác) trong dự án này.
> Rút ra từ incident `AI_OS/incidents/2026-10-02_missing-ads-data.md` (Task_04,
> workflow `Meta_Ads_Daily_Sheet_Update`): mất 5–8 ad/ngày trong nhiều tuần do
> thiếu pagination — không có error, không ai nhận ra bằng log, chỉ phát hiện
> được khi nhìn báo cáo thấy thiếu ad bằng mắt.

## Vì sao checklist này tồn tại

Lỗi pagination là lớp lỗi **nguy hiểm nhất** trong tích hợp API: nó **không throw
error**, workflow vẫn báo `success`, số liệu vẫn có vẻ hợp lý — chỉ thiếu phần đuôi.
Không có log nào tự nhiên cảnh báo trừ khi người review chủ động đếm số record
trả về và so với con số tròn nghi ngờ (25, 100, 1000...).

## Checklist bắt buộc — trước khi tích hợp API mới

- [ ] **API có pagination không?** Đọc docs chính thức, tìm từ khóa: `paging`,
      `cursor`, `next_page_token`, `offset/limit`, `Link` header (GitHub-style).
- [ ] **Default page size là bao nhiêu khi KHÔNG set limit?** (Meta Graph API:
      25. Nhiều API khác: 20, 50, 100 — không bao giờ giả định "chắc đủ".)
- [ ] **Code có set `limit`/`page_size` tường minh không?** Không dựa vào default.
- [ ] **Code có loop đọc HẾT các page không?** (đọc `paging.next`/cursor, lặp tới
      khi hết, không chỉ gọi 1 lần rồi dùng `response.data`).
- [ ] **Có `MAX_PAGES` guard chống infinite loop không?** (ví dụ 10) — để lỡ API
      trả cursor lỗi/loop vô hạn thì dừng có kiểm soát, không treo workflow.
- [ ] **Log số page + số record mỗi lần gọi** để debug dễ khi cần (không bắt buộc
      log phải hiển thị trong mọi UI, nhưng phải có để truy vết).
- [ ] **Rate limit của API là gì?** Số call/giờ/ngày — tính trước workflow sẽ gọi
      bao nhiêu lần/chu kỳ × số page, đảm bảo không chạm giới hạn.
- [ ] **Endpoint khác trong CÙNG workflow có set `limit` khác không?** — kiểm tra
      kỹ: một node có `limit=1000` **không có nghĩa** node khác trong cùng workflow
      cũng vậy. Mỗi node/endpoint phải tự verify riêng (xem "Bẫy ngụy trang" dưới).

## Dấu hiệu nhận biết lỗi pagination đã xảy ra (khi audit workflow cũ)

- Số record trả về **luôn đúng một con số tròn cố định** qua nhiều lần chạy khác
  nhau (ví dụ luôn đúng 25, hoặc luôn đúng 100) — không bao giờ ít hơn, không
  bao giờ nhiều hơn → gần như chắc chắn đang bị cắt ở default page size.
- Code gọi `httpRequest` rồi dùng thẳng `response.data` mà **không có vòng lặp
  nào** đọc `response.paging` hoặc tương đương.

## Bẫy ngụy trang — "có limit ở đâu đó" ≠ "đã xử lý page size"

Trong Task_04, workflow có **2 node** gọi Meta API:
- node `/ads` có `limit=1000` — khiến người đọc code tưởng đã xử lý page size.
- node `/insights` (node thực sự sinh ra dữ liệu ghi vào Sheet) **không có
  `limit` nào** — đây là nơi mất dữ liệu thật.

**Bài học:** khi audit, phải xác định rõ **node nào quyết định output cuối cùng**
(ghi vào Sheet/DB), rồi kiểm pagination đúng tại node đó — không suy luận từ
một node "nhìn có vẻ ổn" khác trong cùng workflow.

## Khi sửa — luôn verify bằng số, không chỉ bằng "chạy không lỗi"

"Workflow chạy xong, không error" **không phải bằng chứng** đã lấy đủ dữ liệu.
Verify bắt buộc:
1. Đo số record **trước fix** vs **sau fix** cho cùng khoảng thời gian — phải
   tăng lên (hoặc bằng nếu thật sự đã đủ).
2. Diff danh sách ID (ví dụ `ad_id`) giữa nguồn API và đích ghi (Sheet/DB) —
   diff phải bằng 0.
3. Nếu có thể, đo trực tiếp trong môi trường thật (container/server chạy
   workflow) để loại trừ sai khác môi trường — không chỉ test local.

## Tham chiếu

- Incident gốc: `AI_OS/incidents/2026-10-02_missing-ads-data.md`
- Task chi tiết: `Project/Module_02_Stabilize/Task_04_Fix_Missing_Ads_Pagination/`
- Bài học trong Task_03 (gốc, trước khi tách sang đây): `LESSONS_LEARNED.md` Bài học 4
