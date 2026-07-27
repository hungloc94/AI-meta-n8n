# CASE Index — n8n Meta Ads

> Danh sách toàn bộ CASE. Mỗi CASE ghi lại một quyết định, sự cố, hoặc bài học đã xác minh.

## Decisions (CASE-001 → CASE-019)

| CASE | File | Mô tả |
|------|------|-------|
| 001 | CASE-001_Polling_Thay_Vi_Webhook.md | Chọn polling Telegram thay vì webhook cho bot reporting |
| 002 | CASE-002_Normalize_Layer_Sau_Google_Sheet.md | Normalize raw array rows thành stable object schema |
| 003 | CASE-003_Snake_Case_ASCII_Schema.md | Dùng snake_case ASCII tránh key mismatch với Vietnamese headers |
| 004 | CASE-004_Workflow_Inactive_Khi_Testing.md | Workflow phải inactive khi test — tránh trigger polling/duplicate |
| 005 | CASE-005_ISO_Date_Format.md | Convert tất cả dates sang ISO YYYY-MM-DD |
| 006 | CASE-006_TEST_Workflow_Truoc_Production.md | Bắt buộc TEST workflow trước khi patch production |
| 007 | CASE-007_Normalize_Ngay_Sau_Ingestion.md | Normalize ngay sau khi nhận data từ external source |
| 008 | CASE-008_Validate_Polling_Real_Runtime.md | Execute Step không đủ — phải validate dưới real runtime |
| 009 | CASE-009_Memory_Project_Scoped.md | Project memory phải nằm trong folder project, không dùng global |
| 010 | CASE-010_Google_Sheet_La_Source_Of_Truth.md | Google Sheet là nguồn sự thật duy nhất cho KPI reporting |
| 011 | CASE-011_Protected_Columns_Nhom_B.md | Nhóm B (business data) không được workflow ghi đè |
| 012 | CASE-012_Kien_Truc_Yesterday_Va_Today.md | Tách riêng workflow Yesterday và Today — tránh lỗi im lặng |
| 013 | CASE-013_Scheduled_Dung_Env_Chat_ID.md | Scheduled workflow dùng $env.TELEGRAM_CHAT_ID, không dùng $json |
| 014 | CASE-014_Thu_Tu_Activate_Dependency.md | Sync phải chạy trước Report — đảm bảo dependency |
| 015 | CASE-015_Exact_Match_Action_Type.md | Exact match === cho action_type — fuzzy inflate Mess_Comment 18x |
| 016 | CASE-016_NumDays_7_Daily_Sync.md | numDays=7 cho Daily Sync — giảm API calls không cần thiết |
| 017 | CASE-017_Today_Report_Truc_Tiep_Meta_API.md | Today Report gọi Meta API trực tiếp, không qua Sheet |
| 018 | CASE-018_CPC_Metric_Bat_Buoc.md | CPC là metric bắt buộc trong báo cáo Telegram |
| 019 | CASE-019_Sheet_Used_Range_Cleanup.md | Cleanup used range khi Ctrl+End nhảy quá xa |

## Incidents (CASE-020 → CASE-026)

| CASE | File | Mô tả |
|------|------|-------|
| 020 | CASE-020_KPI_Reports_Returning_Zero.md | KPI = 0 toàn bộ do pipeline failure |
| 021 | CASE-021_NODE_UNKNOWN_Routing_Spam.md | Double consumer gây duplicate reports + [NODE=UNKNOWN] spam |
| 022 | CASE-022_Google_OAuth_Silent_Expiry.md | OAuth expire im lặng — UI vẫn hiện "connected" |
| 023 | CASE-023_ECONNRESET_Telegram_Send.md | ECONNRESET làm fail Send Report tại 1/3 slot |
| 024 | CASE-024_Duplicate_Key_Trong_Sheet.md | Key trùng trong Sheet — dòng cũ KPI sai, dòng mới đúng |
| 025 | CASE-025_Migration_Verification_Gap.md | Bỏ sót node khi migrate — Health Check fail sáng hôm sau |
| 026 | CASE-026_Meta_API_403_Bearer_Prefix.md | Token mới nhưng thiếu prefix "Bearer " → 403 |

## Lessons (CASE-027 → CASE-038)

