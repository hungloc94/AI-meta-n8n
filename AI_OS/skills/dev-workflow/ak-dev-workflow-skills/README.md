# Bộ Skill Workflow Dev cho AI Agent (Claude Code)

6 skill `/ak` theo đúng vòng đời làm phần mềm cùng AI — từ **ý tưởng** tới **sửa lỗi**.
Đây là tài liệu học cho lớp **AI Blackboard**: đọc `SKILL.md` trong mỗi thư mục để hiểu
cách một AI agent được "giao việc" bài bản thay vì hỏi bừa.

## Thứ tự dùng (workflow chuẩn)

| Bước | Skill | Dùng khi |
|------|-------|----------|
| 1 | `ak-brainstorm` | Bàn ý tưởng, cân nhắc phương án TRƯỚC khi code |
| 2 | `ak-plan` | Chốt phương án → chia thành kế hoạch từng phase |
| 3 | `ak-cook` | Thực thi kế hoạch — viết code thật theo plan |
| 4 | `ak-test` | Chạy test, kiểm tra coverage, xác minh hành vi |
| 5 | `ak-code-review` | Soát chất lượng, bảo mật, hiệu năng trước khi merge |
| 6 | `ak-fix` | Sửa bug (tự scout + chẩn đoán rồi vá) |

Chuỗi điển hình: `brainstorm → plan → cook → test → code-review`, và khi có lỗi thì `fix`.

## Cách cài (Claude Code)

Chép các thư mục skill vào `~/.claude/skills/` rồi gọi bằng lệnh gạch chéo, ví dụ
`/ak:brainstorm`, `/ak:plan`. Mỗi skill có `SKILL.md` mô tả quy tắc + quy trình.

> Lưu ý: một vài skill tham chiếu tới file quy tắc chung (`~/.claude/rules/*`) và skill
> phụ trợ trong hệ thống gốc — bản đóng gói này tập trung vào 6 skill cốt lõi để HỌC
> cách tư duy workflow, không phải bản chạy production đầy đủ.

— Bình Vương · hoc.binhvuong.vn
