# LESSONS LEARNED — Task 03 (Fix Duplicate Row)

> Bài học rút ra từ sự cố duplicate row + báo cáo nhân đôi (2026-09). Case chi tiết: `CASES/CASE-051`.
> Ghi chú: PROJECT_BRAIN.md của project hiện nằm trong `ARCHIVE/` — index bài học này đặt ở `CASE_INDEX.md` (active).

---

## Bài học 1 — Audit duplicate PHẢI quét mọi workflow active ghi cùng đích, TRƯỚC TIÊN
Khi điều tra lỗi duplicate, bước **đầu tiên** phải là: **liệt kê TẤT CẢ workflow đang `active=true` có quyền ghi vào cùng Sheet/tab**.
- **Vì sao:** lần này bỏ qua bước đó → tốn nhiều thời gian sửa đúng workflow canonical (`rz3Wya5lFay7ShVL`) nhưng thủ phạm thật là **bản clone `T0x1qzedADrmj1LA` ("TEST Daily Sheet Update Task09")** vẫn active, chạy 07:30 song song bằng logic append cũ.
- **Cách áp dụng:** `export:workflow --all` → lọc node ghi cùng `spreadsheetId`+`gid` với `active=true`. Chỉ khi biết chắc **chỉ 1 writer** mới kết luận root cause.

## Bài học 2 — Không hy sinh tính năng chính để chống tác dụng phụ nhỏ
Khi cân nhắc **tắt `retryOnFail`** ở node gửi Telegram để tránh gửi tin trùng:
- Mục tiêu chính của workflow báo cáo là **RA ĐƯỢC BÁO CÁO**.
- Tắt retry → chỉ cần mạng giật 1 giây là **mất báo cáo hoàn toàn**.
- Thừa 1 tin trùng (tác dụng phụ nhỏ) **không thể** đánh đổi với mất báo cáo.
- **Giải pháp đúng:** tăng `timeout` node (→ 30s) để giảm xác suất "gửi thành công nhưng n8n không nhận response" xuống gần 0, **GIỮ `retryOnFail=true`**.
- **Nguyên tắc:** không đề xuất giải pháp làm suy yếu mục tiêu chính chỉ để xử lý tác dụng phụ nhỏ.

## Bài học 3 — Hệ thống phải CHỦ ĐỘNG cảnh báo data trùng (không để người dùng phát hiện bằng mắt)
Lần này duplicate được phát hiện do anh Lộc thấy bằng mắt (báo cáo nhân đôi) → **quá muộn**.
Nguyên tắc thiết kế cho tương lai:
- **Data trên Google Sheet:** cuối mỗi workflow run → check `COUNTIF` trùng Key; nếu > 0 → **bôi màu dòng trùng ngay** (dùng tool `AI_OS/skills/highlight_duplicate_rows`) + gửi Telegram `⚠️ Phát hiện X dòng trùng Key — cần xử lý`.
- **Data trên app/database:** phải có alert tự động (unique constraint, monitoring query, hoặc notification).
- **Tuyệt đối không** để người dùng tự phát hiện data trùng bằng mắt.

> **Cập nhật 2026-09-15 (DEFERRED):** anh Lộc đã tự làm **conditional formatting** trên Sheet — tự bôi màu các dòng trùng Key (cột S) khi trùng nhau. Đã có cơ chế phát hiện trùng ở tầng Sheet → workflow auto-alert Telegram **chưa cần** lúc này. Bài học vẫn đúng làm nguyên tắc; chỉ hoãn phần triển khai auto-alert.

---

## Phụ lục — chuỗi sự cố (tóm tắt)
- Root cause thật: 2 workflow active cùng append 07:30 (canonical + clone TEST task09) → trùng dòng hàng ngày.
- Hệ quả: báo cáo "hôm qua" đọc Sheet trùng → nội dung nhân đôi (`filteredCount` 32-34 vs bình thường ~16).
- Fix: upsert (appendOrUpdate by Key) + tắt clone + dọn dup + Mess_Comment phương án 3 + `slice(-2)` + timeout Telegram 30s.
