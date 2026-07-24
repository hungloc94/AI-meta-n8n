# CASE-039: Scheduled Health Check Trước Workflow Phụ Thuộc

## Ngày phát hiện
2026-07-22

## Vấn đề
Các workflow production như Daily Sheet Update, Yesterday Report và Today Report phụ thuộc vào credential/API bên ngoài. Nếu Google Sheets Service Account hoặc Meta API token lỗi, workflow chính có thể fail tại runtime và chỉ phát hiện sau khi lịch chạy quan trọng đã bị ảnh hưởng.

## Nguyên nhân
Credential/API health không thể tin bằng trạng thái UI. n8n UI có thể hiển thị credential còn kết nối, nhưng runtime request thật vẫn fail do token hết hạn, permission thay đổi, service account mất quyền, hoặc API bên ngoài lỗi tạm thời.

## Cách xử lý
Tạo các scheduled health check chạy trước workflow phụ thuộc:
- Google Sheets Health Check chạy 07:00, trước Daily Sheet Update 07:30 và Yesterday Report 08:13.
- Meta API Health Check chạy 07:05, trước Daily Sheet Update 07:30 và Today Report 11:31/16:31/21:13.
- Health check phải gọi runtime API thật.
- Nếu fail thì gửi Telegram Alert và dừng bằng Stop And Error.

## Bài học
- Observability phải đi trước workflow chính.
- Credential/API chỉ được coi là OK khi runtime request thật PASS.
- Health check nên chạy đủ sớm để Human có thời gian xử lý trước lịch production.
- Khi rebuild workflow phụ thuộc credential/API, luôn rebuild cả health check tương ứng.

## Chi tiết fail path chuẩn
- Runtime check FAIL
  → Gửi Telegram Alert nói rõ workflow nào fail
  → Stop And Error để execution hiển thị đỏ rõ ràng
- Không để execution kết thúc PASS khi thực tế FAIL
  → false confidence nguy hiểm hơn không có alert