# AI_OS Changelog

## 2026-07-28

### New Skills

- Added `skills/investigate_fail2ban/` for evidence-only Fail2ban investigations.
- Added `skills/investigate_ssh/` for OpenSSH service, listener, log, and TCP-state investigations.
- Added `skills/investigate_tailscale/` for Tailscale node, peer, and path investigations.
- Added `skills/verify_network/` for independent PASS/FAIL verification after network incidents.
- Added `skills/rollback_fail2ban/` for controlled Fail2ban rollback workflows.

### Updated Documentation

- Added `skills/README.md` as the operational skill directory index.
- Added `SKILLS.md` as the AI_OS operational skills index and structure review.
- Added `templates/ops/README.md` as the shared operational template index.

### Reusable Components Extracted

- Added `templates/ops/SAFETY_GUARDRAILS.md`.
- Added `templates/ops/COMMAND_EVIDENCE.md`.
- Added `templates/ops/NETWORK_VERIFICATION_MATRIX.md`.
- Added `templates/ops/INCIDENT_REPORT_SECTIONS.md`.
- Added `templates/ops/ROLLBACK_PROCEDURE.md`.

### Reason For Changes

The SSH-over-Tailscale incident showed that future incidents need reusable workflows for evidence collection, confidence-scored root-cause analysis, safe recovery, permanent prevention, independent verification, and rollback.

### Future Improvement Opportunities

- Add executable evidence-collection scripts that produce structured JSON.
- Add parsers for Fail2ban status, nftables, iptables, and Tailscale output.
- Add policy checks that detect trusted private networks missing from Fail2ban `ignoreip`.
- Add an approval-gated remediation runner for safe Fail2ban unban and ignore-list changes.
- Add tests for generated incident reports to verify required sections are present.

## 2026-10-07

### New Templates

- Added `templates/ops/API_INTEGRATION_CHECKLIST.md` — checklist bắt buộc trước khi tích hợp
  bất kỳ API ngoài nào mới (pagination, default page size, `MAX_PAGES` guard, rate limit).

### Reason For Changes

Incident 2026-10-02 (`incidents/2026-10-02_missing-ads-data.md`, Task_04): node `Lấy dữ liệu
Meta` trong workflow `Meta_Ads_Daily_Sheet_Update` gọi Meta Insights API không set `limit`
và không xử lý pagination → chốt cứng ở default page size (25 ad/ngày) → mất 5–8 ad/ngày,
không có error, không ai nhận ra qua log. Phát hiện bằng mắt khi xem báo cáo thiếu ad.
Fix: thêm `limit=1000` + pagination loop (đọc `paging.cursors.after`, `MAX_PAGES=10` guard).
Verify PASS trên production (242 ad, 0 error, 8/8 ngày không bị truncate).

Bài học quan trọng nhất không riêng Meta API: **"có `limit` ở một node không có nghĩa node
khác trong cùng workflow cũng đã xử lý page size"** — workflow này có 2 node gọi API, chỉ 1
node có `limit` (và đó không phải node sinh ra dữ liệu bị mất). Checklist mới buộc kiểm
pagination **tại từng endpoint**, không suy luận từ node khác trông có vẻ ổn.

### Future Improvement Opportunities

- Thêm script tự động quét mọi node `httpRequest`/Code gọi API trong các workflow n8n hiện có,
  cảnh báo node nào thiếu `limit` hoặc thiếu xử lý `paging`/`cursor`/`next`.
- Áp checklist này hồi tố cho các workflow khác đang chạy (Today Report, Yesterday Report...)
  để xác nhận không có lỗi pagination tương tự ở nơi khác.
