# PATTERN-001: Scheduled Report From Sheet → Normalize → KPI → Telegram

## Mục đích
Dùng cho workflow báo cáo định kỳ khi dữ liệu đã ổn định trong Google Sheet. Pattern phù hợp với báo cáo “hôm qua” hoặc báo cáo lịch sử, nơi Sheet là nguồn sự thật vận hành.

## Cấu trúc node
```text
Schedule Trigger
→ Tính ngày báo cáo
→ Đọc Google Sheet
→ Normalize Sheet Data
→ Lọc dữ liệu theo ngày
→ Tính KPI từ raw metrics
→ Render message Telegram
→ Send Report Telegram
```

## Code mẫu
Trích từ workflow `Meta Report Yesterday Scheduled 1`.

### Schedule
```json
{
  "name": "Schedule 08:13",
  "type": "n8n-nodes-base.scheduleTrigger",
  "parameters": {
    "rule": {
      "interval": [
        { "field": "cronExpression", "expression": "13 8 * * *" }
      ]
    }
  }
}
```

### Tính ngày hôm qua theo timezone Việt Nam
```javascript
const vnNow = new Date(new Date().toLocaleString('en-US', { timeZone: 'Asia/Ho_Chi_Minh' }));
const yesterday = new Date(vnNow);
yesterday.setDate(vnNow.getDate() - 1);

const yyyy = yesterday.getFullYear();
const mm = String(yesterday.getMonth() + 1).padStart(2, '0');
const dd = String(yesterday.getDate()).padStart(2, '0');

return [{ json: { reportDate: `${yyyy}-${mm}-${dd}` } }];
```

### Đọc Google Sheet qua Service Account
```json
{
  "name": "Đọc Google Sheet",
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "url": "https://sheets.googleapis.com/v4/spreadsheets/<SPREADSHEET_ID>/values/BÁO_CÁO_QUẢNG_CÁO",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "googleApi",
    "options": {}
  }
}
```

### Normalize Sheet Data
```javascript
const values = $('Đọc Google Sheet').first().json.values || [];

function toNumber(value) {
  const raw = String(value ?? '').trim();
  if (!raw || raw === '#VALUE!' || raw === '#REF!') return 0;
  const normalized = raw.replace(/\./g, '').replace(',', '.');
  const n = Number(normalized);
  return Number.isFinite(n) ? n : 0;
}

if (values.length === 0) {
  return [{ json: { records: [], rowCount: 0, headers: [], headerMap: {} } }];
}

const headerRow = values[0];
const headerMap = {};
headerRow.forEach((h, i) => {
  const name = String(h).trim();
  if (name) headerMap[name] = i;
});

const requiredHeaders = [
  "Ma_quang_cao", "Ngay", "Chi_tieu", "Mess_Comment",
  "Khach_hop_le", "SDT", "Khach_chot", "Doanh_thu",
  "Luot_hien_thi", "Click_All"
];

for (const h of requiredHeaders) {
  if (headerMap[h] === undefined) throw new Error(`Thiếu cột bắt buộc trong Sheet: ${h}`);
}

const col = (row, name) => headerMap[name] === undefined ? '' : row?.[headerMap[name]] ?? '';
const colNum = (row, name) => headerMap[name] === undefined ? 0 : toNumber(row[headerMap[name]]);
const dataRows = values.slice(1);

const records = dataRows.map(row => ({
  ma_quang_cao: col(row, 'Ma_quang_cao'),
  ngay: col(row, 'Ngay'),
  chi_tieu: colNum(row, 'Chi_tieu'),
  mess_comment: colNum(row, 'Mess_Comment'),
  khach_hop_le: colNum(row, 'Khach_hop_le'),
  sdt: colNum(row, 'SDT'),
  khach_chot: colNum(row, 'Khach_chot'),
  doanh_thu: colNum(row, 'Doanh_thu'),
  luot_hien_thi: colNum(row, 'Luot_hien_thi'),
  click: colNum(row, 'click'),
  click_all: colNum(row, 'Click_All'),
  key: col(row, 'Key')
}));

return [{ json: { headers: headerRow, headerMap, records, rowCount: records.length } }];
```

