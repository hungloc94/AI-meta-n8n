# PATTERN-003: Daily Sync Meta → Sheet Append/Update By Key

## Mục đích
Đồng bộ dữ liệu Meta Ads vào Google Sheet hằng ngày.
Idempotent: chạy nhiều lần không tạo duplicate.

## Cấu trúc node
Schedule Trigger / Manual Trigger
→ Đọc toàn bộ Sheet
→ Lấy cấu hình Meta
→ Tạo mảng ngày (numDays=7)
→ Gọi Meta API từng ngày
→ Xử lý dữ liệu
→ IF Append? → Clean → Ghi mới → Summary
→ IF Update? → Clean → Cập nhật → Summary

## Code mẫu

### Tạo mảng ngày
const numDays = 7;
const tz = 'Asia/Ho_Chi_Minh';
const dates = [];
for (let i = 1; i <= numDays; i++) {
  const d = new Date();
  d.setDate(d.getDate() - i);
  dates.push(d.toLocaleDateString('en-CA', {timeZone: tz}));
}
return dates.map(date => ({json: {date}}));

### Safe Count Summary
const items = $input.all();
const appended = items.filter(i =>
  i.json.operation === 'append').length;
const updated = items.filter(i =>
  i.json.operation === 'update').length;
return [{json: {
  append: appended,
  update: updated,
  total: appended + updated,
  summary: `Append: ${appended} | Update: ${updated}`
}}];

## Lưu ý quan trọng
- numDays=7 cho production (attribution window Meta)
- Key = ad_id + "_" + date — match để tránh duplicate
- Chỉ ghi Nhóm A — tuyệt đối không đụng Nhóm B
- Dùng Map Each Column Manually, không autoMapInputData
- Chạy lần 2 phải Append=0 — verify bằng COUNTIF(Key)
- safeCount tránh lỗi khi nhánh không chạy

## CASE liên quan
CASE-010, CASE-011, CASE-016, CASE-024, CASE-028, CASE-033, CASE-041