# PLAN — Task 03: Fix Duplicate Row (Daily Sheet Update)

> **State canonical:** `CURRENT_STATUS.md` (execution state + gate). File này là **master plan** — không duplicate state ở đây.
> **Ngày tạo plan:** 2026-09-08 (Claude Code) — tái cấu trúc từ bản 7-Bước (2026-09-07) sang PHASE 0–5 dựa trên audit report + `CASES/CASE-051`.
> **Hướng đã chốt với anh Lộc (2026-09-07):** gộp append/update → **1 node `appendOrUpdate` (upsert) match theo cột Key**. Yêu cầu bắt buộc: **phải backup**, **test tới khi hết lỗi mới thôi**.

---

## Root cause (tóm tắt)
Kiến trúc ghi Sheet **không idempotent ở tầng write**: quyết định append/update dựa trên **snapshot Sheet đọc lúc bắt đầu mỗi run** (`existingMap`), còn node ghi mới `operation: append` **vô điều kiện, không match Key** → **2 execution khác nhau (manual 22:00 + auto 07:30) cùng append 1 Key → 2 dòng trùng**. Node `Check Dedup trước Ghi` lọc lại chính snapshot đó nên **vô tác dụng với trùng chéo-run**; `retryOnFail` trên node append là rủi ro trùng thứ hai còn treo. Chi tiết: audit report + `CASES/CASE-051_Duplicate_Row_Cross_Run_Append.md`.

## Fix strategy (vì sao upsert)
- Key có → update; Key mới → insert → **không thể đẻ dòng trùng** dù chạy tay/auto/lại/chồng.
- Chỉ map **Nhóm A** → cột Nhóm B không map ⇒ update không ghi đè ⇒ **an toàn business data**.
- Upsert idempotent ⇒ giữ được `retryOnFail` mà retry lúc này vô hại.

## Quy ước approval trong plan này
- **CONFIRM** = anh Lộc duyệt (`⏳ CHỜ DUYỆT` → `✅ ĐÃ DUYỆT`) — theo AI_OS RULE4: AI đề xuất → Human duyệt → AI thực thi.
- **APPLY** = thao tác ghi/sửa/import/xóa **chỉ được chạy SAU khi CONFIRM**.
- **Evidence** = bằng chứng bắt buộc phải capture cho mỗi phase (screenshot / execution id / count / md5 / danh sách Key). Không có evidence ⇒ phase chưa DONE.
- Ràng buộc chung: không đụng Nhóm B · không đụng workflow khác · không in token/credential · **1 file JSON canonical duy nhất** trong `Project/WORKFLOWS/` (không đa bản/đa tên).

---

## PHASE 0 — Hotfix: tắt `retryOnFail` node append (giảm rủi ro ngay)

**Mục tiêu:** loại rủi ro trùng do retry-append (Google Sheets `append` không idempotent) trong khi chờ fix upsert.

**Các bước:**
1. Mở workflow `rz3Wya5lFay7ShVL` → node **"Ghi mới vào Google Sheet"**.
2. Tắt `retryOnFail` (từ `true` → `false`) trên node này. **Không** đụng node khác.

- **Approval gate:** `⏳ CHỜ DUYỆT` — **CONFIRM** trước khi **APPLY** (đây là sửa trực tiếp node trên production ACTIVE).
- **Evidence:** screenshot config node "Ghi mới vào Google Sheet" sau khi tắt (thấy `Retry On Fail = off`).
- **Risk:** THẤP. Chỉ tắt retry, không đổi logic ghi. Trong lúc chờ Phase 1, nếu Google trả lỗi mạng → run fail (không ghi) thay vì có nguy cơ ghi 2 lần → an toàn hơn với duplicate.
- **Rollback:** bật lại `retryOnFail=true` trên đúng node đó (1 thao tác).

---

## PHASE 1 — Sửa workflow: chuyển sang `appendOrUpdate`

**Mục tiêu:** thay toàn bộ cơ chế 2-nhánh bằng 1 node upsert idempotent.

