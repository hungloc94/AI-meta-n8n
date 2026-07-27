# CASE-048: safeCount() Không Khớp Message Lỗi Thực Tế Của n8n 1.103.2

- **Ngày phát hiện:** 2026-07-27
- **Ngày xác minh:** 2026-07-27

## Vấn đề
Node "Summary" trong workflow "Meta Ads Daily Sheet Update 7:30" báo execution **FAILED** với lỗi `Referenced node is unexecuted` khi nhánh "IF: Update?" không nhận item nào (chỉ nhánh "IF: Append?" chạy) — mặc dù dữ liệu đã được ghi đúng vào Google Sheet trước đó (Append thành công).

## Nguyên nhân
Helper `safeCount()` (theo đúng khuyến nghị [[CASE-033]] — chỉ catch lỗi cụ thể, không catch trống) được viết để bắt lỗi có chứa chuỗi con `"hasn't been executed"`:

```javascript
function safeCount(nodeName) {
  try {
    return $(nodeName).all().length;
  } catch (error) {
    if (String(error?.message || error).includes("hasn't been executed")) {
      return 0;
    }
    throw error;
  }
}
```

Nhưng n8n bản **1.103.2** (chạy trên Home Server) trả về message thực tế là `"Referenced node is unexecuted"` — không chứa chuỗi con `"hasn't been executed"` — nên điều kiện catch không khớp, lỗi bị `throw` lại thay vì trả về `0`, khiến cả node và cả workflow báo FAILED.

Khả năng cao: message lỗi của n8n đã đổi cách diễn đạt giữa các version (CASE-033 được ghi nhận 2026-06-09 trên version cũ hơn) — code phòng thủ dựa vào chuỗi lỗi cụ thể (string matching) dễ vỡ khi nâng cấp n8n.

## Cách xử lý
Chưa fix trong phạm vi Task 07 (ngoài phạm vi — chỉ test, không sửa logic workflow khác). Đề xuất hướng fix cho lần sau:
- Không dựa vào so khớp chuỗi message lỗi cụ thể — dùng cách kiểm tra không phụ thuộc version, ví dụ kiểm tra qua `error.name` / `error.constructor.name`, hoặc dùng `$(nodeName).isExecuted` nếu n8n hỗ trợ, hoặc bọc rộng hơn: catch mọi lỗi liên quan tới "chưa chạy" bằng cách kiểm tra nhiều chuỗi con khả dĩ (`["hasn't been executed", "is unexecuted", "not been executed"]`).
- Sau khi nâng cấp n8n version, phải test lại các node dùng string-matching trên error message.

## Bài học
- Code phòng thủ dựa vào so khớp chuỗi (string matching) message lỗi của framework/thư viện bên thứ 3 là mong manh — message có thể đổi giữa các version mà không cảnh báo.
- Node "Summary" báo FAILED không đồng nghĩa dữ liệu ghi sai — đã verify bằng snapshot trước/sau: 11 dòng Append đúng, không trùng lặp, dữ liệu cũ không đổi. Đây là lỗi ở tầng báo cáo/log, không phải lỗi ghi dữ liệu — nhưng vẫn nguy hiểm vì gây false alarm giám sát (tưởng lỗi thật, thực ra việc đã xong đúng).
- Liên quan: [[CASE-033]] (nguồn gốc pattern), [[PATTERN-005]] (branch summary safe count).

## Status
Lesson learned — chưa fix, chờ Human quyết định phạm vi xử lý (ngoài Task 07).
