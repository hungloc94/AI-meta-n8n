# 1. Overview

## Purpose

Định nghĩa AI Operating System và cấu trúc tổ chức công việc theo mô hình Project → Module → Task.

---

## Architecture

```text
AI Operating System
        │
        ▼
     Project
        │
        ▼
      Module
        │
        ▼
       Task
```

| Level | Definition | Example |
|-------|------------|---------|
| **AI Operating System** | Bộ tiêu chuẩn chung giúp tất cả AI Agent làm việc theo cùng một quy trình và cấu trúc. | AI_OS |
| **Project** | Một sản phẩm hoặc hệ thống hoàn chỉnh, được cấu thành từ nhiều Module. Project hoàn thành khi tất cả Module trong phạm vi của Project hoàn thành. | Meta Report Automation, CRM App |
| **Module** | Một nhóm chức năng trong Project, được cấu thành từ nhiều Task. Module hoàn thành khi tất cả Task trong phạm vi của Module hoàn thành. | KPI Engine, Data Collection |
| **Task** | Đơn vị công việc nhỏ nhất có thể thực hiện và hoàn thành độc lập. | Build Date Range Engine, Fix Google Sheet Range Error |
# 2. File Standard

## Mục tiêu

Định nghĩa vai trò của từng file trong hệ thống, AI đọc file đó để làm gì, khi nào bắt buộc cập nhật và cách cập nhật.

> **Nguyên tắc cốt lõi:** AI chỉ đọc đúng file cần thiết cho công việc hiện tại. Không đọc thừa. Không bỏ sót.

---

## Safety Rules

> ⚠️ **Đây là quy tắc bắt buộc. AI phải đọc và tuân thủ trước khi thực hiện bất kỳ thao tác nào với file.**

### RULE 1 — Không xóa file khi chưa được phép
Tuyệt đối không xóa bất kỳ file nào.
AI muốn xóa file → hỏi Human → chờ Human duyệt → mới được xóa.

### RULE 2 — Không thay đổi file khi chưa backup
Trước khi thay đổi bất kỳ file nào → backup file đó trước → mới được thay đổi.

### RULE 3 — Ngoại lệ backup
File được miễn backup khi và chỉ khi:
Human xác nhận rõ ràng **"file này lỗi, không cần backup"**.
- AI không được tự phán đoán file lỗi.
- AI không được tự miễn backup.
- Mọi trường hợp còn lại đều phải backup trước khi thay đổi.

### RULE 4 — Quy tắc cập nhật thống nhất
Mọi cập nhật file đều theo một quy trình duy nhất:
> **AI phát hiện → AI đề xuất với Human → Human duyệt → AI cập nhật.**
Không có ngoại lệ. Trigger giúp AI biết lúc nào cần đề xuất — xem Chương 4.

### RULE 5 — Không tạo file ngoài cấu trúc chuẩn khi chưa được phép
Cấu trúc file chuẩn đã được định nghĩa trong Chương 2. AI không được tự tạo bất kỳ file nào ngoài cấu trúc này.
- AI muốn tạo file mới ngoài chuẩn → đề xuất với Human, nêu rõ lý do → chờ Human duyệt → mới được tạo.
- AI không được tự phán đoán file mới là cần thiết.
- Nếu Human duyệt → ghi rõ lý do ngoại lệ vào `Project/RULES.md` của Project đó.

---

## Cấu trúc file toàn hệ thống

```text
AI_OS/
└── README.md

Project/
├── README.md
├── ROADMAP.md
├── STATUS.md
├── RULES.md
├── MODULE_INDEX.md     ← điều hướng đến các Module trong Project
└── OPS/
    └── <file vận hành>.md

Module/
├── README.md
├── PLAN.md
├── STATUS.md
├── RULES.md        ← không bắt buộc, chỉ tạo khi có quy tắc riêng
└── TASK_INDEX.md

Task/
├── README.md
├── PLAN.md         ← bắt buộc, dù Task chỉ có 1 bước
├── STATUS.md
├── RULES.md        ← không bắt buộc, chỉ tạo khi có quy tắc riêng
├── WORKLOG.md
├── CASE_INDEX.md
└── CASES/
    ├── CASE-001_<Case_Name>.md
    ├── CASE-002_<Case_Name>.md
    └── ...
```

---

## Thứ tự ưu tiên RULES

Nếu có xung đột giữa các RULES ở các tầng khác nhau, tầng gần thực thi nhất sẽ thắng vì quy tắc càng cụ thể càng phản ánh đúng thực tế hơn:

```
Task/RULES.md         (ưu tiên cao nhất — cụ thể nhất)
    ↑
Module/RULES.md
    ↑
Project/RULES.md
    ↑
AI_OS/README.md       (ưu tiên thấp nhất — quy chuẩn chung)
```

---

## Thứ tự đọc file theo tình huống

```
Lần đầu làm việc với Project mới:
1. AI_OS/README.md
2. Project/README.md
3. Project/RULES.md (nếu có)
4. Module/README.md
5. Module/RULES.md  (nếu có)
6. Task/README.md
7. Task/RULES.md    (nếu có)
8. Task/STATUS.md        ← đang ở đâu, HANDOVER là gì
9. Task/WORKLOG.md       ← chỉ đọc phiên có ⚠️
10. Task/PLAN.md         ← làm gì tiếp

Phiên tiếp theo trong cùng Task:
1. Task/STATUS.md        ← đang ở đâu, HANDOVER là gì
2. Task/WORKLOG.md       ← chỉ đọc phiên có ⚠️
3. Task/PLAN.md          ← làm gì tiếp

Quay lại Task sau thời gian dài / nâng cấp / mở rộng:
1. Project/README.md     ← nhắc lại bối cảnh tổng thể
2. Module/README.md      ← nhắc lại phạm vi Module
3. Task/README.md        ← nhắc lại mục tiêu Task
4. Task/STATUS.md        ← đang ở đâu, HANDOVER là gì
5. Task/WORKLOG.md       ← chỉ đọc phiên có ⚠️
6. Task/PLAN.md          ← làm gì tiếp

Nếu không rõ đang ở tình huống nào → hỏi Human trước khi đọc.
```

