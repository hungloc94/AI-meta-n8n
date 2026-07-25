# Plan — Task 04: Design Docker Compose

| Bước | Mô tả | Kết quả mong đợi |
|------|-------|-------------------|
| 1 | Tạo project directory trên Home Server (vd `~/n8n-docker/`) | Thư mục sẵn sàng |
| 2 | Thiết kế docker-compose.yml — image, volume, bind `127.0.0.1:5678:5678`, timezone env | File compose hoàn chỉnh, chưa chạy |
| 3 | Thiết kế cấu trúc .env — liệt kê đủ tên biến theo ENVIRONMENT_REGISTRY.md, không điền giá trị thật | .env template đầy đủ tên biến |
| 4 | Xác nhận N8N_ENCRYPTION_KEY sẽ dùng đúng key từ Task 01 (không generate mới) | Ghi chú rõ nguồn key, tránh rủi ro continuity |
| 5 | Review toàn bộ thiết kế đối chiếu Migration Runbook (New-Machine Deployment Order bước 1-6) | Thiết kế PASS review, sẵn sàng handover Task 05 |
