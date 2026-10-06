# Task 04 — Fix Missing Ads Data (Pagination)

**Workflow:** `rz3Wya5lFay7ShVL` — "Meta Ads Daily Sheet Update 7:30 2"
**Incident gốc:** 2026-10-02 — `AI_OS/incidents/2026-10-02_missing-ads-data.md`

## Root cause

Node `Lấy dữ liệu Meta` gọi Meta Insights API `/insights?level=ad` không set `limit`
và không xử lý pagination (`paging.next`) → chỉ đọc page 1, các ad còn lại mất im lặng.

## Các file

- `PLAN.md` — master plan: root cause, fix đề xuất, risk, test plan, DoD.
- `CURRENT_STATUS.md` — execution state canonical (phase gate + facts).
