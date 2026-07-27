# CASE-011: Protected Columns — Nhóm B không được ghi đè

## Vấn đề
Workflow sync Meta có thể vô tình xóa dữ liệu bán hàng, đánh giá lead, và công thức tính chi phí.

## Nguyên nhân
Sheet có 2 nhóm cột:
- **Nhóm A** (Meta được ghi): `Ma_quang_cao`, `Ngay`, `Chien_dich`, `Ten_quang_cao`, `Chi_tieu`, `Nguoi_tiep_can`, `Ngan_sach`, `click`, `Mess_Comment`, `Trang_Thai`, `Key`, `Thoi_diem_cap_nhat`
- **Nhóm B** (PROTECTED): `Khach_sai_tep`, `Khach_hop_le`, `SDT`, `Khach_chot`, `Chi_phi_tren_khach_hop_le`, `Chi_phi_khach_tren_SDT`, `Chi_phi_khach_chot`, `ghi_chu`

Nhóm B do người vận hành quản lý. Ghi đè = mất data vĩnh viễn.

## Cách xử lý
- Mọi workflow sync Meta chỉ cập nhật Nhóm A.
- Không reset, ghi đè, hoặc xóa Nhóm B trừ khi có phê duyệt rõ ràng.
- Bất kỳ thay đổi nào ảnh hưởng Nhóm B đều cần backup Sheet trước.

## Bài học
- Mapping mode phải là `Map Each Column Manually` — không dùng `Map Automatically`.
- Khi đổi credential, n8n có thể reset mapping → phải verify lại.
- Cập nhật 2026-06-02: `Chien_dich`, `Ten_quang_cao`, `Mess_Comment` xác nhận thuộc Nhóm A.

## Status
Accepted — cập nhật 2026-06-02.

## Xem thêm
- CASE-041: autoMapInputData có thể ghi đè   Nhóm B mà không báo lỗi — rebuild phải map tay