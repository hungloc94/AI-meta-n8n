# CASE-004: Workflow phải Inactive khi Testing

## Vấn đề
Workflow active trong lúc test sẽ trigger Telegram polling, gửi duplicate messages, gây production side effects.

## Nguyên nhân
n8n workflow khi active sẽ tự động chạy trigger nodes (polling, cron). Trong giai đoạn forensic recovery, việc này tạo ra noise và risk.

## Cách xử lý
Imported workflow phải giữ `active=false`. Chỉ activate sau khi có explicit approval và đã verify node-by-node.

## Bài học
- Cần manual node-by-node testing riêng biệt.
- Supervised runtime activation cần quy trình riêng cho polling continuity validation.
- Mandatory rule trong forensic recovery.

## Status
Mandatory.
