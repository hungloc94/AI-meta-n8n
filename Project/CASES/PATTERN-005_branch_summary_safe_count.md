# PATTERN-005: Branch Summary với Safe Count

## Mục đích
Tổng hợp kết quả từ nhiều nhánh IF mà không bị lỗi
khi một nhánh không chạy.

## Cấu trúc node
IF Node
→ Nhánh True  → Xử lý → Merge
→ Nhánh False → Xử lý → Merge
→ Summary Node (safe count)

## Code mẫu

### Safe Count Summary
const items = $input.all();
const appended = items.filter(i =>
  i.json.operation === 'append').length;
const updated = items.filter(i =>
  i.json.operation === 'update').length;
const skipped = items.filter(i =>
  i.json.operation === 'skip').length;
return [{json: {
  total: items.length,
  appended,
  updated,
  skipped,
  summary: `Append: ${appended} | Update: ${updated} | Skip: ${skipped}`
}}];

## Lưu ý quan trọng
- Không dùng $items().length trực tiếp
  khi nhánh có thể rỗng → lỗi runtime
- Luôn filter theo operation type để đếm an toàn
- Log summary để debug sau khi chạy
- Merge node phải nhận từ tất cả nhánh

## CASE liên quan
CASE-033, CASE-030