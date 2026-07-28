# Status — Task 01: System User Token vĩnh viễn

## Trạng thái hiện tại
- **Tiến độ:** 0% — README.md và PLAN.md của Task này đang trống
- **Bước đang làm:** Chưa bắt đầu — chưa có Brainstorm, chưa có quyết định kỹ thuật nào
- **Blocker:** Chưa xác định rõ phạm vi kỹ thuật (xem mục "Câu hỏi mở" bên dưới)
- **Cập nhật lần cuối:** 2026-07-28 10:49 (+07), Claude Code

## HANDOVER
- Người giao: Claude Code (theo yêu cầu anh Lộc — ghi nhật ký để tiếp tục ở phiên CLI mới)
- Người nhận: AI session tiếp theo
- Đã làm xong: Chưa có gì — README.md, PLAN.md, WORKLOG.md, CASE_INDEX.md của Task này
  đều đang trống
- Cần làm tiếp:
  1. Đọc theo đúng thứ tự AI_OS quy định cho "lần đầu làm việc với Task mới":
     `Project/README.md` → `Project/RULES.md` → `Module_02/README.md` → `Module_02/STATUS.md`
     → file này
  2. Đọc `Project/OPS/CREDENTIAL_INVENTORY.md` mục 5 (Credential Ownership) — token liên
     quan đang có ghi chú hạn sử dụng
  3. Brainstorm trước khi viết PLAN.md — dùng tinh thần skill `ak-brainstorm`
     (`AI_OS/skills/dev-workflow/ak-dev-workflow-skills/ak-brainstorm/SKILL.md`): scout
     trước, hỏi anh Lộc để chốt EXACT requirement, rồi mới lên PLAN.md
  4. Chưa được động vào bất kỳ credential/token thật nào trước khi có PLAN.md và anh Lộc
     duyệt
- Xong khi nào: Chưa xác định — cần Brainstorm trước mới biết acceptance criteria cụ thể
- Trạng thái: ⏳ Chờ Brainstorm

## Câu hỏi mở (cần làm rõ ngay từ Brainstorm)
- "System User Token vĩnh viễn" (theo mục tiêu ghi trong `Module_02/README.md`) — là:
  - Meta System User Token (long-lived, tạo qua Meta Business Settings, không hết hạn
    hoặc hạn rất dài) để thay cho `Header Auth account` / `META_ACCESS_TOKEN` hiện tại
    (đang ghi "⚠️ TOKEN CÓ HẠN — hết hạn sau 60-90 ngày" trong CREDENTIAL_INVENTORY.md)?
  - Hay một loại token/credential khác?
- Phạm vi: đổi ở cả `Header Auth account` (n8n credential) VÀ `$env.META_ACCESS_TOKEN`
  (.env biến môi trường) — vì CREDENTIAL_INVENTORY.md ghi 2 nơi dùng cùng loại token
  nhưng khác cơ chế, "cần đồng bộ khi rotate"?
- Ảnh hưởng workflow nào đang ACTIVE — nếu có, đây là loại thay đổi credential trên
  production, phải theo đúng Quy tắc 8 (AI_OS/README.md): phát hiện vấn đề gì ảnh hưởng
  workflow ACTIVE trong lúc làm → hỏi anh Lộc ngay, không tự quyết.
