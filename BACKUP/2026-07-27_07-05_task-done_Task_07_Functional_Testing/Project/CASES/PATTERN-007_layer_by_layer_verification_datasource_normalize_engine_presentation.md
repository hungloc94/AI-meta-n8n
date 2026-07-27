# PATTERN-007: Layer-by-Layer Verification DataSource → Normalize → Engine → Presentation

## Mục đích
Dùng để rebuild, debug hoặc migrate workflow n8n theo từng layer độc lập. Pattern giúp cô lập lỗi nhanh và tránh ghép nhiều thay đổi cùng lúc.

## Cấu trúc node
```text
Data Source Layer
→ Normalize Layer
→ Engine Layer
→ Presentation Layer
→ Delivery Layer
```

Ví dụ Yesterday Report:
```text
Đọc Google Sheet
→ Normalize Sheet Data
→ Lọc & Tính KPI Hôm Qua
→ Render message
→ Send Report Telegram
```

Ví dụ Today Report:
```text
Lấy Insights Meta Hôm Nay
→ Parse raw Meta response
→ Tính KPI Hôm Nay
→ Render message
→ Send Report Telegram
```

## Code mẫu
Trích và rút gọn từ `Meta Report Yesterday Scheduled 1` và `Meta Report Today Scheduled 11:31 16:31 21:13`.

### Data Source Layer: kiểm tra có data
```javascript
const values = $('Đọc Google Sheet').first().json.values || [];
if (values.length === 0) {
  return [{ json: { layer: 'data_source', status: 'EMPTY', records: [] } }];
}
return [{ json: { layer: 'data_source', status: 'PASS', rowCount: values.length, values } }];
```

### Normalize Layer: map header sang object
```javascript
const values = $json.values || [];
const headers = values[0] || [];
const headerMap = {};
headers.forEach((h, i) => { if (String(h).trim()) headerMap[String(h).trim()] = i; });

for (const h of ['Ma_quang_cao', 'Ngay', 'Chi_tieu', 'Mess_Comment']) {
  if (headerMap[h] === undefined) throw new Error(`Thiếu cột bắt buộc trong Sheet: ${h}`);
}

const records = values.slice(1).map(row => ({
  ma_quang_cao: row[headerMap.Ma_quang_cao],
  ngay: row[headerMap.Ngay],
  chi_tieu: Number(String(row[headerMap.Chi_tieu] || '0').replace(/\./g, '').replace(',', '.')) || 0,
  mess_comment: Number(row[headerMap.Mess_Comment]) || 0
}));

return [{ json: { layer: 'normalize', status: 'PASS', records, rowCount: records.length } }];
```

### Engine Layer: tính KPI có guard divide-by-zero
```javascript
const records = $json.records || [];
const totals = records.reduce((acc, r) => {
  acc.spend += r.chi_tieu || 0;
  acc.mess += r.mess_comment || 0;
  return acc;
}, { spend: 0, mess: 0 });

const kpis = {
  costPerMess: totals.mess > 0 ? Math.round(totals.spend / totals.mess) : null
};

return [{ json: { layer: 'engine', status: 'PASS', totals, kpis } }];
```

### Presentation Layer: render message cuối
```javascript
const fmt = n => Math.round(n).toLocaleString('vi-VN');
const fmtMoney = n => n === null ? 'N/A' : fmt(n) + 'đ';
const message = [
  '📊 *BÁO CÁO META ADS*',
  `💰 Chi tiêu: *${fmt($json.totals.spend)}đ*`,
  `📩 Mess: *${$json.totals.mess}*`,
  `💵 Cost/Mess: *${fmtMoney($json.kpis.costPerMess)}*`
].join('\n');

return [{ json: { ...$json, layer: 'presentation', message } }];
```

## Lưu ý quan trọng
- Không ghép layer tiếp theo khi layer hiện tại chưa PASS bằng runtime execution thật.
- Với mỗi layer, output phải có dữ liệu kiểm chứng được: `rowCount`, `filteredCount`, `totals`, `message`.
- Khi schema thay đổi, verify lại từ Data Source đến Normalize, không chỉ sửa Engine.
- KPI = 0 chưa chắc là lỗi; cần kiểm tra pipeline và dữ liệu business thật.
- Debug bằng evidence runtime, không đoán từ code.

## CASE liên quan
- CASE-020: KPI reports returning zero
- CASE-030: Node PASS không đảm bảo workflow PASS
- CASE-036: Layer-by-layer verification
- CASE-037: Schema change verify toàn bộ
- CASE-038: Business zero vs pipeline failure
