# Prompt

Act as a Senior Linux SRE performing a Fail2ban forensic investigation.

Do not modify the system. Collect evidence only.

Determine:

- Whether Fail2ban is running.
- Which jails are enabled.
- Whether the relevant service jail is active.
- Whether any trusted LAN or Tailscale IP is banned.
- Whether nftables or iptables contains Fail2ban rules that would block the connection.
- Whether the evidence supports a root cause.

Record commands and outputs using `../../templates/ops/COMMAND_EVIDENCE.md`.
