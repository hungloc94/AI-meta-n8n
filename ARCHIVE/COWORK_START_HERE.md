# COWORK START HERE

## 1. Identity

Project này là n8n automation system cho Meta Ads reporting.

- Telegram bot nhận commands, fetch Google Sheet data, và deliver KPI reports.
- Chạy trên Windows + Docker.
- n8n container: `n8n-msp`
- Project path: `D:\AI_Project\n8n-meta-ads`

## 2. Reading Order For Cowork

Đọc theo thứ tự:

1. File này trước
2. `CURRENT_STATUS.md` - operational truth hiện tại
3. `GOVERNANCE_RULES.md` - non-overridable rules
4. `DECISIONS.md` - architecture reasoning
5. Các project memory files khác chỉ đọc khi cần

Project memory location: `D:\AI_Project\Obdisian_2\My Vault\Project\AI_Meta_n8n_autoamation\`

## 3. Daily Tasks

Cowork daily responsibilities:

- Đọc Meta Ads KPI data từ Google Sheet.
- Đọc lead/customer data từ Google Sheet.
- Analyze KPI trends.
- Suggest campaign actions như pause hoặc activate.
- Log decisions và reasoning vào `DECISIONS.md`.

## 4. Data Sources

- Meta Ads KPI Sheet: `[REPLACE_WITH_ACTUAL_LINK]`
- Lead/Customer Sheet: `[REPLACE_WITH_ACTUAL_LINK]`
- Decision history: `DECISIONS.md`
- Architecture: `SYSTEM_ARCHITECTURE.md`

## 5. Operational Notes

- n8n fetch previous day data tự động mỗi sáng.
- Intraday data cần direct Meta API call.
- Workflow phải remain inactive trừ khi được explicit approval.
- Không modify credentials nếu chưa explicit approval.
- Luôn log decisions với date và reasoning.

## 6. Current System State

> Full detail nằm trong `CURRENT_STATUS.md`. Đây chỉ là tóm tắt nhanh.

**Cập nhật lần cuối: 2026-06-02**

| Hạng mục | Trạng thái |
|----------|-----------|
| Pipeline E2E | ✅ Verified (2026-05-27) |
| Telegram KPI delivery | ✅ Hoạt động |
| Polling continuity | ✅ Đã verify dưới supervised runtime |
| Telegram offset | ✅ Dynamic expression — KHÔNG còn hardcoded |
| Workflow `Meta Report VERIFIED` | ✅ **ĐANG ACTIVE** trên n8n |
| Production promotion | ⏳ Chờ — còn 1 blocker cuối |

**Blocker duy nhất còn lại:**
Chưa test stale replay sau khi restart workflow. Cần deactivate → activate lại → gửi command → xác nhận không có báo cáo cũ bị gửi lại.

**Hạn chế đã biết (chưa fix):**
- Year rollover ambiguity chưa giải quyết.
- `DD/MM/YYYY` explicit format chưa được hỗ trợ.
- Business-intent dedupe chưa có (deferred).

---

## 7. Việc Tiếp Theo Theo Thứ Tự Ưu Tiên

### Ưu tiên 1 — Blocker production promotion
> Test stale replay sau restart:
> 1. Deactivate `Meta Report VERIFIED`
> 2. Activate lại
> 3. Gửi `báo cáo hôm qua` qua Telegram
> 4. Xác nhận nhận đúng 1 response, không có report cũ
> 5. Tick checklist → xem xét GO/NO-GO với người có thẩm quyền

### Ưu tiên 2 — Phase B (24h observation)
> Sau khi pass stale replay test:
> - Quan sát 24h liên tục theo `PRODUCTION_PROMOTION_CHECKLIST.md`
> - Không có duplicate report, không crash, không KPI corruption

### Ưu tiên 3 — Sau production stable
> - Thiết kế workflow sync Meta API (Option B: Yesterday + Today — xem `DECISIONS.md`)
> - Review 3 phiên bản "Meta Ads Daily Sheet Update" đang inactive trong n8n
> - Credential validity check (Meta token, Google OAuth)

## 8. Governance Reminder

- Không activate workflow nếu chưa explicit approval.
- Không rotate encryption key nếu chưa có full credential recreation plan.
- Không patch production trực tiếp.
- Luôn backup trước mọi changes.
- Không để hardcoded Telegram offsets trong active production workflows sau testing.
- Full rules: `GOVERNANCE_RULES.md`
