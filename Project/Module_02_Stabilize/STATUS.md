# Status — Module 02: Stabilize

## Trạng thái hiện tại
- **Tiến độ:** ~30% — Task 01 chờ deploy token; Task 03 đã có PLAN PHASE 0–5 + backup, đang thực thi.
- **Bước đang làm:** Task 03 — PHASE 0 (tắt `retryOnFail` node append) chờ anh Lộc CONFIRM; đang dựng bản fix upsert (PHASE 1) ở scratchpad.
- **Cập nhật lần cuối:** 2026-09-08 (+07), Claude Code

## Tiến độ Task

| Task | Trạng thái |
|------|-----------|
| Task 01 — System User Token vĩnh viễn | 🔄 Đang làm (70%, chờ deploy — xem `Task_01/CURRENT_STATUS.md`) |
| Task 02 — Revoke Telegram Token cũ | ⏳ Chưa bắt đầu |
| Task 03 — Fix Duplicate Row (Daily Sheet Update) | 🔄 Đang làm — PLAN PHASE 0–5 điền xong, PHASE 0 chờ CONFIRM (xem `Task_03/CURRENT_STATUS.md`) |

## HANDOVER
- Người giao: Claude Code
- Người nhận: AI session tiếp theo (anh Lộc sẽ mở CLI mới tối nay hoặc mai)
- Đã làm xong: Chưa có gì trong Module này — chỉ mới xác nhận đây là việc tiếp theo cần làm
- Cần làm tiếp: Bắt đầu Task 01 theo đúng quy trình AI OS — Brainstorm trước (chưa có
  README/PLAN cho Task 01), không nhảy thẳng vào code/thao tác credential
- Xong khi nào: Task 01 xong (System User Token vĩnh viễn hoạt động, verify) rồi tới Task 02
- Trạng thái: ⏳ Sắp bắt đầu — chờ Brainstorm

## Lưu ý
- Module 02 độc lập về mặt kỹ thuật với `Module_00/Task_08` (đang Phase 2 quan sát thụ
  động) — không xung đột tài nguyên, có thể làm song song.
- Trước khi động vào bất kỳ credential nào, đọc `Project/OPS/CREDENTIAL_INVENTORY.md`
  mục "5. Credential Ownership" — Meta token liên quan (Header Auth account /
  `META_ACCESS_TOKEN`) đang ghi "⚠️ TOKEN CÓ HẠN — token Meta thường hết hạn sau 60-90
  ngày". Cần làm rõ ngay từ Brainstorm: "System User Token vĩnh viễn" ở đây có phải
  Meta System User Token (long-lived, khác với short-lived user token đang dùng) hay
  là việc khác — chưa có quyết định kỹ thuật nào được chốt.
