# Decisions

## Decision: Use Polling Instead Of Webhook
WHY:
Polling dễ test local hơn và tránh public exposure trong recovery.

TRADEOFF:
Polling yêu cầu staticData/offset persistence rất cẩn thận để tránh duplicate processing.

STATUS:
Accepted for current MSP recovery/testing phase.

## Decision: Place Normalize Layer Immediately After Google Sheet
WHY:
Google Sheets API trả raw array rows. Downstream KPI logic cần stable object schema.

TRADEOFF:
Thêm một node, nhưng ngăn schema drift lan xuống downstream.

STATUS:
Accepted and implemented as `Normalize Sheet Data`.

## Decision: Adopt snake_case ASCII Schema
WHY:
ASCII snake_case tránh accented key mismatch, typing errors, và expression ambiguity.

TRADEOFF:
Cần one-time migration từ Vietnamese display headers sang canonical internal field names.

STATUS:
Accepted.

## Decision: Keep Workflow Inactive During Testing
WHY:
Tránh unintended Telegram polling, duplicate messages, và production side effects.

TRADEOFF:
Cần manual node-by-node testing và supervised runtime activation riêng cho polling continuity.

STATUS:
Mandatory during forensic recovery.

## Decision: Convert All Dates Into ISO Format
WHY:
ISO dates stable, sortable, và dễ compare hơn DD/MM strings.

TRADEOFF:
Parser phải handle local input formats và year assumptions.

STATUS:
Accepted.

## Decision: TEST Workflow Required Before Production Changes
WHY:
Hệ thống từng có runtime propagation uncertainty và duplicate-processing risk.

TRADEOFF:
Thêm import/testing step trước production activation.

STATUS:
Mandatory.

## Decision: Normalize Downstream Operational Processing Into Canonical Schema Objects Immediately After External Ingestion
WHY:
Array-based spreadsheet structures gây KPI corruption risk, schema drift, và silent aggregation failures.

TRADEOFF:
Thêm một normalization node, nhưng giảm mạnh downstream reliability risk và forensic debugging time.

STATUS:
ACCEPTED AND VERIFIED IN END-TO-END RECOVERY.

## Decision: Validate Polling Systems Under Real Runtime Activation

WHY:
Execute Step testing không validate đủ polling continuity vì staticData persistence behave khác trong real workflow activation.

TRADEOFF:
Cần supervised runtime activation và tighter operational controls, nhưng evidence tốt hơn để chống stale replay, duplicate processing, và offset drift.

STATUS:
ACCEPTED AND VERIFIED FOR POLLING CONTINUITY.

## Decision: Keep Project Memory Project-Scoped

WHY:
Operational memory cho project này phải self-contained trong Obsidian project folder để giữ context, authority, và handoff clarity.

TRADEOFF:
Mọi agents phải dùng `D:\AI_Project\Obdisian_2\My Vault\Project\AI_Meta_n8n_autoamation` thay vì global `AI_MEMORY` folder.

STATUS:
ACCEPTED.

---

## Decision: Google Sheet Là Nguồn Sự Thật Duy Nhất

WHY:
Google Sheet chứa cả dữ liệu từ Meta Ads API lẫn dữ liệu kinh doanh do người vận hành nhập tay (chất lượng khách, ghi chú, công thức tính chi phí). Nếu workflow ghi đè toàn bộ hàng, dữ liệu kinh doanh sẽ bị xóa mà không có cảnh báo.

Vì vậy: **Workflow phải thích nghi với cấu trúc Sheet — không phải ngược lại.**

TRADEOFF:
Workflow cần logic selective update (chỉ ghi vào cột thuộc về Meta API), phức tạp hơn ghi đè toàn bộ hàng.

STATUS:
ACCEPTED — 2026-05-29. Đã document trong `DATA_SCHEMA_RULES.md`.

---

## Decision: Protected Columns — Chỉ Nhóm B Không Được Meta Sync Ghi Đè

WHY:
Có hai nhóm cột trong Sheet:

**Nhóm A — Meta được phép ghi:**
`Ma_quang_cao`, `Ngay`, `Chien_dich`, `Ten_quang_cao`, `Chi_tieu`, `Nguoi_tiep_can`, `Ngan_sach`, `click`, `Mess_Comment`, `Trang_Thai`, `Key`, `Thoi_diem_cap_nhat`