---

## AI_OS

### `AI_OS/README.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Toàn bộ quy chuẩn chung của AI OS — 5 chương định nghĩa cách AI làm việc |
| **AI đọc khi nào** | Lần đầu làm việc với Project mới. Quay lại sau thời gian dài |
| **AI đọc để làm gì** | Hiểu quy trình, cấu trúc file, và quy tắc làm việc trước khi bắt đầu bất kỳ Task nào |
| **Cập nhật khi nào** | Khi có quyết định thay đổi quy chuẩn chung |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

## PROJECT

### `Project/README.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Mục tiêu, phạm vi, bối cảnh và tổng quan của Project |
| **AI đọc khi nào** | Lần đầu làm việc với Project. Quay lại sau thời gian dài. Khi nâng cấp hoặc mở rộng Project |
| **AI đọc để làm gì** | Hiểu Project đang giải quyết vấn đề gì và phạm vi bao gồm những gì |
| **Cập nhật khi nào** | Khi mục tiêu hoặc phạm vi Project thay đổi |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

### `Project/ROADMAP.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Định hướng phát triển dài hạn — các Module dự kiến và thứ tự ưu tiên |
| **AI đọc khi nào** | Khi cần hiểu bức tranh tổng thể. Khi lên kế hoạch Module mới. Khi nâng cấp Project |
| **AI đọc để làm gì** | Biết Project đang đi đến đâu, Module nào tiếp theo |
| **Cập nhật khi nào** | Khi định hướng phát triển thay đổi hoặc có Module mới được thêm vào |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

### `Project/STATUS.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Tiến độ tổng thể — Module nào xong, đang làm, chưa bắt đầu |
| **AI đọc khi nào** | Khi cần nắm tiến độ tổng thể. Khi quay lại Project sau thời gian dài |
| **AI đọc để làm gì** | Biết Project đang ở đâu mà không cần đọc toàn bộ lịch sử |
| **Cập nhật khi nào** | Khi một Module hoàn thành hoặc trạng thái Module thay đổi |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

### `Project/RULES.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Quy tắc riêng của Project — ưu tiên hơn AI_OS nếu có xung đột |
| **AI đọc khi nào** | Lần đầu làm việc với Project. Quay lại sau thời gian dài |
| **AI đọc để làm gì** | Biết các quy tắc đặc thù cần tuân theo trong Project này |
| **Cập nhật khi nào** | Khi có quyết định thêm hoặc thay đổi quy tắc riêng |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

### `Project/OPS/`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Tài liệu vận hành đặc thù — xem chi tiết trong Project/RULES.md |
| **AI đọc khi nào** | Khi cần tra cứu credential, môi trường, checklist, quy trình vận hành |
| **Lưu ý** | Không bắt buộc — chỉ tạo khi được duyệt trong Project/RULES.md |

---

## MODULE

### `Module/README.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Mục tiêu và phạm vi của Module |
| **AI đọc khi nào** | Lần đầu làm việc với Module. Quay lại sau thời gian dài. Khi nâng cấp Module |
| **AI đọc để làm gì** | Hiểu Module đang giải quyết vấn đề gì trong phạm vi Project |
| **Cập nhật khi nào** | Khi mục tiêu hoặc phạm vi Module thay đổi |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

### `Module/PLAN.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Kế hoạch thực hiện Module — danh sách Task và thứ tự thực hiện |
| **AI đọc khi nào** | Khi bắt đầu Module. Khi cần xác định Task tiếp theo. Khi quay lại Module |
| **AI đọc để làm gì** | Biết cần làm gì tiếp theo trong Module |
| **Cập nhật khi nào** | Khi phát sinh Task mới hoặc thứ tự thực hiện thay đổi |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

### `Module/STATUS.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Tiến độ của Module — Task nào xong, đang làm, bị chặn |
| **AI đọc khi nào** | Đầu mỗi phiên làm việc trong Module. Khi quay lại Module sau thời gian dài |
| **AI đọc để làm gì** | Nắm nhanh tiến độ mà không cần đọc từng Task |
| **Cập nhật khi nào** | Khi trạng thái của bất kỳ Task nào trong Module thay đổi |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

### `Module/RULES.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Quy tắc riêng của Module — ưu tiên hơn Project/RULES.md nếu có xung đột |
| **AI đọc khi nào** | Lần đầu làm việc với Module, nếu file tồn tại. Quay lại sau thời gian dài |
| **AI đọc để làm gì** | Biết các quy tắc đặc thù của Module |
| **Cập nhật khi nào** | Khi có quyết định thêm hoặc thay đổi quy tắc riêng |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |
| **Lưu ý** | Không bắt buộc — chỉ tạo khi Module có quy tắc riêng khác Project |

---

### `Module/TASK_INDEX.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Danh sách tất cả Task trong Module kèm trạng thái và đường dẫn đến thư mục Task |
| **AI đọc khi nào** | Khi cần tìm hoặc điều hướng đến một Task cụ thể. Khi kiểm tra Module có hoàn thành chưa |
| **AI đọc để làm gì** | Tìm Task nhanh. Phát hiện khi tất cả Task đều `[x]` để hỏi Human xác nhận Module xong |
| **Cập nhật khi nào** | Khi có Task mới hoặc trạng thái Task thay đổi |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