**Các bước:**
1. **Backup:** đã có `BACKUP/task-fix-duplicate/Meta_Ads_Daily_Sheet_Update.LIVE.bak_2026-09-07T22-47-30.json` + `...repo.bak...`. Trước khi import → **export lại bản LIVE mới nhất** (phòng khi Phase 0 đã đổi node).
2. **Xóa** các node: `Check Dedup trước Ghi`, `IF: Append?`, `IF: Update?`, `Clean Append`, `Clean Update`, `Ghi mới vào Google Sheet`, `Cập nhật vào Google Sheet`.
3. **Thêm** 1 node Google Sheets **"Ghi vào Sheet"**: `operation = appendOrUpdate`, `matchingColumns = ["Key"]`, `mappingMode = autoMapInputData`.
4. **Sửa** node **"Xử lý dữ liệu"**: bỏ nhánh `__actionType` (append/update), xuất **1 loại item** map đúng bộ **Nhóm A + Key**:
   - Insert/update fields: `Ma_quang_cao, Ngay, Chien_dich, Ten_quang_cao, Ngan_sach, Trang_Thai, Key`, `Chi_tieu, Nguoi_tiep_can, click, Thoi_diem_cap_nhat, Luot_hien_thi, Tan_suat, Xem_3s, Video_25/50/75/95, ThruPlay, Click_Ra_Web, Click_All`.
   - **`Mess_Comment` (quyết định anh Lộc 2026-09-08 — phương án 3):** chỉ ghi khi `date_start` là **hôm nay hoặc hôm qua** (VN); từ **2 ngày trước trở về trước → KHÔNG map → đóng băng** (không ghi đè). Cơ chế: item bỏ field `Mess_Comment` cho ngày cũ → `autoMapInputData` giữ nguyên ô cũ.
   - **TUYỆT ĐỐI KHÔNG map (Nhóm B):** `Khach_sai_tep, Khach_hop_le, SDT, Khach_chot, Chi_phi_*, Doanh_thu, ghi_chu`.
5. **Kết nối lại flow:** `Xử lý dữ liệu → Ghi vào Sheet → Summary`. Giữ nguyên: Schedule/Manual → Đọc Sheet → Lấy cấu hình Meta → Tạo mảng ngày → Lấy dữ liệu Meta → Xử lý dữ liệu.
6. **Validate JSON:** parse JSON hợp lệ; đếm node = 9 (giảm từ 15); không còn tham chiếu tới node đã xóa trong `connections`.

- **Approval gate:** `⏳ CHỜ DUYỆT` — **CONFIRM diff** (node xóa/thêm/sửa + `matchingColumns=["Key"]` + không map Nhóm B) **trước khi APPLY import**. Import giữ `active=false` tới khi Phase 2 PASS (Project/RULES).
- **Evidence:** (a) đường dẫn backup LIVE mới; (b) diff danh sách node trước/sau; (c) kết quả validate JSON PASS (node count, connections sạch).
- **Risk:** TRUNG BÌNH. Sửa cấu trúc 7 node. Sai mapping có thể ghi thiếu/thừa cột hoặc chạm Nhóm B → mitigate bằng review diff + test Phase 2 trên tab TEST trước.
- **Rollback:** import lại `...LIVE.bak_2026-09-07T22-47-30.json` (hoặc bản export ở bước 1) → `active` như cũ → verify workflow chạy lại bình thường.

---

## PHASE 2 — Test idempotency trên tab `TEST_DEUP`

**Mục tiêu:** chứng minh không sinh trùng & không đụng Nhóm B **trước khi** chạm Sheet chính.

**Chuẩn bị:** nhân bản tab `BÁO_CÁO_QUẢNG_CÁO` → tab **`TEST_DEUP`**; trỏ node "Đọc toàn bộ Sheet" + "Ghi vào Sheet" sang `TEST_DEUP`. Giữ workflow `active=false`, chạy Manual/Execute.

| # | Test | Kỳ vọng |
|---|------|---------|
| T1 | Chạy lần 1 | Ghi nhận `appendCount` / `updateCount` (baseline) |
| T2 | Chạy lần 2 ngay sau | **`appendCount = 0`**, không tạo dòng mới |
| T3 | Nhập tay giá trị Nhóm B (Mess_Comment, SDT, Khach_chot...) vào vài dòng → chạy lại | Nhóm B **còn nguyên**, chỉ Nhóm A cập nhật |
| T4 | Simulate manual(22:00) rồi auto(07:30) trên cùng Key | **0 dòng trùng** (COUNTIF(Key)=1) |