**Nhóm B — PROTECTED, không được ghi:**
`Khach_sai_tep`, `Khach_hop_le`, `SDT`, `Khach_chot`, `Chi_phi_tren_khach_hop_le`, `Chi_phi_khach_tren_SDT`, `Chi_phi_khach_chot`, `ghi_chu`

Nhóm B chứa dữ liệu bán hàng, đánh giá chất lượng lead, và công thức Sheet do người vận hành quản lý. Ghi đè nhóm này = mất dữ liệu kinh doanh vĩnh viễn.

Lưu ý cập nhật (2026-06-02): `Chien_dich`, `Ten_quang_cao`, `Mess_Comment` đã được xác nhận là Meta có thể ghi — chuyển từ Nhóm B sang Nhóm A dựa trên thực tế vận hành.

RULE:
- Mọi workflow sync Meta chỉ được cập nhật **Nhóm A**.
- Không được reset, ghi đè, hoặc xóa giá trị Nhóm B trừ khi có **phê duyệt rõ ràng từ người vận hành**.
- Bất kỳ thay đổi nào ảnh hưởng Nhóm B đều cần backup Sheet trước.

STATUS:
ACCEPTED — cập nhật phân loại cột 2026-06-02. Chi tiết trong `DATA_SCHEMA_RULES.md`.

---

## Decision: Kiến Trúc Sync Meta — Chọn Option B (Workflow Yesterday + Workflow Today)

WHY:
Có hai loại nhu cầu báo cáo khác nhau:
1. **Báo cáo ngày hôm qua** — dữ liệu đã ổn định, đã có trong Sheet → đọc từ Sheet là đủ và an toàn.
2. **Báo cáo ngày hôm nay** — dữ liệu chưa có trong Sheet (chưa sync) → phải gọi trực tiếp Meta API để lấy số realtime.

Nếu dùng chung một workflow, logic sẽ phức tạp và dễ gây lỗi im lặng khi dữ liệu hôm nay chưa vào Sheet.

TRADEOFF:
Cần xây và maintain hai workflow riêng, nhưng mỗi workflow có trách nhiệm rõ ràng và dễ debug hơn.

OPTION B — Kiến trúc đã chọn:
- **Workflow Yesterday**: đọc data từ Google Sheet → normalize → KPI aggregation → Telegram. Dùng cho `báo cáo hôm qua`, `báo cáo 24/03`, range reports.
- **Workflow Today**: gọi Meta Ads API trực tiếp → KPI aggregation → Telegram. Dùng cho `báo cáo hôm nay`.

STATUS:
ACCEPTED — đã implement 2026-06-07. Workflow Yesterday test PASS 2026-06-08. Workflow Today chưa test.

---

## Decision: Scheduled Workflow Dùng $env.TELEGRAM_CHAT_ID, Không Dùng $json.chat_id

WHY:
Scheduled (cron) workflow không có user trigger. Không có Telegram message context → `$json.chat_id` không tồn tại. Nếu dùng `$json.chat_id` trong Send Report node, workflow sẽ fail silently hoặc gửi sai chat.

RULE:
- Bot-triggered workflow (user nhắn tin) → dùng `$json.chat_id` từ message context.
- Scheduled workflow (cron) → bắt buộc dùng `$env.TELEGRAM_CHAT_ID`.
- `TELEGRAM_CHAT_ID` phải có trong `.env` container. Giá trị: `5977057100`.

TRADEOFF:
Hardcode chat_id trong env — nếu muốn gửi nhiều chat thì phải thêm logic.

STATUS:
ACCEPTED — xác minh 2026-06-08.

---

## Decision: Thứ Tự Activate Scheduled Workflows Phải Đảm Bảo Dependency

WHY:
`Meta Report Yesterday` đọc dữ liệu từ Sheet. Nếu `Meta Ads Daily Sheet Update` chưa chạy thì Sheet chưa có dữ liệu mới nhất → báo cáo sẽ thiếu hoặc sai.

RULE:
- `Meta Ads Daily Sheet Update` (07:30) phải chạy TRƯỚC.
- `Meta Report Yesterday Scheduled` (08:13) chạy SAU — đủ thời gian để Sheet cập nhật xong.
- Khoảng cách tối thiểu khuyến nghị: 30 phút.
- Không activate Report trước Sync.

TRADEOFF:
Nếu Sync chạy quá chậm hoặc lỗi, Report 08:13 vẫn chạy nhưng dữ liệu có thể thiếu. Cần monitor cả hai workflow.