**Cấu trúc mẫu:**
```markdown
| Task | Trạng thái | Đường dẫn |
|------|------------|-----------|
| Build Date Range Engine | ✅ Hoàn thành | /Task/Date_Range_Engine/ |
| Fix Google Sheet Range  | 🔄 Đang làm  | /Task/Fix_Sheet_Range/   |
| Parser Rewrite          | ⏳ Chưa bắt đầu | /Task/Parser_Rewrite/ |
```

> **Quy tắc Module hoàn thành:** Khi tất cả Task đều `✅` → AI hỏi Human: *"Module X có vẻ đã xong tất cả Task, Human xác nhận hoàn thành chưa?"* → Human xác nhận → cập nhật Module/STATUS.md và Project/STATUS.md → Backup.

---

## TASK

### `Task/README.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Mục tiêu, phạm vi và mô tả chi tiết của Task |
| **AI đọc khi nào** | Lần đầu làm việc với Task. Quay lại sau thời gian dài. Khi nâng cấp Task |
| **AI đọc để làm gì** | Hiểu Task cần đạt được kết quả gì và giới hạn phạm vi ở đâu |
| **Cập nhật khi nào** | Khi mục tiêu hoặc phạm vi Task thay đổi |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

---

### `Task/PLAN.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Kế hoạch thực hiện Task — các bước cụ thể cần làm theo thứ tự |
| **AI đọc khi nào** | Đầu mỗi phiên làm việc. Quay lại Task sau thời gian dài |
| **AI đọc để làm gì** | Biết bước tiếp theo cần thực hiện |
| **Cập nhật khi nào** | Khi phát hiện cần thêm bước mới hoặc thứ tự bước thay đổi |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |
| **Lưu ý** | Bắt buộc có — dù Task chỉ có 1 bước. Giúp AI không cần phán đoán |

---

### `Task/STATUS.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Trạng thái hiện tại — đang ở bước nào, blocker là gì, tiến độ bao nhiêu %, HANDOVER |
| **AI đọc khi nào** | Bắt buộc đọc đầu tiên mỗi phiên — trước cả WORKLOG và PLAN |
| **AI đọc để làm gì** | Biết ngay Task đang ở đâu, ai đang cầm việc, bước tiếp theo là gì |
| **Cập nhật khi nào** | Sau mỗi bước hoàn thành, khi có blocker mới, khi bàn giao sang Agent khác |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

**Cấu trúc mẫu:**
```markdown
## Trạng thái hiện tại
- **Tiến độ:** 60%
- **Bước đang làm:** Bước 3 — Viết parser
- **Blocker:** Chưa có dữ liệu tháng 7
- **Cập nhật lần cuối:** 2026-07-19

## HANDOVER
- Người giao: Claude (vừa lên xong kế hoạch)
- Người nhận: Codex (sẽ viết code)
- Đã làm xong: PLAN.md đầy đủ 5 bước
- Cần làm tiếp: Viết code theo bước 3 trong PLAN.md
- Xong khi nào: Code chạy không lỗi, Claude Code review PASS
- Trạng thái: ⏳ Chờ Codex bắt đầu
```

---

### `Task/RULES.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Quy tắc riêng của Task — ưu tiên cao nhất nếu có xung đột |
| **AI đọc khi nào** | Lần đầu làm việc với Task, nếu file tồn tại. Quay lại sau thời gian dài |
| **AI đọc để làm gì** | Biết các ràng buộc đặc thù của Task |
| **Cập nhật khi nào** | Khi có quyết định thêm hoặc thay đổi quy tắc |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |
| **Lưu ý** | Không bắt buộc — chỉ tạo khi Task có quy tắc riêng khác Module |

---

### `Task/WORKLOG.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Nhật ký làm việc theo phiên + Signal các việc còn dở, file có thể lỗi thời, Proposal chờ duyệt |
| **AI đọc khi nào** | Đầu mỗi phiên — chỉ đọc phiên có ⚠️, bỏ qua phiên đã xong ✅ |
| **AI đọc để làm gì** | Biết ngay việc còn dở, file nào cần xem lại, Proposal nào đang chờ Human duyệt |
| **Cập nhật khi nào** | Ghi Signal ngay khi phát hiện. Cuối phiên bổ sung Nhật ký |
| **Ai cập nhật** | Signal: AI ghi ngay khi phát hiện. Nhật ký và Proposal: AI đề xuất → Human duyệt → AI cập nhật |

**Cấu trúc mẫu:**
```markdown
## [2026-07-10] ✅

### Nhật ký
- Hoàn thành bước 2: viết hàm parse date
- Không có vấn đề phát sinh

### Signal
OPEN_TASKS:
- [x] Viết hàm parse date
      → Hoàn thành lúc: kết thúc phiên

STALE_DOCS:
- [x] PLAN.md → đã cập nhật bước 2 hoàn thành

PROPOSAL:
- [x] STATUS.md → đã cập nhật tiến độ lên 40%

---

## [2026-07-19] ⚠️

### Nhật ký
- Bắt đầu bước 3: viết parser
- Phát hiện thiếu dữ liệu tháng 7, đã báo Human
- Human chưa phản hồi, tạm dừng

### Signal
OPEN_TASKS:
- [ ] Cần thu thập dữ liệu tháng 7 trước
      → Phát hiện khi: đang làm bước 3
      → Lý do: chưa có file dữ liệu đầu vào

STALE_DOCS:
- [ ] PLAN.md → cần bổ sung bước thu thập dữ liệu
      → Phát hiện khi: đang làm bước 3

PROPOSAL:
- [ ] PLAN.md → thêm bước thu thập dữ liệu tháng 7
      → Sửa chỗ: sau bước 2
      → Lý do: thiếu bước này Cook không chạy được
      → Tạo lúc: kết thúc bước 3
```

> **Quy tắc Signal:**
> - Ghi ngay khi phát hiện — không đợi cuối phiên
> - Đánh dấu `[x]` khi đã xử lý xong
> - AI chỉ đọc phiên có `⚠️` — bỏ qua phiên đã xong `✅`
> - Phiên không còn dòng `[ ]` nào → đổi sang `✅`

