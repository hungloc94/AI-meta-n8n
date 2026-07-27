# WORKLOG — Task 08: Switch Production

## [2026-07-27] ⚠️

### Nhật ký
- PLAN.md đang trống → dựng lại dựa trên bối cảnh Human cung cấp + đối chiếu
  `Task_07/STATUS.md`, `Project/OPS/PRODUCTION_PROMOTION_CHECKLIST.md`,
  `Project/OPS/MIGRATION_RUNBOOK.md`, `Project/OPS/WORKFLOW_SCHEDULES.md`.
- Human đề xuất same-day smoke test thay vì chờ mù 24h — đã đưa vào PLAN.md.
- Thử `n8n execute --id=<id>` để smoke test → lỗi `Missing node to start execution`
  (CLI execute cần node Execute Workflow Trigger làm entry point, không chạy trực tiếp
  từ Schedule Trigger được — khớp với cách Task 06 Bước 6 đã từng làm bằng node tạm).
- Quyết định: không làm lại smoke test kiểu thêm-node-tạm cho 4 workflow buổi sáng vì
  Task 07 vừa test thật đầy đủ (real send) rất gần đây, chưa có gì đổi trên Home Server.
  Giờ thật hôm nay của 4 workflow đã qua (12:36) → activate thẳng an toàn tuyệt đối.
- Activate 4/5 workflow (Google Sheets Health Check, Meta API Health Check, Meta Ads
  Daily Sheet Update, Yesterday Report) qua `n8n update:workflow --active=true`.
- Phát hiện: `update:workflow --active=true` chỉ đổi DB, không có hiệu lực runtime khi
  n8n đang chạy (CLI tự cảnh báo). Đã `docker compose restart n8n` (không dùng
  `--force-recreate` vì không đổi `.env`, theo đúng bài học CASE-049) → verify lại bằng
  `n8n list:workflow --active=true`: đúng 4/4 workflow đã active thật trên runtime.
- "Today Report" chưa activate — chờ anh Lộc deactivate Windows trước 16:31 hôm nay để
  tránh gửi trùng Telegram (Windows và Home Server cùng có lịch 16:31 thật hôm nay).

- 13:52 — Anh Lộc xác nhận đã deactivate "Today Report" bên Windows → activate "Today
  Report" (id `VKZzSlAvsb4GiyMs`) bên Home Server, restart n8n, verify runtime active
  đúng 5/5 workflow (`n8n list:workflow --active=true`), container healthy.

- 13:55 — Anh Lộc xác nhận đã tắt hết 4 workflow còn lại bên Windows → Phase 1 hoàn
  thành, cả 5 workflow chỉ còn 1 nguồn chạy (Home Server). Bắt đầu tính Phase 2 (24–48h
  quan sát) từ mốc này.

### Signal
OPEN_TASKS:
- [x] Anh Lộc xác nhận đã deactivate 4 workflow buổi sáng bên Windows
      → Hoàn thành lúc: 2026-07-27 13:55
- [x] Anh Lộc deactivate "Today Report" bên Windows TRƯỚC 16:31 hôm nay → Claude Code
      activate "Today Report" bên Home Server ngay sau đó
      → Hoàn thành lúc: 2026-07-27 13:52
- [x] Quan sát lần chạy thật 16:31 hôm nay của Today Report — bằng chứng same-day
      → Hoàn thành lúc: 2026-07-27 ~16:31 — anh Lộc xác nhận tự động báo cáo, đúng nội dung
- [x] Quan sát lần chạy thật 21:13 hôm nay của Today Report
      → Hoàn thành lúc: 2026-07-27 ~21:13 — anh Lộc xác nhận tự động báo cáo, đúng nội dung
      (anh Lộc ghi "21:31", cron thật là 21:13 — chỉ lệch cách ghi giờ, không phải lỗi hệ thống)
- [ ] Quan sát chu kỳ sáng mai (07:00 → 08:13) — lần đầu 4 workflow buổi sáng chạy dưới
      schedule thật trên Home Server, không phải activate-tay giữa trưa
- [ ] Theo dõi đủ 24–48h theo PRODUCTION_PROMOTION_CHECKLIST Phase B, mốc bắt đầu:
      2026-07-27 13:55
      → Phát hiện khi: Phase 1 vừa hoàn thành

STALE_DOCS:
- [ ] `Project/STATUS.md` → cập nhật lần cuối 2026-07-03, vẫn ghi "Module 00 ⏳ Chưa bắt
      đầu" — đã lỗi thời so với thực tế (Task 07 xong, Task 08 đang triển khai)
      → Phát hiện khi: đọc context trước khi viết PLAN.md Task 08
- [ ] `Task_01_Audit_Windows_n8n/STATUS.md` vẫn "0% chưa bắt đầu" → chưa có inventory
      Windows chính thức, và `Task_02_Backup_Windows_n8n` cũng 0% → Task 08 không phụ
      thuộc 2 task này (rollback dựa vào Windows còn sống, không dựa vào file backup),
      nhưng đây là gap rộng hơn Module 00 nên chỉ ghi nhận, không xử lý ở Task 08
      → Phát hiện khi: pre-check rollback readiness

PROPOSAL:
- [ ] Chưa có — sẽ đề xuất cập nhật `Project/STATUS.md` sau khi Task 08 hoàn thành hẳn
      (gộp 1 lần, tránh sửa nhiều lần)
