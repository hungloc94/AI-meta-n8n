# Network Verification Matrix

Use this matrix to verify SSH, firewall, and overlay-network incidents.

| Area | Evidence | PASS Criteria | FAIL Criteria |
| --- | --- | --- | --- |
| SSH service | `systemctl status ssh` | Service active/running | Inactive, failed, restarting, masked |
| SSH listener | `ss -tlnp` | Port 22 listening on expected addresses | No listener, wrong port, unexpected bind |
| SSH sessions | `ss -tan` | Expected established or attempted flows visible | SYNs stuck, resets, unexpected drops |
| Fail2ban | `fail2ban-client status`, jail status | Expected jail enabled; no trusted IP banned | Trusted IP banned or jail missing unexpectedly |
| nftables | `nft list ruleset` | No trusted-network drop path | Drop/reject path for trusted network |
| iptables | `iptables-save` | No trusted-network drop path | Drop/reject path for trusted network |
| UFW | `ufw status verbose` | Expected allow policy remains | SSH or trusted path blocked |
| Tailscale | `tailscale status`, `tailscale ping` | Peer visible and reachable | Peer absent, ACL denied, no connectivity |
| LAN SSH | `ssh -v user@LAN_IP` | Auth prompt or login succeeds | TCP connect timeout/refused unexpectedly |
| Tailscale SSH | `ssh -v user@TS_IP` | Auth prompt or login succeeds | TCP connect timeout/refused unexpectedly |

## Result Format

```text
Area: <name>
Result: PASS|FAIL
Evidence: <command and decisive output>
Notes: <remaining risk or none>
```
