# CASE-002: Normalize Layer ngay sau Google Sheet

## Vấn đề
Downstream KPI logic cần stable object schema, nhưng Google Sheets API trả raw array rows.

## Nguyên nhân
Nếu không normalize, mọi node downstream phải tự xử lý array → dễ gây schema drift và silent aggregation failures.

## Cách xử lý
Đặt node `Normalize Sheet Data` ngay sau mỗi lần đọc Google Sheet. Convert array rows thành canonical objects với field names chuẩn.

## Bài học
- Thêm 1 node nhưng ngăn schema drift lan xuống downstream.
- Mọi thay đổi schema chỉ cần sửa tại 1 điểm duy nhất.
- Đã verify trong end-to-end recovery.

## Status
Accepted and implemented.
