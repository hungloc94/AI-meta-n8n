# CONTEXT SNAPSHOT — 2026-06-02

> Báo cáo tổng hợp trạng thái dự án. Được tạo tự động từ đọc toàn bộ Markdown files trong project folder.
> Không có file nào bị sửa trong quá trình tạo báo cáo này.

---

## 1. CURRENT STATE — Dự án đang ở đâu

**Phase:** Stable Operational Baseline — chờ Production Promotion Governance

**Workflow đang chạy trong n8n:**
- `Meta Report VERIFIED` → **ACTIVE** (workflow duy nhất đang chạy)
- `Meta Report TEST new` → inactive/deprecated
- `Telegram Bot & Reports` → inactive/deprecated
- `My workflow` → inactive/non-Telegram test artifact

**Trạng thái thực tế:**
- Toàn bộ pipeline đã verified end-to-end: Telegram polling → routing → date parsing → Google Sheet → normalize → KPI aggregation → Telegram delivery.
- Single-consumer Telegram polling đã đảm bảo (không còn double-consumer chaos).
- KPI aggregation chạy đúng; kết quả KPI=0 cho ngày không có data là **expected business behavior**, không phải lỗi pipeline.
- Stable baseline export đã freeze: `META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json`.

**Production activation:** **CHƯA ĐƯỢC APPROVE.** Vẫn trong giai đoạn governance review.

---

## 2. LAST KNOWN ACTIONS — Việc cuối cùng đã làm

Thứ tự theo thời gian gần nhất:

### 2026-05-29 (mới nhất)
- Phát hiện và xác nhận Google Sheet là Source of Truth; phân loại columns thành Group A (Meta-owned) và Group B (business/formula/manual).
- Tạo `DATA_SCHEMA_RULES.md` định nghĩa ownership rules cho từng column.
- Freeze stable baseline: `META_REPORT_VERIFIED_STABLE_BASELINE_2026-05-29.json`.
- Resolve `[NODE=UNKNOWN]` routing spam do double-consumer (hai workflow cùng poll Telegram queue).
- Xác nhận chỉ còn một active polling consumer (`Meta Report VERIFIED`).
- Quyết định defer business-intent dedupe, Meta realtime sync.

### 2026-05-27
- Supervised runtime activation: verify polling continuity thật sự (không chỉ Execute Step).
- Verified command `báo cáo hôm qua` → exactly one Telegram response, no stale replay.
- Temporary hardcoded offset dùng cho backlog draining: `81218650`.
- First full E2E verified test recovery baseline: `META_REPORT_TEST_E2E_VERIFIED_2026-05-27.json`.
- Khởi tạo governance system: `STAGING_TEST_MATRIX.md`, `PRODUCTION_PROMOTION_CHECKLIST.md`.
- Fix parser DD/MM, routing chain, Normalize Sheet Data node, Send Report Telegram payload.

---

## 3. OPEN ISSUES — Vấn đề chưa giải quyết

### Critical (blockers cho production promotion)
| # | Vấn đề | File tham chiếu |
|---|--------|----------------|
| 1 | **Chưa restore offset expressions** sau khi dùng temporary hardcoded offset `81218650` trong testing | `PRODUCTION_PROMOTION_CHECKLIST.md` |
| 2 | **Chưa confirm no stale replay sau restart** (chỉ verify trong một session, chưa test sau restart thật) | `STAGING_TEST_MATRIX.md`, `CURRENT_STATUS.md` |
| 3 | **Phase B (24h sustained observation) chưa hoàn thành** — tất cả checklist items của Phase B vẫn unchecked | `PRODUCTION_PROMOTION_CHECKLIST.md` |

### High (cần làm trước hoặc ngay sau production)
| # | Vấn đề | Ghi chú |
|---|--------|---------|
| 4 | Rapid repeated command stress test chưa làm (chỉ verify single command) | `STAGING_TEST_MATRIX.md` |
| 5 | Invalid command handling test chưa verify | `STAGING_TEST_MATRIX.md` |
| 6 | Credential validity chưa verify đầy đủ (Telegram, Meta long-lived token, Meta App ID/Secret, Google OAuth) | `BACKUP_INVENTORY.md` |
| 7 | n8n database/volume backup chưa có recurring procedure | `BACKUP_INVENTORY.md` |

### Deferred (không phải blocker hiện tại)
| # | Vấn đề |
|---|--------|
| 8 | DD/MM/YYYY explicit format chưa support |
| 9 | Year rollover ambiguity chưa giải quyết |
| 10 | Business-intent dedupe deferred |
| 11 | Meta realtime sync (current-day reporting) chưa có |
| 12 | Webhook/Cloudflare public exposure deferred |
| 13 | Credential/key rotation plan chưa có |
| 14 | Long runtime stability test (extended observation window) chưa làm |

---

## 4. NEXT SAFE STEP — Bước tiếp theo an toàn nhất

**Bước 1 (Critical, phải làm trước production promotion):**
> Restore Telegram offset expressions về dynamic expressions (xóa hardcoded `81218650`), sau đó verify lại workflow sau restart → confirm no stale replay.

**Tại sao bước này trước tiên:**
- Đây là blocker rõ ràng nhất còn lại trong Production Promotion Checklist.
- Hardcoded offset trong active workflow là NO-GO condition theo Governance Rules.
- Governance rules cấm để hardcoded Telegram offset values trong active production workflows.

**Sau khi bước 1 xong:**
- Thực hiện Phase B (24h sustained observation).
- Sau Phase B pass → mới có thể xem xét GO/NO-GO production promotion decision.

---

## 5. MISSING INFO — Thiếu thông tin gì để tiếp tục

| # | Thông tin còn thiếu | Ảnh hưởng |
|---|---------------------|-----------|
| 1 | **Offset expression hiện tại trong workflow có còn hardcoded `81218650` không?** Cần mở n8n UI để check node `Telegram Trigger` hoặc polling node. | Blocker cho production promotion |
| 2 | **Credential validity hiện tại:** Meta long-lived token còn sống không? Google OAuth đã re-authorize chưa? | Có thể gây silent data failure |
| 3 | **`báo cáo 7 ngày` và custom date ranges** chưa verify — behavior thực tế ra sao? | Unknown production behavior |
| 4 | **Google Sheet có data thực tế cho ngày hiện tại (2026-06-02) không?** Hay vẫn chỉ có data lịch sử tháng 3? | Ảnh hưởng test `báo cáo hôm nay` |
| 5 | **n8n version và Docker compose hiện tại** — không rõ environment spec đầy đủ | Cần cho migration/recovery nếu server có vấn đề |
| 6 | **Ai là người có thể approve GO decision** cho production promotion? (human authority chưa được document rõ) | Governance gap |

---

## Tham chiếu nhanh

| File | Vai trò |
|------|---------|
| `CURRENT_STATUS.md` | Operational truth cao nhất |
| `GOVERNANCE_RULES.md` | Safety constraints không được override |
| `PRODUCTION_PROMOTION_CHECKLIST.md` | Checklist GO/NO-GO production |
| `STAGING_TEST_MATRIX.md` | Các test cases cần pass |
| `BACKUP_INVENTORY.md` | Danh sách backup artifacts |
| `DATA_SCHEMA_RULES.md` | Sheet column ownership rules |
| `DECISIONS.md` | Lý do kiến trúc |
| `INCIDENTS_AND_RECOVERY.md` | Lịch sử forensic recovery |

---

*Snapshot này được tạo ngày 2026-06-02 bằng cách đọc toàn bộ 17 Markdown files trong project folder. Không có file nào bị sửa đổi.*
