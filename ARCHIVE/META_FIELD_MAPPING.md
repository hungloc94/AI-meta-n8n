# META FIELD MAPPING — n8n-meta-ads

## Mục đích
File này ghi lại mapping chính xác giữa Meta Graph API, biến trong Workflow parser, và tên cột Google Sheet. Dùng để tra cứu khi cần debug hoặc khi Meta thay đổi cấu trúc API.

## API Version
Verified với version đang sử dụng trong workflow tại thời điểm 2026-06-30 (Meta Graph API v19.0 theo URL hiện tại).
Workflow: Meta Ads Daily Sheet Update 7:30 2

## Bảng Mapping

| Meta API field | Cấu trúc response | Biến Parser | Google Sheet |
|---|---|---|---|
| impressions | string | impressions | Luot_hien_thi |
| clicks | string | clickAll | Click_All |
| inline_link_clicks | string | clickLink | click |
| frequency | string | frequency | Tan_suat |
| spend | string | spend | Chi_tieu |
| reach | string | reach | Nguoi_tiep_can |
| actions[].action_type="video_view" | array, tìm trong actions | xem3s | Xem_3s |
| video_p25_watched_actions | array | video25 | Video_25 |
| video_p50_watched_actions | array | video50 | Video_50 |
| video_p75_watched_actions | array | video75 | Video_75 |
| video_p95_watched_actions | array | video95 | Video_95 |
| video_thruplay_watched_actions | array | thruplay | ThruPlay |
| outbound_clicks | array, UNVERIFIED structure | clickRaWeb | Click_Ra_Web |

## Field đã thử nhưng KHÔNG dùng

| Meta field | Lý do không dùng |
|---|---|
| ctr | HTTP 400 khi request, đã quyết định tự tính CTR trong Sheet |
| outbound_clicks_all | HTTP 400 khi request |
| video_3_sec_watched_actions | Sai tên, không tồn tại |
| video_3_sec_watched | Sai tên, không tồn tại |
| video_view (như field độc lập) | Không xuất hiện độc lập trong response — chỉ tồn tại trong actions[] |

## Design Constraint quan trọng

video_view không phải field độc lập trong response, dù được dùng trong fields= để Meta trả thêm dữ liệu video. Giá trị Xem_3s phải lấy từ actions[] tìm action_type == "video_view".

Các field video_p25/p50/p75/p95_watched_actions và video_thruplay_watched_actions là mảng. Parser hiện lấy phần tử đầu tiên của mảng ([0].value), đã verify với sample hiện tại. Nếu Meta thay đổi cấu trúc cần kiểm tra lại logic này.

## fields= URL đã verify PASS toàn bộ
fields=ad_id,ad_name,campaign_id,campaign_name,account_id,spend,reach,clicks,inline_link_clicks,actions,date_start,impressions,frequency,video_view,video_p25_watched_actions,video_p50_watched_actions,video_p75_watched_actions,video_p95_watched_actions,video_thruplay_watched_actions,outbound_clicks
