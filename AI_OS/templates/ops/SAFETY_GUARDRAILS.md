# Operational Safety Guardrails

Use these guardrails for Linux network, SSH, Tailscale, and Fail2ban incidents.

## Investigation Mode

- Collect evidence before drawing conclusions.
- Do not modify configuration, restart services, unban IPs, or flush firewall rules.
- Record every command that was executed and preserve relevant output.
- Separate observed facts from inferences.
- Stop when evidence is insufficient for the requested confidence threshold.

## Recovery Mode

- Proceed only when root cause confidence meets the stated threshold.
- Back up every configuration file before editing.
- Make the smallest possible change.
- Restart only services required by the change.
- Document before state, after state, result, and rollback command.

## Permanent Fix Mode

- Preserve existing security controls.
- Never overwrite local configuration without reviewing it.
- Prefer additive drop-in files over editing package-managed defaults.
- Bypass Fail2ban only for explicitly trusted private networks.
- Keep Internet source addresses protected by Fail2ban.

## Verification Mode

- Verify service state, listener state, firewall state, network path, and application behavior.
- Confirm there are no unexpected bans or broad allow rules.
- Record PASS or FAIL with evidence.
- If verification fails, stop and report the failure instead of attempting another fix.
