# CASE-008: Validate Polling dưới Real Runtime Activation

## Vấn đề
Execute Step testing không đủ validate polling continuity.

## Nguyên nhân
staticData persistence behave khác trong real workflow activation vs Execute Step. Execute Step không lưu offset vào staticData → không thể verify duplicate-processing protection.

## Cách xử lý
Validate bằng supervised real runtime activation: activate workflow → gửi command thật → verify chỉ nhận đúng 1 response, không có stale replay.

## Bài học
- Cần tighter operational controls (chỉ 1 consumer, drain backlog trước).
- Evidence tốt hơn để chống stale replay, duplicate processing, và offset drift.
- Sau test xong phải verify hardcoded offset đã được xóa.

## Status
Accepted and verified for polling continuity.
