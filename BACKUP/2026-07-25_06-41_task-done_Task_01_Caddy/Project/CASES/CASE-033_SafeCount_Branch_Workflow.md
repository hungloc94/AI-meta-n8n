# CASE-033: safeCount Helper cho Branch Workflow

## Vấn đề
Summary node throw error khi gọi `$('IF: Update?')` — node đó chưa execute trong context hiện tại.

## Nguyên nhân
Workflow có 2 nhánh song song (Append + Update) hội tụ vào Summary node. Không phải lúc nào cả 2 nhánh đều execute — nếu data match Key → chỉ Update chạy, Append không chạy (và ngược lại).

`$('NodeName')` trong n8n sẽ throw nếu node chưa execute trong execution context.

## Cách xử lý
Dùng `safeCount` helper với try/catch:

```javascript
function safeCount(nodeName) {
  try { return $items(nodeName).length; } catch(e) { return 0; }
}
```

Chỉ catch lỗi `hasn't been executed` — không dùng catch trống để tránh che lỗi thật.

## Bài học
- Summary node phải defensive với cả 2 khả năng: nhánh chạy hoặc không chạy.
- Verify idempotent trước khi activate sync: chạy 2 lần liên tiếp, lần 2 phải Append=0.
- Nếu lần 2 vẫn có Append → có bug tạo trùng.

## Status
Lesson learned — 2026-06-09.
