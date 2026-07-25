# Rules — n8n Meta Ads

## 1. Safety Rules — Không được vi phạm

### Production
- Không patch production trực tiếp — luôn dùng TEST workflow trước.
- Luôn export backup trước khi patch.
- Imported workflow phải giữ `active=false` cho đến khi có explicit approval.
- Không activate workflow nếu chưa đọc file RULES.md trong session đó.

### Credential
- Không modify credentials tùy tiện.
- Không rotate `N8N_ENCRYPTION_KEY` nếu chưa có full credential recreation plan.
- Không lưu secrets, tokens, cookies, encryption keys vào bất kỳ file nào trong vault.

### Polling & Runtime
- Không rewrite polling architecture nếu chưa review.
- Không reset staticData/Telegram offset tùy tiện.
- Một Telegram queue chỉ được có đúng một active polling consumer.
- Không để hardcoded Telegram offset values trong active production workflows.
- Polling continuity phải verify bằng real runtime activation, không chỉ Execute Step.

### Data
- Không ghi đè cột Nhóm B trong Google Sheet (business data protected).
- Không tin silent success — luôn verify output data thực tế.

## 2. Quy tắc vận hành — Cách AI làm việc trong Project này

### Thứ tự ưu tiên
1. Continuity before optimization.
2. Observability before patching.
3. TEST workflow before production workflow.
4. Canonical schemas before downstream KPI logic.

### Khi thay đổi hệ thống
- Test node-by-node trong recovery để isolate lỗi.
- Execute Step để isolate, sau đó real runtime activation để verify.
- Khi fix một workflow, kiểm tra ngay các workflow tương tự có cùng pattern.

### Ghi nhớ và cập nhật
- Không lưu raw conversations — chỉ lưu distilled operational knowledge.
- Không invent incidents, decisions, hoặc deployment details.
- Uncertain information phải mark là `GIẢ THUYẾT` hoặc `TODO verification`.
- Ưu tiên cập nhật file cũ — chỉ tạo file mới khi thông tin sẽ xuất hiện thường xuyên.

### Sau mỗi nhiệm vụ quan trọng
Tự hỏi: "AI mới có phải điều tra lại việc này không?"
- Nếu CÓ → cập nhật file phù hợp (STATUS, DECISIONS, RULES, hoặc ROADMAP).
- Mức ưu tiên: CAO (production state, credential, data source) > TRUNG BÌNH (architecture decisions) > THẤP (giả thuyết bị bác bỏ).

## 3. Constraints — Giới hạn kỹ thuật

### Schema
- snake_case ASCII cho tất cả operational field names.
- Dates: ISO `YYYY-MM-DD`. Timezone: `Asia/Ho_Chi_Minh`.
- Normalize external schemas ngay sau ingestion — downstream dùng canonical format.

### Workflow scheduling
- `Meta Ads Daily Sheet Update` (07:30) phải chạy TRƯỚC `Yesterday Report` (08:13).
- Khoảng cách tối thiểu giữa Sync và Report: 30 phút.
- Scheduled workflow dùng `$env.TELEGRAM_CHAT_ID`, không dùng `$json.chat_id`.

### Meta API
- Luôn dùng exact match `===` cho action_type — không dùng `includes()` hay `startsWith()`.
- `numDays = 7` cho Daily Sync (attribution window Meta chủ yếu trong 7 ngày).
- Today Report gọi Meta API trực tiếp, không qua Google Sheet.

### Google Sheet
- Workflow chỉ ghi vào Nhóm A (cột Meta owns).
- Dùng cột `Key` để match hàng — tránh tạo trùng.
- Mapping mode: `Map Each Column Manually` (không dùng `Map Automatically`).

### Forensic
- Webhook reference files là FORENSIC ONLY — không modify.
- Project operational memory self-contained trong folder project, không dùng global AI_MEMORY.

## Cấu trúc mở rộng — Ngoại lệ được duyệt

OPS/ — thư mục vận hành được phép tạo theo chuẩn AI OS mở rộng.
Lý do: chứa tài liệu vận hành đặc thù không thuộc cấu trúc AI OS chuẩn
nhưng AI cần đọc khi tra cứu thông tin vận hành.
Chứa:
- CREDENTIAL_INVENTORY.md — credential và workflow mapping
- ENVIRONMENT_REGISTRY.md — biến môi trường đang dùng
- MIGRATION_RUNBOOK.md — quy trình deploy và recovery
- PRODUCTION_PROMOTION_CHECKLIST.md — checklist GO/NO-GO
- STAGING_TEST_MATRIX.md — test matrix
- TEST_PLAYBOOK.md — quy trình test

Khi gặp vấn đề liên quan đến credential, môi trường, deploy hoặc test → đọc OPS/ trước khi xử lý.

---

## Quy tắc vận hành AI

### Anti-context-drift
Nếu AI định rewrite hoặc redesign khi chưa hiểu logic cũ → dừng lại, đọc README + RULES + STATUS + CASES liên quan trước, hỏi Human nếu vẫn chưa rõ.

### Memory routing — Ghi thông tin vào đâu
- Trạng thái runtime → Project/STATUS.md
- Credential, môi trường, deploy, test → Project/OPS/
- Bài học đã xác minh → CASES/
- Định hướng dài hạn → Project/ROADMAP.md

### Human final authority
AI chỉ được: cảnh báo, so sánh, ghi nhớ, đề xuất, tóm tắt.
Human quyết định cuối cho: production, credential, ngân sách, chiến lược.

### Source of truth — Thứ tự ưu tiên khi đọc
Project/RULES.md → Project/STATUS.md → CASES/ → Project/OPS/


## Cấu trúc mở rộng — CASES/ cấp Project
CASES/ nằm trong Project/ — ngoại lệ được duyệt.
Lý do: chứa knowledge base toàn Project (42 CASE + 7 PATTERN)
không thuộc riêng Task nào — đặt ở Project/ hợp lý hơn.
AI đọc khi: gặp vấn đề cần tra cứu CASE hoặc PATTERN.
Điều hướng qua: Project/CASES/CASE_INDEX.md

## Quy tắc Git — bắt buộc

### Thứ tự làm việc chuẩn
1. Mở máy lên → git pull TRƯỚC
2. Làm việc
3. git push NGAY sau khi xong
4. Chuyển sang máy khác → git pull TRƯỚC khi làm

### Không bao giờ
- Làm việc trên máy chưa pull
- Để "ahead by N commits" qua đêm
- Push từ 2 máy cùng lúc mà không pull trước

### Sau git pull --rebase
Luôn push ngay — không được quên:
```bash
git pull --rebase
git push
```

### Nếu thấy "ahead by N commits"
→ git push ngay, không làm việc tiếp.
