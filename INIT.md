# INIT

## Hướng dẫn sử dụng

Mỗi khi mở terminal mới hoặc bắt đầu dự án mới, Human chỉ cần gõ:

```
"Đọc INIT.md và làm theo."
```

---

## Quy trình khởi động

### Bước 1 — Đọc quy chuẩn chung
Đọc `AI_OS/README.md` trong thư mục hiện tại.

---

### Bước 2 — Xác định ngữ cảnh & tìm hiểu Project

**Mục tiêu:** xác định đúng Project, Module và Task hiện tại để hiểu đủ ngữ cảnh trước khi Scout và Brainstorm.

AI trò chuyện tự nhiên với Human — không hỏi theo form cố định:

- Trước tiên xác định:
  - Đây là Project mới hay Project đang triển khai?
  - Nếu là Project đang triển khai:
    - Đang tiếp tục Task nào?
    - Hay bắt đầu Task mới?
- Sau khi xác định được ngữ cảnh, tiếp tục hỏi mở và đào sâu khi cần.
- Nếu Human trả lời chưa rõ → hỏi thêm để làm rõ.
- Nếu Human nảy sinh ý mới → theo dòng đó.
- Dừng khi đã hiểu đủ:
  - Project giải quyết vấn đề gì.
  - Task hiện tại là gì.
  - Kết quả mong đợi là gì.
  - Đã có gì rồi.
  - Chưa có gì.

> Không chốt vội. Thà hỏi thêm còn hơn Scout và Brainstorm sai hướng.
---

### Bước 3 — Scout
Dựa trên thông tin Human vừa cung cấp:
- Dò toàn bộ thư mục hiện tại
- Xác định file và thư mục nào đã có
- Xác định điểm cần chú ý trước khi Brainstorm

Báo Human tóm tắt những gì tìm được.

---

### Bước 3.5 — Kiểm tra CASES/ nếu gặp vấn đề
Nếu trong quá trình làm việc gặp vấn đề kỹ thuật:
- Đọc `Project/CASES/CASE_INDEX.md` trước
- Tìm CASE hoặc PATTERN liên quan
- Áp dụng lại — không mò lại từ đầu

### Bước 4 — Brainstorm
Dựa trên kết quả Scout và thông tin từ Human:
- Đề xuất hướng triển khai
- Phân tích ưu nhược điểm từng hướng
- Đề xuất hướng tối ưu nhất

Hỏi Human:
```
"Tôi đề xuất hướng [X] vì [lý do].
Human xác nhận hướng này không?"
```

Chờ Human xác nhận mới đi tiếp.

---

### Bước 5 — Plan
Dựa trên hướng đi đã được Human xác nhận:
- Lên kế hoạch chi tiết từng Module
- Xác định Task trong Module đầu tiên
- Xác định thứ tự thực hiện

---

### Bước 6 — Đề xuất điền file
Sau khi Plan xong, AI đề xuất nội dung từng file — Human xác nhận từng file trước khi AI ghi:

```
Thứ tự đề xuất:
1. Project/README.md
2. Project/ROADMAP.md
3. Module/README.md    ← Module đầu tiên
4. Module/PLAN.md
5. Module/TASK_INDEX.md
6. Task/README.md      ← Task đầu tiên
7. Task/PLAN.md
```

> Quy tắc: AI đề xuất nội dung → Human xác nhận → AI ghi vào file.
> Human không xác nhận → không ghi.

---

### Bước 7 — Báo sẵn sàng
Sau khi tất cả file được xác nhận và ghi xong:

```
"Tôi đã sẵn sàng. Cấu trúc Project đã được tạo:
- Project: [tên]
- Module đầu tiên: [tên]
- Task đầu tiên: [tên] — [mô tả ngắn]

Bắt đầu thực hiện Task đầu tiên không?"
```

---

## Lưu ý quan trọng

- Hỏi từng câu một — không hỏi tất cả cùng lúc
- Không bỏ qua bước Scout trước Brainstorm
- Không ghi file khi chưa có xác nhận của Human
- Xưng hô với Human theo đúng quy tắc trong `AI_OS/README.md`
- Khi đọc hoặc ghi file .md trên Windows: 
  bắt buộc dùng encoding UTF-8.
  Đọc: Get-Content -Encoding UTF8
  Ghi: Set-Content -Encoding UTF8
  Ghi thêm: Add-Content -Encoding UTF8

