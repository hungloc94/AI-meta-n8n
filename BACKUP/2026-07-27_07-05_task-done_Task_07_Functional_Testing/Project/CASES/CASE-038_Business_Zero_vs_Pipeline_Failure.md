# CASE-038: Business Zero KPI ≠ Pipeline Failure

## Vấn đề
KPI = 0 trong Telegram report → phản xạ đầu tiên là "pipeline hỏng". Nhưng thực tế campaign date được chọn thật sự có zero valid leads.

## Nguyên nhân
Hai nguyên nhân hoàn toàn khác nhau cho KPI = 0:
1. **Pipeline failure:** data không chảy qua pipeline → output = 0 (lỗi kỹ thuật)
2. **Business reality:** pipeline hoạt động đúng, nhưng campaign date có 0 leads thật (dữ liệu đúng)

Nếu không phân biệt → mất thời gian debug pipeline trong khi không có gì hỏng.

## Cách xử lý
Khi gặp KPI = 0, verify theo thứ tự:
1. Kiểm tra pipeline: từng node có output không? Data có chảy qua toàn bộ chain?
2. Kiểm tra data: Ads Manager cho ngày đó có data không? Sheet có dòng cho ngày đó không?
3. Nếu pipeline OK + Ads Manager cũng = 0 → đây là business reality, không phải lỗi.

## Bài học
- KPI = 0 có thể hoàn toàn đúng — phải chứng minh pipeline broken bằng evidence khác.
- Silent data corruption nguy hiểm hơn KPI = 0 — vì KPI = 0 bị nghi ngay, nhưng KPI = 18 (inflate) trông "bình thường".
- End-to-end verification là bắt buộc trước khi declare recovery success — không chỉ nhìn output cuối.

## Status
Lesson learned — 2026-05-27.
