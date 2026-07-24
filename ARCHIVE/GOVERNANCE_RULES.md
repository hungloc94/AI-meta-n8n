# Governance Rules

## Role
Các operational safety rules không được override.

## Production Safety
- Không patch production trực tiếp.
- Luôn export backup trước khi patch.
- Luôn dùng TEST workflow trước.
- Imported workflow phải giữ `active=false` trong forensic testing.
- Không activate workflow trong recovery nếu chưa có explicit approval.

## Forensic Safety
- Webhook reference files là FORENSIC ONLY.
- Không modify `WEBHOOK_CLOUDFLARE_REFERENCE_20260508.json` nếu chưa được yêu cầu rõ.
- Không tin silent success nếu chưa verify output data.
- Luôn test node-by-node trong recovery để isolate lỗi.
- Execute Step testing được dùng cho forensic isolation.
- Polling continuity phải verify bằng real runtime activation, không chỉ Execute Step.

## Credential Safety
- Không modify credentials tùy tiện.
- Không rotate `N8N_ENCRYPTION_KEY` nếu chưa có full credential recreation plan.
- n8n không auto-migrate encrypted credentials sau khi đổi key.
- Nếu encryption key đổi, existing credentials có thể undecryptable.
- Không lưu secrets, tokens, cookies, encryption keys, hoặc sensitive values vào memory files.

## Polling and Runtime Safety
- Không rewrite polling architecture nếu chưa review.
- Không reset staticData/offset state tùy tiện.
- Polling systems phải preserve update continuity.
- Phải avoid duplicate-processing risk trước activation.
- Queue/polling systems phải được validate dưới real runtime activation, không chỉ manual execution.
- Không bao giờ để hardcoded Telegram offset values trong active production workflows sau testing.

## Schema Safety
- Luôn inspect downstream dependencies trước schema changes.
- Normalize external schemas ngay sau ingestion.
- Downstream systems phải dùng canonical schema formats.
- Ưu tiên snake_case ASCII schema names cho operational data.
- Convert human-friendly dates sang canonical ISO dates một lần tại parser boundary.

## Documentation Safety
- Không lưu raw conversations.
- Không lưu repetitive debug noise.
- Không invent incidents, decisions, hoặc deployment details.
- Uncertain information phải mark là assumption hoặc TODO verification.
- Project operational memory là project-scoped và self-contained trong `D:\AI_Project\Obdisian_2\My Vault\Project\AI_Meta_n8n_autoamation`.
- Không dùng `D:\AI_Project\Obdisian_2\My Vault\AI_MEMORY` cho project này.

## Cập Nhật Bộ Nhớ

Cuối mỗi nhiệm vụ, tự hỏi: **"AI mới có phải điều tra lại việc này không?"**

Nếu CÓ → đề xuất cập nhật file hiện có (không tạo file mới trừ khi thực sự cần).

**Mức ưu tiên cập nhật:**
- CAO: trạng thái production, credential, nguồn dữ liệu chính, protected columns
- TRUNG BÌNH: quyết định kiến trúc, lessons learned
- THẤP: giả thuyết đã bị bác bỏ

**Phân loại thông tin khi ghi:**
- `ĐÃ XÁC MINH` — có bằng chứng runtime hoặc test thực tế
- `GIẢ THUYẾT` — chưa verify, cần xác nhận
- `QUYẾT ĐỊNH` — đã chốt, có lý do rõ ràng
- `KHUYẾN NGHỊ` — gợi ý, chưa phải quyết định cuối
- `ĐÃ LỖI THỜI` — không còn áp dụng, giữ để forensic reference

**Quy tắc tạo file:**
- Ưu tiên cập nhật file cũ.
- Chỉ đề xuất tạo file mới khi phải sửa từ 3 file trở lên VÀ loại thông tin đó sẽ xuất hiện thường xuyên.
- Không sao chép cùng một nội dung đầy đủ vào nhiều file — dùng tham chiếu thay thế.

**Phân công file:**
- Trạng thái hệ thống hiện tại → `CURRENT_STATUS.md`
- Quyết định kiến trúc + lý do → `DECISIONS.md`
- Quy tắc vận hành → `GOVERNANCE_RULES.md`