- **Approval gate:** **tất cả T1–T4 PASS** mới được tiếp tục Phase 3. Nếu bất kỳ test FAIL → quay lại Phase 1 (Closed Loop: EVIDENCE → ROOT CAUSE → FIX), **không** đi tiếp. Max 3 auto-fix/root-cause → ESCALATE.
- **Evidence:** execution id + `appendCount/updateCount` từng lần chạy; ảnh tab `TEST_DEUP` cho T2 (không dòng mới) và T3 (Nhóm B nguyên vẹn); COUNTIF(Key) cho T4.
- **Risk:** THẤP — ghi vào tab TEST, không đụng production data.
- **Rollback:** xóa/reset tab `TEST_DEUP` (không ảnh hưởng Sheet chính).

---

## PHASE 3 — Dọn duplicate tồn đọng trên `TEST_DEUP`

**Mục tiêu:** validate quy trình dọn dup (giữ/xóa) an toàn trên tab TEST trước khi áp lên Sheet chính.

**Các bước:**
1. Script liệt kê mọi Key có `COUNTIF(Key) > 1` trên `TEST_DEUP`.
2. **Logic giữ/xóa:** ưu tiên **giữ dòng có Nhóm B có giá trị**; nếu tất cả dòng trùng đều rỗng Nhóm B → **giữ dòng mới nhất** (`Thoi_diem_cap_nhat` lớn nhất), xóa các dòng còn lại.
3. **In danh sách đề xuất giữ/xóa trước — KHÔNG xóa ngay.**

- **Approval gate:** `⏳ CHỜ DUYỆT` — anh Lộc **duyệt danh sách** → **CONFIRM** → mới **APPLY** xóa (AI_OS RULE1: không xóa khi chưa duyệt).
- **Evidence:** danh sách Key trùng + cột "giữ/xóa" + lý do; ảnh `TEST_DEUP` sau khi xóa (mọi Key COUNTIF=1).
- **Risk:** TRUNG BÌNH (thao tác xóa) — nhưng trên tab TEST nên hệ quả cô lập.
- **Rollback:** `TEST_DEUP` là bản sao → tái tạo lại từ `BÁO_CÁO_QUẢNG_CÁO` nếu sai.

---

## PHASE 4 — Chuyển sang Sheet chính (`BÁO_CÁO_QUẢNG_CÁO`) + dọn dup + production

**Mục tiêu:** áp bản đã verify lên production, dọn dup thật, giám sát ổn định.

**Các bước:**
1. Backup lại LIVE trước khi đổi (lần nữa).
2. Trỏ node "Đọc toàn bộ Sheet" + "Ghi vào Sheet" từ `TEST_DEUP` → `BÁO_CÁO_QUẢNG_CÁO`.
3. Dọn duplicate trên Sheet chính bằng **đúng script + logic Phase 3** (in danh sách → CONFIRM → xóa).
4. Import workflow production `rz3Wya5lFay7ShVL` → `Activate`.
5. Monitor **2 chu kỳ 07:30 liên tiếp** → xác nhận `appendCount=0` (sau ngày đầu), không phát sinh Key COUNTIF>1.

- **Approval gate:** 2 CONFIRM tách biệt — (a) duyệt danh sách dup Sheet chính trước khi xóa; (b) CONFIRM **sau khi monitor 2 cycle xong** mới coi production PASS.
- **Evidence:** đường dẫn backup; ảnh danh sách dup Sheet chính + sau khi xóa (COUNTIF=1); execution id 2 cycle 07:30 + `appendCount`.
- **Risk:** CAO (chạm production Sheet business + xóa dòng thật). Mitigate: đã PASS toàn bộ Phase 2–3 trên TEST; backup Sheet + workflow trước; xóa theo danh sách đã duyệt.
- **Rollback:** import lại `...LIVE.bak...json` + `active` như cũ; với Sheet, khôi phục từ Google Sheets version history / bản backup tab trước khi xóa.

---

## PHASE 5 — Fix phụ + đóng task

**Mục tiêu:** vá 2 sai lệch phụ (không phải nguyên nhân trùng) + cập nhật docs.