---

### `Task/CASE_INDEX.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Danh sách tất cả CASE của Task kèm mô tả ngắn và đường dẫn vào thư mục CASES/ |
| **AI đọc khi nào** | Khi gặp vấn đề và cần tra cứu xem đã có CASE tương tự chưa |
| **AI đọc để làm gì** | Điều hướng nhanh đến đúng CASE — không cần duyệt toàn bộ CASES/ |
| **Cập nhật khi nào** | Khi có CASE mới được tạo |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

**Cấu trúc mẫu:**
```markdown
| CASE | Mô tả ngắn | Đường dẫn |
|------|------------|-----------|
| CASE-001 | Google Sheet trả về sai range | CASES/CASE-001_Google_Sheet_Range_Error.md |
| CASE-002 | Date filter bỏ sót record    | CASES/CASE-002_Date_Filter_Missing_Record.md |
```

---

## TASK / CASES

### `CASES/CASE-XXX_<Case_Name>.md`

| Mục | Nội dung |
|-----|----------|
| **Chứa gì** | Một trường hợp đã được xác minh: vấn đề → nguyên nhân → cách xử lý → bài học |
| **AI đọc khi nào** | Khi gặp vấn đề tương tự và được điều hướng từ CASE_INDEX.md |
| **AI đọc để làm gì** | Áp dụng lại cách xử lý đã xác minh — không mò lại từ đầu |
| **Cập nhật khi nào** | Khi có thông tin mới hoặc cách xử lý được cải thiện |
| **Ai cập nhật** | AI đề xuất → Human duyệt → AI cập nhật |

**Cấu trúc mẫu:**
```markdown
## CASE-001: Google Sheet Range Error
- **Ngày phát hiện:** 2026-07-10
- **Ngày xác minh:** 2026-07-12

### Vấn đề
Sheet trả về dữ liệu sai range khi query ngày cuối tháng.

### Nguyên nhân
Header row bị tính vào range — offset lệch 1 dòng.

### Cách xử lý
Thêm `startRowIndex: 1` để bỏ qua header row.

### Bài học
Luôn kiểm tra offset header khi query Google Sheet theo range động.
```

> **Lưu ý:** CASE ghi ngày phát hiện và ngày xác minh để AI đánh giá thông tin còn phù hợp không — đặc biệt khi thư viện hoặc API thay đổi theo thời gian.
# 3. AI Workflow

## Mục tiêu

Định nghĩa quy trình AI làm việc từ lúc nhận Task đến khi hoàn thành, bao gồm phân vai Agent, thứ tự thực hiện, cập nhật tài liệu và bàn giao giữa các Agent.

> **Nguyên tắc cốt lõi:** AI phát hiện → AI đề xuất với Human → Human duyệt → AI cập nhật 100%.

---

## Các Agent và vai trò

| Agent | Vai trò | Bước thực hiện |
|---|---|---|
| Claude | Brainstorm, Plan, chốt quyết định | Brainstorm → Plan |
| Grok | Brainstorm, Cook, Fix | Brainstorm → Cook → Fix |
| Claude Code | Scout, Review, Update Docs, Debug | Scout → Review → Update Docs → Debug |
| Codex | Cook, Fix | Cook → Fix |

---

## Workflow tổng thể

```
Brainstorm (Claude + Grok)
        ↓
Scout (Claude Code)
        ↓
Plan (Claude chốt)
        ↓
Human duyệt PLAN.md
        ↓
Cook (Codex hoặc Grok)
        ↓
Review (Claude Code)
        ↓
Review PASS?
    ↓ YES                    ↓ NO
Update Docs             Debug (Claude Code)
(Claude Code)                ↓
        ↓               Fix (Grok hoặc Codex)
   Task xong                 ↓
                        Review lại (Claude Code)
                             ↓
                        Update Docs (Claude Code)
```

---

## Bảng File Mapping — Mỗi bước cập nhật file nào

| Bước | Agent | File có thể cần cập nhật |
|---|---|---|
| Brainstorm | Claude + Grok | `Project/README.md` — nếu có quyết định thay đổi hướng đi |
| Scout | Claude Code | `Task/PLAN.md` — danh sách file liên quan vừa tìm được |
| Plan | Claude | `Task/PLAN.md` — kế hoạch từng bước + `Task/STATUS.md` HANDOVER |
| Cook | Codex / Grok | `Task/STATUS.md` tiến độ + `Task/WORKLOG.md` Signal |
| Review PASS | Claude Code | chuyển sang Update Docs |
| Review FAIL | Claude Code | `Task/WORKLOG.md` Signal blocker + `Task/STATUS.md` HANDOVER |
| Update Docs | Claude Code | `Task/WORKLOG.md` Nhật ký + `Task/CASE_INDEX.md` + `CASES/` nếu có |
| Debug | Claude Code | `Task/WORKLOG.md` Signal + `Task/STATUS.md` HANDOVER |
| Fix | Grok / Codex | `Task/STATUS.md` HANDOVER + `Task/WORKLOG.md` Signal nếu có thêm vấn đề |

> **Lưu ý:** Mọi cập nhật đều theo quy trình: AI đề xuất → Human duyệt → AI cập nhật. Không có ngoại lệ.

---

## Chi tiết từng bước

### Bước 1 — Brainstorm
**Agent:** Claude + Grok

**Làm gì:**
- Phân tích yêu cầu từ Human
- Đề xuất các hướng giải quyết
- Phản biện lẫn nhau
- Chốt hướng đi trước khi tiếp tục

**File đọc:**
- `Project/README.md` — hiểu bối cảnh Project
- `Project/RULES.md` — quy tắc Project
- `Task/README.md` — hiểu yêu cầu Task

