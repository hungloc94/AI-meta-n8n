# Lịch Workflow — n8n Meta Ads

## Thứ tự chạy hằng ngày

| Giờ | Workflow | Phụ thuộc |
|-----|----------|-----------|
| 07:00 | Google Sheets Health Check | Không có |
| 07:05 | Meta API Health Check | Không có |
| 07:30 | Meta Ads Daily Sheet Update | GS Health PASS |
| 08:13 | Yesterday Report | Daily Sync PASS |
| 11:31 | Today Report | Meta Health PASS |
| 16:31 | Today Report | Meta Health PASS |
| 21:13 | Today Report | Meta Health PASS |

## Quy tắc dependency
- Health Check FAIL → không chạy workflow phụ thuộc
- Daily Sync phải xong trước Yesterday Report
- Today Report gọi Meta API trực tiếp — không phụ thuộc Daily Sync

## Workflow đang lỗi
- Meta Report VERIFIED — ⚠️ ĐANG LỖI, không activate