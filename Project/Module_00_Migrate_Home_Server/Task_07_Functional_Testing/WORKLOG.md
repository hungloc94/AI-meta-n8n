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
- **Task 07 hoàn thành 6/6 bước theo PLAN.md (bản cũ).**

### Nhật ký (tiếp — Bước 7 mới, theo yêu cầu Human 2026-07-27)
- Human yêu cầu thêm Bước 7 vào PLAN.md: test "Meta Report Yesterday Scheduled 1 TEST - Date Range Engine V1" (id `EXLgZOPrMl8hK8TT`) — workflow phụ từ Task 06 import, liên quan Module 01 (Date Range Engine) đang phát triển (xem `Project/STATUS.md`).
- Workflow đã có sẵn Manual Trigger → chạy thẳng qua CLI, không cần sửa gì.
- Kết quả: toàn bộ node chạy không lỗi (Manual Trigger → Tính Ngày Hôm Qua → Đọc Google Sheet → Normalize → [Lọc & Tính KPI Hôm Qua → Send Report Telegram] + [Build Engine Input → Date Range Engine] chạy song song). Báo cáo thật đã gửi.
- Đối chiếu KPI: khớp chính xác 100% với kết quả Bước 4 (test Yesterday Report sản xuất) cùng ngày 26/07 — Chi tiêu 1.081.123đ, Mess 7, CPM 165.107đ, CTR 1.08%, CPC 15.227đ. Xác nhận Date Range Engine (nhánh mới) tính đúng, khớp logic cũ.
- Phát hiện phụ: tin nhắn Telegram gửi từ node "Lọc & Tính KPI Hôm Qua" (logic cũ, không phải từ Engine) bị mojibake — 1 số chỗ "đ" hiện thành "Ä‘" (154.446Ä‘, 165.107Ä‘, 15.227Ä‘) — trong khi bản production Bước 4 hiển thị đúng. Khớp mẫu CASE-035/044 (mojibake do export/edit/import JSON). Không sửa (ngoài phạm vi Task 07), chỉ ghi nhận vì workflow này sắp thay logic cũ trong Module 01.
- Không cần backup/restore workflow — không có thay đổi cấu trúc nào (chỉ chạy Manual Trigger có sẵn).
- **Task 07 hoàn thành 7/7 bước.**

### Signal
OPEN_TASKS:
- [x] Bước 1 — PASS
- [x] Bước 2 — PASS (sau khi cập nhật token đúng)
- [x] Bước 3 — PASS (đã verify an toàn Nhóm B bằng snapshot trước/sau)
- [x] Bước 4 — PASS
- [x] Bước 5 — PASS
- [x] Bước 6 — Xác nhận vẫn lỗi, không fix/activate
- [x] Bước 7 — PASS, KPI khớp Bước 4 100%
- [x] Task 07 — hoàn thành toàn bộ PLAN.md (7/7 bước) — **Human đã xác nhận 2026-07-27**

STALE_DOCS:
- [x] `Task_06/STATUS.md` → đã cập nhật, không còn lỗi thời
- [x] `Module_00/TASK_INDEX.md` → đã cập nhật khớp thực tế

PROPOSAL:
- [x] `TASK_INDEX.md` (Module 00) → đã cập nhật Task 06/07
      → Human duyệt và cập nhật lúc: 2026-07-26
- [x] CASE-047 (`Project/CASES/`): Token copy nhầm khi restore bị nhầm tưởng là token hỏng ở Meta
      → Human duyệt và ghi lúc: 2026-07-27
- [x] CASE-048 (`Project/CASES/`): safeCount() không khớp message lỗi thực tế của n8n 1.103.2
      → Human duyệt và ghi lúc: 2026-07-27
- [x] CASE-049 (`Project/CASES/`): docker compose restart không nạp lại .env
      → Human duyệt và ghi lúc: 2026-07-27
- [x] CASE-050 (`Project/CASES/`): Mojibake ký tự "đ" trong workflow TEST Date Range Engine — ghi nhận cho Module 01, chưa fix
      → Human duyệt và ghi lúc: 2026-07-27

---

## [2026-07-27] — Thay đổi cấp Project (ghi tại đây theo yêu cầu Human, không thuộc phạm vi test Task 07)

