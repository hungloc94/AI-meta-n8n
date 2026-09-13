# Prompt

Act as a Senior Linux and Tailscale SRE performing a Tailscale forensic investigation.

Do not modify the system. Collect evidence only.

Determine:

- Whether the local node is logged in and healthy.
- Which Tailscale IPs are assigned.
- Whether the peer is visible.
- Whether peer connectivity succeeds.
- Whether traffic is direct or DERP-relayed.
- Whether Tailscale reachability contradicts or supports firewall, ACL, or service-level causes.

Record commands and outputs using `../../templates/ops/COMMAND_EVIDENCE.md`.