### Lọc ngày, tính KPI và render Telegram
```javascript
const records = $('Normalize Sheet Data').first().json.records || [];
const reportDate = $('Tính Ngày Hôm Qua').first().json.reportDate;

function normalizeDate(value) {
  const raw = String(value ?? '').trim();
  const dmy = raw.match(/^(\d{1,2})\/(\d{1,2})\/(\d{4})$/);
  if (dmy) return `${dmy[3]}-${dmy[2].padStart(2, '0')}-${dmy[1].padStart(2, '0')}`;
  return raw;
}

const filtered = records.filter(r => normalizeDate(r.ngay) === reportDate);
if (filtered.length === 0) {
  return [{ json: { reportDate, hasData: false, message: `📊 *BÁO CÁO META ADS — HÔM QUA*\n📅 Ngày: ${reportDate}\n⚠️ Không có dữ liệu trong Sheet cho ngày này.` } }];
}

const totals = { spend: 0, luotHienThi: 0, click: 0, clickAll: 0, mess: 0, khachHopLe: 0, sdt: 0, khachChot: 0, doanhThu: 0 };
for (const r of filtered) {
  totals.spend += r.chi_tieu;
  totals.luotHienThi += r.luot_hien_thi;
  totals.click += r.click;
  totals.clickAll += r.click_all;
  totals.mess += r.mess_comment;
  totals.khachHopLe += r.khach_hop_le;
  totals.sdt += r.sdt;
  totals.khachChot += r.khach_chot;
  totals.doanhThu += r.doanh_thu;
}

const kpis = {
  cpm: totals.luotHienThi > 0 ? Math.round(totals.spend / totals.luotHienThi * 1000) : null,
  ctrLink: totals.luotHienThi > 0 ? +(totals.click / totals.luotHienThi * 100).toFixed(2) : null,
  cpc: totals.click > 0 ? Math.round(totals.spend / totals.click) : null,
  costPerMess: totals.mess > 0 ? Math.round(totals.spend / totals.mess) : null,
  cpl: totals.khachHopLe > 0 ? Math.round(totals.spend / totals.khachHopLe) : null
};

const fmt = n => Math.round(n).toLocaleString('vi-VN');
const fmtMoney = n => n === null ? 'N/A' : fmt(n) + 'đ';
const message = [
  `📊 *BÁO CÁO META ADS — HÔM QUA*`,
  `📅 Ngày: ${reportDate}`,
  `💰 Chi tiêu: *${fmt(totals.spend)}đ*`,
  `📩 Mess: *${totals.mess}*`,
  `💵 Cost/Mess: *${fmtMoney(kpis.costPerMess)}*`,
  `📈 CPM: *${fmtMoney(kpis.cpm)}*`,
  `📈 CPC: *${fmtMoney(kpis.cpc)}*`
].join('\n');

return [{ json: { reportDate, hasData: true, filteredCount: filtered.length, totals, kpis, message } }];
```

### Gửi Telegram bằng env
```json
{
  "method": "POST",
  "url": "=https://api.telegram.org/bot{{ $env.TELEGRAM_BOT_TOKEN }}/sendMessage",
  "sendBody": true,
  "specifyBody": "json",
  "jsonBody": "={{ { chat_id: $env.TELEGRAM_CHAT_ID.toString(), text: $json.message, parse_mode: \"Markdown\" } }}"
}
```

## Lưu ý quan trọng
- Dữ liệu từ Google Sheets API thường là raw array, cần normalize layer ngay sau node đọc Sheet.
- Date output nên dùng ISO `YYYY-MM-DD`, timezone `Asia/Ho_Chi_Minh`.
- KPI phải tính từ raw metrics, không phụ thuộc cột công thức nếu có thể.
- Scheduled workflow phải gửi Telegram bằng `$env.TELEGRAM_CHAT_ID`, không dùng `$json.chat_id`.
- Node gửi Telegram trong production nên bật retry để chống lỗi mạng tạm thời.
- Nếu KPI = 0, phải phân biệt business zero và pipeline failure trước khi kết luận lỗi.

## CASE liên quan
- CASE-012: Kiến trúc Yesterday và Today
- CASE-013: Scheduled dùng env chat id
- CASE-020: KPI reports returning zero
- CASE-023: ECONNRESET Telegram send
- CASE-036: Layer-by-layer verification
- CASE-038: Business zero vs pipeline failure
