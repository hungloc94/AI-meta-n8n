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

### Nhật ký (tiếp — 2026-07-27, sau khi có token đúng từ Human)
- Human cung cấp token Meta dài hạn đúng, yêu cầu Claude Code tự cập nhật toàn bộ.
- Cập nhật `.env` (biến `META_ACCESS_TOKEN`) trên Home Server.
- Vì n8n public API không cho sửa credential đã tạo (chỉ POST/DELETE, không PATCH/PUT) → tạo credential "Header Auth account" MỚI (id `LK6jk3B1TtPIEiBH`) với token đúng, cập nhật node "Lấy cấu hình Meta" (trong Daily Sheet Update) trỏ sang id mới, xoá credential cũ hỏng (id `HAOqERh15wR0pB4r`). Đã rescan toàn bộ 7 workflow xác nhận chỉ có 1 node dùng credential này trước khi xoá.
- Restart n8n bằng `docker compose restart` — **sai lệnh**, không nạp `.env` mới (container giữ nguyên biến môi trường cũ). Retest Bước 2 vẫn FAIL, gửi thêm 1 Telegram alert thật không cần thiết. Phát hiện qua `docker exec n8n printenv` khác với `.env` trên host → sửa bằng `docker compose up -d --force-recreate`, xác nhận khớp, retest lại PASS.
- **Bước 2 (Meta API Health Check) retest:** PASS — Schedule → Code Node → End, không lỗi, không alert.
- **Bước 3 (Meta Ads Daily Sheet Update 07:30):** rủi ro cao nhất (có ghi Sheet thật, phải bảo vệ Nhóm B theo CASE-011). Trước khi chạy: kiểm tra schema cột của node ghi (39-40 cột khai báo, không có cột nào thuộc Nhóm B) → an toàn về cấu trúc. Tạo 1 workflow tạm độc lập "TEMP Snapshot Reader" (Manual Trigger + Execute Workflow Trigger + HTTP Request đọc Sheet, không đụng workflow thật) để chụp snapshot TRƯỚC (1042 dòng). Chạy Daily Sheet Update thật qua CLI — chạy qua nhánh Schedule 07:30 thật, ghi thật 11 dòng mới (Append). Chụp snapshot SAU (1053 dòng). Đối chiếu: 1042 dòng cũ y hệt trước/sau (kể cả cột không tên nghi là Nhóm B), 11 dòng mới đều ngày 27/07 (0 dòng ngày này tồn tại trước đó — không phải trùng lặp), 0 cặp (Mã QC, Ngày) trùng trong toàn sheet → PASS, an toàn.
  - Phát hiện phụ (bug có sẵn, không phải do Claude Code gây ra): node "Summary" cuối chuỗi lỗi `Referenced node is unexecuted` vì nhánh "IF: Update?" không nhận item nào (chỉ "IF: Append?" chạy). Helper `safeCount()` trong node Code chỉ bắt lỗi có chứa chuỗi "hasn't been executed", nhưng n8n bản 1.103.2 trả về message "Referenced node is unexecuted" — không khớp, nên không được catch, khiến cả workflow báo execution FAILED dù dữ liệu đã ghi đúng. Rủi ro: có thể gây false alarm giám sát (workflow tưởng lỗi nhưng thực ra đã làm đúng việc).
  - Đã xoá workflow tạm "TEMP Snapshot Reader" sau khi dùng xong.
- **Bước 5 (Today Report 11:31/16:31/21:13):** PASS — toàn chuỗi chạy thành công, báo cáo thật đã gửi (Chi tiêu 116.085đ, 27/07/2026 06:08).
- Tất cả node tạm dùng để test đều đã xoá, đối chiếu khớp 100% với backup ở từng bước.
- **Task 07 hoàn thành 6/6 bước theo PLAN.md.**

### Signal
OPEN_TASKS:
- [x] Bước 1 — PASS
- [x] Bước 2 — PASS (sau khi cập nhật token đúng)
- [x] Bước 3 — PASS (đã verify an toàn Nhóm B bằng snapshot trước/sau)
- [x] Bước 4 — PASS
- [x] Bước 5 — PASS
- [x] Bước 6 — Xác nhận vẫn lỗi, không fix/activate
- [x] Task 07 — hoàn thành toàn bộ PLAN.md, chờ Human xác nhận

STALE_DOCS:
- [x] `Task_06/STATUS.md` → đã cập nhật, không còn lỗi thời
- [x] `Module_00/TASK_INDEX.md` → đã cập nhật khớp thực tế

PROPOSAL:
- [x] `TASK_INDEX.md` (Module 00) → đã cập nhật Task 06/07
      → Human duyệt và cập nhật lúc: 2026-07-26
- [ ] Tạo CASE mới #1: "Token copy nhầm khi restore bị nhầm tưởng là token hỏng ở Meta — phải đối chiếu production trước khi kết luận"
      → Chờ Human duyệt nội dung trước khi ghi vào `CASES/`
- [ ] Tạo CASE mới #2: "safeCount() không bắt được message lỗi thực tế của n8n 1.103.2 (Referenced node is unexecuted vs hasn't been executed) → workflow báo FAILED dù đã ghi Sheet đúng"
      → Chờ Human duyệt nội dung trước khi ghi vào `CASES/`
- [ ] Tạo CASE mới #3 (tuỳ chọn): "docker compose restart không nạp lại .env — phải dùng up -d --force-recreate"
      → Chờ Human quyết định có cần ghi thành CASE riêng hay gộp vào ghi chú vận hành
