# CASE-049: `docker compose restart` Không Nạp Lại `.env` Mới

- **Ngày phát hiện:** 2026-07-27
- **Ngày xác minh:** 2026-07-27

## Vấn đề
Sau khi sửa `META_ACCESS_TOKEN` trong `.env` và chạy `docker compose restart`, container n8n khởi động lại thành công nhưng vẫn dùng **giá trị biến môi trường CŨ** — không phải giá trị mới vừa sửa trong `.env`. Test lại Meta API Health Check vẫn FAIL với token cũ, gửi thêm 1 Telegram alert thật không cần thiết.

## Nguyên nhân
`docker compose restart` chỉ khởi động lại **tiến trình bên trong container đã tồn tại** — không đọc lại file `.env` trên host. Biến môi trường của container được "đóng băng" từ lúc container được **tạo** (qua `up`/`create`), không phải từ lúc `restart`.

Xác nhận qua `docker exec n8n printenv META_ACCESS_TOKEN` khác với `.env` trên host sau khi restart.

## Cách xử lý
Khi sửa `.env` và cần container nạp giá trị mới, dùng:
```
docker compose up -d --force-recreate
```
(không dùng `docker compose restart`). Đã xác nhận: sau `up -d --force-recreate`, `docker exec n8n printenv <biến>` khớp đúng `.env` trên host.

## Bài học
- `restart` ≠ nạp lại cấu hình môi trường — chỉ dùng `restart` khi biến môi trường không đổi (ví dụ: chỉ cần container hết treo).
- Sau bất kỳ thay đổi nào trong `.env`, luôn xác minh lại bằng `docker exec <container> printenv <biến>` so với `.env` trên host **trước khi** kết luận việc cập nhật đã có hiệu lực — tránh test với giá trị cũ mà tưởng đang test giá trị mới.
- Áp dụng cho mọi lần sửa `.env` trong toàn bộ vòng đời Module 00 (và các Module sau).

## Status
Lesson learned — 2026-07-27.
