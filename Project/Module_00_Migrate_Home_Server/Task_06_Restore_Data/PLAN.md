# Plan — Task 06: Restore Data

| Bước | Mô tả | Kết quả mong đợi |
|------|-------|-------------------|
| 1 | Restore file .env vào project directory trên Home Server | .env khớp template Task 04, đủ biến |
| 2 | Import toàn bộ workflow JSON từ backup Task 02, xác nhận `active=false` | Đủ workflow, tất cả inactive |
| 3 | Recreate credential Telegram | Credential hoạt động, chưa gán vào node |
| 4 | Recreate credential Meta Graph (Header Auth) | Credential hoạt động |
| 5 | Re-authorize Google Sheets OAuth2 (hoặc Service Account nếu đã migrate) | Credential hoạt động |
| 6 | Verify credentials decrypt đúng, không lỗi N8N_ENCRYPTION_KEY | Không có lỗi decrypt |
| 7 | Đối chiếu toàn bộ workflow + credential đã restore với inventory Task 01 | Checklist khớp 100%, không thiếu node nào (tránh CASE-025) |
