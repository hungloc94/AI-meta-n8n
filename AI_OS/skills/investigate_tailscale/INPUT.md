# Inputs

Required:

- Server Tailscale IP.
- Client Tailscale IP or peer name if known.
- Application port being tested, for example SSH port `22`.

Optional:

- Tailnet policy or ACL excerpt if available.
- Client operating system.
- Prior packet-capture evidence.

## Default Commands

```bash
tailscale status
tailscale version
tailscale ip
tailscale ip -4
tailscale netcheck
tailscale ping <peer>
tailscale debug prefs
```

Use `tailscale bugreport` only when an incident report needs vendor-quality diagnostics.
