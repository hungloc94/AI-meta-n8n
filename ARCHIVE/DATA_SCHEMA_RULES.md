# Data Schema Rules

## Mục Đích

File này định nghĩa quy tắc sở hữu và bảo vệ dữ liệu trong Google Sheet, dùng bởi các workflow n8n Meta Ads.

**Mục tiêu:** Ngăn workflow ghi đè dữ liệu kinh doanh chỉ vì tiện cho implementation.

Google Sheet chứa hai loại dữ liệu:
- Dữ liệu từ Meta Ads API (workflow có thể cập nhật)
- Dữ liệu kinh doanh do người vận hành nhập tay / công thức Sheet (workflow KHÔNG được chạm vào)

## Google Sheet Data Ownership Rules: 2026-05-29

Authoritative Sheet header order:

```text
Ma_quang_cao, Ngay, Chien_dich, Ten_quang_cao, Chi_tieu,
Nguoi_tiep_can, Ngan_sach, click, Mess_Comment, Khach_sai_tep,
Khach_hop_le, SDT, Khach_chot, Chi_phi_tren_khach_hop_le,
Chi_phi_khach_tren_SDT, Doanh_thu, ghi_chu,
Trang_Thai, Key, Thoi_diem_cap_nhat
```

### Quy Tắc 1: Google Sheet Là Nguồn Sự Thật

Google Sheet là nguồn dữ liệu chính thức cho toàn bộ hệ thống báo cáo.

**Workflow phải thích nghi với cấu trúc Sheet hiện tại.**
Không đổi tên cột, không di chuyển cột, không viết lại cấu trúc Sheet chỉ để workflow dễ implement hơn.

Lý do: Sheet còn chứa công thức và dữ liệu kinh doanh không thể tái tạo nếu bị xóa.

### Nhóm A: Cột Do Meta Ads API Quản Lý

Các cột này có thể được cập nhật bởi workflow sync từ Meta API:

- Ma_quang_cao
- Ngay
- Chi_tieu
- Nguoi_tiep_can
- Ngan_sach
- click
- Trang_Thai
- Key
- Thoi_diem_cap_nhat

### Nhóm B: Cột Kinh Doanh / Công Thức / Nhập Tay — PROTECTED

Các cột này thuộc về người vận hành hoặc do công thức Sheet tính toán. Workflow Meta Ads **KHÔNG phải** nguồn sự thật cho các cột này và **KHÔNG được** ghi đè hoặc reset trừ khi có phê duyệt rõ ràng:

- Chien_dich
- Ten_quang_cao
- Mess_Comment
- Khach_sai_tep
- Khach_hop_le
- SDT
- Khach_chot
- Chi_phi_tren_khach_hop_le
- Chi_phi_khach_tren_SDT
- Doanh_thu
- ghi_chu

### Ghi Chú Từng Cột Nhóm B

- `Chien_dich` và `Ten_quang_cao`: Dùng để tra cứu và hiển thị trong báo cáo. Được coi là protected trừ khi có review riêng.
- `Mess_Comment`: Dữ liệu tương tác thực tế — protected.
- `Khach_sai_tep`, `Khach_hop_le`, `SDT`, `Khach_chot`: Dữ liệu chất lượng lead và bán hàng, nhập tay sau khi lead xảy ra. Không được workflow tự động ghi đè.
- `Chi_phi_tren_khach_hop_le`, `Chi_phi_khach_tren_SDT`: Công thức trong Sheet, không phải dữ liệu Meta API.
- `Doanh_thu`: Tổng doanh thu thực tế thu được từ khách hàng trong kỳ báo cáo — nhập tay, không phải công thức và không lấy từ Meta API.
- `ghi_chu`: Ghi chú kinh doanh nhập tay.

### Quy Tắc Thiết Kế Workflow

Mọi workflow sync Meta trong tương lai phải dùng **selective update** (chỉ cập nhật những gì mình sở hữu):

