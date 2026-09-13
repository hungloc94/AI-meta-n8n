# investigate_fail2ban

## Purpose

Collect forensic evidence about Fail2ban state without changing the host.

Use this skill when SSH, Tailscale SSH, web admin access, or other network services may be blocked by Fail2ban.

## Scope

- Identify enabled jails.
- Identify banned IPs.
- Confirm whether trusted private networks are ignored.
- Inspect Fail2ban-generated firewall rules through nftables or iptables.
- Produce evidence suitable for root-cause analysis.

## Out of Scope

- Unbanning IPs.
- Editing Fail2ban configuration.
- Restarting services.
- Flushing firewall rules.

## Shared Templates

- `../../templates/ops/SAFETY_GUARDRAILS.md`
- `../../templates/ops/COMMAND_EVIDENCE.md`
- `../../templates/ops/INCIDENT_REPORT_SECTIONS.md`
