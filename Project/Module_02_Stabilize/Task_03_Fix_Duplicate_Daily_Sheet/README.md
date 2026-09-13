# Task 03 — Fix Duplicate Row: Meta Ads Daily Sheet Update

## Mục tiêu
Làm cho workflow **Meta Ads Daily Sheet Update** (`rz3Wya5lFay7ShVL`, ACTIVE 07:30) **idempotent tuyệt đối**:
chạy tay hoặc auto, chạy lại nhiều lần, chạy chồng lần — đều **không sinh dòng trùng** trên Google Sheet
và **không đụng Nhóm B** (dữ liệu business nhập tay).

## Phạm vi
**Trong scope:**
- Sửa cơ chế ghi Sheet của đúng workflow Daily Sheet Update.
- Dọn các dòng đang bị trùng Key trên Sheet (COUNTIF(Key) > 1).
- Rà điểm phụ: manual mode đang `slice(0,2)` lấy 2 ngày cũ nhất.

**Ngoài scope:**
- Không đụng workflow khác (Yesterday/Today/Health Check/VERIFIED).
- Không đổi schema cột Sheet.
- Không đổi token/credential (thuộc Task 01).

## Bối cảnh — sự cố
- Hệ thống chống trùng dựa trên **Key ở cột S** (`{date_start}_{ad_id}`), tách 2 nhánh append/update.
- Sự cố: 1 Key xuất hiện **2 dòng** — 1 dòng cập nhật lúc 22:00 (anh Lộc chạy tay để xem chỉ số) + 1 dòng lúc 07:30 (auto hôm sau).
- Tức là **2 lần chạy khác nhau cùng APPEND 1 Key** — dedup không bắt được dòng do lần chạy trước tạo.

## Kết quả mong đợi
| Kết quả | Tiêu chí thành công |
|---------|---------------------|
| Idempotent | Chạy 2 lần liên tiếp: lần 2 `appendCount = 0` |
| Không trùng | Mọi Key trên Sheet có COUNTIF = 1 |
| Nhóm B an toàn | Mess_Comment, Khach_sai_tep, SDT, Khach_chot... không bị ghi đè |
| Production ổn | 7/7 workflow active, chu kỳ 07:30 chạy sạch |
