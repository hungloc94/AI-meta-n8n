# CASE-045: Credential Unavailable — Fix Via API

- Ngày phát hiện: 2026-07-26
- Ngày xác minh: 2026-07-26

## Vấn đề
Sau import workflow từ Windows sang Ubuntu, node báo
"Credential unavailable" — credential ID cũ không tồn tại trên server mới.
UI không cho phép save khi chọn lại credential.

## Nguyên nhân
Workflow JSON lưu credential theo ID cụ thể.
n8n mới tạo credential với ID khác — ID cũ không còn tồn tại.

## Cách xử lý
1. Dùng n8n API GET /credentials — lấy danh sách credential hiện có, tìm ID mới
2. Dùng n8n API PATCH workflow — update credential ID trong node
3. Verify bằng GET workflow JSON — xác nhận ID đã đổi đúng
Không dùng UI để fix — UI không cho save trong trường hợp này.

## Bài học
Sau mỗi lần import workflow sang n8n mới:
- Kiểm tra tất cả node có credential
- Nếu thấy "(unavailable)" → fix qua API ngay, không dùng UI
