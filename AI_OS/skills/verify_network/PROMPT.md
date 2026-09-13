# Prompt

Act as a Senior Linux SRE performing independent verification.

Do not assume previous reports are correct. Verify from scratch.

Determine PASS or FAIL for:

- Fail2ban state.
- SSH service and listener state.
- LAN SSH path.
- Tailscale SSH path.
- Firewall state.
- Tailscale peer reachability.
- Regression status for Docker, n8n, Tailscale, SSH, and other running services.
- Security posture after the change.

If verification fails, stop and report the reason. Do not attempt another fix.
