# PATTERN-001: Scheduled Report Sheet → Normalize → KPI → Telegram

## Mục đích
Báo cáo định kỳ khi dữ liệu đã ổn định trong Google Sheet.
Phù hợp báo cáo hôm qua hoặc lịch sử.

## Cấu trúc node
Schedule Trigger
→ Tính ngày báo cáo
→ Đọc Google Sheet
→ Normalize Sheet Data
→ Lọc dữ liệu theo ngày
→ Tính KPI từ raw metrics
→ Render message Telegram
→ Send Report Telegram

## Code mẫu

### Normalize Sheet Data
const rows = $input.all();
return rows.map(row => ({
  json: {
    ngay: row.json.Ngay,
    chi_tieu: parseFloat(row.json.Chi_tieu) || 0,
    nguoi_tiep_can: parseFloat(row.json.Nguoi_tiep_can) || 0,
    click: parseFloat(row.json.click) || 0,
    luot_hien_thi: parseFloat(row.json.Luot_hien_thi) || 0,
    key: row.json.Key
  }
}));

### Tính KPI
const spend = $json.chi_tieu;
const impressions = $json.luot_hien_thi;
const clicks = $json.click;
return [{json: {
  ...$json,
  cpm: impressions > 0 ? (spend/impressions*1000).toFixed(0) : 0,
  ctr: impressions > 0 ? (clicks/impressions*100).toFixed(2) : 0,
  cpc: clicks > 0 ? (spend/clicks).toFixed(0) : 0
}}];

## Lưu ý quan trọng
- Sheets API trả raw array → bắt buộc normalize
- Date ISO YYYY-MM-DD, timezone Asia/Ho_Chi_Minh
- KPI tính từ raw metrics, không dùng cột công thức
- Scheduled dùng $env.TELEGRAM_CHAT_ID
- Node Telegram phải bật retry
- KPI=0 → phân biệt business zero vs pipeline failure

## CASE liên quan
CASE-012, CASE-013, CASE-020, CASE-023, CASE-036, CASE-038