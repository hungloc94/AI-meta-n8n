# Inputs

Required:

- Server IPs to test: LAN IP and/or Tailscale IP.
- Client IP if known.
- Failing SSH command and symptom.

Optional:

- SSH port if not `22`.
- Incident start time.
- Username used for testing.

## Default Commands

```bash
systemctl status ssh
journalctl -u ssh --since "1 hour ago"
ss -tlnp
ss -tan
sudo sshd -T
sudo sshd -t
```

Use remote SSH tests only when operator access and target identity are known.
