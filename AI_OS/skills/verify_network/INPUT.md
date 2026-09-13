# Inputs

Required:

- Services to verify.
- Trusted network ranges.
- LAN IP and Tailscale IP for SSH checks.
- Expected Fail2ban jails.

Optional:

- Previous incident reports.
- Expected Docker containers.
- Expected n8n URL or process manager.
- External test host details.

## Default Commands

```bash
systemctl status fail2ban
fail2ban-client status
fail2ban-client status sshd
systemctl status ssh
ss -tlnp
ss -tan
nft list ruleset
iptables-save
ufw status verbose
tailscale status
tailscale ping <peer>
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```
