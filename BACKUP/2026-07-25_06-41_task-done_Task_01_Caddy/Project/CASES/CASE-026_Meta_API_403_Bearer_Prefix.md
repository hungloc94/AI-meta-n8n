# CASE-026: Meta API 403 — Bearer Prefix Missing + Token Expired

## Vấn đề
Meta API trả 403 `(#200) Provide valid app ID` dù vừa renew token mới. Các workflow Daily Sheet Update, Today Report, Meta API Health Check đều fail.

## Nguyên nhân
Hai lỗi chồng nhau:

**Root Cause #1:** Meta Access Token hết hạn (error_subcode 463). Đang dùng User Token có thời hạn.

**Root Cause #2:** Sau khi renew token, Header Auth credential thiếu prefix `Bearer `:
- Sai: `Authorization: EAASxxxx`
- Đúng: `Authorization: Bearer EAASxxxx`

**Root Cause sâu:** Đang dùng User Access Token trong production thay vì System User Token.

## Cách xử lý
1. Thêm `Bearer ` prefix vào Header Auth credential
2. Exchange Short-lived → Long-lived Token (60 ngày)
3. Verify tất cả workflow gọi Meta API hoạt động

## Bài học
- Lỗi `(#200) Provide valid app ID` **không phải** lỗi App ID — kiểm tra Authorization header format trước.
- Khi renew token: luôn verify `Bearer ` prefix còn nguyên, không chỉ thay giá trị token.
- Token từ Graph API Explorer là Short-lived (vài giờ) — **không dùng cho production**.
- Bảng tra lỗi Meta: `401 error_subcode 463` = token hết hạn; `403 (#200)` = header sai format; `ECONNRESET` = mạng tạm.
- Mục tiêu dài hạn: System User Token (không expire).

## Status
Resolved — 2026-06-22.
