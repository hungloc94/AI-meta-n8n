# Test Playbook

## Role
Quy trình test an toàn, lặp lại được cho recovery và regression checks.

## Standard Recovery Test Flow
1. Import workflow dưới dạng TEST branch.
2. Verify workflow là `active=false`.
3. Verify không bật public webhook hoặc tunnel.
4. Execute nodes tuần tự bằng Execute Step để isolate lỗi.
5. Verify output sau từng node.
6. Không test production trước.
7. Sau khi node-level pass, verify polling continuity bằng supervised runtime activation thật.

## Node Execution Order
```text
Telegram API getUpdates
→ Process Updates
→ Main Router Code
→ IF Report
→ Parse & Route
→ Đọc Google Sheet
→ Normalize Sheet Data
→ Lọc Dữ Liệu Thiếu
→ Tạo Báo Cáo
→ Send Report Telegram
```

## Required Checks Per Node
- input shape
- output shape
- route/branch selected
- date normalization
- KPI field values
- Telegram payload text
- no duplicate-processing side effects

## Offset Testing Rule
Nếu dùng hardcoded offset testing, phải restore offset expression trước mọi activation review.

Không bao giờ để hardcoded Telegram offset values trong active production workflows sau testing.

## Schema Testing Rule
Trước khi patch KPI logic, phải validate downstream schema dependencies.

## Send Report Rule
Trước final import testing, verify `Send Report Telegram` gửi `$json.message`, không phải debug placeholder.

## Known Good Commands

Dùng các command này để controlled TEST workflow verification:

- `menu`
- `báo cáo hôm nay`
- `báo cáo hôm qua`
- `báo cáo 24/03`
- `báo cáo từ 2026-03-24 đến 2026-03-24`

Expected verified output cho `báo cáo 24/03`:
```json
{
  "dates": ["2026-03-24"]
}
```

## Known Limitations

- DD/MM/YYYY format chưa support.
- Year rollover ambiguity vẫn tồn tại cho DD/MM inputs.
- Execute Workflow không recommended trong forensic recovery.
- Polling systems cần offset/staticData continuity.
- Restart no-stale-replay check vẫn cần verify trước production promotion.

## Runtime Polling Continuity Validation

Manual Execute Step testing không đủ để validate polling continuity. Queue/polling systems phải được validate dưới real runtime activation vì staticData persistence behave khác khi workflow thật sự active.

Validated runtime command:
```text
báo cáo hôm qua
```

Validated runtime output:
```text
📊 BÁO CÁO META ADS
📅 Ngày: 2026-05-26
💰 Tổng chi tiêu: 0đ
📩 Tổng kết quả: 0
🎯 CPA: 0đ
```

Required runtime checks:
- no stale replay
- no duplicate reports
- no replay of old historical reports
- exactly one Telegram response per command
- offset/staticData continuity preserved
- temporary hardcoded offsets restored before activation review

Temporary testing offset used during verified backlog drain:
```text
81218650
```

KPI=0 cho 2026-05-26 được accept là expected business reality khi sheet data missing/empty, không phải pipeline failure.

## End-to-End Recovery Verification Checklist

Dùng checklist này trước khi declare recovery success:

- [ ] Verify Telegram command reception.
- [ ] Verify route classification.
- [ ] Verify Parse & Route ISO date output.
- [ ] Verify Normalize Sheet Data numeric conversion.
- [ ] Verify `loadedRecords` không empty khi có matching data.
- [ ] Verify KPI aggregation totals.
- [ ] Verify Telegram final formatting.
- [ ] Verify divide-by-zero protection.
- [ ] Verify final Telegram payload không có invalid artifacts.

Final output không được có:
- `NaN`
- `Infinity`
- `undefined`
- `[NODE=REPORT]`

Với date `2026-03-24`, KPI zero valid leads được accept là verified business reality, không phải pipeline failure.
