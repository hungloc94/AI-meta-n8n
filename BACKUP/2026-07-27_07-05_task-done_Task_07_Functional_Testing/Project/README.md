# n8n Meta Ads — Marketing Intelligence Operating System

## 1. Mục tiêu Project

Xây dựng hệ thống Marketing Intelligence tự vận hành, dùng n8n làm orchestrator để:
- Tự động đồng bộ dữ liệu Meta Ads → Google Sheet hằng ngày
- Gửi báo cáo KPI qua Telegram theo lịch (hôm qua, hôm nay, khoảng ngày)
- Giám sát sức khỏe credential và API, alert khi có sự cố
- Hỗ trợ forensic debugging và recovery khi hệ thống gặp lỗi

Human vẫn là final authority cho mọi quyết định production.

## 2. Phạm vi

**Trong scope:**
- n8n workflows (scheduling, polling, data sync, reporting)
- Meta Marketing API integration (KPI: spend, reach, messages, clicks, CPC, CPM, CTR, video metrics)
- Google Sheets làm operational source of truth (Nhóm A: Meta ghi / Nhóm B: business data protected)
- Telegram bot: báo cáo tự động + command-driven reporting
- Monitoring layer: Health Check credential + API mỗi sáng

**Ngoài scope:**
- Dashboard UI (backlog, chưa build)
- Multi-channel ads (chỉ Meta Ads)
- CRM integration
- Public-facing webhook/Cloudflare exposure (deferred)

## 3. Bối cảnh

**Tech stack:**
- n8n (Docker, self-hosted) — workflow orchestrator
- Meta Marketing API — data source
- Google Sheets (Service Account) — operational data store
- Telegram Bot API — reporting channel + alert channel

**Kiến trúc chính:**
- **Daily Sheet Update** (07:30): Meta API → Google Sheet (numDays=7, idempotent)
- **Yesterday Report** (08:13): Google Sheet → KPI aggregation → Telegram
- **Today Report** (11:31 / 16:31 / 21:13): Meta API trực tiếp → Telegram
- **Meta Report VERIFIED**: Telegram bot polling, command-driven (báo cáo hôm qua, báo cáo 24/03...)
- **Health Checks** (07:00, 07:05): Google Sheets + Meta API credential validation

**Trạng thái hiện tại:** Production stable từ 2026-06-16. Date Range Engine V1 verified, đang tích hợp vào Yesterday Report.

## 4. Kết quả mong đợi

| Kết quả | Tiêu chí thành công |
|---------|---------------------|
| Báo cáo tự động hằng ngày | Telegram nhận đúng lịch, KPI khớp Ads Manager |
| Dữ liệu Sheet luôn fresh | Sync 07:30 thành công, không mất data Nhóm B |
| Alert sớm khi có sự cố | Health Check phát hiện credential/API fail trước giờ sync |
| Recoverable cho AI agent mới | Đọc README → hiểu hệ thống trong 1 session, không cần đào chat cũ |
| Mở rộng được | Date Range Engine cho multi-day reports, thêm metric không cần redesign |