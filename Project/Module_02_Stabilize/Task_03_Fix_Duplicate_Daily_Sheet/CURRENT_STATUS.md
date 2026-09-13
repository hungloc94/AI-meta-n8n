# CURRENT_STATUS — Task 03: Fix Duplicate Row (Daily Sheet Update)

> **Canonical execution state** của AIOS cho Task này. Master plan: `PLAN.md`. Case gốc: `CASES/CASE-051`.
> Không duplicate state sang file khác (`STATUS.md` chỉ giữ tóm tắt).

**Cập nhật lần cuối:** 2026-09-08 — Claude Code

## Trạng thái
- **Task status:** 🟡 IN_PROGRESS
- **Plan đã điền:** 2026-09-08 (tái cấu trúc sang PHASE 0–5 + Risk/Rollback/DoD).
- **Phase hiện tại:** **PHASE 4 — ĐANG DỞ.** 🔑 Đã tìm ra **root cause thật (2026-09-09)**: có **workflow clone lỗi `T0x1qzedADrmj1LA` ("TEST Daily Sheet Update Task09") vẫn active + chạy 07:30**, dùng append cũ → nguồn đẻ trùng hàng ngày. Đã **tắt clone + bật bản upsert + restart n8n**. Verify: chỉ còn 1 workflow production active = upsert.
- **Còn lại:** (a) dọn nốt **62 dup tồn** (danh sách 61 auto + 1 gộp đã bôi màu; anh đang xoá tay) → verify **0 dup**; (b) monitor 07:30 mai (chỉ upsert chạy); (c) PHASE 5: xoá hẳn clone `T0x1qzedADrmj1LA` + các workflow test, commit canonical, slice/timezone, update docs.
- **n8n test artifacts cần dọn ở PHASE 5:** workflow `baIwZg6ZSu5sWfDS` (TEST upsert), `5MUMksN5LrdulcLi` (delete), và bản có sẵn `Task03TestDedup1`.

## Phase gate

| Phase | Nội dung | Trạng thái | Evidence |
|-------|----------|-----------|----------|
| PHASE 0 | Tắt `retryOnFail` node append (hotfix) | ⏳ CHỜ DUYỆT (chưa APPLY) | — |
| PHASE 1 | Chuyển sang `appendOrUpdate` (upsert by Key) | ✅ BUILD+VALIDATE+IMPORT(TEST) DONE | Candidate `BACKUP/.../FIX_upsert_candidate.json`; workflow TEST `baIwZg6ZSu5sWfDS` import n8n (active=false) |
| PHASE 2 | Test idempotency T1–T4 trên `TEST_DEUP` | ✅ **PASS** (2026-09-08) | Xem block "PHASE 2 — Test results" |
| PHASE 3 | Dọn duplicate trên `TEST_DEUP` | ✅ **DONE** (2026-09-08) | Xoá 78 dòng trùng qua workflow `5MUMksN5LrdulcLi` (HTTP batchUpdate deleteDimension). Verify: 1658→**1580 dòng**, **0 Key trùng**, Nhóm B giữ nguyên |
| PHASE 4 | Chuyển Sheet chính + dọn dup + production 2 cycle | 🟡 IN_PROGRESS — apply+run thử PASS, dup đã xoá tay; **chưa verify sau xoá, chưa bật lại workflow** | prod run: appendCount=0/updateCount=133; 94 dup phân tích; anh Lộc tự xoá |
| PHASE 5 | Fix phụ + đóng task | 🟢 gần xong: slice(-2) + phương án 3 deployed (verify3 ĐẠT); Telegram timeout=30s (giữ retry); backup tạm đã dọn; bài học ghi `LESSONS_LEARNED.md`; commit git DONE | còn: xoá workflow test trong UI (tay) |
| PHASE 5b | Giữ & document tool highlight duplicate | ✅ **DONE** (2026-09-13) | Tool `QWowp4oT9PA1iJTO` = "TOOL — Highlight Duplicate Rows" (active=false); doc `AI_OS/skills/highlight_duplicate_rows/README.md` + index SKILLS.md/skills/README.md; 2 bản thừa đổi tên `[XOA]` |

