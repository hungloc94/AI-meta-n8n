# CASE-028: Đổi Credential Reset Mapping Config

## Vấn đề
Hai triệu chứng sau khi đổi credential Google Sheets:
1. Update Row fail: "Column to Match On parameter is required"
2. Update Row trả "No output data returned" — bị nhầm là lỗi credential

## Nguyên nhân
n8n reset các field cấu hình khi đổi credential:
- Column to match on → trống
- Mapping mode → reset
- Values to Update / Values to Send → trống
- Sheet target có thể đổi

"No output data returned" không phải lỗi credential — node tìm được dòng nhưng không có field nào để update vì mapping bị reset.

## Cách xử lý
Checklist bắt buộc sau mỗi lần đổi credential Google Sheets:
- ☐ Column to match on = `Key`
- ☐ Mapping mode = `Map Each Column Manually`
- ☐ Values to Update / Values to Send không trống (chỉ map Nhóm A)
- ☐ Sheet target đúng (production vs TEST)

## Bài học
- "No output data" ≠ lỗi xác thực — kiểm tra "Values to Update" trước khi nghi credential.
- Migration credential không chỉ là đổi loại — phải reconfigure toàn bộ mapping.
- Checklist này áp dụng cho mọi lần đổi credential, không chỉ migration.

## Status
Lesson learned — 2026-06-22.

## Xem thêm
- CASE-041: legacy workflow dùng autoMapInputData
  đặc biệt nguy hiểm khi đổi credential vì   mapping bị reset silent