**File có thể cần cập nhật:**
- `Project/README.md` — nếu Brainstorm dẫn đến quyết định thay đổi hướng đi

**Kết thúc khi:** Human xác nhận hướng đi.

---

### Bước 2 — Scout
**Agent:** Claude Code

**Làm gì:**
- Dò toàn bộ codebase liên quan
- Xác định file nào sẽ bị ảnh hưởng
- Kiểm tra CASE_INDEX.md xem đã có vấn đề tương tự chưa
- Liệt kê các điểm cần chú ý trước khi động vào code

**File đọc:**
- `Task/PLAN.md` — biết mục tiêu cần scout
- `Task/CASE_INDEX.md` — kiểm tra đã có CASE tương tự chưa

**File có thể cần cập nhật:**
- `Task/PLAN.md` — bổ sung danh sách file liên quan vừa tìm được

**Kết thúc khi:** Claude Code liệt kê đủ file liên quan và báo Human.

---

### Bước 3 — Plan
**Agent:** Claude

**Làm gì:**
- Biến yêu cầu thành kế hoạch triển khai từng bước rõ ràng
- Chưa viết code — chỉ lên kế hoạch
- Ghi rõ từng bước, thứ tự, và kết quả mong đợi của mỗi bước

**File đọc:**
- `Task/README.md` — yêu cầu Task
- `Task/PLAN.md` — kế hoạch hiện tại nếu có
- `Task/STATUS.md` — đang ở đâu

**File có thể cần cập nhật:**
- `Task/PLAN.md` — kế hoạch đầy đủ từng bước
- `Task/STATUS.md` — cập nhật HANDOVER cho Codex/Grok

**Cấu trúc HANDOVER sau Plan:**
```markdown
## HANDOVER
- Người giao: Claude (vừa lên xong kế hoạch)
- Người nhận: Codex hoặc Grok (sẽ viết code)
- Đã làm xong: PLAN.md đầy đủ N bước
- Cần làm tiếp: Viết code theo từng bước trong PLAN.md
- Xong khi nào: Code chạy không lỗi, Claude Code review PASS
- Trạng thái: ⏳ Chờ Human duyệt
```

**Kết thúc khi:** Human đọc PLAN.md và xác nhận "ok".

---

### Bước 4 — Cook
**Agent:** Codex hoặc Grok

**Làm gì:**
- Viết code theo đúng kế hoạch trong PLAN.md
- Không tự ý thay đổi phạm vi ngoài PLAN.md
- Nếu phát hiện cần làm thêm việc ngoài kế hoạch → ghi vào WORKLOG Signal ngay, báo Human

**File đọc:**
- `Task/STATUS.md` — đọc HANDOVER để biết bắt đầu từ đâu
- `Task/PLAN.md` — làm theo từng bước
- `Task/RULES.md` — quy tắc riêng của Task nếu có

**File có thể cần cập nhật:**
- `Task/STATUS.md` — cập nhật tiến độ sau mỗi bước hoàn thành + HANDOVER sang Claude Code
- `Task/WORKLOG.md` — ghi Signal ngay khi phát hiện blocker hoặc việc ngoài kế hoạch

**Cấu trúc HANDOVER sau Cook:**
```markdown
## HANDOVER
- Người giao: Codex (vừa viết xong code)
- Người nhận: Claude Code (sẽ review)
- Đã làm xong: Code theo PLAN.md bước 1–N
- Cần làm tiếp: Review code, kiểm tra đúng yêu cầu
- Xong khi nào: Review PASS, không có lỗi
- Trạng thái: ⏳ Chờ Claude Code review
```

**Kết thúc khi:** Codex/Grok báo "done" và cập nhật HANDOVER trong STATUS.md.

---

### Bước 5 — Review
**Agent:** Claude Code

**Làm gì:**
- Đọc code vừa được Cook
- Kiểm tra đúng yêu cầu trong PLAN.md chưa
- Kiểm tra lỗi, edge case, vấn đề tiềm ẩn

**File đọc:**
- `Task/STATUS.md` — đọc HANDOVER từ Codex/Grok
- `Task/PLAN.md` — đối chiếu code với kế hoạch

**Kết thúc khi:**
- **PASS** → chuyển sang Update Docs
- **FAIL** → chuyển sang Debug

---

### Bước 6a — Update Docs
**Agent:** Claude Code

**Làm gì:**
- Ghi lại những gì đã làm trong phiên này
- Tạo CASE mới nếu phát hiện vấn đề đáng ghi nhớ
- Đề xuất với Human: *"Mảng X đã xong, tôi ghi tổng kết vào WORKLOG nhé?"*
- Chờ Human xác nhận rồi mới ghi

**File có thể cần cập nhật:**
- `Task/WORKLOG.md` — ghi Nhật ký phiên, đánh dấu `[x]` Signal đã xử lý
- `Task/STATUS.md` — cập nhật tiến độ, xóa HANDOVER cũ
- `Task/CASE_INDEX.md` + `CASES/` — nếu có CASE mới

**Kết thúc khi:** Human xác nhận "ok" hoặc giao Task tiếp theo.

---

### Bước 6b — Debug
**Agent:** Claude Code

**Làm gì:**
- Tìm nguyên nhân gốc của lỗi — không vá bừa
- Ghi blocker vào WORKLOG Signal ngay khi phát hiện
- Báo Human nếu lỗi vượt quá phạm vi tự xử lý

**File có thể cần cập nhật:**
- `Task/WORKLOG.md` — ghi Signal blocker ngay khi phát hiện
- `Task/STATUS.md` — cập nhật HANDOVER sang Grok/Codex để fix

