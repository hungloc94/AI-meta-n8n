# investigate_tailscale

## Purpose

Collect forensic evidence about Tailscale state, addresses, peer reachability, and path selection without changing the host.

Use this skill when Tailscale ICMP or `tailscale ping` works but application traffic such as SSH fails.

## Scope

- Confirm local Tailscale state.
- Record Tailscale IPs.
- Check peer visibility and connectivity.
- Inspect netcheck and path characteristics.
- Distinguish overlay reachability from service or firewall blocking.

## Out of Scope

- Changing Tailscale ACLs.
- Re-authenticating the node.
- Enabling subnet routes or exit nodes.
- Restarting Tailscale.

## Shared Templates

- `../../templates/ops/SAFETY_GUARDRAILS.md`
- `../../templates/ops/COMMAND_EVIDENCE.md`
- `../../templates/ops/NETWORK_VERIFICATION_MATRIX.md`
