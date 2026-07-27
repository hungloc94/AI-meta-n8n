# Plan — Task 08: Switch Production

Cutover tuần tự từng workflow theo đúng thứ tự dependency. Ưu tiên chứng minh Home Server
chạy đúng TRƯỚC khi tắt Windows (Windows là fallback cho tới khi có bằng chứng PASS) —
không tắt Windows rồi mới test.

- **Phase 1 (chiều/tối nay, sau 21:13):** Same-day smoke test cho từng workflow — biết kết
  quả trong vài giờ thay vì chờ qua đêm.
- **Phase 2 (24–48h sau khi Phase 1 xong):** Quan sát passive theo PRODUCTION_PROMOTION_CHECKLIST
  Phase B — anh Lộc theo dõi, Claude Code hỗ trợ kiểm tra khi cần.

Windows n8n **không gỡ cài đặt** trong suốt Phase 1 + Phase 2 — chỉ deactivate từng workflow
sau khi workflow tương ứng bên Home Server đã PASS smoke test. Nếu bất kỳ bước nào FAIL →
giữ nguyên Windows active cho workflow đó, không deactivate, dừng lại báo Human.

---

## Phase 1 — Same-day smoke test (theo đúng thứ tự dependency)

**Phương pháp smoke test (chốt sau khi kiểm tra thực tế server 2026-07-27 12:09 +07 —
máy này chính là Home Server, n8n container đang chạy, cả 7 workflow hiện đều inactive):**
Thử `n8n execute --id=<id>` trực tiếp → lỗi `Missing node to start execution` (CLI execute
cần node `Execute Workflow Trigger` làm entry point, không chạy được trực tiếp từ Schedule
Trigger — đúng như CASE ghi nhận ở Task 06 Bước 6, phải thêm node tạm mới chạy được qua CLI).

**Quyết định:** Không làm lại smoke test kiểu thêm-node-tạm cho 4 workflow buổi sáng
(07:00/07:05/07:30/08:13) — vì Task 07 vừa test thật đầy đủ end-to-end (gửi Telegram/ghi
Sheet thật) cách đây rất gần, chưa có gì thay đổi trên Home Server từ đó tới giờ. Thêm 1
lần nữa qua CLI (phải sửa JSON workflow tạm thời) chỉ tạo thêm rủi ro thao tác mà không có
thông tin mới. Giờ thật hôm nay của 4 workflow này đã qua (hiện 12:36) → activate ngay bây
giờ **an toàn tuyệt đối** (không có rủi ro trùng lệnh, lần chạy thật kế tiếp là 07:00 sáng
mai — nằm trong 24-48h quan sát anh Lộc đã nhận theo dõi). Riêng **Today Report** còn 2 lần
chạy thật hôm nay (16:31, 21:13) — đây chính là phép thử same-day thật không cần hack gì:
chỉ cần deactivate Windows xong TRƯỚC 16:31 rồi activate Home Server, lần chạy 16:31 sẽ tự
nhiên là bằng chứng thật trong chiều nay.

| Bước | Mô tả | Người thực hiện | Kết quả mong đợi |
|------|-------|------------------|-------------------|
| 1 | Pre-check: xác nhận Task 07 PASS, .env/credentials Home Server đúng cho 5 workflow | Claude Code | Đủ điều kiện GO theo PRODUCTION_PROMOTION_CHECKLIST Phase A |
| 2 | Activate "Google Sheets Health Check" (id `EWvw3tX4oV11xXbs`) trên Home Server (cron giữ nguyên 07:00) | Claude Code | Workflow active |
| 3 | Deactivate "Google Sheets Health Check" bên Windows | Anh Lộc (thủ công) | Windows không còn chạy 07:00 |
| 4 | Activate "Meta API Health Check" (id `sgzJXYTLQKk4hWp3`) trên Home Server (cron giữ nguyên 07:05) | Claude Code | Workflow active |
| 5 | Deactivate "Meta API Health Check" bên Windows | Anh Lộc (thủ công) | Windows không còn chạy 07:05 |
| 6 | Activate "Meta Ads Daily Sheet Update" (id `rz3Wya5lFay7ShVL`) trên Home Server (cron giữ nguyên 07:30) | Claude Code | Workflow active |
| 7 | Deactivate "Meta Ads Daily Sheet Update" bên Windows | Anh Lộc (thủ công) | Windows không còn chạy 07:30 |
| 8 | Activate "Yesterday Report" (id `9l8RHgYuRZYz0TVr`) trên Home Server (cron giữ nguyên 08:13) | Claude Code | Workflow active |
| 9 | Deactivate "Yesterday Report" bên Windows | Anh Lộc (thủ công) | Windows không còn chạy 08:13 |
| 10 | **Deactivate "Today Report" bên Windows TRƯỚC** — bắt buộc xong trước 16:31 chiều nay | Anh Lộc (thủ công) | Windows không còn chạy Today Report |
| 11 | Activate "Today Report" (id `VKZzSlAvsb4GiyMs`) trên Home Server (cron giữ nguyên 11:31/16:31/21:13) — chỉ sau khi bước 10 xong | Claude Code | Workflow active, lần chạy 16:31 hôm nay là bằng chứng same-day thật |
| 12 | Xác nhận "Meta Report VERIFIED" giữ inactive cả hai bên — không đổi | Claude Code | Đúng phạm vi đã chốt ở Task 07 |

## Phase 2 — Quan sát 24–48h

| Bước | Mô tả | Người thực hiện | Kết quả mong đợi |
|------|-------|------------------|-------------------|
| 18 | Theo dõi đủ 1–2 chu kỳ đầy đủ (07:00 → 21:13, lặp lại 1–2 ngày) theo PRODUCTION_PROMOTION_CHECKLIST Phase B: không trùng Telegram, không stale replay, không lỗi KPI, không crash, không delay | Anh Lộc (chính) + Claude Code (hỗ trợ check khi cần) | 24H/48H OBSERVATION CHECKS đều PASS |
| 19 | Human xác nhận GO — cập nhật `STATUS.md` Task 08 + `Module_00/STATUS.md` + `Project/STATUS.md`, Backup | Claude Code | Task 08 hoàn thành, Module 00 hoàn thành |

---

## Rollback

Nếu bất kỳ bước Phase 1 nào FAIL: không deactivate Windows cho workflow đó, giữ nguyên
Windows active, dừng lại báo Human trước khi tiếp tục các bước sau.

Nếu Phase 2 phát hiện lỗi: re-activate lại workflow tương ứng bên Windows, deactivate bên
Home Server — không xoá dữ liệu Home Server, giữ nguyên để debug.
