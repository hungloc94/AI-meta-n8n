# Roadmap — n8n Meta Ads

> Cập nhật lần cuối: 2026-07-03

## Định hướng phát triển dài hạn

Hệ thống hướng tới Marketing Intelligence tự vận hành hoàn toàn:
1. **Reporting tự động** — báo cáo KPI qua Telegram theo lịch, không cần thao tác thủ công
2. **Multi-range reporting** — báo cáo hôm qua, hôm nay, 7 ngày, 30 ngày, khoảng ngày tùy chọn
3. **Infrastructure hardening** — credential không expire, alert phân loại severity, source control toàn bộ workflow
4. **Dashboard KPI** — tổng quan visual, không phụ thuộc Telegram

## Danh sách Module đã hoàn thành

| # | Module | Ngày hoàn tất | Ghi chú |
|---|--------|---------------|---------|
| 1 | Daily Sheet Update (Meta → Sheet) | 2026-06-09 | numDays=7, idempotent, Nhóm B protected |
| 2 | Yesterday Report (Sheet → Telegram) | 2026-06-09 | Scheduled 08:13, KPI khớp Sheet |
| 3 | Today Report (Meta API → Telegram) | 2026-06-11 | 3 slot/ngày, KPI 100% khớp Ads Manager |
| 4 | Meta Report VERIFIED (Bot polling) | 2026-05-29 | Command-driven, source controlled |
| 5 | Monitoring Layer (Health Checks) | 2026-06-15 | GS 07:00 + Meta API 07:05, PASS/FAIL alert |
| 6 | Stable Production State | 2026-06-16 | 7 workflows verified, 24-48h observation PASS |
| 7 | Service Account Migration | 2026-06-25 | Google OAuth → Service Account, không expire |
| 8 | Meta KPI Expansion | 2026-06-30 | Impressions, CPM, CTR, Video metrics |
| 9 | Date Range Engine V1 | 2026-07-03 | Filter + Aggregate + KPI, runtime verified |

## Module đang làm

### KPI Engine Integration — ACTIVE DEVELOPMENT

**Mục tiêu:** Tích hợp Date Range Engine V1 vào Yesterday Report, thay thế logic cũ.

**Các bước:**
1. Presentation Layer (Render Telegram) sử dụng output của Date Range Engine
2. Production verify PASS → loại bỏ node cũ "Lọc & Tính KPI Hôm Qua"
3. Tái sử dụng Engine cho 7 ngày, 30 ngày, khoảng ngày

**Trạng thái:** Engine đã verified trên TEST workflow (inactive). Chưa tích hợp vào production.

## Module tiếp theo — theo thứ tự ưu tiên

| # | Module | Priority | Mô tả |
|---|--------|----------|-------|
| 1 | Yesterday Report + Engine | HIGH | Tích hợp Date Range Engine, verify production |
| 2 | Multi-range Reports | HIGH | 7 ngày, 30 ngày, khoảng ngày tùy chọn — tái sử dụng Engine |
| 3 | Xác minh loại Meta Token | MEDIUM | Token Debugger → ghi CREDENTIAL_INVENTORY.md |
| 4 | Revoke Telegram Bot Token cũ | MEDIUM | Token đã lộ trong chat |
| 5 | Patch format báo cáo ngắn hơn | MEDIUM | Today + Yesterday Report |
| 6 | Meta System User Token | LOW | Chuyển từ User Token → System User Token (BM ID: 187499196519080) |
| 7 | Telegram Alert severity levels | LOW | Phân loại mức độ lỗi trong alert |
| 8 | Dashboard tổng quan KPI | LOW | Visual dashboard, không phụ thuộc Telegram |

## Backlog kỹ thuật — Deferred

| # | Item | Priority | Ghi chú |
|---|------|----------|---------|
| 1 | Revoke Telegram Bot Token cũ | MEDIUM | Token đã lộ trong chat — cần revoke và verify token mới |
| 2 | DD/MM/YYYY explicit format support | LOW | Parser hiện chỉ hỗ trợ DD/MM, chưa có DD/MM/YYYY |
| 3 | Year rollover ambiguity | LOW | Edge case cuối năm: 31/12 → năm nào? Chưa xử lý |
| 4 | Schema validation layer | LOW | Validate input data trước khi ghi vào Sheet |
| 5 | Preflight validation cho protected columns | LOW | Detect ghi đè Nhóm B/Derived trước khi execute |
| 6 | Automated regression testing | LOW | Chưa có test suite cho workflows |
| 7 | Parser unit tests | LOW | Date parser chưa có unit tests |
| 8 | Credential rotation dry-run checklist | LOW | Quy trình rotate Meta/Telegram token an toàn |
| 9 | Import validation script cho workflow JSON | LOW | Detect mojibake tiếng Việt trước khi import |
| 10 | Cleanup dedupe memory (staticData bloat) | LOW | Entries >60s chưa được dọn tự động |
| 11 | Replace hardcoded Telegram strings với constants | LOW | Giảm magic strings trong Code nodes |
## 4 Phase phát triển

| Phase | Tên | Mô tả |
|-------|-----|-------|
| 1 | Reporting Assistant | Báo cáo KPI tự động qua Telegram theo lịch — đang ở đây |
| 2 | Advertising Intelligence | Phân tích hiệu quả quảng cáo, anomaly detection, creative fatigue |
| 3 | Lead Quality Intelligence | Đo quality leads, cost per qualified phone/message, CRM feedback |
| 4 | Cross-business Intelligence | So sánh đa kênh, đa chiến dịch, báo cáo chiến lược |

## Business priorities
- Ưu tiên: quality leads, potential customers,
  cost per qualified phone, cost per potential message
- Tránh: vanity metrics, cheap leads không chuyển đổi
- Telegram = reporting interface +
  operational command center +
  future AI interaction layer