**Cấu trúc HANDOVER sau Debug:**
```markdown
## HANDOVER
- Người giao: Claude Code (đã tìm ra nguyên nhân lỗi)
- Người nhận: Grok hoặc Codex (sẽ fix)
- Đã làm xong: Xác định nguyên nhân — [mô tả lỗi]
- Cần làm tiếp: Fix theo hướng — [mô tả hướng fix]
- Xong khi nào: Code chạy lại không lỗi, Claude Code review PASS lần 2
- Trạng thái: ⏳ Chờ Grok/Codex fix
```

**Kết thúc khi:** Claude Code xác định được nguyên nhân và cập nhật HANDOVER.

---

### Bước 7 — Fix
**Agent:** Grok hoặc Codex

**Làm gì:**
- Fix đúng theo hướng Claude Code đã xác định trong HANDOVER
- Không tự ý thay đổi ngoài phạm vi fix
- Sau fix → Claude Code review lại

**File đọc:**
- `Task/STATUS.md` — đọc HANDOVER từ Claude Code
- `Task/WORKLOG.md` — đọc Signal blocker

**File có thể cần cập nhật:**
- `Task/STATUS.md` — cập nhật HANDOVER cho Claude Code review lại
- `Task/WORKLOG.md` — ghi Signal nếu phát hiện thêm vấn đề

**Kết thúc khi:** Grok/Codex báo "fixed" và cập nhật HANDOVER.

Sau Fix → quay lại **Bước 5 — Review**.

---

## Quy tắc bắt buộc

### Quy tắc 1 — Đọc STATUS.md trước tiên
Mọi Agent bắt đầu phiên mới đều phải đọc `Task/STATUS.md` đầu tiên — biết đang ở bước nào, HANDOVER đang chờ ai. Không tự đoán.

### Quy tắc 2 — Ghi WORKLOG Signal ngay khi phát hiện
Không đợi cuối phiên. Phát hiện blocker hoặc việc ngoài kế hoạch → ghi ngay vào WORKLOG Signal → báo Human.

### Quy tắc 3 — Không tự ý mở rộng phạm vi
Cook hoặc Fix phát hiện cần làm thêm việc ngoài PLAN.md → không tự làm → ghi vào WORKLOG Signal → hỏi Human.

### Quy tắc 4 — HANDOVER phải cập nhật trước khi bàn giao
Agent giao việc phải cập nhật HANDOVER trong STATUS.md trước khi báo Agent nhận bắt đầu.

### Quy tắc 5 — Update Docs phải hỏi trước khi ghi
Claude Code không tự ghi WORKLOG mà phải hỏi Human trước:
> *"Mảng [tên mảng] đã xong, tôi ghi tổng kết vào WORKLOG nhé?"*
Chờ Human xác nhận mới ghi.

### Quy tắc 6 — Proposal đúng lúc, đúng nội dung

**AI đề xuất cập nhật khi:**
- Vừa hoàn thành một bước
- Vừa giải quyết xong blocker
- Chuẩn bị bàn giao sang Agent khác
- Chuẩn bị Backup

**AI không đề xuất khi:**
- Đang giữa chừng thực hiện — chưa có kết quả rõ ràng
- Human đang chờ kết quả khác quan trọng hơn

**Nội dung Proposal phải ghi rõ:**
- Sửa file nào
- Sửa chỗ nào
- Lý do tại sao
- Tạo lúc nào

> AI không được đề xuất chung chung "file này cần cập nhật" — phải cụ thể đủ để Human duyệt ngay mà không cần hỏi thêm.

### Quy tắc 7 — Hỏi khi không rõ bước hiện tại
Nếu STATUS.md không ghi rõ đang ở bước nào → AI hỏi Human ngay, không tự suy đoán.

---

## Quy tắc xưng hô — Dấu hiệu nhận biết AI đọc đủ tài liệu

> **Bất kỳ AI Agent nào làm việc trong hệ thống này đều phải xưng hô với người dùng là "anh Lộc".**

Nếu AI không gọi "anh Lộc" → anh Lộc biết ngay AI đó chưa đọc đủ tài liệu → yêu cầu AI đọc lại `AI_OS/README.md` trước khi tiếp tục.

Đây là dấu hiệu kiểm tra đơn giản nhất để phát hiện AI bị miss context.
# 4. Documentation Workflow

## Mục tiêu

AI tự phát hiện khi tài liệu không còn phản ánh đúng thực tế và tự đề xuất cập nhật — không cần Human nhắc.

> **Nguyên tắc cốt lõi:** AI phát hiện → AI đề xuất với Human → Human duyệt → AI cập nhật 100%.

---

## Trigger — Khi nào bắt đầu Documentation Review

AI bắt đầu kiểm tra tài liệu khi xuất hiện một trong các trigger sau:

| Trigger | Ví dụ cụ thể | AI tự phát hiện? |
|---|---|---|
| Human giao Task mới | "Làm tiếp bước 4 đi" | ✅ Dễ — Human nói trực tiếp |
| AI hoàn thành một bước | Cook xong, Fix xong, Scout xong | ✅ Dễ — AI tự biết vừa xong gì |
| Phát hiện blocker | Thiếu dữ liệu, lỗi không tự xử lý được | ✅ Dễ — AI gặp vấn đề là biết ngay |
| Task hoàn thành | Task đánh dấu `[x]` Done | ✅ Dễ — Backup ngay sau khi Task xong |
| Tất cả Task trong Module xong | TASK_INDEX.md toàn bộ là `[x]` | ✅ Dễ — AI đọc TASK_INDEX thấy toàn `[x]` |

> **Quy tắc Backup:** Xong 1 Task = Backup ngay — không cần Human nhắc.

> **Quy tắc Module hoàn thành:** Khi tất cả Task trong TASK_INDEX.md đều `[x]`, AI hỏi Human: *"Module X có vẻ đã xong tất cả Task, Human xác nhận Module này hoàn thành chưa?"* → Human xác nhận → cập nhật `Module/STATUS.md` và `Project/STATUS.md` → Backup.