| CASE | File | Mô tả |
|------|------|-------|
| 027 | CASE-027_Credential_Display_Name_Khong_Dang_Tin.md | Tên hiện "Service Account" nhưng thực tế vẫn dùng OAuth2 |
| 028 | CASE-028_Doi_Credential_Reset_Mapping.md | Đổi credential có thể reset mapping config của node |
| 029 | CASE-029_Verify_Update_Row_Dung_Chuan.md | Execute OK không đảm bảo data thực sự được ghi |
| 030 | CASE-030_Node_PASS_Khong_Dam_Bao_Workflow_PASS.md | Node PASS riêng lẻ không đảm bảo full workflow PASS |
| 031 | CASE-031_Hai_Nguon_Meta_Token.md | Workflow dùng 2 nguồn Meta token — đổi 1 chỗ vẫn lỗi |
| 032 | CASE-032_Migration_Definition_of_Done.md | Migration cần 6 bước bắt buộc — không đánh dấu xong sớm |
| 033 | CASE-033_SafeCount_Branch_Workflow.md | safeCount helper tránh throw error khi node chưa execute |
| 034 | CASE-034_Env_Var_UI_Error_Khong_Phai_Runtime.md | $env lỗi trên UI editor nhưng runtime vẫn chạy đúng |
| 035 | CASE-035_Mojibake_Workflow_JSON_Tieng_Viet.md | Export → edit → import workflow JSON gây mojibake tiếng Việt |
| 036 | CASE-036_Layer_by_Layer_Verification.md | Verify từng layer riêng — không tích hợp nhiều layer cùng lúc |
| 037 | CASE-037_Schema_Change_Verify_Toan_Bo.md | Thêm/đổi cột Sheet → verify toàn bộ chuỗi đọc dữ liệu |
| 038 | CASE-038_Business_Zero_vs_Pipeline_Failure.md | KPI = 0 có thể là business reality, không phải pipeline hỏng |

## Patterns (CASE-039 → CASE-046)

| CASE | File | Mô tả |
|------|------|-------|
| 039 | CASE-039_Scheduled_Health_Check_Truoc_Workflow_Phu_Thuoc.md | Health check phải chạy trước workflow phụ thuộc |
| 040 | CASE-040_Workflow_Artifact_Active_Khong_Bang_Runtime_Active.md | Artifact active khác runtime active |
| 041 | CASE-041_Legacy_AutoMapInputData_Risk.md | Node Sheets dùng autoMapInputData legacy — rủi ro ghi sai/ghi thừa cột |
| 042 | CASE-042_Health_Check_Fail_Path_Alert_StopAndError.md | Health check fail phải stopAndError — không để fail im lặng |
| 043 | CASE-043_Google_Service_Account_Scope_Required.md | Google SA scope bắt buộc khai báo |
| 044 | CASE-044_Mojibake_Node_Code_Sau_Import_Windows_Ubuntu.md | Mojibake node Code sau import workflow Windows → Ubuntu |
| 045 | CASE-045_Credential_Unavailable_Fix_Via_API.md | Credential unavailable sau import — fix qua API |
| 046 | CASE-046_AI_Must_Confirm_Workflow_Name_Before_Fix.md | AI phải confirm tên workflow trước khi fix |

## Module 00 — Migrate Home Server (CASE-047 → CASE-050)

| CASE | File | Mô tả |
|------|------|-------|
| 047 | CASE-047_Token_Copy_Nham_Bi_Tuong_Token_Hong.md | Token copy sai khi restore bị nhầm tưởng token hỏng ở Meta — phải đối chiếu production trước khi kết luận |
| 048 | CASE-048_SafeCount_Sai_Message_Loi_Theo_Version_n8n.md | safeCount() không khớp message lỗi thực tế của n8n 1.103.2 — workflow báo FAILED dù ghi Sheet đúng |
| 049 | CASE-049_Docker_Compose_Restart_Khong_Nap_Lai_Env.md | docker compose restart không nạp lại .env — phải dùng up -d --force-recreate |
| 050 | CASE-050_Mojibake_Dau_D_Trong_Workflow_TEST_Date_Range_Engine.md | Mojibake ký tự "đ" trong workflow TEST Date Range Engine — chưa fix, cần xử lý trước khi Module 01 dùng |
