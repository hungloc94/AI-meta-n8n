# CASE-030: Node PASS Không Đảm Bảo Full Workflow PASS

## Vấn đề
Từng node Google Sheets PASS với Service Account. Nhưng full workflow fail vì Meta token hết hạn ở node khác. Mất nhiều vòng execute riêng lẻ node trong khi chạy full workflow sẽ phát hiện ngay.

## Nguyên nhân
Node-by-node testing không phát hiện:
- Cross-node failures (node A pass, node B fail)
- Lỗi mapping/field propagation giữa các node
- Data test không phản ánh luồng thực

## Cách xử lý

### Quy trình debug chuẩn (Auth First):
1. Verify credential/access token hợp lệ
2. Verify permission đúng (scope, share, format)
3. **Chạy toàn bộ workflow TEST** — n8n sẽ dừng tại node đỏ đầu tiên
4. Chỉ execute riêng node khi cần forensic chuyên sâu

### So sánh:
| Phương pháp | Phát hiện node lỗi | Kiểm tra luồng thực | Phát hiện lỗi propagation | Tốc độ |
|-------------|-------------------|--------------------|-----------------------------------------|--------|
| Execute từng node | ✅ | ❌ | ❌ | Chậm |
| Full workflow TEST | ✅ | ✅ | ✅ | Nhanh |

## Bài học
- Node PASS riêng lẻ ≠ workflow PASS — cross-node failures chỉ lộ khi chạy toàn bộ.
- Sau khi auth/permission đã verify → dừng execute riêng, chạy full workflow.
- Sai lầm điển hình: kết luận "node pass" từ Execute Step rồi ngạc nhiên khi full workflow fail.

## Status
Lesson learned — 2026-06-22.
