# Status — Task 07: Functional Testing

## Trạng thái hiện tại
- **Tiến độ:** 100% (6/6 bước xong — 5 workflow PASS, 1 workflow (Meta Report VERIFIED) xác nhận vẫn lỗi đúng như dự kiến)
- **Bước đang làm:** Đã xong toàn bộ PLAN.md — chờ Human xác nhận Task 07 hoàn thành trước khi chuyển Task 08
- **Blocker:** `META_ACCESS_TOKEN` trong `.env` Home Server bị Meta từ chối (`401 - "could not be decrypted"`, code 190). **Đã đối chiếu với production Windows (Human xác nhận 2026-07-27): workflow tương ứng trên Windows chạy hoàn toàn chính xác cùng ngày** → token thật vẫn sống bình thường phía Meta. Vậy nguyên nhân là **giá trị token điền tay vào `.env` Home Server lúc Task 06 bị sai/copy nhầm**, không phải Meta thu hồi token. Cần: đối chiếu và copy đúng token đang chạy thật từ Windows sang `.env` Home Server — không cần tạo token mới từ Meta console.
- **Ảnh hưởng dây chuyền:** Bước 3 (Meta Ads Daily Sheet Update 07:30) và Bước 5 (Today Report — gọi Meta API trực tiếp theo CASE-017) chắc chắn cũng sẽ FAIL vì dùng chung `META_ACCESS_TOKEN`. Bước 4 (Yesterday Report) có thể bị ảnh hưởng gián tiếp nếu Bước 3 không cập nhật được Sheet mới (CASE-014).
- **Sự cố phát sinh:** Test Bước 2 đã khiến workflow đi nhánh FAIL và **gửi 1 tin nhắn Telegram thật** tới chat cá nhân anh Lộc ("🚨 META API HEALTH CHECK FAILED..."). Đây là hành vi thiết kế đúng của Health Check (CASE-042 — fail phải alert, không im lặng), không phải lỗi do AI, nhưng anh sẽ thấy tin nhắn này trên điện thoại.
- **Cập nhật lần cuối:** 2026-07-26 (Claude Code)

## Đã hoàn thành
- ✅ Bước 1 — Google Sheets Health Check (07:00): PASS.
- ✅ Bước 2 — Meta API Health Check (07:05): PASS sau khi cập nhật token đúng (retest 2026-07-27).
- ✅ Bước 3 — Meta Ads Daily Sheet Update (07:30): PASS. Append 11 dòng mới (ngày 27/07, chưa từng có trước đó — không phải trùng lặp, đã đối chiếu snapshot trước/sau: 1042 dòng cũ giữ nguyên 100% kể cả cột chưa đặt tên (nghi là Nhóm B), 0 cặp Mã QC+Ngày bị trùng trong toàn sheet). Phát hiện phụ: node "Summary" (cuối chuỗi) bị lỗi `Referenced node is unexecuted` khi nhánh "IF: Update?" không có item nào chạy qua — bug tồn tại sẵn trong helper `safeCount` (chỉ bắt substring "hasn't been executed", nhưng lỗi n8n thực tế là "Referenced node is unexecuted") khiến workflow báo FAILED dù dữ liệu đã ghi đúng. Không ảnh hưởng kết quả ghi Sheet — chỉ ảnh hưởng node báo cáo cuối. Đề xuất ghi CASE, không fix (ngoài phạm vi Task 07).
- ✅ Bước 4 — Yesterday Report (08:13): PASS.
- ✅ Bước 5 — Today Report (11:31/16:31/21:13): PASS — báo cáo thật đã gửi (Chi tiêu 116.085đ, ngày 27/07/2026 06:08).
- ✅ Bước 6 — Meta Report VERIFIED: xác nhận vẫn lỗi (credential cũ không còn tồn tại). Không fix, không activate.

## Sự cố đã xử lý trong quá trình test
- Token `META_ACCESS_TOKEN` bị copy sai lúc Task 06 → đã xác định đúng nguyên nhân (đối chiếu với Windows production), Human cung cấp token đúng, Claude Code cập nhật `.env` + tạo lại credential "Header Auth account" (id mới `LK6jk3B1TtPIEiBH`, đã xoá id cũ hỏng `HAOqERh15wR0pB4r`) + cập nhật node "Lấy cấu hình Meta" trỏ đúng credential mới.
- Lỗi thao tác: `docker compose restart` không nạp `.env` mới (chỉ `up -d --force-recreate` mới nạp) → khiến 1 lần test Bước 2 chạy nhầm với token cũ, gửi thêm 1 Telegram alert không cần thiết. Đã sửa đúng cách và retest thành công.

## HANDOVER
- Người giao: Claude Code
- Người nhận: Human (anh Lộc)
- Đã làm xong: Toàn bộ 6/6 bước PLAN.md — 5 workflow PASS, Meta Report VERIFIED xác nhận vẫn lỗi (đúng dự kiến, không fix/activate).
- Cần làm tiếp: Anh xác nhận Task 07 hoàn thành → GO cho Task 08 (Switch Production), trừ Meta Report VERIFIED (README Task 07 đã ghi rõ). Cũng cần duyệt 2 đề xuất CASE mới (token copy nhầm; bug safeCount ở node Summary).
- Xong khi nào: Human xác nhận
- Trạng thái: ✅ Hoàn thành kỹ thuật — chờ Human xác nhận
