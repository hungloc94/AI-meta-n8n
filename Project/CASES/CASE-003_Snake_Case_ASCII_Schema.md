# CASE-003: snake_case ASCII Schema

## Vấn đề
Key mismatch và expression ambiguity khi dùng Vietnamese accented headers trong code.

## Nguyên nhân
Accented characters (ế, ạ, ổ...) gây typing errors, copy-paste issues, và expression evaluation failures trong n8n Code nodes.

## Cách xử lý
Adopt snake_case ASCII cho tất cả operational field names. Ví dụ: `Chi_tieu`, `Khach_hop_le`, `Thoi_diem_cap_nhat`.

## Bài học
- Cần one-time migration từ Vietnamese display headers sang canonical internal field names.
- Google Sheet vẫn giữ header tiếng Việt cho user — normalize layer handle mapping.
- Tất cả downstream code chỉ dùng ASCII field names.

## Status
Accepted.
