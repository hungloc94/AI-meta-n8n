# CASE-031: Hai Nguồn Meta Token Trong Một Workflow

## Vấn đề
Đổi Meta token ở 1 chỗ nhưng workflow vẫn lỗi. Phát hiện workflow đang dùng 2 nguồn token khác nhau.

## Nguyên nhân
- Node "Lấy cấu hình Meta" dùng **n8n credential UI** (Header Auth)
- Node "Lấy dữ liệu Meta" dùng **`$env.META_ACCESS_TOKEN`** (file `.env` Docker)

Hai nguồn này độc lập nhau. Cập nhật 1 nguồn không ảnh hưởng nguồn kia.

## Cách xử lý
Khi đổi Meta token, phải kiểm tra workflow đang dùng nguồn nào:
1. Cập nhật n8n credential trong UI (nếu workflow dùng)
2. Cập nhật `META_ACCESS_TOKEN` trong file `.env` (nếu workflow dùng)
3. Restart Docker container sau khi sửa `.env`

## Bài học
- Không giả định toàn bộ workflow dùng cùng 1 nguồn token.
- Luôn kiểm tra trực tiếp từng node hoặc code node để biết đang dùng nguồn nào.
- Khi renew token: verify cả 2 nguồn, không chỉ 1.

## Status
Lesson learned — 2026-06-22.
