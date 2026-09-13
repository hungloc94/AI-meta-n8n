# Outputs

Produce a report with:

- Local Tailscale state.
- Assigned Tailscale IPs.
- Peer visibility.
- Peer reachability result.
- Direct or DERP path evidence.
- ACL or routing suspicion if supported by evidence.
- Relationship between overlay reachability and failing application traffic.

## Required Conclusion Format

```text
Tailscale State: <healthy|degraded|failed|unknown>
Peer Reachable: <yes|no|unknown>
Application Path Cause: <tailscale|firewall|service|client|unknown>
Confidence Score: <0-100>
Evidence:
- <fact>
```
