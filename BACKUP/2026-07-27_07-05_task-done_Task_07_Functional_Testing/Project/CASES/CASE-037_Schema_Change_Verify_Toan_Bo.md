# CASE-037: Schema Change Cần Verify Toàn Bộ Chuỗi Đọc

## Vấn đề
Vừa thêm cột mới (Click_All) vào Google Sheet. Node parser báo "THIẾU CỘT BẮT BUỘC: Click_All". Sau khi execute lại, lỗi tự hết.

## Nguyên nhân
Root cause chính xác **chưa được xác minh** bằng bằng chứng runtime đầy đủ. Không kết luận là do cache hay timing khi chưa có bằng chứng cụ thể.

## Cách xử lý
Sau khi thay đổi schema Google Sheet (thêm/xóa/đổi tên cột), verify theo thứ tự:
1. Execute riêng node đọc Sheet → xem output có cột mới chưa
2. Execute riêng node parser → xem header row parser nhận được là gì
3. Kiểm tra `requiredHeaders` trong code có khớp tên cột thật trong Sheet không
4. Chỉ kết luận PASS/FAIL dựa trên kết quả thực tế, không đoán nguyên nhân

## Bài học
- Schema change = verify toàn bộ chuỗi đọc dữ liệu, không chỉ node thay đổi.
- Debug hiệu quả: thêm tạm `throw new Error(JSON.stringify({...}))` ngay sau bước nghi ngờ để lấy bằng chứng runtime.
- Không suy luận từ các lần execute trước đó — chỉ tin kết quả execute hiện tại.

## Status
Lesson learned — 2026-06-30.
