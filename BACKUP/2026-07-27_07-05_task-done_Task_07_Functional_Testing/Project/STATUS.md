# Status — n8n Meta Ads

> Cập nhật lần cuối: 2026-07-03

## Tiến độ hiện tại

### Production Workflows — ALL STABLE

| Workflow | Schedule | Trạng thái |
|----------|----------|-----------|
| Google Sheets Health Check | 07:00 | ✅ ACTIVE |
| Meta API Health Check | 07:05 | ✅ ACTIVE |
| Meta Ads Daily Sheet Update | 07:30 | ✅ ACTIVE (numDays=7, Service Account) |
| Meta Report Yesterday Scheduled | 08:13 | ✅ ACTIVE (đọc Sheet → Telegram) |
| Meta Report Today Scheduled | 11:31 / 16:31 / 21:13 | ✅ ACTIVE (Meta API trực tiếp) |
| Meta Report VERIFIED | Realtime polling | ⚠️ ĐANG LỖI — chưa sửa, không activate |

### Credentials

| Credential | Trạng thái |
|------------|-----------|
| Google Sheets (Service Account) | ✅ Hoạt động — không expire |
| Meta Access Token (Header Auth) | ⚠️ Hoạt động — chưa xác minh loại token |
| Telegram Bot Token | ⚠️ Token cũ đã lộ trong chat, cần revoke |

### Modules đã hoàn tất

| Module | Trạng thái |
|--------|-----------|
| Service Account Migration | ✅ DONE (2026-06-25) |
| Meta KPI Expansion (Impressions, CPM, CTR, Video) | ✅ DONE (2026-06-30) |
| Date Range Engine V1 | ✅ RUNTIME VERIFIED (2026-07-03) |
| Parser | 🔒 FEATURE FREEZE |

## Bước đang làm

**KPI Engine — ACTIVE DEVELOPMENT:**
1. Tích hợp Date Range Engine vào Yesterday Report
   - Presentation Layer (Render Telegram) sử dụng output của Engine
   - Chỉ loại bỏ logic cũ ("Lọc & Tính KPI Hôm Qua") sau khi Production verify PASS
2. Sau Yesterday PASS → tái sử dụng Engine cho 7 ngày, 30 ngày, khoảng ngày

## Blocker

| Blocker | Mức độ | Ghi chú |
|---------|--------|---------|
| Chưa xác minh loại Meta token | MEDIUM | Cần check Token Debugger → ghi vào CREDENTIAL_INVENTORY.md |
| Telegram Bot Token cũ đã lộ | MEDIUM | Cần revoke token cũ, verify token mới hoạt động |

## Không được thay đổi khi chưa có lệnh Human

- 5 workflow đang ACTIVE và ổn định — không sửa
- Workflow Meta Report VERIFIED — đang lỗi, không activate cho đến khi được sửa và verify
- Cột Nhóm B trong Google Sheet — không ghi đè
- Service Account credentials — không thay đổi
- Telegram Bot Token đang dùng — không rotate cho đến khi có kế hoạch rõ ràng
## Handover

Nếu AI agent mới tiếp nhận:
1. Đọc `Project/README.md` → hiểu tổng quan
2. Đọc `Project/RULES.md` → hiểu constraints
3. Đọc file này → biết đang ở đâu
4. Việc tiếp theo: tích hợp Date Range Engine vào Yesterday Report (workflow TEST, inactive)
5. Không activate workflow nếu chưa đọc RULES.md
6. Workflow Meta Report VERIFIED đang lỗi — chưa sửa, không activate
7. Toàn bộ 6 workflow đều đã được đóng gói và backup trong BACKUP/workflows/



## Tiến độ Module

| Module | Tên | Trạng thái |
|--------|-----|------------|
| Module 00 | Migrate Home Server | ⏳ Chưa bắt đầu |
| Module 01 | Date Range Engine Integration | ⏳ Chưa bắt đầu |
| Module 02 | Stabilize | ⏳ Chưa bắt đầu |