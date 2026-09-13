# verify_network

## Purpose

Verify network service health after investigation, recovery, or permanent configuration changes.

Use this skill to produce PASS/FAIL evidence for SSH, Tailscale, firewall, Fail2ban, and dependent services.

## Scope

- Verify service active states.
- Verify listeners and connection states.
- Verify LAN and Tailscale SSH paths.
- Verify firewall rules do not block trusted networks.
- Verify Internet-facing protection remains active.
- Regression-test important services.

## Out of Scope

- Applying fixes.
- Editing firewall or Fail2ban configuration.
- Restarting services unless explicitly part of a separate recovery task.

## Shared Templates

- `../../templates/ops/SAFETY_GUARDRAILS.md`
- `../../templates/ops/NETWORK_VERIFICATION_MATRIX.md`
- `../../templates/ops/COMMAND_EVIDENCE.md`
