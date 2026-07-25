# PATTERN-002: Scheduled Today Report Direct Meta API Retry → KPI → Telegram

## Mục đích
Dùng cho báo cáo trong ngày khi dữ liệu chưa ổn định hoặc chưa được sync vào Google Sheet. Pattern gọi Meta Marketing API trực tiếp, tính KPI realtime và gửi Telegram theo nhiều khung giờ.

## Cấu trúc node
```text
Schedule Trigger nhiều khung giờ
→ Tính ngày hôm nay + giờ hiện tại
→ Lấy Insights Meta hôm nay qua Meta API
→ Tính KPI từ raw metrics
→ Render message Telegram
→ Send Report Telegram
```

## Code mẫu
Trích từ workflow `Meta Report Today Scheduled 11:31 16:31 21:13`.

### Schedule nhiều khung giờ
```json
{
  "rule": {
    "interval": [
      { "field": "cronExpression", "expression": "31 11 * * *" },
      { "field": "cronExpression", "expression": "31 16 * * *" },
      { "field": "cronExpression", "expression": "31 21 * * *" }
    ]
  }
}
```

### Tính ngày hôm nay
```javascript
const vnNow = new Date(new Date().toLocaleString('en-US', { timeZone: 'Asia/Ho_Chi_Minh' }));
const yyyy = vnNow.getFullYear();
const mm = String(vnNow.getMonth() + 1).padStart(2, '0');
const dd = String(vnNow.getDate()).padStart(2, '0');
const HH = String(vnNow.getHours()).padStart(2, '0');
const min = String(vnNow.getMinutes()).padStart(2, '0');

return [{ json: { reportDate: `${yyyy}-${mm}-${dd}`, reportTime: `${HH}:${min}` } }];
```

### Gọi Meta API có retry cho rate limit và network error
```javascript
const reportDate = $('Tính Ngày Hôm Nay').first().json.reportDate;
const token = $env.META_ACCESS_TOKEN;
if (!token) throw new Error('Missing META_ACCESS_TOKEN environment variable.');

const timeRange = encodeURIComponent(JSON.stringify({ since: reportDate, until: reportDate }));
const url = `https://graph.facebook.com/v19.0/act_<AD_ACCOUNT_ID>/insights?time_range=${timeRange}&level=ad&fields=ad_id,ad_name,campaign_name,spend,reach,clicks,inline_link_clicks,impressions,frequency,actions,video_view,video_p25_watched_actions,video_p50_watched_actions,video_p75_watched_actions,video_p95_watched_actions,video_thruplay_watched_actions,outbound_clicks,date_start&limit=500`;

let success = false;
let attempts = 0;
let responseData = {};

while (!success && attempts < 3) {
  attempts++;
  try {
    responseData = await this.helpers.httpRequest({
      method: 'GET',
      url,
      json: true,
      headers: { Authorization: `Bearer ${token}` }
    });
    success = true;
  } catch (error) {
    const metaErrorBody = error.response?.data ? JSON.stringify(error.response.data) : '';
    const errMsg = (error.message || '') + ' ' + metaErrorBody;
    const isRateLimitError = errMsg.includes('Application request limit reached');
    const networkErrorPatterns = ['Client network socket disconnected', 'ECONNRESET', 'ETIMEDOUT', 'socket hang up'];
    const isNetworkError = networkErrorPatterns.some(p => errMsg.toLowerCase().includes(p.toLowerCase()));

    if (isRateLimitError) {
      if (attempts >= 3) throw new Error('Rate limit. Vẫn lỗi sau 3 lần retry.');
      const wait = Math.floor(Math.random() * 60000) + 60000;
      await new Promise(r => setTimeout(r, wait));
    } else if (isNetworkError) {
      if (attempts >= 3) throw new Error('Lỗi mạng tạm thời. Vẫn lỗi sau 3 lần retry.');
      await new Promise(r => setTimeout(r, attempts * 3000));
    } else {
      throw new Error('Meta API Error: ' + JSON.stringify(error.response?.data || error.message, null, 2));
    }
  }
}

return [{ json: responseData }];
```

### Tính KPI hôm nay bằng exact match action_type
```javascript
const reportDate = $('Tính Ngày Hôm Nay').first().json.reportDate;
const reportTime = $('Tính Ngày Hôm Nay').first().json.reportTime;
const insights = $input.first().json.data || [];

const toInt = v => parseInt(v, 10) || 0;
const toFloat = v => parseFloat(v) || 0;
const totals = { spend: 0, reach: 0, luotHienThi: 0, click: 0, clickAll: 0, mess: 0 };

for (const row of insights) {
  totals.spend += toFloat(row.spend);
  totals.reach += toInt(row.reach);
  totals.luotHienThi += toInt(row.impressions);
  totals.click += toInt(row.inline_link_clicks);
  totals.clickAll += toInt(row.clicks);
  if (row.actions) {
    const messAction = row.actions.find(a => a.action_type === 'onsite_conversion.messaging_conversation_started_7d');
    totals.mess += messAction ? toInt(messAction.value) : 0;
  }
}

const kpis = {
  cpm: totals.luotHienThi > 0 ? Math.round(totals.spend / totals.luotHienThi * 1000) : null,
  ctrLink: totals.luotHienThi > 0 ? +(totals.click / totals.luotHienThi * 100).toFixed(2) : null,
  cpc: totals.click > 0 ? Math.round(totals.spend / totals.click) : null,
  costPerMess: totals.mess > 0 ? Math.round(totals.spend / totals.mess) : null
};

const fmt = n => Math.round(n).toLocaleString('vi-VN');
const fmtMoney = n => n === null ? 'N/A' : fmt(n) + 'đ';
const message = [
  `📊 *BÁO CÁO META ADS — HÔM NAY*`,
  `📅 Ngày: ${reportDate}  ⏰ ${reportTime} VN`,
  `💰 Chi tiêu: *${fmt(totals.spend)}đ*`,
  `📩 Mess: *${totals.mess}*`,
  `💵 Cost/Mess: *${fmtMoney(kpis.costPerMess)}*`,
  `📈 CPM: *${fmtMoney(kpis.cpm)}* — 👥 Tiếp cận: *${fmt(totals.reach)}*`,
  `📈 CPC: *${fmtMoney(kpis.cpc)}* — 🖱️ Click: *${totals.click}*`
].join('\n');

return [{ json: { reportDate, reportTime, hasData: insights.length > 0, campaignCount: insights.length, totals, kpis, message } }];
```

## Lưu ý quan trọng
- Today Report không đọc Google Sheet vì dữ liệu hôm nay có thể chưa được sync.
- Meta token phải đến từ env/runtime, không hardcode trong workflow JSON.
- `action_type` cho message phải exact match, không dùng `includes()`.
- Cần retry cho Meta API rate limit và lỗi mạng tạm thời.
- Scheduled Telegram phải dùng `$env.TELEGRAM_CHAT_ID`.

## CASE liên quan
- CASE-012: Kiến trúc Yesterday và Today
- CASE-013: Scheduled dùng env chat id
- CASE-015: Exact match action_type
- CASE-017: Today Report trực tiếp Meta API
- CASE-018: CPC metric bắt buộc
- CASE-023: ECONNRESET Telegram send
- CASE-026: Meta API 403 Bearer prefix