**Các bước:**
1. **`Lấy dữ liệu Meta` — manual slice:** `items.slice(0, 2)` lấy 2 ngày **cũ nhất** → chạy tay không cập nhật ngày mới nhất. Đổi `slice(-2)` (2 ngày mới nhất). CONFIRM riêng trước khi APPLY.
2. **Timezone:** kiểm tra `TZ`/`GENERIC_TIMEZONE` của container n8n; node `Tạo mảng ngày` dùng `new Date()` = giờ server. Nếu server ≠ UTC+7 → ép `Asia/Ho_Chi_Minh` cho biên ngày (không ảnh hưởng format Key vì Key lấy `date_start` từ Meta).
3. Export bản PASS → cập nhật **file canonical duy nhất** `Project/WORKFLOWS/Meta_Ads_Daily_Sheet_Update.json` → commit.
4. Update docs: `CURRENT_STATUS.md`, `STATUS.md`, `WORKLOG.md`, `CASE_INDEX.md`/`CASE-051`; cập nhật Module `STATUS.md` + `TASK_INDEX.md` (thêm Task 03).
5. Xóa backup tạm của task sau khi verify OK (Project/RULES — backup vòng đời 1 task).

- **Approval gate:** `⏳ CHỜ DUYỆT` cho mỗi thay đổi code (slice, timezone) trước khi APPLY.
- **Evidence:** diff node slice; giá trị `TZ` đọc được; commit hash bản canonical PASS.
- **Risk:** THẤP–TRUNG BÌNH. `slice(-2)` chỉ ảnh hưởng manual mode.
- **Rollback:** revert commit / import lại bản trước fix phụ.

---

## Rollback plan (tổng)
| Phase fail | Hành động |
|-----------|-----------|
| 0 | Bật lại `retryOnFail=true` node "Ghi mới vào Google Sheet". |
| 1 | Import lại `...LIVE.bak_2026-09-07T22-47-30.json` → verify workflow chạy như cũ. |
| 2 | Fix node theo Closed Loop rồi test lại; tab `TEST_DEUP` cô lập, reset tự do. Sau 3 lần fail cùng root cause → ESCALATE anh Lộc. |
| 3 | Không xóa khi chưa CONFIRM; tab TEST tái tạo được. |
| 4 | Import lại LIVE backup + khôi phục Sheet từ version history/bản backup tab. |
| 5 | Revert commit / import bản trước fix phụ. |

## Definition of Done
- [ ] PHASE 0: `retryOnFail` node append = off (evidence screenshot).
- [ ] PHASE 1: workflow còn 1 node ghi `appendOrUpdate` matchingColumns=`["Key"]`; JSON validate PASS; không map Nhóm B.
- [ ] PHASE 2: T1–T4 PASS — chạy 2 lần liên tiếp `appendCount=0`; Nhóm B nguyên vẹn; T4 không sinh trùng.
- [ ] PHASE 3: `TEST_DEUP` mọi Key COUNTIF=1 (theo danh sách đã CONFIRM).
- [ ] PHASE 4: Sheet chính mọi Key COUNTIF=1; production active; **2 cycle 07:30 liên tiếp** sạch (appendCount=0, không Key mới trùng).
- [ ] PHASE 5: slice + timezone xử lý xong; bản canonical PASS đã commit; docs + Module STATUS/TASK_INDEX cập nhật; backup tạm đã xóa.
- **Task DONE khi:** toàn bộ checkbox trên PASS **và** anh Lộc CONFIRM production ổn định sau 2 chu kỳ.

---

## PHASE 5b — Giữ & Document workflow highlight duplicate (thêm 2026-09-12)

> Bổ sung sau PHASE 5. Không sửa các phase cũ. Mục tiêu: biến workflow bôi màu tạm (dùng trong PHASE 3/4) thành **tool tái sử dụng** và lưu vào AI OS.
> Chỗ lưu đã chốt với anh Lộc: **(A) lightweight** — `AI_OS/skills/highlight_duplicate_rows/README.md` (không có thư mục `Tools/`; `AI_OS/skills/` là tương đương).

