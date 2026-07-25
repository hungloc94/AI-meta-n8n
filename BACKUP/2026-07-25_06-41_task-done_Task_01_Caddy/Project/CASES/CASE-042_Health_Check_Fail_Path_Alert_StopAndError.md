# CASE-042: Health Check Fail Path Alert + Stop And Error

## Ngày phát hiện
2026-07-22

## Vấn đề
Health check chỉ có giá trị nếu failure được nhìn thấy rõ. Nếu workflow health check fail im lặng, hoặc chỉ gửi alert mà execution vẫn PASS, AI/Human có thể bỏ sót lỗi credential/API trước khi workflow production phụ thuộc chạy.

## Nguyên nhân
Một số workflow kiểm tra kết nối chỉ dừng ở request test hoặc gửi thông báo nhưng không đánh dấu execution là failed. Khi đó execution history không phản ánh đúng tình trạng hệ thống, làm observability bị sai.

## Cách xử lý
Fail path chuẩn cho health check:
```text
Schedule
→ Runtime Check
→ End nếu PASS
→ Telegram Alert nếu FAIL
→ Stop And Error
```

Yêu cầu:
- Alert phải nói rõ hệ thống nào fail và workflow nào có nguy cơ bị ảnh hưởng.
- Sau alert phải đi vào `Stop And Error`.
- Execution fail là tín hiệu vận hành bắt buộc, không chỉ là lỗi kỹ thuật.
- Khi rebuild health check, test cả PASS path và FAIL path nếu có thể.

## Bài học
- Alert mà không fail execution có thể tạo false confidence.
- Health check phải vừa báo cho Human, vừa để lại dấu vết đỏ trong execution logs.
- Observability tốt cần tín hiệu kép: notification + runtime failure state.
- Pattern này nên dùng cho mọi credential/API health check production.
