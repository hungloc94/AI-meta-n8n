# System Architecture

## Telegram Polling Flow
```text
Cron Polling 30s
→ Telegram API getUpdates
→ Process Updates
→ Main Router Code
```

Responsibilities:
- fetch Telegram updates
- preserve update continuity
- classify command type
- avoid duplicate processing

## Routing Architecture
```text
Main Router Code
→ IF Menu
→ IF Report
→ IF Range
→ Parse & Route
→ Switch Route
```

Routing tách riêng menu/report/range/unknown command behavior.

## Date Parsing Pipeline
```text
human input: báo cáo 24/03
→ parser regex
→ canonical ISO date: 2026-03-24
→ dates: ["2026-03-24"]
```

Internal systems nên compare ISO dates.

## Google Sheets Dependency Flow
```text
Đọc Google Sheet
→ Normalize Sheet Data
```

Google Sheets trả array rows. Normalize layer convert rows thành canonical schema objects.

## Normalize Layer Architecture
Canonical fields:
```text
ma_quang_cao
ngay
chien_dich
ten_quang_cao
chi_tieu
nguoi_tiep_can
ngan_sach
click
mess_comment
khach_sai_tep
khach_hop_le
sdt
khach_chot
chi_phi_tren_khach_hop_le
chi_phi_khach_tren_sdt
chi_phi_khach_chot
ghi_chu
trang_thai
key
thoi_diem_cap_nhat
```

## KPI Aggregation Flow
```text
Normalize Sheet Data
→ Lọc Dữ Liệu Thiếu
→ Tạo Báo Cáo
```

KPI logic nên dùng:
- `r.chi_tieu`
- `r.khach_hop_le`

Không dùng raw array indexes downstream, ngoại trừ bên trong normalization.

## Telegram Report Delivery Pipeline
```text
Tạo Báo Cáo
→ Send Report Telegram
→ ACK Telegram Update
```

`Send Report Telegram` phải send `$json.message` với Markdown parse mode.

## Runtime State Persistence Flow
- n8n runtime data lưu trong Docker volume `n8n_data`.
- n8n credentials depend vào `N8N_ENCRYPTION_KEY` continuity.
- Polling duplicate protection depend vào staticData/offset continuity.
- Polling continuity phải validate bằng real runtime activation, không chỉ manual Execute Step testing.
- Temporary hardcoded Telegram offsets chỉ được dùng cho supervised backlog draining và không được nằm trong active production workflows.

## Finalized End-to-End Recovery Architecture

```text
Telegram Input
→ Polling Layer
→ Command Parsing
→ Routing Layer
→ Date Canonicalization
→ Google Sheet Ingestion
→ Normalize Sheet Data
→ Data Filtering
→ KPI Aggregation
→ Telegram Delivery
```

## Canonical ISO Date Strategy
- Human-friendly formats được allow tại parser layer.
- Internal operational processing dùng ISO dates only.
- `báo cáo 24/03` normalize thành `2026-03-24` trong parsing.
- Downstream filtering compare canonical date representations, không compare raw user text.

## Canonical Data Contract Strategy
- Raw Google Sheets array rows normalize một lần ngay sau ingestion.
- Downstream KPI logic consume canonical object fields như `chi_tieu` và `khach_hop_le`.
- Downstream systems không depend trực tiếp vào raw external array indexes.

## Project-Scoped Operational Memory
- Operational memory cho project này self-contained trong `D:\AI_Project\Obdisian_2\My Vault\Project\AI_Meta_n8n_autoamation`.
- Không dùng `D:\AI_Project\Obdisian_2\My Vault\AI_MEMORY` cho project này.
- Memory files không được chứa secrets, credentials, tokens, cookies, encryption keys, hoặc sensitive values.
