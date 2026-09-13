# Inputs

Required:

- Service name or suspected jail name, for example `sshd`.
- Client IP address if known.
- Trusted network ranges if known, for example LAN CIDR and Tailscale CIDR.

Optional:

- Incident start time.
- Relevant previous reports.
- Expected firewall backend: nftables, iptables, or unknown.

## Default Commands

```bash
systemctl status fail2ban
fail2ban-client status
fail2ban-client status sshd
fail2ban-client get sshd ignoreip
nft list ruleset
iptables-save
journalctl -u fail2ban --since "1 hour ago"
```
