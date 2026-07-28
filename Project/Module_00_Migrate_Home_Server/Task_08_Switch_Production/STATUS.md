# Status — Task 08: Switch Production

## Trạng thái hiện tại
- **Tiến độ:** ~90% (Phase 1 HOÀN THÀNH; Phase 2 đang chạy — chu kỳ sáng 2026-07-28 đã qua: 3/4 workflow success, 1/4 lỗi CASE-048 đã fix + verify lại thành công. Còn chờ 11:31/16:31/21:13 hôm nay + 1 chu kỳ sáng mai chạy sạch hoàn toàn để đủ điều kiện GO)
- **Bước đang làm:** Phase 2 — quan sát 24–48h theo PRODUCTION_PROMOTION_CHECKLIST Phase B. Mốc bắt đầu tính: 2026-07-27 13:55 (+07). Có 1 lần restart n8n chủ động lúc 2026-07-28 ~10:11 (+07) để fix CASE-048 — không phải crash, nhưng là gián đoạn runtime cần loại trừ khỏi tiêu chí "không crash" khi đánh giá. Mốc quan trọng tiếp theo: 11:31/16:31/21:13 hôm nay (Today Report), rồi 07:00–08:13 sáng mai (2026-07-29) chạy hoàn toàn tự động không có fix xen vào.
- **Cập nhật lần cuối:** 2026-07-28 10:49 (+07), Claude Code

## Đã hoàn thành (Phase 1 — HOÀN THÀNH)
- ✅ Pre-check: n8n container healthy, .env/credentials đã verify PASS ở Task 07
- ✅ Activate cả 5 workflow trên Home Server (Google Sheets Health Check 07:00, Meta API Health Check 07:05, Meta Ads Daily Sheet Update 07:30, Yesterday Report 08:13, Today Report 11:31/16:31/21:13) — cron giữ nguyên, không đổi
- ✅ Verify runtime active thật sau `docker compose restart n8n` (2 lần) — `n8n list:workflow --active=true` khớp đúng 5/5, container healthy cả 2 lần
- ✅ "Meta Report VERIFIED" và bản TEST giữ inactive — không đổi
- ✅ Anh Lộc xác nhận đã deactivate "Today Report" bên Windows trước 16:31 (2026-07-27 ~13:52)
- ✅ Anh Lộc xác nhận đã deactivate 4 workflow còn lại bên Windows (2026-07-27 13:55) — Phase 1 xong, mỗi workflow chỉ còn 1 nguồn chạy (Home Server)

## Đã hoàn thành (Phase 2 — đang chạy)
- ✅ "Today Report" 16:31 hôm nay — anh Lộc xác nhận tự động báo cáo đúng, không trùng, không thiếu
- ✅ "Today Report" 21:13 hôm nay — anh Lộc xác nhận tự động báo cáo đúng, không trùng, không thiếu (2/2 lần thật còn lại trong ngày đều PASS)

## Chưa hoàn thành (Phase 2)
- ⏳ Quan sát chu kỳ đầy đủ sáng mai (07:00 → 08:13) — lần đầu 4 workflow buổi sáng (Google Sheets Health Check, Meta API Health Check, Meta Ads Daily Sheet Update, Yesterday Report) chạy dưới schedule thật trên Home Server, không phải activate-tay giữa trưa
- ⏳ Theo dõi đủ 24–48h theo Phase B checklist: không trùng Telegram, không stale replay, không lỗi KPI, không crash, không delay
- ⏳ Sau 24–48h PASS → Human xác nhận GO → cập nhật `Module_00/STATUS.md` + `Project/STATUS.md` (đang stale từ 2026-07-03) → Backup

## HANDOVER
- Người giao: Claude Code
- Người nhận: Anh Lộc (theo dõi thụ động) + AI session tiếp theo nếu được gọi lại vào Task này
- Đã làm xong: Phase 1 xong toàn bộ; chu kỳ sáng 2026-07-28 đã qua (3/4 success, 1/4 lỗi
  CASE-048 đã fix + verify lại thành công qua execution log thật, không phải chỉ đoán)
- Cần làm tiếp:
  1. Theo dõi Today Report 11:31, 16:31, 21:13 hôm nay (2026-07-28) — xác nhận Telegram đúng
  2. Theo dõi chu kỳ sáng mai 07:00–08:13 (2026-07-29) — mốc quan trọng nhất còn lại,
     phải chạy hoàn toàn tự động, không có fix nào xen giữa chừng như hôm nay
  3. Sau đủ PASS → hỏi anh Lộc xác nhận GO → cập nhật STATUS.md này + Module_00/STATUS.md
     + Project/STATUS.md (đang stale từ 2026-07-03) → Backup
- Xong khi nào: Qua 24–48h không phát sinh lỗi/trùng lặp mới → Human xác nhận GO
- Trạng thái: 🔄 Đang làm — Phase 2 quan sát tiếp tục thụ động. **AI session hiện tại
  đã chuyển sang làm `Module_02_Stabilize/Task_01_System_User_Token` theo yêu cầu anh
  Lộc — Task 08 không cần AI action chủ động, chỉ cần kiểm tra khi được hỏi hoặc khi
  có sự cố mới.**
