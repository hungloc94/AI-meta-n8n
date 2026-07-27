# Task 07: Functional Testing

## Mục tiêu
Test thủ công từng workflow trong 6 workflow thực tế của hệ thống trên n8n Home Server
(đã restore ở Task 06), theo đúng thứ tự dependency, trước khi switch production (Task 08).

## Lý do
MIGRATION_RUNBOOK.md — Workflow Import Order bước 4-6: test node-by-node, verify
polling continuity dưới supervised runtime activation, chỉ import production sau
khi TEST pass. CASE-014 (Thứ Tự Activate Dependency): Sync phải chạy trước Report.
CASE-008: Execute Step không đủ — phải validate dưới real runtime.

## Phạm vi — 6 Workflow, test theo đúng thứ tự dependency
1. **Google Sheets Health Check (07:00)** — test thủ công, verify PASS, không có alert
2. **Meta API Health Check (07:05)** — test thủ công, verify PASS, không có alert
3. **Meta Ads Daily Sheet Update (07:30)** — test thủ công, verify Sheet cập nhật đúng
   (phụ thuộc: GS Health Check PASS)
4. **Yesterday Report (08:13)** — test thủ công, verify Telegram nhận báo cáo đúng KPI
   (phụ thuộc: Daily Sync PASS)
5. **Today Report (11:31 / 16:31 / 21:13)** — test thủ công, verify Telegram nhận báo cáo
   (phụ thuộc: Meta API Health Check PASS, gọi Meta API trực tiếp không qua Sheet)
6. **Meta Report VERIFIED** — xác nhận vẫn đang lỗi (theo Project/STATUS.md hiện tại),
   ghi nhận vào WORKLOG — KHÔNG fix ở Task này, KHÔNG activate

## Ngoài phạm vi
- Switch traffic production sang Home Server (Task 08)
- Fix lỗi Meta Report VERIFIED — nằm ngoài phạm vi Module 00, cần Task/Module riêng
- Sửa lỗi logic workflow khác nếu phát hiện — báo Human trước khi xử lý

## Kết quả mong đợi
5/6 workflow (trừ Meta Report VERIFIED) PASS test thủ công theo đúng thứ tự dependency
07:00 → 07:05 → 07:30 → 08:13 → Today Report. Meta Report VERIFIED xác nhận vẫn lỗi,
ghi nhận rõ ràng, không activate. GO cho Task 08 Switch Production (trừ Meta Report VERIFIED).
