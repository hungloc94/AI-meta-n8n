# CASE Index — Task 03

| CASE | Mô tả ngắn | Đường dẫn |
|------|------------|-----------|
| CASE-051 | Duplicate row do 2 execution cùng append 1 Key (dedup chỉ theo từng run) | CASES/CASE-051_Duplicate_Row_Cross_Run_Append.md |

## Bài học (Lessons Learned)
| # | Bài học | Đường dẫn |
|---|---------|-----------|
| 1 | Audit duplicate phải quét mọi workflow active ghi cùng đích trước tiên | LESSONS_LEARNED.md |
| 2 | Không hy sinh tính năng chính (retry báo cáo) để chống tác dụng phụ nhỏ (tin trùng) — tăng timeout thay vì tắt retry | LESSONS_LEARNED.md |
| 3 | Hệ thống phải chủ động cảnh báo data trùng (COUNTIF + bôi màu + Telegram), không để người dùng phát hiện bằng mắt | LESSONS_LEARNED.md |
