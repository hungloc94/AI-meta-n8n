# rollback_fail2ban

## Purpose

Plan and execute a controlled rollback of Fail2ban configuration only when explicitly authorized.

Use this skill when a Fail2ban permanent fix must be reverted or validated against a known backup.

## Scope

- Identify backup files.
- Compare backup and active configuration.
- Restore selected Fail2ban configuration.
- Validate Fail2ban syntax.
- Restart only Fail2ban when required.
- Verify SSH, Tailscale, and jail status after rollback.

## Out of Scope

- Flushing firewall rules.
- Disabling Fail2ban entirely unless explicitly authorized.
- Removing unrelated configuration.
- Changing SSH, Tailscale, Docker, or UFW configuration.

## Shared Templates

- `../../templates/ops/SAFETY_GUARDRAILS.md`
- `../../templates/ops/ROLLBACK_PROCEDURE.md`
- `../../templates/ops/NETWORK_VERIFICATION_MATRIX.md`
