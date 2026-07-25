# PATTERN-002: Today Report Direct Meta API → KPI → Telegram

## Mục đích
Báo cáo realtime trong ngày, gọi Meta API trực tiếp.
Không phụ thuộc Google Sheet.

## Cấu trúc node
Schedule Trigger (11:31 / 16:31 / 21:13)
→ Tính ngày hôm nay
→ Gọi Meta Insights API
→ Parse actions (Mess_Comment, video metrics)
→ Tính KPI hôm nay
→ Render message Telegram
→ Send Report Telegram

## Code mẫu

### Tính ngày hôm nay
const today = new Date().toLocaleDateString('en-CA',
  {timeZone: 'Asia/Ho_Chi_Minh'});
return [{json: {today}}];

### Parse Mess_Comment từ actions
const actions = $json.actions || [];
const mess = actions.find(a =>
  a.action_type === 'onsite_conversion.messaging_conversation_started_7d'
);
return [{json: {
  ...$json,
  mess_comment: mess ? parseFloat(mess.value) : 0
}}];

### Parse video metrics
const actions = $json.actions || [];
const xem3s = actions.find(a => a.action_type === 'video_view');
return [{json: {
  ...$json,
  xem_3s: xem3s ? parseFloat(xem3s.value) : 0,
  video_25: parseFloat(($json.video_p25_watched_actions?.[0]?.value)) || 0,
  video_50: parseFloat(($json.video_p50_watched_actions?.[0]?.value)) || 0,
  video_75: parseFloat(($json.video_p75_watched_actions?.[0]?.value)) || 0,
  video_95: parseFloat(($json.video_p95_watched_actions?.[0]?.value)) || 0,
  thruplay: parseFloat(($json.video_thruplay_watched_actions?.[0]?.value)) || 0
}}];

## Lưu ý quan trọng
- Exact match === cho action_type, không dùng includes()
- video_view không phải field độc lập, lấy từ actions[]
- video watched fields là array, đọc [0].value
- Bearer prefix trong Authorization header
- Scheduled dùng $env.TELEGRAM_CHAT_ID
- Có retry cho ECONNRESET và rate limit
- Token 2 tháng → theo dõi hạn

## CASE liên quan
CASE-012, CASE-013, CASE-015, CASE-017, CASE-018, CASE-023, CASE-026