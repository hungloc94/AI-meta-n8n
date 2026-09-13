# TOOL: Highlight Duplicate Rows — Google Sheets

## Mục đích
Bôi màu các dòng duplicate trên Google Sheet để review trực quan trước khi xóa.
Dùng khi phát hiện trùng dữ liệu và cần kiểm tra bằng mắt trước khi xóa hàng loạt.

## Workflow n8n
- **ID:** `QWowp4oT9PA1iJTO`
- **Tên:** TOOL — Highlight Duplicate Rows
- **Trigger:** Manual (không schedule)
- **Flow:** Manual Trigger → Read Input File → Build Color Requests (Code) → Apply Colors (HTTP batchUpdate)
- **Project gốc:** AI_Meta_n8n_autoamation
- **active:** false (chỉ chạy tay)

## Màu quy ước
| Màu | Hex | Ý nghĩa |
|-----|-----|---------|
| Vàng nhạt | `#FFF9C4` | dòng sẽ XÓA (cần review) |
| Cam nhạt | `#FFE0B2` | dòng cần GỘP / xử lý đặc biệt |
| Trắng | `#FFFFFF` | reset màu |

## Input file chuẩn
Đường dẫn (trong container n8n): `/tmp/dup_highlight.json`
```json
[{"row":123,"color":"yellow"},{"row":456,"color":"orange"},{"row":789,"color":"white"}]
```
> Lưu ý: file phải nằm trong **filesystem của container n8n**. Đưa vào bằng:
> `docker cp dup_highlight.json n8n:/tmp/dup_highlight.json`

## Tham số cần đổi khi dùng lại
- **spreadsheetId** — trong node "Apply Colors" (URL `.../spreadsheets/<ID>:batchUpdate`)
- **GID** (gid tab) — trong node "Build Color Requests" (biến `const GID = ...`)
- **File input** — `/tmp/dup_highlight.json`

## Cách chạy
1. Chuẩn bị `/tmp/dup_highlight.json` (danh sách `{row, color}`) trong container n8n.
2. Đổi `spreadsheetId` + `GID` cho đúng Sheet/tab target.
3. Trigger manual trong n8n UI.
4. Mở Sheet kiểm tra trực quan.
5. Xác nhận → tiến hành xóa (bằng tool xóa riêng / thủ công).

## An toàn
- Chỉ đổi `backgroundColor` — KHÔNG đụng giá trị ô, KHÔNG xóa dòng.
- Manual trigger, active=false → không tự chạy.

## Lịch sử dùng
| Ngày | Task | Sheet target | Kết quả |
|------|------|--------------|---------|
| 2026-09 | Task_03_Fix_Duplicate_Daily_Sheet | BÁO_CÁO_QUẢNG_CÁO | Review 62→46 dup trước khi xóa |
