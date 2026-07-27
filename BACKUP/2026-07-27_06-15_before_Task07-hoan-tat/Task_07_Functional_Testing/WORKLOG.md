# WORKLOG — Task 07: Functional Testing

## [2026-07-26 → 2026-07-27] ⚠️

### Nhật ký
- Xác minh Task 06 thực tế (đối chiếu tài liệu vs trạng thái thật trên n8n): phát hiện `Task_06/STATUS.md` lỗi thời — credential Header Auth (Meta) và Google Service Account thực ra đã tồn tại, không còn là blocker.
- **Bước 1 (Google Sheets Health Check 07:00):** thêm tạm node Execute Workflow Trigger, chạy qua `n8n execute` CLI, PASS — không lỗi, không alert Telegram. Đã xoá node tạm, đối chiếu khớp 100% với backup trước khi sửa.
- Cập nhật `Task_06/STATUS.md` + `WORKLOG.md`, `Task_07/STATUS.md`, `Module_00/TASK_INDEX.md` phản ánh đúng tiến độ thật (đã backup trước khi sửa).
- **Bước 2 (Meta API Health Check 07:05):** cùng phương pháp → workflow đi nhánh **FAIL**, gửi **1 tin Telegram thật** tới chat Human ("META API HEALTH CHECK FAILED"). Đã xoá node tạm, đối chiếu khớp 100% với backup.
- Điều tra nguyên nhân: gọi trực tiếp Meta Graph API bằng curl (dùng `META_ACCESS_TOKEN` từ `.env`, không qua workflow — tránh gửi thêm alert trùng) → nhận `401 "The access token could not be decrypted" (code 190, OAuthException)`.
- **[SỬA LẠI kết luận — 2026-07-27, theo phản hồi Human]:** Kết luận ban đầu "token đã chết ở Meta" là **vội, thiếu bước đối chiếu**. Human kiểm tra: workflow tương ứng trên Windows (production) chạy ra kết quả hoàn toàn chính xác cùng ngày → token thật vẫn sống bình thường phía Meta, KHÔNG bị thu hồi. Nguyên nhân đúng: **giá trị `META_ACCESS_TOKEN` điền tay vào `.env` Home Server ở Task 06 Bước 1 bị sai/copy nhầm** — lỗi nhập liệu khi restore, không phải sự cố Meta. Hướng xử lý đúng: đối chiếu và copy chính xác token đang chạy thật từ Windows sang `.env` Home Server, KHÔNG cần tạo token mới từ Meta console.
- **Bài học quy trình:** khi 1 credential fail sau restore/migrate, phải đối chiếu với hệ thống nguồn (production) đang chạy tốt TRƯỚC khi kết luận credential/token tự nó hỏng — tránh kết luận sai hướng khiến xử lý sai (đi xin token mới trong khi chỉ cần sửa lỗi copy).
- **Bước 4 (Yesterday Report 08:13):** không phụ thuộc Meta API trực tiếp nên test được ngay dù Bước 2/3 đang kẹt. Thêm node Execute Workflow Trigger tạm, chạy CLI → PASS toàn chuỗi (Đọc Sheet → Normalize → Tính KPI → Gửi Telegram), không lỗi. Báo cáo thật đã gửi (Chi tiêu 1.081.123đ, Mess 7, ngày 26/07/2026 — số liệu không toàn 0 nên không phải lỗi pipeline câm lặng, CASE-038). Xoá node tạm, đối chiếu khớp 100% với backup.
  - Giới hạn: chưa đối chiếu số liệu byte-by-byte với Google Sheet thật (không có đường đọc Sheet độc lập ngoài credential trong workflow) — nếu cần chắc chắn 100%, Human tự đối chiếu dòng 26/07 trong Sheet.
- **Bước 6 (Meta Report VERIFIED):** không chạy thử trực tiếp (workflow có vòng polling Telegram thật — ACK update, Send Menu... rủi ro cao hơn cần thiết). Xác nhận qua bằng chứng cấu hình: credential `googleSheetsOAuth2Api` (id `iGuF5SVznNPI7ihl`) mà node "Đọc Google Sheet" tham chiếu không còn tồn tại (`n8n export:credentials --all`) → xác nhận vẫn lỗi đúng như README ghi nhận. Không fix, không activate.

### Signal
OPEN_TASKS:
- [x] Bước 1 — PASS
- [x] Bước 2 — FAIL (đúng, phát hiện lỗi copy token thật) — không phải bug workflow
- [ ] **BLOCKER — Bước 3, Bước 5:** chờ Human đối chiếu và copy đúng `META_ACCESS_TOKEN` từ Windows sang `.env` Home Server + credential "Header Auth account" trong n8n UI, rồi mới retest được
      → Phát hiện khi: test Bước 2, 2026-07-26; xác nhận nguyên nhân 2026-07-27
- [x] Bước 4 — PASS
- [x] Bước 6 — Xác nhận vẫn lỗi, không fix/activate

STALE_DOCS:
- [x] `Task_06/STATUS.md` → đã cập nhật, không còn lỗi thời
- [x] `Module_00/TASK_INDEX.md` → đã cập nhật khớp thực tế (Task 06 ~85%, Task 07 đang làm)

PROPOSAL:
- [x] `TASK_INDEX.md` (Module 00) → đã cập nhật Task 06 = 🔄 Đang làm (~85%), Task 07 = 🔄 Đang làm (~50%)
      → Human duyệt và cập nhật lúc: 2026-07-26
- [ ] Tạo CASE mới ghi lại sự cố "token copy nhầm khi restore bị nhầm là token hỏng ở Meta" — đề xuất, chờ Human duyệt nội dung trước khi ghi vào `CASES/`
