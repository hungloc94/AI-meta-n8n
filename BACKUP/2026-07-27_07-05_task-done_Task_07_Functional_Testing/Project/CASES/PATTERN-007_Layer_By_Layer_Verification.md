# PATTERN-007: Layer-by-Layer Verification

## Mục đích
Verify từng layer riêng biệt trước khi kết luận lỗi.
Tránh blame nhầm layer, tiết kiệm thời gian debug.

## 4 Layer cần verify
Layer 1: Data Source
  → Meta API trả đúng data chưa?
  → Google Sheet đọc đúng range chưa?
  → Field names đúng chưa?

Layer 2: Normalize
  → Data sau normalize đúng type chưa?
  → Null/undefined được handle chưa?
  → parseFloat trả 0 thay vì NaN chưa?

Layer 3: KPI Engine
  → Công thức tính đúng chưa?
  → Divide by zero được handle chưa?
  → Output đúng format chưa?

Layer 4: Presentation
  → Message render đúng chưa?
  → Telegram nhận được chưa?
  → Chat ID đúng chưa?

## Code mẫu

### Verify Data Source
const fields = Object.keys($json);
const sample = JSON.stringify($json, null, 2);
console.log('Fields:', fields);
console.log('Sample:', sample);
return [{json: {fields, sample}}];

### Verify Normalize
const required = [
  'chi_tieu','nguoi_tiep_can',
  'click','luot_hien_thi','mess_comment'
];
const missing = required.filter(f =>
  $json[f] === undefined || $json[f] === null
);
if (missing.length > 0) {
  throw new Error(`Missing fields: ${missing.join(', ')}`);
}
return [{json: $json}];

### Verify KPI Engine
const spend = $json.chi_tieu;
const impressions = $json.luot_hien_thi;
if (spend === 0 && impressions === 0) {
  console.log('WARNING: Both spend and impressions are 0');
  console.log('Check Layer 1 (Data Source) first');
}
return [{json: $json}];

## Lưu ý quan trọng
- Dùng Execute Step để isolate từng layer
- Sau Execute Step → real runtime activation để verify
- KPI=0 không có nghĩa là lỗi
  → check Layer 1 trước khi kết luận
- Không fix Layer 3 khi Layer 1 chưa verify
- Ghi lại kết quả verify từng layer vào WORKLOG

## CASE liên quan
CASE-020, CASE-030, CASE-036, CASE-038