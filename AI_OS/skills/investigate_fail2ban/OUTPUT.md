# Outputs

Produce a report with:

- Fail2ban service state.
- Enabled jail list.
- Relevant jail state.
- Banned IP list.
- Loaded `ignoreip` values.
- Firewall backend evidence.
- Rules that affect the suspect source IP or trusted network.
- Root-cause assessment with confidence score.

## Required Conclusion Format

```text
Root Cause: <proven|not proven> - <statement>
Confidence Score: <0-100>
Evidence Basis:
- <fact>
- <fact>
Remaining Gaps:
- <gap or none>
```
