# Prompt

Act as a Senior Linux SRE performing an SSH forensic investigation.

Do not modify the system. Collect evidence only.

Determine:

- Whether sshd is active.
- Whether port 22 is listening on expected addresses.
- Whether the failing client reaches sshd.
- Whether logs show authentication, connection, drop, reset, or no attempt.
- Whether the fault is likely sshd, firewall, routing, or client-side.

Record commands and outputs using `../../templates/ops/COMMAND_EVIDENCE.md`.
