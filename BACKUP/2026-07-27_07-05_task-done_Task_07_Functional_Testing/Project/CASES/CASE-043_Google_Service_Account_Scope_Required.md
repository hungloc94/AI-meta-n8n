# CASE-043: Google Service Account — Phải khai báo Scope

- **Ngày phát hiện:** 2026-07-26
- **Ngày xác minh:** 2026-07-26

## Vấn đề
Node HTTP Request dùng Google Service Account API bị lỗi 403:
"Method doesn't allow unregistered callers"
Dù credential đã tạo và map vào node đúng.

## Nguyên nhân
n8n yêu cầu khai báo rõ scope trong credential Google Service Account.
Nếu không có scope → API call không có identity → Google từ chối 403.

## Cách xử lý
1. Vào credential Google Service Account → Edit
2. Thêm scope: https://www.googleapis.com/auth/spreadsheets
3. Nếu cần đọc cả Drive: https://www.googleapis.com/auth/drive.readonly
4. Save credential
5. Chạy lại node → PASS

## Bài học
Khi tạo credential Google Service Account trong n8n:
- Luôn thêm scope ngay khi tạo — đừng bỏ trống
- Scope tối thiểu cho Google Sheets: https://www.googleapis.com/auth/spreadsheets
- Cảnh báo vàng "Make sure you have specified the scope(s)" = chưa có scope → phải thêm
