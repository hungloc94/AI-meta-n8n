# investigate_ssh

## Purpose

Collect forensic evidence about OpenSSH service, listeners, sessions, and logs without changing the host.

Use this skill when SSH hangs, times out, refuses connections, or behaves differently across LAN, VPN, and Internet paths.

## Scope

- Confirm sshd service state.
- Confirm port 22 listener bindings.
- Inspect recent SSH logs.
- Inspect active and half-open TCP sessions.
- Distinguish sshd failures from network or firewall failures.

## Out of Scope

- Editing `sshd_config`.
- Restarting SSH.
- Changing authentication policy.
- Adding users or keys.

## Shared Templates

- `../../templates/ops/SAFETY_GUARDRAILS.md`
- `../../templates/ops/COMMAND_EVIDENCE.md`
- `../../templates/ops/NETWORK_VERIFICATION_MATRIX.md`
