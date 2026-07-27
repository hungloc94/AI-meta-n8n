# CASE-029: Verify Update Row Đúng Chuẩn

## Vấn đề
Execute thành công nhưng không biết node có thực sự ghi dữ liệu không. Dữ liệu cũ = dữ liệu mới nên không thấy thay đổi — tạo false confidence.

## Nguyên nhân
Khi data trong Sheet đã đúng sẵn, Update Row sẽ "ghi đè" cùng giá trị → output trông giống thành công nhưng không chứng minh được node thật sự ghi.

## Cách xử lý
Quy trình verify chuẩn:
1. Sửa tay 1 giá trị trong Sheet (ví dụ: `Chi_tieu` = 999999)
2. Execute workflow
3. Xác nhận giá trị quay về số gốc
4. Kiểm tra Version History Google Sheets để có bằng chứng rõ ràng

## Bài học
- "Execute thành công" không đủ — phải verify bằng thay đổi dữ liệu thực tế.
- Version History Google Sheets là bằng chứng tốt nhất.
- Áp dụng cho mọi node ghi dữ liệu, không chỉ Update Row.

## Status
Lesson learned — 2026-06-22.
