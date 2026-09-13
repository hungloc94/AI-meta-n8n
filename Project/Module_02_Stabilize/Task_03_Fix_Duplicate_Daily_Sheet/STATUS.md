# STATUS — Task 03: Fix Duplicate Row (Daily Sheet Update)

> **Execution state canonical:** `CURRENT_STATUS.md`. File này chỉ giữ tóm tắt. Master plan: `PLAN.md`.

## Trạng thái hiện tại
- **Tiến độ:** 25% — Audit xong, root cause xác định, PLAN tái cấu trúc sang PHASE 0–5 (Risk/Rollback/DoD), backup xong.
- **Phase đang làm:** PHASE 0 — tắt `retryOnFail` node append → **chờ anh Lộc CONFIRM** mới APPLY.
- **Blocker:** cần CONFIRM PHASE 0 (sửa production ACTIVE); PHASE 2 sau đó cần tab `TEST_DEUP` trên Google Sheet.
- **Cập nhật lần cuối:** 2026-09-08, Claude Code

## Đã hoàn thành
- ✅ Audit workflow `rz3Wya5lFay7ShVL` — xác định root cause: 2 execution cùng append 1 Key (dedup chỉ theo từng run).
- ✅ Bác bỏ giả thuyết append-retry (2 timestamp 22:00 vs 07:30 = 2 lần chạy).
- ✅ Backup 2 lớp: repo JSON + bản LIVE export từ n8n (md5/logic khớp). Thư mục `BACKUP/task-fix-duplicate/`.

## Facts
| Entity | Giá trị |
|--------|---------|
| Workflow id | `rz3Wya5lFay7ShVL` — "Meta Ads Daily Sheet Update 7:30 2", active |
| Key format | `{date_start}_{ad_id}` — cột **S** |
| Sheet | `1EHpEws60xWjJUaBfd_qDFLoeikMYvhUc9f9fR-HcL_s`, tab `BÁO_CÁO_QUẢNG_CÁO` |
| Nhóm B (không đụng) | Mess_Comment, Khach_sai_tep, Khach_hop_le, SDT, Khach_chot, Chi_phi_*, Doanh_thu, ghi_chu |

## HANDOVER
- Người giao: Claude Code (audit + plan + backup xong)
- Người nhận: session tiếp theo / Cook
- Đã làm xong: Root cause, PLAN đầy đủ 7 bước, backup 2 lớp.
- Cần làm tiếp: Dựng bản fix upsert (Bước 2) → xin anh Lộc tab TEST → test idempotency (Bước 3).
- Xong khi nào: 2 lần chạy liên tiếp appendCount=0, COUNTIF(Key)=1, Nhóm B nguyên vẹn, production 1 chu kỳ sạch.
- Trạng thái: ⏳ Đang dựng bản fix; chờ anh Lộc duyệt cách test (tab TEST).
