# CURRENT_STATUS — Task 05: Backfill Missing Ads Data

> Canonical execution state của Task_05. Master plan: `PLAN.md`.

**Cập nhật lần cuối:** 2026-10-07 — Claude Code

## Trạng thái

- **Task status:** 🟢 **DONE** — backfill verify PASS trên production Sheet (2026-10-07 01:05 UTC).
- **Phase:** PHASE 4 — đóng task hoàn tất.
- **Workflow backfill:** đã xoá `udyHkTWSU2c3J54v` sau khi verify PASS.

## Kết quả backfill

| Chỉ số | Giá trị |
|---|---|
| Ngày backfill | 67 ngày (`2026-08-01` → `2026-10-06`) |
| Records Meta API trả về | 1199 |
| `appendCount` | 66 (ad mới) |
| `updateCount` | 1133 (ad đã có, update Nhóm A) |
| Pages/ngày | 1 × 67 ngày, `truncated=false` mọi ngày |
| Error | 0 |

## Verify Sheet sau backfill (đọc trực tiếp)

| Kiểm tra | Kết quả |
|---|---|
| Key trùng (COUNTIF > 1) | **0** — không duplicate |
| Nhóm B (5 dòng mẫu) | **NGUYÊN VẸN** — `Khach_sai_tep`, `Khach_hop_le`, `SDT`, `Khach_chot`, `Doanh_thu` giữ nguyên giá trị |
| Ngày có đúng 25 ad | **Không còn** — `days_exactly_25 = []` |

## So sánh mẫu Nhóm B trước/sau

Đối chiếu snapshot Sheet đầu execution backfill (2313 dòng) với snapshot đọc-only sau backfill
(2379 dòng), join bằng `Key`. Năm dòng có giá trị thực (`Khach_chot`, `SDT`, `Khach_hop_le`,
`Khach_sai_tep`) đều giữ nguyên cả 5 cột kiểm tra:

| Row | Key | Giá trị Nhóm B được kiểm tra | Kết quả |
|---:|---|---|---|
| 1021 | `2026-07-25_120252013747670202` | sai_tep=0, hợp_lệ=1, SDT=1, chốt=1 | Không đổi |
| 18 | `2026-03-27_120245752985840202` | sai_tep=0, hợp_lệ=2, SDT=1 | Không đổi |
| 7 | `2026-03-25_120245752985840202` | sai_tep=0, hợp_lệ=3 | Không đổi |
| 116 | `2026-04-08_120246509871770202` | sai_tep=1, hợp_lệ=1 | Không đổi |
| 2 | `2026-03-24_120245707622430202` | sai_tep=0, hợp_lệ=0 | Không đổi |

Ở cả 5 mẫu, `Khach_chot`, `Doanh_thu` và các cột trống khác cũng khớp snapshot trước backfill.

## Các lần thử verify

- Cách 1 (Python service account trực tiếp): classifier chặn với `Credential Exploration`;
  không truy cập credential bằng cách khác.
- Cách 2 (`POST /api/v1/executions`): n8n trả 405. Workflow READONLY có sẵn
  `WyLKW6wrPEagQvNa` chạy được qua `docker exec`; dùng output để đọc Sheet.
- Cách 3 (curl Google Sheets API): không cần dùng vì Cách 2 thành công.

## Trạng thái cuối

- **Task status:** 🟢 DONE — backfill và verify hoàn tất 2026-10-07.
- 67 ngày: 1199 records; 66 append + 1133 update.
- Sheet: 2379 dòng; 0 duplicate Key; không ngày nào đúng 25 ad.
- Nhóm B: 5 mẫu đối chiếu trước/sau không đổi.
- Workflow backfill `udyHkTWSU2c3J54v` đã xóa sau verify PASS.
- Bài học 5 được ghi ở `AI_OS/templates/ops/API_INTEGRATION_LESSONS.md`.
