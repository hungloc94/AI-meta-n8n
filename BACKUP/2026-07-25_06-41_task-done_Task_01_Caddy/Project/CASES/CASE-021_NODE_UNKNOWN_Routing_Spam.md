# CASE-021: [NODE=UNKNOWN] Routing Spam — Double Consumer Chaos

## Vấn đề
Telegram nhận duplicate reports, `[NODE=UNKNOWN]` messages spam, và routing behavior không nhất quán. Valid report và unknown branch xuất hiện cùng lúc cho cùng một command.

## Nguyên nhân
Hai workflow cùng active và poll cùng một Telegram update queue:
- `Meta Report lỗi`
- `Meta Report TEST new`

Tạo ra double consumer chaos — mỗi message bị xử lý bởi cả 2 workflow với routing logic khác nhau.

## Cách xử lý
- Isolate 1 workflow stable duy nhất: `Meta Report VERIFIED`
- Deactivate toàn bộ workflow deprecated
- Verify single-consumer state: chỉ 1 workflow active poll Telegram

## Bài học
- Một Telegram queue chỉ được có đúng 1 active polling consumer.
- Topology bugs trong event-driven systems khó hơn code bugs — triệu chứng rất misleading.
- Runtime UI workflow state có thể drift khỏi exported verified artifacts.
- Khi thấy duplicate/spam → kiểm tra số lượng consumer trước khi nghi ngờ parser.

## Status
Resolved — 2026-05-29.