---

## Workflow

```
Trigger xảy ra
        ↓
AI đọc STATUS.md — xác định đang ở bước nào
(đọc HANDOVER để biết bước hiện tại và bước tiếp theo)
        ↓
AI đọc WORKLOG — tìm phiên có dòng [ ] chưa xử lý
(nhìn vào ngày — chỉ đọc phiên có ⚠️, bỏ qua phiên đã xong ✅)
        ↓
Có STALE_DOCS [ ] không?
        ↓
      CÓ
        ↓
AI đọc đúng file được đánh dấu trong STALE_DOCS
        ↓
AI tạo Proposal — đề xuất cụ thể sửa gì, ở đâu
        ↓
AI báo Human:
"File X cần cập nhật chỗ [mô tả cụ thể] — Human duyệt không?"
        ↓
Human duyệt
        ↓
AI cập nhật 100%
        ↓
Đánh dấu [x] trong WORKLOG Signal
```

> **Cách AI xác định bước hiện tại:**
> AI đọc `Task/STATUS.md` trước tiên — phần HANDOVER ghi rõ:
> - Bước đang làm là gì
> - Agent nào đang cầm việc
> - Bước tiếp theo là gì
>
> Nếu STATUS.md không rõ → AI hỏi Human trước khi làm bất cứ thứ gì.

> **Cách AI đọc WORKLOG tiết kiệm token:**
> Mỗi phiên WORKLOG có ghi ngày + trạng thái:
> - `## [2026-07-10] ✅` → không còn `[ ]` → bỏ qua
> - `## [2026-07-19] ⚠️` → còn `[ ]` chưa xử lý → đọc phiên này
>
> AI chỉ đọc các phiên có `⚠️` — không đọc toàn bộ lịch sử.

---

## File Mapping — Thay đổi gì cập nhật vào file nào

| Trigger | File có thể cần cập nhật |
|---|---|
| Brainstorm chốt hướng mới | `Task/README.md` — nếu có quyết định thay đổi hướng đi |
| Scout xong | `Task/PLAN.md` — danh sách file liên quan |
| Plan xong | `Task/PLAN.md` — kế hoạch từng bước |
| Cook xong | `Task/STATUS.md` + `Task/WORKLOG.md` Signal |
| Update Docs xong | `Task/WORKLOG.md` Nhật ký + `CASES/` nếu có |
| Debug xong | `Task/WORKLOG.md` Signal — ghi blocker |
| Fix xong | `Task/CASE_INDEX.md` + `CASES/` mới + `Task/STATUS.md` |
| Mục tiêu hoặc phạm vi Task thay đổi | `Task/README.md` |
| Thêm bước mới vào Task đang làm | `Task/PLAN.md` |
| Thêm công việc mới ngoài kế hoạch | `Task/WORKLOG.md` Signal → `Module/TASK_INDEX.md` sau khi Human duyệt |
| Hoàn thành một bước hoặc có blocker mới | `Task/STATUS.md` |
| Phát hiện blocker hoặc việc ngoài kế hoạch | `Task/WORKLOG.md` Signal |
| Làm xong một mảng hoặc giải quyết được blocker | `Task/WORKLOG.md` Nhật ký |
| Phát hiện lỗi có thể tái diễn | `Task/CASE_INDEX.md` + `CASES/` |
| Trạng thái Task thay đổi | `Module/STATUS.md` |
| Module hoàn thành | `Project/STATUS.md` |
| Định hướng Project thay đổi | `Project/ROADMAP.md` |

> **Lưu ý:** Mọi cập nhật trong bảng này đều phải qua quy trình: AI đề xuất → Human duyệt → AI cập nhật. Không có ngoại lệ.

---

## STALE_DOCS là gì

**STALE_DOCS** = danh sách các file mà AI nghi ngờ đang bị lỗi thời — chưa được cập nhật theo thực tế.

Không phải file riêng. Nằm trong phần Signal của `Task/WORKLOG.md`.

Khi phát hiện file có thể lỗi thời, AI ghi ngay — không đợi cuối phiên:

```markdown
## [2026-07-19] ⚠️

### Signal
OPEN_TASKS:
- [ ] Cần thu thập dữ liệu tháng 7 trước
      → Phát hiện khi: đang Cook bước 3

STALE_DOCS:
- [ ] PLAN.md → cần bổ sung bước thu thập dữ liệu
      → Phát hiện khi: đang Cook bước 3
- [ ] STATUS.md → tiến độ chưa cập nhật sau bước 2 hoàn thành
      → Phát hiện khi: bắt đầu bước 3

PROPOSAL:
- [ ] PLAN.md → thêm bước thu thập dữ liệu tháng 7
      → Sửa chỗ: sau bước 2
      → Lý do: thiếu bước này Cook không chạy được
      → Tạo lúc: kết thúc bước 3
```

---

## Cách AI tạo Proposal

Khi đọc STALE_DOCS và đã đọc file liên quan, AI báo Human theo mẫu:

```
"Tôi phát hiện [số] file cần cập nhật:

1. PLAN.md — cần thêm bước thu thập dữ liệu tháng 7
   Lý do: phát hiện khi đang Cook bước 3,
   thiếu bước này thì không chạy được

2. STATUS.md — cần cập nhật tiến độ từ 40% lên 60%
   Lý do: bước 2 đã xong nhưng chưa được ghi nhận

Human duyệt thì tôi cập nhật luôn ạ."
```

---

## Quy tắc bắt buộc

### Quy tắc 1 — Đọc STATUS.md trước tiên
Mọi Agent bắt đầu phiên mới đều phải đọc `Task/STATUS.md` đầu tiên để biết đang ở bước nào — không tự đoán.

