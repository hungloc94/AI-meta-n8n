# Outputs

Produce a report with:

- SSH service state.
- Listener evidence.
- Recent SSH log evidence.
- TCP connection-state evidence.
- LAN SSH result if tested.
- Overlay or VPN SSH result if tested.
- Ruling on whether sshd is the failing component.

## Required Conclusion Format

```text
SSH State: <healthy|degraded|failed|unknown>
Root Cause Candidate: <component and reason>
Confidence Score: <0-100>
Evidence:
- <fact>
Next Evidence Needed:
- <command or none>
```
