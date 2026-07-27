# PATTERN-006: Schedule Dependency Chain

## Mục đích
Đảm bảo các workflow chạy đúng thứ tự phụ thuộc.
Tránh Report chạy khi Sync chưa xong hoặc credential lỗi.

## Thứ tự chuẩn
07:00 Google Sheets Health Check
  ↓ PASS
07:05 Meta API Health Check
  ↓ PASS
07:30 Meta Ads Daily Sheet Update
  ↓ PASS (buffer 43 phút)
08:13 Yesterday Report

11:31 Today Report (độc lập)
16:31 Today Report (độc lập)
21:13 Today Report (độc lập)

## Quy tắc dependency
- Health Check FAIL → không chạy workflow phụ thuộc
- Daily Sync phải xong trước Yesterday Report
  buffer tối thiểu 30 phút (07:30 → 08:13)
- Today Report gọi Meta API trực tiếp
  không phụ thuộc Daily Sync hay Sheet

## Lưu ý quan trọng
- Mỗi workflow dùng Schedule Trigger riêng
  không dùng chung trigger cho 2 workflow
- Health Check phải Stop And Error khi FAIL
  để workflow sau không chạy theo lịch
- Không đặt lịch quá sát nhau
  Sync cần thời gian xử lý 7 ngày x N ads
- Today Report không cần đợi Sync
  vì lấy dữ liệu trực tiếp từ Meta API

## CASE liên quan
CASE-012, CASE-013, CASE-014, CASE-039