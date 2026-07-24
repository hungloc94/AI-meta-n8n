# PRODUCTION PROMOTION CHECKLIST

## PHASE A — CONTROLLED ACTIVATION

### PRE-ACTIVATION CHECKS

* [x] TEST workflow fully verified
* [x] active=false import verified
* [x] rollback export exists
* [x] LAST KNOWN GOOD baseline exists
* [x] credentials correctly bound
* [ ] no hardcoded test offsets remain
* [ ] no debug-only code remains
* [x] polling continuity verified
* [x] duplicate-processing guard verified
* [x] project operational memory updated
* [ ] restore offset expressions after testing
* [ ] confirm no stale replay after restart

---

### CONTROLLED ACTIVATION PLAN

* [x] activate workflow under supervision
* [x] restrict first run observation window
* [x] monitor Telegram output
* [x] monitor polling continuity
* [x] verify no replay behavior
* [x] verify KPI integrity under live runtime

---

### VERIFIED SUPERVISED RUNTIME ACTIVATION

* [x] command tested: `báo cáo hôm qua`
* [x] exactly one Telegram response generated
* [x] no stale replay
* [x] no duplicate reports
* [x] no replay of old 2026-03-24 report
* [x] KPI=0 for 2026-05-26 confirmed as expected business reality
* [x] temporary hardcoded offset documented: `81218650`
* [ ] temporary offset expression restoration verified

---

## PHASE B — SUSTAINED OBSERVATION

### 24H OBSERVATION CHECKS

* [ ] no duplicate Telegram reports
* [ ] no stale queue replay
* [ ] no memory drift
* [ ] no KPI corruption
* [ ] no runtime crashes
* [ ] no delayed command processing

---

## ROLLBACK READINESS

* [ ] rollback export verified
* [ ] previous stable workflow retained
* [ ] recovery runbook updated
* [ ] backup inventory updated

---

## GO / NO-GO CRITERIA

GO ONLY IF:

* all staging tests pass
* polling continuity verified
* no duplicate processing observed
* KPI integrity verified
* rollback readiness confirmed

NO-GO IF:

* any replay behavior exists
* KPI integrity uncertain
* runtime instability observed
* credential continuity uncertain
* hardcoded Telegram offset remains in an active production workflow

Operational note:
Production promotion chỉ được xem xét khi tất cả open checklist items đã resolved. Production activation hiện vẫn NOT approved.