### Step 1 — Chọn & giữ 1 workflow màu tốt nhất
- Kiểm 3 workflow màu đã tạo: `QWowp4oT9PA1iJTO` (COLOR v1), `1eFjFdCjXZRCRV0x` (COLOR v2), `ORFWlIrApPASnALp` (COLOR final).
- **Tiêu chí giữ:** bản logic đầy đủ nhất = **reset màu cũ (trắng) + bôi vàng + bôi cam** trong 1 batch. (COLOR v2 `1eFjFdCjXZRCRV0x` có cả reset trắng + vàng; COLOR v1 có vàng+cam; final chỉ 2 dòng. → ứng viên: hợp nhất logic thành 1 bản chuẩn, đọc input từ file thay vì hardcode row.)
- Đổi tên bản giữ → **"TOOL — Highlight Duplicate Rows"**; đảm bảo `active=false`; đổi input sang đọc `/tmp/dup_highlight.json` (`[{"row":N,"color":"yellow|orange|white"}]`).
- 2 bản còn lại: đổi tên tiền tố `[XOA]` (hoặc xoá trong UI — CLI không có `delete:workflow`).
- **Approval gate:** ⏳ CONFIRM **id bản giữ lại** trước khi đổi tên/sửa.
- **Evidence:** id + tên sau đổi; ảnh/loại xác nhận `active=false` + node input trỏ `/tmp/dup_highlight.json`.
- **Risk:** THẤP (workflow manual, không schedule, không tự chạy).
- **Rollback:** các bản màu đều là artifact tạm; import lại từ `scratchpad/Task03_color_*.json` nếu cần.

### Step 2 — Tạo file document trong AI OS
- Tạo `AI_OS/skills/highlight_duplicate_rows/README.md` + thêm 1 dòng index vào `AI_OS/SKILLS.md` và `AI_OS/skills/README.md`.
- **Nội dung bắt buộc của README.md:**
  ```
  # TOOL: Highlight Duplicate Rows — Google Sheets

  ## Mục đích
  Bôi màu các dòng duplicate trên Google Sheet để review trực quan trước khi xóa.
  Dùng khi phát hiện trùng dữ liệu và cần kiểm tra mắt trước khi thực thi xóa hàng loạt.

  ## Workflow n8n
  - ID: [id bản giữ lại]
  - Tên: TOOL — Highlight Duplicate Rows
  - Trigger: Manual (không schedule)
  - Project gốc: AI_Meta_n8n_autoamation

  ## Màu quy ước
  - Vàng nhạt #FFF9C4 — dòng sẽ XÓA (cần review)
  - Cam nhạt #FFE0B2 — dòng cần GỘP / xử lý đặc biệt
  - Trắng   #FFFFFF — reset màu

  ## Input file chuẩn
  /tmp/dup_highlight.json
  [{"row":123,"color":"yellow"},{"row":456,"color":"orange"}]

  ## Tham số cần đổi khi dùng lại
  - spreadsheetId : ID Sheet target
  - sheetName / gid : tab target
  - File JSON input: /tmp/dup_highlight.json

  ## Cách chạy
  1. Chuẩn bị /tmp/dup_highlight.json (danh sách row + color)
  2. Trigger manual trong n8n UI
  3. Mở Sheet kiểm tra trực quan
  4. Xác nhận → tiến hành xóa

  ## Lịch sử dùng
  | Ngày | Task | Sheet target | Kết quả |
  |------|------|--------------|---------|
  | 2026-09 | Task_03_Fix_Duplicate_Daily_Sheet | BÁO_CÁO_QUẢNG_CÁO | Review 62→46 dup trước khi xóa |
  ```
- **Approval gate:** ⏳ CONFIRM **nội dung file** (nhất là `[id bản giữ lại]`) trước khi tạo.
- **Evidence:** path file thực tế + dòng index đã thêm vào `SKILLS.md`/`skills/README.md`.

### Step 3 — Cập nhật CURRENT_STATUS.md
- Ghi nhận: TOOL `highlight_duplicate_rows` đã document vào AI OS; ghi **path thực tế**.

### Definition of Done (PHASE 5b)
- [ ] Chỉ còn **1 workflow màu**, đã đổi tên **"TOOL — Highlight Duplicate Rows"**, `active=false`, input đọc `/tmp/dup_highlight.json`.
- [ ] File `AI_OS/skills/highlight_duplicate_rows/README.md` tồn tại (đúng nội dung) + index cập nhật trong `SKILLS.md` & `skills/README.md`.
- [ ] `CURRENT_STATUS.md` cập nhật (đã document tool + path).

### Ràng buộc thực thi
- Dừng tại **mỗi approval gate** (Step 1 id giữ lại, Step 2 nội dung file). Không tự chạy PHASE 5b khi chưa CONFIRM.