## PHASE 2 — Test results (workflow TEST `baIwZg6ZSu5sWfDS` → tab `TEST_DEUP`, mode cli = full 8 ngày)
| Test | Kết quả | Đánh giá |
|------|---------|----------|
| T1 (chạy lần 1) | ghi vào `TEST_DEUP` thành công | baseline |
| T2 (chạy lần 2) | `appendCount=0, updateCount=133` | ✅ idempotent — 0 dòng mới |
| T3 (chạy lần 3) | `appendCount=0, updateCount=133`; Nhóm B STRICT thay đổi = **0**; Mess_Comment ngày cũ đổi = **0** | ✅ không đụng Nhóm B; đóng băng đúng |
| T4 (không trùng) | row count giữ nguyên 1658, upsert không sinh Key mới trùng | ✅ (78 dup là **có sẵn từ copy production**, không do upsert) |
| Mess_Comment phương án 3 | `2026-09-07`+`2026-09-08` CÓ ghi (16+16); `09-01`→`09-06` KHÔNG ghi | ✅ đúng quy tắc anh Lộc |

**Ghi chú:** `TEST_DEUP` (copy production) đang có **78 Key trùng** tồn đọng → xử lý ở PHASE 3/4. Upsert **không** tự xóa dòng trùng cũ (chỉ update dòng match đầu tiên).

## Build PHASE 1 — chi tiết (scratchpad, chưa import)
- **Xóa 7 node:** Check Dedup trước Ghi, IF: Append?, IF: Update?, Clean Append, Clean Update, Ghi mới vào Google Sheet, Cập nhật vào Google Sheet.
- **Thêm 1 node** `Ghi vào Sheet` = base trên node update cũ (đã có schema + `matchingColumns=["Key"]`), đổi `operation=appendOrUpdate`, giữ `retryOnFail` (upsert idempotent → vô hại).
- **Sửa `Xử lý dữ liệu`:** 1 item/insight, chỉ Nhóm A + Key + Meta metrics; **KHÔNG map Nhóm B**. `Mess_Comment` giữ = totalMess (như prod hiện tại — flag CONFIRM).
- **Sửa `Summary`:** đếm append/update từ snapshot start (giữ hợp đồng test `appendCount=0` ở lần chạy 2), vì 2 node IF cũ đã bị xóa.
- **Flow mới:** `… → Xử lý dữ liệu → Ghi vào Sheet → Summary`. `active=false` (import giữ inactive tới khi PHASE 2 PASS).

## Đã hoàn thành (pre-plan)
- ✅ Audit độc lập workflow `rz3Wya5lFay7ShVL` — root cause: 2 execution cùng append 1 Key (dedup chỉ theo từng run); `Check Dedup` no-op cross-run; `retryOnFail` append là rủi ro thứ hai.
- ✅ Backup 2 lớp: repo JSON + LIVE export (`BACKUP/task-fix-duplicate/`).
- ✅ Backup `PLAN.md` cũ → `BACKUP/task-fix-duplicate/PLAN.md.bak_2026-09-08T09-05-39` trước khi tái cấu trúc.
- ✅ `CASES/CASE-051` ghi nhận root cause + fix.

## Facts
| Entity | Giá trị |
|--------|---------|
| Workflow id | `rz3Wya5lFay7ShVL` — "Meta Ads Daily Sheet Update 7:30 2", active |
| Key format | `{date_start}_{ad_id}` — cột **S** |
| Sheet | `1EHpEws60xWjJUaBfd_qDFLoeikMYvhUc9f9fR-HcL_s` — tab prod `BÁO_CÁO_QUẢNG_CÁO`, tab test `TEST_DEUP` |
| Nhóm B (không đụng) | Mess_Comment, Khach_sai_tep, Khach_hop_le, SDT, Khach_chot, Chi_phi_*, Doanh_thu, ghi_chu |

## Auto-fix attempt counter
| Root cause | Attempts | Trạng thái |
|-----------|----------|-----------|
| Duplicate cross-run append | 0 / 3 | Chưa có auto-fix (fix chính thức = upsert, chưa execute) |

## Next action
→ Anh Lộc **CONFIRM PHASE 0** (tắt `retryOnFail` node "Ghi mới vào Google Sheet") thì tôi mới APPLY. Sau đó tuần tự PHASE 1 với gate CONFIRM diff trước import.
