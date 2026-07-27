# Status — Task 07: Functional Testing

## Trạng thái hiện tại
- **Tiến độ:** ~17% (Bước 1/6 xong; Bước 2 FAIL — blocker credential, không phải lỗi workflow)
- **Bước đang làm:** Bước 2 — Test thủ công Meta API Health Check (07:05) — 🛑 DỪNG, chờ Human
- **Blocker:** `META_ACCESS_TOKEN` trong `.env` Home Server bị Meta từ chối (`401 - "could not be decrypted"`, code 190). **Đã đối chiếu với production Windows (Human xác nhận 2026-07-27): workflow tương ứng trên Windows chạy hoàn toàn chính xác cùng ngày** → token thật vẫn sống bình thường phía Meta. Vậy nguyên nhân là **giá trị token điền tay vào `.env` Home Server lúc Task 06 bị sai/copy nhầm**, không phải Meta thu hồi token. Cần: đối chiếu và copy đúng token đang chạy thật từ Windows sang `.env` Home Server — không cần tạo token mới từ Meta console.
- **Ảnh hưởng dây chuyền:** Bước 3 (Meta Ads Daily Sheet Update 07:30) và Bước 5 (Today Report — gọi Meta API trực tiếp theo CASE-017) chắc chắn cũng sẽ FAIL vì dùng chung `META_ACCESS_TOKEN`. Bước 4 (Yesterday Report) có thể bị ảnh hưởng gián tiếp nếu Bước 3 không cập nhật được Sheet mới (CASE-014).
- **Sự cố phát sinh:** Test Bước 2 đã khiến workflow đi nhánh FAIL và **gửi 1 tin nhắn Telegram thật** tới chat cá nhân anh Lộc ("🚨 META API HEALTH CHECK FAILED..."). Đây là hành vi thiết kế đúng của Health Check (CASE-042 — fail phải alert, không im lặng), không phải lỗi do AI, nhưng anh sẽ thấy tin nhắn này trên điện thoại.
- **Cập nhật lần cuối:** 2026-07-26 (Claude Code)

## Đã hoàn thành
- ✅ Bước 1 — Google Sheets Health Check (07:00): PASS — không lỗi, không alert Telegram. Chi tiết: `Task_06/WORKLOG.md` Bước 6.
- ⚠️ Bước 2 — Meta API Health Check (07:05): **FAIL đúng như thiết kế** (health check phát hiện đúng vấn đề thật — token hỏng), đã gửi Telegram Alert thật. Workflow đã được khôi phục nguyên trạng sau test, không có thay đổi vĩnh viễn nào.

## HANDOVER
- Người giao: Claude Code
- Người nhận: Human (anh Lộc)
- Đã làm xong: Bước 1 PASS; Bước 2 xác định đúng nguyên nhân gốc (token Meta hỏng) qua test trực tiếp Graph API, không qua workflow (tránh gửi thêm alert trùng)
- Cần làm tiếp: Anh lấy `META_ACCESS_TOKEN` mới từ Meta Business/Developer console → cập nhật vào `.env` → báo Claude Code retest Bước 2
- Xong khi nào: Bước 2 PASS với token mới, không alert Telegram
- Trạng thái: 🛑 Dừng — chờ Human cập nhật token Meta