STATUS:
ACCEPTED — xác minh thứ tự vận hành 2026-06-09.

---

## Decision: Dùng Exact Match Cho action_type Meta API

WHY:
Fuzzy matching (`includes('message')`) vô tình gom nhiều action_type không mong muốn từ Meta API, gây KPI Mess_Comment bị inflate nghiêm trọng (ví dụ: thực tế = 1, hệ thống = 18). Ads Manager định nghĩa "Lượt bắt đầu cuộc trò chuyện" = `onsite_conversion.messaging_conversation_started_7d` — không có action_type nào khác.

RULE:
- Luôn dùng exact match `===` cho action_type, không dùng `includes()` hay `startsWith()`.
- Trước khi thêm metric mới, kiểm tra Ads Manager để xác định chính xác tên metric và action_type tương ứng.

TRADEOFF:
Exact match cứng hơn, nhưng đảm bảo data chính xác. Nếu Meta đổi tên action_type sẽ cần update code — nhưng lỗi đó sẽ lộ ra ngay (= 0) thay vì bị inflate im lặng.

STATUS:
ACCEPTED — 2026-06-11. Đã patch toàn bộ workflow và backfill lịch sử.

---

## Decision: numDays = 7 Cho Meta Daily Sync

WHY:
Meta điều chỉnh dữ liệu chậm (attribution window) chủ yếu trong vòng 7 ngày. Dùng numDays=30 tạo API calls không cần thiết, tăng thời gian chạy workflow, và không mang lại lợi ích thực tế.

RULE:
- `numDays = 7` trong workflow Meta Ads Daily Sheet Update.
- Nếu cần backfill dài hơn → dùng workflow backfill riêng, không thay đổi numDays production.

TRADEOFF:
Nếu Meta có điều chỉnh attribution dài hơn 7 ngày trong trường hợp đặc biệt, sẽ bỏ sót. Accepted trade-off vì case này rất hiếm.

STATUS:
ACCEPTED — 2026-06-11. Đã verify production thành công.

---

## Decision: Today Report Lấy Trực Tiếp Từ Meta API, Không Qua Google Sheet

WHY:
Today Report cần dữ liệu realtime của ngày hiện tại — dữ liệu này chưa có trong Sheet (Sheet chỉ được sync lúc 07:30). Nếu đọc Sheet sẽ lấy dữ liệu ngày hôm qua hoặc không có gì. Ngoài ra, tách nguồn dữ liệu ra khỏi Google OAuth giúp Today Report không bị ảnh hưởng khi OAuth expire.

RULE:
- `Meta Report Today Scheduled` gọi Meta Marketing API trực tiếp.
- Không đọc Google Sheet.
- Không phụ thuộc Google Sheets credential.

TRADEOFF:
Cần quản lý Meta API token riêng. Nếu Meta API thay đổi response format, Today Report sẽ bị ảnh hưởng độc lập với Yesterday Report.

STATUS:
ACCEPTED — deploy và verify production 2026-06-11.

---

## Decision: CPC Là Metric Bắt Buộc Trong Báo Cáo Telegram

WHY:
CPC (Cost Per Click = Chi tiêu / Clicks) cho biết hiệu quả chi phí quảng cáo theo lượt click. Metric này cần thiết để đánh giá nhanh campaign mà không cần mở Ads Manager.

RULE:
- CPC = `totalSpend / totalClick`.
- Nếu `totalClick = 0` thì CPC = 0 (bảo vệ divide-by-zero).
- Hiển thị trong tất cả báo cáo Telegram (Today Report và Yesterday Report khi có dữ liệu).

STATUS:
ACCEPTED — implemented và verify 2026-06-11.

---

## Future Cleanup: Google Sheet Used Range

QUAN SÁT:
- Ctrl+End nhảy tới dòng 1011 dù dữ liệu thực tế kết thúc sớm hơn.
- Google Sheet giữ "last used row" lịch sử kể cả khi data đã xóa.

TÁC ĐỘNG:
- Không ảnh hưởng tính đúng đắn dữ liệu.
- Không gây duplicate.
- Không ảnh hưởng workflow.

CÁCH XỬ LÝ KHI CẦN:
- Delete các rows cuối dư thừa (không dùng Clear — Clear không reset used range).
- Verify Ctrl+End sau khi dọn.

STATUS:
DEFERRED / LOW PRIORITY — 2026-06-11.