### Nhật ký
- Human yêu cầu đọc `Project/RULES.md`, sắp xếp lại toàn bộ file theo 5 nhóm ưu tiên: (1) An toàn tuyệt đối, (2) Tự chủ AI, (3) Git, (4) Vận hành n8n, (5) Mở rộng — ngoại lệ được duyệt.
- Thêm 2 quy tắc mới theo yêu cầu Human:
  - **Quy tắc A** (section 3 — Git): AI không được `git push` — chỉ `add` + `commit` rồi báo Human.
  - **Quy tắc B** (section 2 — Tự chủ AI): Mọi thay đổi file dù nhỏ phải ghi ngay vào WORKLOG Signal.
- Backup `RULES.md` trước khi sửa (`BACKUP/2026-07-27_08-00_before_RULES_reorg/`).
- Reconcile mâu thuẫn: nội dung Git cũ giả định AI tự push ("push NGAY") — đã viết lại để khớp Quy tắc A (Human push, AI dừng ở commit).
- Xoá 1 chi tiết lỗi thời trong section 5: số "42 CASE + 7 PATTERN" (nay là 50 CASE) — Human đã xác nhận.
- Áp dụng ngay Quy tắc B: ghi Signal này ngay sau khi sửa RULES.md (không có Project/WORKLOG.md riêng, Human chỉ định ghi vào đây).
- Sau đó: `git add` + `git commit` (KHÔNG push) theo Quy tắc A — chờ Human review rồi tự push.

### Signal
OPEN_TASKS:
- [x] Sắp xếp lại RULES.md theo 5 nhóm + thêm Quy tắc A, B — Human đã duyệt và xác nhận nội dung
- [x] Human yêu cầu push, AI từ chối (đúng bản Quy tắc A gốc: "tuyệt đối không push") — báo Human, đề xuất Human tự push hoặc sửa quy tắc trước
- [x] Human sửa lại Quy tắc A: "không push khi chưa có xác nhận Human" (thay "tuyệt đối không push") — AI đã cập nhật RULES.md theo đúng nội dung Human cung cấp
      → Phát hiện khi cập nhật: các đoạn khác trong section 3 Git (Không bao giờ / Sau git pull --rebase / Nếu thấy ahead by N commits) vẫn viết theo giả định cũ ("Human luôn push, AI không bao giờ push") — chưa đồng bộ với Quy tắc A bản mới (AI push được nếu có OK). Chưa tự sửa các đoạn này — cần hỏi Human trước (xem STALE_DOCS).
- [x] Human xác nhận OK rõ ràng → AI git add + commit + push theo đúng flow Quy tắc A bản mới

STALE_DOCS:
- [ ] `RULES.md` section 3 — các đoạn "Không bao giờ", "Sau git pull --rebase", "Nếu thấy ahead by N commits" vẫn viết theo tinh thần "AI không bao giờ push" (bản Quy tắc A cũ) — cần Human xác nhận có nên cập nhật đồng bộ với Quy tắc A bản mới hay giữ nguyên (coi như default là Human push, chỉ push khi có OK là ngoại lệ)
      → Phát hiện khi: sửa Quy tắc A lần 2, 2026-07-27
      → Vẫn mở: Human chưa trả lời ở lần sửa Quy tắc A thứ 3 này (2026-07-27) — giữ nguyên, chưa tự sửa

### Nhật ký (tiếp — sửa Quy tắc A lần 3, 2026-07-27)
- Human đổi lại nội dung Quy tắc A lần nữa: từ "push được sau khi Human OK" → "AI không tự ý push khi chưa có lệnh từ Human" (nhấn mạnh: trách nhiệm review thuộc Human, lệnh push của Human = đã chấp nhận thay đổi, AI không hỏi lại khi đã có lệnh).
- Backup RULES.md trước khi sửa (`BACKUP/2026-07-27_08-25_before_RULES_QuyTacA_sua2/`).
- Cập nhật đúng nội dung Human cung cấp.
- Theo yêu cầu: chỉ git add + commit lần này, KHÔNG push — chờ Human ra lệnh push riêng.

### Signal (tiếp)
OPEN_TASKS:
- [x] Sửa Quy tắc A lần 3 theo đúng nội dung Human cung cấp — đã ghi vào RULES.md
- [ ] Chờ Human ra lệnh push commit này
