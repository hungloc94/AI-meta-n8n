# CASE-047: Token Copy Sai Khi Restore Bị Nhầm Tưởng Là Token Hỏng Ở Meta

- **Ngày phát hiện:** 2026-07-26
- **Ngày xác minh:** 2026-07-27

## Vấn đề
Sau khi restore `.env` sang Home Server (Task 06), `META_ACCESS_TOKEN` bị Meta Graph API từ chối: `401 - "The access token could not be decrypted" (code 190, OAuthException)`. Kết luận ban đầu (AI): "token đã hết hạn/bị Meta thu hồi, cần lấy token mới từ Meta console".

## Nguyên nhân
Kết luận ban đầu sai vì thiếu bước đối chiếu. Thực tế: workflow tương ứng trên production Windows chạy hoàn toàn chính xác **cùng ngày, cùng giờ** — chứng minh token thật vẫn sống bình thường phía Meta, không hề bị thu hồi. Nguyên nhân thật: giá trị `META_ACCESS_TOKEN` được **điền tay sai/copy nhầm** vào `.env` Home Server lúc Task 06 Bước 1 — lỗi nhập liệu khi restore, không phải sự cố ở Meta.

AI đã test bằng curl trực tiếp tới Graph API và xác nhận đúng token trong `.env` bị từ chối — nhưng dừng lại kết luận ở đó mà không đối chiếu với hệ thống nguồn (Windows) đang chạy tốt, dẫn tới hướng xử lý sai (đề xuất tạo token mới từ Meta, trong khi chỉ cần sửa lỗi copy).

## Cách xử lý
1. Khi 1 credential/token fail ngay sau restore/migrate, **đối chiếu với hệ thống nguồn (production) đang chạy tốt TRƯỚC KHI kết luận** credential/token tự nó hỏng.
2. Nếu production vẫn chạy đúng với "cùng" giá trị lẽ ra phải giống nhau → lỗi nằm ở bản copy, không phải bản gốc.
3. Lấy giá trị đúng trực tiếp từ nguồn (file `.env` plaintext trên máy nguồn — không phải qua n8n Credential UI vì n8n che giá trị đã lưu, không xem lại được).

## Bài học
- **Test kỹ thuật xác nhận "giá trị X sai" không đồng nghĩa với "nguyên nhân X sai là do X tự hỏng"** — luôn cần thêm bước đối chiếu nguồn trước khi kết luận nguyên nhân gốc, đặc biệt trong bối cảnh restore/migrate nơi lỗi nhập liệu là khả năng cao.
- Kết luận vội dẫn tới hướng xử lý sai lệch (đi xin token mới) tốn công hơn nhiều so với hướng đúng (chỉ copy lại đúng giá trị).
- Áp dụng cho mọi loại restore credential trong migration, không riêng Meta token.

## Status
Lesson learned — 2026-07-27.
