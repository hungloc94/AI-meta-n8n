# STAGING TEST MATRIX

## TEST CATEGORY

Runtime Persistence

### TEST:

Restart workflow after successful polling.

EXPECTED RESULT:

* No replay of old Telegram commands
* staticData continuity preserved
* no duplicate Telegram reports

FAIL CONDITIONS:

* duplicate reports
* replayed updates
* offset reset
* stale queue processing

SEVERITY:
CRITICAL

LATEST VALIDATION:
Polling continuity đã pass dưới supervised real workflow activation. Không thấy stale replay, duplicate reports, hoặc old 2026-03-24 replay với `báo cáo hôm qua`.

REMAINING CHECK:
Confirm no stale replay after restart trước production promotion.

---

## TEST CATEGORY

Duplicate Command Stress

### TEST:

Send repeated:

* báo cáo hôm nay
* báo cáo 24/03

multiple times rapidly.

EXPECTED RESULT:

* One clean processing cycle per command
* no looping
* no duplicate delivery

FAIL CONDITIONS:

* repeated sends
* duplicate KPI reports
* queue corruption

SEVERITY:
CRITICAL

LATEST VALIDATION:
Duplicate-processing protection đã pass cho supervised runtime command `báo cáo hôm qua`; exactly one Telegram response generated.

REMAINING CHECK:
Rapid repeated command stress vẫn là staging test riêng.

---

## TEST CATEGORY

Invalid Command Handling

### TEST:

Send invalid commands:

* báo cáo abcxyz
* random text
* malformed dates

EXPECTED RESULT:

* graceful handling
* no crash
* no routing corruption

FAIL CONDITIONS:

* workflow exceptions
* stuck polling
* broken routing state

SEVERITY:
HIGH

---

## TEST CATEGORY

Empty Data Date

### TEST:

Request:

* báo cáo 01/01

for date with no data.

EXPECTED RESULT:

* graceful empty report
* no NaN
* no Infinity
* no undefined

FAIL CONDITIONS:

* crash
* broken report formatting
* invalid KPI calculations

SEVERITY:
HIGH

LATEST VALIDATION:
Runtime command `báo cáo hôm qua` returned KPI=0 cho 2026-05-26. Kết quả này verified là expected business reality do missing/no sheet data, không phải pipeline failure.

---

## TEST CATEGORY

Long Runtime Stability

### TEST:

Run workflow continuously for extended observation window.

EXPECTED RESULT:

* polling stable
* memory stable
* no replay drift
* no delayed queue corruption

FAIL CONDITIONS:

* polling freeze
* duplicate processing
* delayed replay
* memory instability

SEVERITY:
CRITICAL
