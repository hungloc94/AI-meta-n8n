# CASE-051: Duplicate Row — Cross-Run Append (Daily Sheet Update)

- **Ngày phát hiện:** 2026-09-07
- **Ngày xác minh:** 2026-09-07 (evidence cột T do anh Lộc cung cấp)
- **Mức độ ảnh hưởng:** ACTIVE production (workflow `rz3Wya5lFay7ShVL`, ghi vào Sheet business)
- **Đã báo Human:** 2026-09-07 — đã báo và chốt hướng fix với anh Lộc trong cùng phiên

### Vấn đề
Workflow Daily Sheet Update ghi **2 dòng trùng cùng Key** vào Sheet thay vì update 1 dòng.
Evidence: 2 dòng có `Thoi_diem_cap_nhat` (cột T) = 22:00 (anh Lộc chạy tay xem chỉ số) và 07:30
(auto hôm sau).

### Nguyên nhân
Cơ chế chống trùng tách 2 nhánh append/update, dựa trên `existingMap`/`Check Dedup trước Ghi`,
nhưng cả hai chỉ đối chiếu với **snapshot Sheet lúc bắt đầu của chính lần chạy đó**.
Khi có **2 execution khác nhau** (manual 22:00 rồi auto 07:30), mỗi run tự append cho Key mà
tại thời điểm nó phân loại chưa "nhìn thấy" là đã tồn tại theo đúng nhánh update → **2 dòng**.

Không phải do append-retry (giả thuyết ban đầu): 2 timestamp khác nhau = 2 lần chạy, không phải
retry trong cùng 1 lần. Node `Check Dedup trước Ghi` (thêm ở commit c9aff03, 2026-08-11) chạy
trước node append nên không phủ được tình huống cross-run/ retry nội bộ node.

### Cách xử lý
Gộp append + update thành **1 node `appendOrUpdate` (upsert)** `matchingColumns=["Key"]`:
Key có → update; Key mới → insert. Idempotent với mọi nguồn (tay/auto/lại/chồng).
Chỉ map **Nhóm A** → Nhóm B không bị ghi đè. Chi tiết: `../PLAN.md`.

### CẬP NHẬT ROOT CAUSE THẬT — 2026-09-09 (quan trọng)
Sau khi apply upsert mà **vẫn thấy trùng ngày mới**, quét toàn bộ workflow n8n phát hiện: có **HAI workflow cùng ghi Sheet production**, cùng lịch **07:30**:
- `rz3Wya5lFay7ShVL` — "Meta Ads Daily Sheet Update 7:30 2" (canonical).
- `T0x1qzedADrmj1LA` — **"TEST Daily Sheet Update Task09"** — bản clone TEST từ task09 (11/08), **để active=TRUE**, dùng node cũ `Ghi mới (append)`.

→ **Root cause thật:** hai workflow active cùng chạy 07:30, cả hai cùng append ⇒ **ngày nào cũng trùng**, độc lập với giả thuyết "manual 22:00 + auto 07:30". Đây là lý do "sửa upsert rồi vẫn trùng": bản fix bị để `active=false`, còn bản clone lỗi vẫn nắm quyền chạy.

**Xử lý (2026-09-09):** `update:workflow` → tắt `T0x1qzedADrmj1LA`, bật `rz3Wya5lFay7ShVL`; **restart container** n8n để scheduler nạp lại (đổi active chỉ hiệu lực sau restart). Verify: chỉ còn 1 workflow production active = bản upsert.

**Bài học bổ sung:** khi điều tra trùng, PHẢI liệt kê **mọi workflow active ghi cùng Sheet/tab**, không chỉ workflow canonical. Bản TEST/clone để active là bẫy kinh điển.

### Bài học
- Dedup "đọc-rồi-ghi" theo từng run KHÔNG idempotent giữa nhiều execution — luôn dùng upsert
  match key ở tầng ghi.
- Append của Google Sheets không idempotent; chỉ nên retry trên thao tác upsert/update.
- Khi điều tra: evidence timestamp (cột T) phân biệt "retry cùng run" vs "2 run" — hỏi Human sớm.
