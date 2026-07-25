# Worklog — Task 01: Cài Caddy

<!-- Ghi lại theo từng phiên làm việc -->
<!-- ✅ = không còn [ ] chưa xử lý -->
<!-- ⚠️ = còn [ ] chưa xử lý -->

## [2026-07-25] ✅

### Nhật ký
- Bước 1 (SSH vào server): AI CLI đang chạy trực tiếp trên Home Server (hostname `homeserver`, Tailscale IP 100.105.119.88, Ubuntu 24.04.4 LTS khớp Module README) — không cần SSH riêng. Xác nhận có quyền sudo (passwordless).
- Bước 2 (Cài Caddy qua apt): Thêm official Caddy apt repo (dl.cloudsmith.io) + GPG key theo hướng dẫn chính thức, `apt update`, `apt install caddy`. Kết quả: Caddy v2.11.4 cài thành công, service `caddy.service` tự động enabled (symlink multi-user.target.wants).
- Bước 3 (Tạo Caddyfile cơ bản): Package apt đã tự tạo `/etc/caddy/Caddyfile` mặc định — lắng nghe `:80`, `file_server` phục vụ `/usr/share/caddy`. Validate bằng `caddy validate --config /etc/caddy/Caddyfile --adapter caddyfile` → "Valid configuration". Đây đúng phạm vi "cơ bản" của Task (chưa cấu hình domain/n8n — nằm ở Task 02, 03).
- Bước 4 (Start và enable Caddy): `systemctl start caddy` + `systemctl enable caddy` → `active` + `enabled` (tự chạy khi boot).
- Bước 5 (Verify): `curl -i http://localhost` → `HTTP/1.1 200 OK`, header `Server: Caddy`, trả về trang mặc định "Caddy works!".
- Toàn bộ 5 bước trong PLAN.md đã hoàn tất, không có blocker.

### Signal
OPEN_TASKS:
- [x] Bước 3 — Tạo Caddyfile cơ bản
      → Hoàn thành lúc: kết thúc phiên
- [x] Bước 4 — Start và enable Caddy
      → Hoàn thành lúc: kết thúc phiên
- [x] Bước 5 — Verify Caddy hoạt động (curl localhost)
      → Hoàn thành lúc: kết thúc phiên

STALE_DOCS:
- [x] STATUS.md → cần cập nhật tiến độ 100%, Task Done
      → Phát hiện khi: vừa xong Bước 5
- [x] Module_00 TASK_INDEX.md → Task 01 chuyển từ "⏳ Chưa bắt đầu" sang "✅ Hoàn thành"
      → Phát hiện khi: vừa xong Bước 5

PROPOSAL:
- [x] STATUS.md → cập nhật tiến độ 100%, Task Done
      → Sửa chỗ: phần "Trạng thái hiện tại" + HANDOVER
      → Lý do: cả 5 bước PLAN.md đã verify PASS
      → Tạo lúc: kết thúc Bước 5
- [ ] Module_00/TASK_INDEX.md → đánh dấu Task 01 hoàn thành
      → Sửa chỗ: dòng "Task 01 | Phase 9 — Caddy"
      → Lý do: Task 01 đã Done, cần Human xác nhận trước khi cập nhật (theo AI_OS Rule 4)
      → Tạo lúc: kết thúc Bước 5