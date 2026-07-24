# CASE-015: Exact Match cho action_type Meta API

## Vấn đề
KPI Mess_Comment bị inflate nghiêm trọng — thực tế = 1, hệ thống báo = 18. Toàn bộ lịch sử từ 24/03 → 10/06 bị sai.

## Nguyên nhân
Fuzzy matching `includes('message')` vô tình gom nhiều action_type không mong muốn từ Meta API:
- `messaging_conversation_replied_7d`
- `messaging_welcome_message_view`
- `total_messaging_connection`
- `messaging_first_reply`
- và nhiều action khác chứa keyword "message"/"conversation"

Ads Manager chỉ hiển thị `onsite_conversion.messaging_conversation_started_7d` — duy nhất action_type này.

## Cách xử lý

### Patch workflow:
- Thay toàn bộ fuzzy matching bằng exact match `===` trong 3 workflow:
  - `META_REPORT_TODAY_SCHEDULED`
  - `META_ADS_DAILY_SHEET_UPDATE`
  - `META_ADS_BACKFILL`

### Backfill lịch sử:
- Tạo workflow backfill riêng với exact match
- Backfill 4 giai đoạn: 24/03→31/03, 01/04→30/04, 01/05→31/05, 01/06→10/06
- Verify: toàn bộ appendCount=0, updateCount>0 (không tạo trùng)
- Spot check: 01/04 trước=18, sau=6, Ads Manager=6 ✅

## Bài học
- Luôn dùng exact match `===` cho action_type — không dùng `includes()` hay `startsWith()`.
- Silent data inflation (số quá cao) nguy hiểm hơn silent zero — trông có vẻ "có data" nên không bị challenge.
- Trước khi thêm metric mới, kiểm tra Ads Manager để xác định chính xác action_type.
- Backfill thành công = appendCount=0 + spot check nhiều ngày khác nhau.
- Sau mỗi patch → chạy COUNTIF(Key) detect duplicate ngay.

## Status
Resolved — 2026-06-11. Toàn bộ lịch sử đã backfill và verify.
