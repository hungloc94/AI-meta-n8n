# AI_OS Operational Skills Index

## Structure Review

The current AI_OS tree already contained a `skills/` directory with a `dev-workflow/` bundle. The incident workflow showed a gap for reusable Linux operations skills covering SSH, Tailscale, Fail2ban, verification, and rollback.

This update adds modular operational skills and shared templates:

```text
AI_OS/
├── skills/
│   ├── investigate_fail2ban/
│   ├── investigate_ssh/
│   ├── investigate_tailscale/
│   ├── verify_network/
│   └── rollback_fail2ban/
└── templates/
    └── ops/
```

## Reusable Skills

| Skill | Purpose | Inputs | Outputs | Required Permissions | Dependencies |
| --- | --- | --- | --- | --- | --- |
| `investigate_fail2ban` | Collect Fail2ban jail, ban, ignore list, and firewall evidence. | Jail name, client IP, trusted networks. | Fail2ban evidence report and confidence-scored root-cause assessment. | Read-only shell; sudo may be needed for firewall and service status. | Fail2ban, systemd, nftables or iptables. |
| `investigate_ssh` | Determine whether SSH service, listener, logs, or TCP state explains a failure. | Server IPs, client IP, SSH port, symptom. | SSH state report and component-level root-cause candidate. | Read-only shell; sudo may be needed for `sshd -T` and logs. | OpenSSH, systemd, ss. |
| `investigate_tailscale` | Determine whether Tailscale overlay state or peer path explains a failure. | Server Tailscale IP, peer name/IP, application port. | Tailscale reachability and path report. | Read-only shell. | Tailscale CLI. |
| `verify_network` | Independently verify service, firewall, SSH, Tailscale, and regression status. | Expected services, trusted networks, LAN/Tailscale test targets. | PASS/FAIL verification report with remaining risks. | Read-only shell; remote SSH test permission if used. | systemd, Fail2ban, Tailscale, firewall tools, optional Docker. |
| `rollback_fail2ban` | Restore a known Fail2ban backup and verify rollback when authorized. | Active config path, backup path, jail name, trusted networks. | Rollback report with commands, files, services, and verification. | Explicit write approval and sudo. | Fail2ban, systemd, backup file. |
| `highlight_duplicate_rows` | Bôi màu dòng duplicate trên Google Sheet để review trực quan trước khi xóa. | `/tmp/dup_highlight.json` (`[{row,color}]`); spreadsheetId; gid tab. | Các dòng được bôi màu (vàng=xóa, cam=gộp, trắng=reset) trên Sheet. | n8n manual run + Google service account credential. | n8n, Google Sheets API, workflow `QWowp4oT9PA1iJTO`. |

## Template Reuse

Common logic is centralized in `templates/ops/`:

- Safety rules for investigation, recovery, permanent fixes, and verification.
- Command evidence format.
- Network verification matrix.
- Incident report section model.
- Rollback procedure template.

Skills should reference these shared templates instead of duplicating broad procedural text.
