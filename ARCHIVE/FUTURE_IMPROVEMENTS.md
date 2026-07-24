# Future Improvements

## Deferred Backlog
- Support DD/MM/YYYY ranges.
- Resolve year rollover ambiguity.
- Create centralized date parser utility.
- Add schema validation layer.
- Add structured telemetry/logging.
- Add automated regression testing.
- Replace hardcoded Telegram strings with constants.
- Add multi-report batching optimization.
- Add parser unit tests.
- Add runtime health monitoring.
- Add credential rotation dry-run checklist.
- Cleanup dedupe memory entries older than 60s to prevent staticData bloat over long runtime.
- Add import validation script cho workflow JSON exports.

## Not For Current Forensic Patch
Các improvements này intentionally deferred để tránh architecture drift trong recovery.

## Deferred After Stable Baseline Freeze: 2026-05-29

1. Revisit business-intent dedupe architecture after production stabilization.
2. Implement Meta realtime sync for current-day reporting.
3. Use Google Sheet historical cache for previous-day reporting.

These are deferred enhancements, not blockers for the current stable operational baseline.


## Roadmap Giai Đoạn 1 — Làm Ngay (Reliability): 2026-06-10

1. **Giảm `numDays` 30 → 7** trong Meta Ads Daily Sheet Update.
   - Meta adjust data chậm 1-7 ngày. 30 ngày không cần thiết và tốn API calls.
   - Lợi ích: giảm runtime đáng kể, ít risk rate limit hơn.

2. **Global Telegram Error Alert** — khi bất kỳ workflow nào fail:
   - Gửi Telegram ngay với: tên workflow, node lỗi, thời gian, error message.
   - Dùng n8n Error Workflow (global) để bắt tất cả mà không cần patch từng workflow.
   - Priority: cao nhất — incident 10/06 cho thấy silent failure rất nguy hiểm.

3. **Thêm CPC** vào Telegram report (CPC = Chi tiêu / Click).

## Roadmap Giai Đoạn 2 — Sau Khi Ổn Định 1 Tuần: 2026-06-10

4. **Chuyển Google OAuth → Google Service Account.**
   - Service Account không expire, không cần reconnect.
   - Loại bỏ hoàn toàn single point of failure hiện tại.
   - Cần tạo Service Account, share Sheet với service account email, cấu hình credential mới trong n8n.

5. **Test + Activate Meta Report Today Scheduled** (11:31 / 16:31 / 21:13).

6. Tối ưu báo cáo: CTR, CPM nếu cần.

## Roadmap Giai Đoạn 3 — Chưa Cần Làm Ngay: 2026-06-10

7. Revoke Telegram Bot Token cũ → lấy token mới (token đã lộ trong chat session).
8. Health Check workflow định kỳ riêng biệt (ping Sheet + Meta API, gửi heartbeat).

---

## Data Schema / Sheet Ownership Improvements: 2026-05-29

- Add selective update safeguards so Meta Sync updates only Meta-owned columns by default.
- Add preflight validation that detects attempted overwrite of formula/manual/business columns.
- Add a protected-column allowlist/denylist before any production Meta Sync activation.
- Review current append/update workflow behavior against DATA_SCHEMA_RULES.md before future import testing.
- Consider documenting formulas for protected cost columns so future agents can distinguish API values from Sheet-derived values.