### Quy tắc 2 — Ghi STALE_DOCS ngay khi phát hiện
Không đợi cuối phiên. Phát hiện file có thể lỗi thời → ghi ngay vào WORKLOG Signal.

### Quy tắc 3 — Chỉ đọc phiên WORKLOG có ⚠️
AI chỉ đọc các phiên có dòng `[ ]` chưa xử lý — không đọc toàn bộ lịch sử WORKLOG.

### Quy tắc 4 — Không tự cập nhật khi chưa được duyệt
AI không được tự ý sửa bất kỳ file nào trước khi Human xác nhận — dù AI chắc chắn là đúng.

### Quy tắc 5 — Đánh dấu xong sau khi cập nhật
Sau khi cập nhật file xong → đánh dấu `[x]` trong WORKLOG Signal ngay lập tức.

### Quy tắc 6 — Proposal phải cụ thể và đúng thời điểm

**AI đề xuất khi:**
- Vừa hoàn thành một bước
- Vừa giải quyết xong blocker
- Chuẩn bị bàn giao sang Agent khác
- Chuẩn bị Backup

**AI không đề xuất khi:**
- Đang giữa chừng thực hiện — chưa có kết quả rõ ràng
- Human đang chờ kết quả khác quan trọng hơn

**Nội dung Proposal phải ghi rõ:**
- Sửa file nào
- Sửa chỗ nào
- Lý do tại sao
- Tạo lúc nào

> AI không được báo chung chung "file này cần cập nhật" — phải cụ thể đủ để Human duyệt ngay mà không cần hỏi thêm.

### Quy tắc 7 — Hỏi khi không rõ bước hiện tại
Nếu STATUS.md không ghi rõ đang ở bước nào → AI hỏi Human ngay, không tự suy đoán.
# 5. Backup Strategy

## Mục tiêu

Định nghĩa thời điểm, phạm vi và quy tắc Backup để luôn có thể khôi phục khi cần.

> **Nguyên tắc cốt lõi:** Backup trước khi thay đổi. Backup sau khi hoàn thành. Không có ngoại lệ.

---

## Khi nào phải Backup

| Trigger | Mô tả |
|---|---|
| Trước khi thay đổi bất kỳ file nào | Backup file đó trước — xem Safety Rules Chương 2 |
| Task hoàn thành | Xong 1 Task = Backup ngay toàn bộ Project |
| Module hoàn thành | Human xác nhận Module xong → Backup ngay |
| Human yêu cầu | Human nói "backup đi" → Backup ngay |

> **Lưu ý:** AI không tự quyết định bỏ qua Backup. Mọi trường hợp nghi ngờ → Backup.

---

## Phạm vi Backup

> **Backup toàn bộ thư mục Project** — không chọn lọc file.

Lý do: đơn giản, AI không cần phán đoán file nào cần backup. Đến trigger là backup hết.

---

## Vị trí lưu Backup

Backup nằm trong thư mục `BACKUP/` cùng cấp với Project:

```text
Project/
├── README.md
├── ROADMAP.md
├── STATUS.md
├── RULES.md
├── Module_1/
├── Module_2/
└── BACKUP/
    ├── 2026-07-10_14-30_task-done_Build-Date-Range-Engine/
    ├── 2026-07-15_09-00_module-done_Data-Collection/
    └── 2026-07-19_16-45_human-request/
```

---

## Quy tắc đặt tên Backup

```
YYYY-MM-DD_HH-MM_<lý-do>_<tên-cụ-thể-nếu-có>/
```

| Thành phần | Ý nghĩa | Ví dụ |
|---|---|---|
| `YYYY-MM-DD` | Ngày backup | `2026-07-19` |
| `HH-MM` | Giờ backup | `16-45` |
| `<lý-do>` | Tại sao backup | `task-done`, `module-done`, `before-change`, `human-request` |
| `<tên-cụ-thể>` | Tên Task hoặc Module nếu có | `Build-Date-Range-Engine` |

**Ví dụ tên backup:**

```
2026-07-19_16-45_task-done_Build-Date-Range-Engine/
2026-07-19_17-00_before-change_PLAN/
2026-07-20_09-00_module-done_Data-Collection/
2026-07-20_10-30_human-request/
```

---

## Quy trình Backup

```
Trigger xảy ra
        ↓
AI thông báo Human:
"Tôi sẽ backup toàn bộ Project trước khi tiếp tục."
        ↓
AI tạo thư mục backup theo đúng tên quy chuẩn
        ↓
AI copy toàn bộ thư mục Project vào BACKUP/
(không copy thư mục BACKUP/ vào trong backup)
        ↓
AI xác nhận với Human: "Backup xong tại BACKUP/<tên-backup>/"
        ↓
Tiếp tục công việc
```

---

## Quy tắc bắt buộc

### Quy tắc 1 — Không bao giờ bỏ qua Backup
AI không được tự ý bỏ qua bước Backup dù lý do gì. Nếu muốn bỏ qua → hỏi Human.

### Quy tắc 2 — Không copy BACKUP/ vào trong backup
Khi backup, chỉ copy nội dung Project — không bao gồm thư mục `BACKUP/` để tránh backup lồng nhau.

### Quy tắc 3 — Thông báo trước và sau
AI phải thông báo Human trước khi backup và xác nhận sau khi backup xong — không làm im lặng.

### Quy tắc 4 — Đặt tên đúng quy chuẩn
Tên backup phải theo đúng format `YYYY-MM-DD_HH-MM_<lý-do>_<tên-cụ-thể>/` để dễ tra cứu và khôi phục sau này.

### Quy tắc 5 — Kiểm tra WORKLOG trước khi Backup
Trước khi Backup, AI đọc `Task/WORKLOG.md` Signal — nếu còn dòng `[ ]` chưa xử lý thì xử lý hết trước, sau đó mới Backup. Không Backup khi còn Signal chưa xử lý.