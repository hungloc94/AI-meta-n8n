# Status — Task 07: Functional Testing

## Trạng thái hiện tại
- **Tiến độ:** ~50% (Bước 1, 4, 6 xong; Bước 2/3/5 kẹt cùng 1 nguyên nhân — chờ Human đối chiếu token)
- **Bước đang làm:** Bước 2/3/5 — 🛑 DỪNG, chờ Human copy đúng token Meta từ Windows sang `.env` Home Server
- **Blocker:** `META_ACCESS_TOKEN` trong `.env` Home Server bị Meta từ chối (`401 - "could not be decrypted"`, code 190). **Đã đối chiếu với production Windows (Human xác nhận 2026-07-27): workflow tương ứng trên Windows chạy hoàn toàn chính xác cùng ngày** → token thật vẫn sống bình thường phía Meta. Vậy nguyên nhân là **giá trị token điền tay vào `.env` Home Server lúc Task 06 bị sai/copy nhầm**, không phải Meta thu hồi token. Cần: đối chiếu và copy đúng token đang chạy thật từ Windows sang `.env` Home Server — không cần tạo token mới từ Meta console.
- **Ảnh hưởng dây chuyền:** Bước 3 (Meta Ads Daily Sheet Update 07:30) và Bước 5 (Today Report — gọi Meta API trực tiếp theo CASE-017) chắc chắn cũng sẽ FAIL vì dùng chung `META_ACCESS_TOKEN`. Bước 4 (Yesterday Report) có thể bị ảnh hưởng gián tiếp nếu Bước 3 không cập nhật được Sheet mới (CASE-014).
- **Sự cố phát sinh:** Test Bước 2 đã khiến workflow đi nhánh FAIL và **gửi 1 tin nhắn Telegram thật** tới chat cá nhân anh Lộc ("🚨 META API HEALTH CHECK FAILED..."). Đây là hành vi thiết kế đúng của Health Check (CASE-042 — fail phải alert, không im lặng), không phải lỗi do AI, nhưng anh sẽ thấy tin nhắn này trên điện thoại.
- **Cập nhật lần cuối:** 2026-07-26 (Claude Code)

## Đã hoàn thành
- ✅ Bước 1 — Google Sheets Health Check (07:00): PASS — không lỗi, không alert Telegram.
- ⚠️ Bước 2 — Meta API Health Check (07:05): FAIL — phát hiện đúng vấn đề thật (token `.env` Home Server sai), đã gửi Telegram Alert thật. Nguyên nhân đã xác định lại đúng: lỗi copy token khi restore, không phải Meta thu hồi (xem WORKLOG).
- ✅ Bước 4 — Yesterday Report (08:13): PASS — chạy đúng thứ tự Đọc Sheet → Normalize → Tính KPI → Gửi Telegram, không lỗi. Báo cáo thật đã gửi (Chi tiêu 1.081.123đ, Mess 7, ngày 26/07/2026). Không phụ thuộc token Meta nên không bị chặn bởi Bước 2/3.
- ✅ Bước 6 — Meta Report VERIFIED: xác nhận vẫn lỗi — credential `googleSheetsOAuth2Api` (id `iGuF5SVznNPI7ihl`) mà node "Đọc Google Sheet" tham chiếu **không còn tồn tại** trong hệ thống. Không chạy thử trực tiếp (workflow có vòng polling Telegram thật — rủi ro cao hơn cần thiết). Không fix, không activate — đúng phạm vi Task 07.

## Còn lại
- ⏳ Bước 2, 3, 5 — kẹt chung 1 nguyên nhân: token Meta trong `.env` Home Server không khớp token thật đang chạy trên Windows. Cần Human đối chiếu và copy đúng giá trị.

## HANDOVER
- Người giao: Claude Code
- Người nhận: Human (anh Lộc)
- Đã làm xong: Bước 1, 4, 6 PASS/xác nhận. Bước 2 xác định đúng nguyên nhân gốc (lỗi copy token, không phải Meta thu hồi) sau khi đối chiếu với production Windows.
- Cần làm tiếp: Anh đối chiếu token Meta đang chạy thật trên Windows, copy chính xác sang `.env` Home Server (biến `META_ACCESS_TOKEN`) và credential "Header Auth account" trong n8n UI → báo Claude Code retest Bước 2/3/5
- Xong khi nào: Bước 2/3/5 PASS với token đúng
- Trạng thái: 🛑 Dừng — chờ Human copy đúng token
