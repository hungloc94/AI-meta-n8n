# Schema — n8n Meta Ads

> Cập nhật lần cuối: 2026-06-30

## Header Order trong Google Sheet

```
Ma_quang_cao, Ngay, Chien_dich, Ten_quang_cao, Chi_tieu,
Nguoi_tiep_can, Ngan_sach, click, Mess_Comment, Khach_sai_tep,
Khach_hop_le, SDT, Khach_chot, Chi_phi_tren_khach_hop_le,
Chi_phi_khach_tren_SDT, Doanh_thu, ghi_chu,
Trang_Thai, Key, Thoi_diem_cap_nhat,
Luot_hien_thi, Click_All, Tan_suat,
Xem_3s, Video_25, Video_50, Video_75, Video_95, ThruPlay, Click_Ra_Web
```

## 3 Nhóm Metrics — Ownership

### Nhóm 1: Raw Metrics (Workflow ghi từ Meta API)

| Cột | Nguồn Meta API |
|-----|---------------|
| Ma_quang_cao | ad_id |
| Ngay | date_start |
| Chi_tieu | spend |
| Nguoi_tiep_can | reach |
| Ngan_sach | daily_budget |
| click | inline_link_clicks |
| Mess_Comment | onsite_conversion.messaging_conversation_started_7d |
| Trang_Thai | effective_status |
| Key | ad_id + "_" + date |
| Thoi_diem_cap_nhat | timestamp lúc sync |
| Chien_dich | campaign_name |
| Ten_quang_cao | ad_name |
| Luot_hien_thi | impressions |
| Click_All | clicks |
| Tan_suat | frequency |
| Xem_3s | video_view (actions[]) |
| Video_25 | video_p25_watched_actions |
| Video_50 | video_p50_watched_actions |
| Video_75 | video_p75_watched_actions |
| Video_95 | video_p95_watched_actions |
| ThruPlay | video_thruplay_watched_actions |
| Click_Ra_Web | outbound_clicks |

### Nhóm 2: Derived Metrics (Google Sheet tự tính bằng công thức)

| Cột | Công thức |
|-----|-----------|
| CPM | Chi_tieu / Luot_hien_thi * 1000 |
| CTR_Link | click / Luot_hien_thi * 100 |
| CTR_All | Click_All / Luot_hien_thi * 100 |
| Ty_le_Hook_3s | Xem_3s / Luot_hien_thi * 100 |
| Ty_le_Giu_Chan_25 | Video_25 / Xem_3s * 100 |
| Ty_le_Xem_95 | Video_95 / Xem_3s * 100 |
| Ty_le_Ra_Web | Click_Ra_Web / click * 100 |
| Khach_sai_tep | Mess_Comment - Khach_hop_le |
| Chi_phi_tren_khach_hop_le | công thức Sheet |
| Chi_phi_khach_tren_SDT | công thức Sheet |

Workflow **KHÔNG được ghi** vào nhóm này — công thức Sheet tự tính.

### Nhóm 3: Business Metrics (Người dùng nhập/chỉnh sửa — PROTECTED)

| Cột | Mô tả |
|-----|-------|
| Khach_hop_le | Số khách hợp lệ — nhập tay sau khi đánh giá lead |
| SDT | Số điện thoại thu được — nhập tay |
| Khach_chot | Số khách chốt đơn — nhập tay |
| Doanh_thu | Tổng doanh thu thực tế — nhập tay, dùng tính ROAS |
| ghi_chu | Ghi chú kinh doanh — nhập tay |

Workflow **KHÔNG được ghi đè** nhóm này trừ khi có phê duyệt rõ ràng từ người vận hành.

## Parser Rules

- Field video hoặc field Meta không tồn tại trong API response → parser trả về `0`, không throw Error.
- action_type phải dùng exact match `===`, không dùng `includes()`.
- Dates nội bộ: ISO `YYYY-MM-DD`. Timezone: `Asia/Ho_Chi_Minh`.
- Schema naming: snake_case ASCII.

## Bảng tổng hợp Ownership

| Nhóm | Chủ sở hữu | Workflow được ghi? |
|------|------------|-------------------|
| Raw Metrics | Meta API → Workflow | ✅ Có |
| Derived Metrics | Google Sheet (công thức) | ❌ Không |
| Business Metrics | Người vận hành (nhập tay) | ❌ Không |
## Meta API Field Mapping Notes

Verified date: 2026-06-30  
Meta Graph API version: v19.0  
Workflow: Meta Ads Daily Sheet Update 7:30 2

### Parser Variable Mapping

| Meta API field | Response structure | Parser variable | Google Sheet column |
|---|---|---|---|
| impressions | string | impressions | Luot_hien_thi |
| clicks | string | clickAll | Click_All |
| inline_link_clicks | string | clickLink | click |
| frequency | string | frequency | Tan_suat |
| spend | string | spend | Chi_tieu |
| reach | string | reach | Nguoi_tiep_can |
| actions[].action_type="video_view" | array, find in actions | xem3s | Xem_3s |
| video_p25_watched_actions | array | video25 | Video_25 |
| video_p50_watched_actions | array | video50 | Video_50 |
| video_p75_watched_actions | array | video75 | Video_75 |
| video_p95_watched_actions | array | video95 | Video_95 |
| video_thruplay_watched_actions | array | thruplay | ThruPlay |
| outbound_clicks | array, UNVERIFIED structure | clickRaWeb | Click_Ra_Web |

### Fields Tested But Not Used

| Meta field | Reason |
|---|---|
| ctr | HTTP 400 when requested; CTR is calculated in Google Sheet |
| outbound_clicks_all | HTTP 400 when requested |
| video_3_sec_watched_actions | Invalid field name |
| video_3_sec_watched | Invalid field name |
| video_view as standalone response field | Does not appear independently in response; only exists inside actions[] |

### Meta API Design Constraints

- `video_view` is not a standalone response field, even when included in `fields=`.
- `Xem_3s` must be parsed from `actions[]` where `action_type == "video_view"`.
- `video_p25_watched_actions`, `video_p50_watched_actions`, `video_p75_watched_actions`, `video_p95_watched_actions`, and `video_thruplay_watched_actions` are arrays.
- Parser currently reads `[0].value` from those video arrays; re-check this if Meta changes response structure.
- `outbound_clicks` structure is currently marked UNVERIFIED.

### Verified Meta API fields Parameter

```text
fields=ad_id,ad_name,campaign_id,campaign_name,account_id,spend,reach,clicks,inline_link_clicks,actions,date_start,impressions,frequency,video_view,video_p25_watched_actions,video_p50_watched_actions,video_p75_watched_actions,video_p95_watched_actions,video_thruplay_watched_actions,outbound_clicks
```
