# CASE-005: Convert tất cả Dates sang ISO Format

## Vấn đề
Date comparison không ổn định khi dùng DD/MM strings.

## Nguyên nhân
DD/MM strings không sortable, dễ nhầm tháng/ngày, và không compare được trực tiếp trong code.

## Cách xử lý
Convert tất cả dates sang ISO `YYYY-MM-DD` tại parser boundary. Downstream chỉ dùng ISO format.

## Bài học
- Parser phải handle local input formats (DD/MM, "hôm qua", "hôm nay") và year assumptions.
- Year rollover ambiguity chưa giải quyết hoàn toàn (edge case cuối năm).
- ISO dates stable, sortable, dễ filter theo range.

## Status
Accepted.
