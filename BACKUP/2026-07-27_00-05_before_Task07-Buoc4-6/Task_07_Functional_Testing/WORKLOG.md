# WORKLOG — Task 07: Functional Testing

## [2026-07-26] ⚠️

### Nhật ký
- Xác minh Task 06 thực tế (đối chiếu tài liệu vs trạng thái thật trên n8n): phát hiện `Task_06/STATUS.md` lỗi thời — credential Header Auth (Meta) và Google Service Account thực ra đã tồn tại, không còn là blocker.
- Test Bước 1 (Google Sheets Health Check 07:00): thêm tạm node Execute Workflow Trigger, chạy qua `n8n execute` CLI, PASS — không lỗi, không alert Telegram. Đã xoá node tạm, đối chiếu khớp 100% với backup trước khi sửa.
- Cập nhật `Task_06/STATUS.md` + `WORKLOG.md` và `Task_07/STATUS.md` phản ánh đúng tiến độ thật (đã backup trước khi sửa tại `BACKUP/2026-07-26_23-26_before_STATUS-WORKLOG-Task06-Task07/`).
- Cập nhật `Module_00/TASK_INDEX.md` theo yêu cầu Human (backup trước khi sửa).
- **Test Bước 2 (Meta API Health Check 07:05):** cùng phương pháp (node Execute Workflow Trigger tạm) → workflow đi nhánh **FAIL**, gửi **1 tin Telegram thật** tới chat Human ("META API HEALTH CHECK FAILED"). Đã xoá node tạm, đối chiếu khớp 100% với backup.
- Điều tra nguyên nhân gốc: gọi trực tiếp Meta Graph API bằng curl (dùng `META_ACCESS_TOKEN` từ `.env`, không qua workflow — tránh gửi thêm alert trùng) → nhận `401 "The access token could not be decrypted" (code 190, OAuthException)`. Kiểm tra định dạng token (độ dài, khoảng trắng, ký tự lạ) — không thấy bất thường ở phía chuỗi, lỗi nằm ở phía Meta từ chối token.
- Đây là blocker credential thật, ngoài phạm vi Task 07 tự xử lý — dừng lại, không thử tiếp Bước 3/5 (sẽ FAIL tương tự vì dùng chung token, tránh gửi thêm Telegram alert trùng không cần thiết).
- **[SỬA LẠI kết luận — 2026-07-27, theo phản hồi Human]:** Kết luận ban đầu "token đã chết ở Meta" là **vội, thiếu bước đối chiếu**. Human đã kiểm tra: workflow tương ứng trên máy Windows (production) chạy ra kết quả hoàn toàn chính xác cùng ngày → chứng minh token thật đang hoạt động bình thường ở phía Meta, KHÔNG bị Meta thu hồi. Vậy nguyên nhân đúng nhiều khả năng là: **giá trị `META_ACCESS_TOKEN` được điền tay vào `.env` Home Server ở Task 06 Bước 1 bị sai/copy nhầm** (không phải bản token đang chạy thật trên Windows) — lỗi nhập liệu khi restore, không phải sự cố Meta. Hướng xử lý đúng: đối chiếu và copy chính xác token đang chạy thật từ máy Windows sang `.env` Home Server, KHÔNG cần tạo token mới từ Meta console.
- **Bài học quy trình:** khi 1 credential fail sau restore/migrate, phải đối chiếu với hệ thống nguồn (production) đang chạy tốt TRƯỚC khi kết luận credential/token tự nó hỏng — tránh kết luận sai hướng khiến xử lý sai (đi xin token mới trong khi chỉ cần sửa lỗi copy).

### Signal
OPEN_TASKS:
- [ ] **BLOCKER — Bước 2 Meta API Health Check FAIL:** `META_ACCESS_TOKEN` bị Meta từ chối ("could not be decrypted", code 190). Cần Human lấy token mới từ Meta Business/Developer console, cập nhật `.env`, rồi mới retest được.
      → Phát hiện khi: test Bước 2, 2026-07-26
      → Ảnh hưởng: chặn Bước 2, 3, 5 (cùng dùng token này); có thể ảnh hưởng gián tiếp Bước 4
- [ ] Bước 3 — Test thủ công Meta Ads Daily Sheet Update (07:30) — chờ token mới trước khi thử
- [ ] Bước 4 — Test thủ công Yesterday Report (08:13)
- [ ] Bước 3 — Test thủ công Meta Ads Daily Sheet Update (07:30)
- [ ] Bước 4 — Test thủ công Yesterday Report (08:13)
- [ ] Bước 5 — Test thủ công Today Report (11:31 / 16:31 / 21:13)
- [ ] Bước 6 — Xác nhận Meta Report VERIFIED vẫn lỗi, ghi WORKLOG, không fix/activate

STALE_DOCS:
- [x] `Task_06/STATUS.md` → đã cập nhật, không còn lỗi thời
      → Phát hiện khi: đối chiếu Task 06 trước khi bắt đầu Task 07
- [ ] Module_00 `TASK_INDEX.md` → vẫn ghi Task 06/07 "⏳ Chưa bắt đầu", không khớp thực tế (Task 06 ~85%, Task 07 đang làm)
      → Phát hiện khi: đọc TASK_INDEX.md lúc scout ban đầu
      → Chưa sửa: chưa hỏi Human, để dành đề xuất riêng

PROPOSAL:
- [x] `TASK_INDEX.md` (Module 00) → đã cập nhật Task 06 = 🔄 Đang làm (~85%), Task 07 = 🔄 Đang làm (~17%)
      → Human duyệt và cập nhật lúc: 2026-07-26 (cùng phiên)
