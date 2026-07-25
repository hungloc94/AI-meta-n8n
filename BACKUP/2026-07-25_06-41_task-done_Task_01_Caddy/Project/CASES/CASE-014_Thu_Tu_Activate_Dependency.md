# CASE-014: Thứ tự Activate phải đảm bảo Dependency

## Vấn đề
Report chạy trước Sync → báo cáo dùng dữ liệu cũ hoặc thiếu.

## Nguyên nhân
`Meta Report Yesterday` đọc dữ liệu từ Sheet. Nếu `Meta Ads Daily Sheet Update` chưa chạy xong thì Sheet chưa có dữ liệu mới nhất → báo cáo sẽ thiếu hoặc sai.

## Cách xử lý
- Sync (07:30) phải chạy TRƯỚC Report (08:13).
- Khoảng cách tối thiểu khuyến nghị: 30 phút.
- Không activate Report trước Sync.

## Bài học
- Nếu Sync chạy quá chậm hoặc lỗi, Report 08:13 vẫn chạy nhưng dữ liệu có thể thiếu.
- Cần monitor cả hai workflow — không chỉ Report.
- Health Check 07:00/07:05 giúp phát hiện sớm nếu credential fail trước giờ Sync.

## Status
Accepted — xác minh 2026-06-09.