- Chỉ cập nhật các cột **Nhóm A** theo mặc định.
- Giữ nguyên giá trị **Nhóm B** từ các hàng đã có.
- Không bao giờ reset cột công thức / nhập tay / chất lượng khách trong routine sync.
- Bất kỳ workflow nào chạm vào Nhóm B đều **phải có phê duyệt từ người vận hành** và **backup Sheet trước**.

### Trạng Thái Implementation: 2026-06-02

- Quy tắc này đã được xác lập và document.
- Workflow hiện tại (`Meta Report VERIFIED`) chỉ **đọc** Sheet — không ghi — nên không vi phạm quy tắc này.
- Các workflow "Meta Ads Daily Sheet Update" (đang inactive trong n8n) **chưa được review** theo quy tắc này trước khi activate.
- **Bắt buộc review** `DATA_SCHEMA_RULES.md` trước khi activate bất kỳ workflow nào có ghi vào Sheet.

---

## Cập nhật Schema 2026-06-30 — Meta KPI Expansion

### Architecture Decision
Workflow chỉ ghi Raw Metrics (dữ liệu lấy trực tiếp từ Meta API).
Google Sheet tính toàn bộ Derived Metrics bằng công thức.

Lý do:
- Dễ kiểm chứng — công thức nằm trong Sheet, ai cũng đọc được
- Sửa công thức không cần deploy lại workflow
- Giảm độ phức tạp parser

### Nhóm Raw Metrics (Meta API, workflow ghi)
- Luot_hien_thi (impressions)
- Click_All (clicks)
- Tan_suat (frequency)
- Xem_3s (video_view trong actions[])
- Video_25 (video_p25_watched_actions)
- Video_50 (video_p50_watched_actions)
- Video_75 (video_p75_watched_actions)
- Video_95 (video_p95_watched_actions)
- ThruPlay (video_thruplay_watched_actions)
- Click_Ra_Web (outbound_clicks)

### Nhóm Derived Metrics (Google Sheet tự tính, workflow KHÔNG ghi)
- CPM = Chi_tieu / Luot_hien_thi * 1000
- CTR_Link = click / Luot_hien_thi * 100
- CTR_All = Click_All / Luot_hien_thi * 100
- Ty_le_Hook_3s = Xem_3s / Luot_hien_thi * 100
- Ty_le_Giu_Chan_25 = Video_25 / Xem_3s * 100
- Ty_le_Xem_95 = Video_95 / Xem_3s * 100
- Ty_le_Ra_Web = Click_Ra_Web / click * 100

### Nhóm Business Metrics (người dùng nhập/chỉnh sửa)
- Khach_hop_le, SDT, Khach_chot, ghi_chu, Chien_dich, Ten_quang_cao, Doanh_thu

#### Doanh_thu
- Ý nghĩa: Tổng doanh thu thực tế thu được từ khách hàng trong kỳ báo cáo
- Nguồn: Người dùng nhập hoặc cập nhật thủ công trong Google Sheet. Workflow không tự sinh hoặc tự cập nhật giá trị này.
- Không lấy từ Meta API
- Workflow KHÔNG được tự ghi vào cột này
- Dùng để tính: DT/SDT, DT/Khach_chot, ROAS

### Bảng Ownership

| Nhóm | Chủ sở hữu |
|---|---|
| Raw Metrics | Workflow |
| Derived Metrics | Google Sheet (công thức) |
| Business Metrics | Người dùng nhập/chỉnh sửa |

Workflow KHÔNG được ghi đè vào nhóm Derived và Business Metrics.

### Rule xử lý field thiếu (Parser Rule)
Workflow phải xử lý trường hợp field video hoặc field Meta khác không tồn tại trong response.
Khi field không tồn tại, parser trả về 0, không throw Error.
Đây là rule thiết kế của parser, không phải đảm bảo hành vi cố định từ Meta API.

### Sửa lại phân loại Khach_sai_tep
Khach_sai_tep không phải dữ liệu nhập tay như ghi nhận trước đây.
Đây là công thức: Khach_sai_tep = Mess_Comment - Khach_hop_le
Cần xếp vào nhóm Derived/công thức, không phải nhóm nhập tay.
