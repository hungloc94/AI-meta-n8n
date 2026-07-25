# CASE-032: Migration Definition of Done — 6 Bước Bắt Buộc

## Vấn đề
Migration bị đánh dấu "xong" quá sớm dựa vào cảm tính — "thấy không lỗi là kết luận xong". Hôm sau phát sinh incident vì bỏ sót bước.

## Nguyên nhân
Không có tiêu chí hoàn thành (Definition of Done) rõ ràng. Node PASS chỉ chứng minh node đó hoạt động, không chứng minh toàn bộ workflow hay scheduler thật hoạt động.

## Cách xử lý
Migration chỉ được đánh dấu COMPLETED khi đủ cả 6 bước:

```
☐ Bước 1: Inventory
  Liệt kê đầy đủ toàn bộ nơi cần migrate
  (node, workflow, health check, monitoring, env variable...)

☐ Bước 2: Migration
  Thực hiện migration cho tất cả item trong inventory

☐ Bước 3: Production Verify
  Execute manual toàn bộ workflow production
  Xác nhận dữ liệu thực tế đúng (không chỉ "không báo lỗi")

☐ Bước 4: Monitoring Update
  Cập nhật Health Check theo kiến trúc mới
  Cập nhật message alert — không còn nhắc credential cũ

☐ Bước 5: Scheduler Verify
  Chờ ít nhất 1 lần scheduler thật chạy thành công

☐ Bước 6: Documentation
  Cập nhật STATUS.md, ROADMAP.md
  Ghi Lesson Learned nếu có bài học mới
```

## Bài học
- Không tick COMPLETED khi chưa đủ cả 6 bước — dù chỉ thiếu 1 bước cũng có thể gây incident.
- Áp dụng cho mọi loại migration: credential, endpoint, schema, infrastructure.
- Optional sau DoD: archive workflow TEST, revoke credential cũ (chờ vài ngày quan sát trước).

## Status
Lesson learned — 2026-06-25.
