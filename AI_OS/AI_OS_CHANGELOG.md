# AI_OS Changelog

## 2026-07-28

### New Skills

- Added `skills/investigate_fail2ban/` for evidence-only Fail2ban investigations.
- Added `skills/investigate_ssh/` for OpenSSH service, listener, log, and TCP-state investigations.
- Added `skills/investigate_tailscale/` for Tailscale node, peer, and path investigations.
- Added `skills/verify_network/` for independent PASS/FAIL verification after network incidents.
- Added `skills/rollback_fail2ban/` for controlled Fail2ban rollback workflows.

### Updated Documentation

- Added `skills/README.md` as the operational skill directory index.
- Added `SKILLS.md` as the AI_OS operational skills index and structure review.
- Added `templates/ops/README.md` as the shared operational template index.

### Reusable Components Extracted

- Added `templates/ops/SAFETY_GUARDRAILS.md`.
- Added `templates/ops/COMMAND_EVIDENCE.md`.
- Added `templates/ops/NETWORK_VERIFICATION_MATRIX.md`.
- Added `templates/ops/INCIDENT_REPORT_SECTIONS.md`.
- Added `templates/ops/ROLLBACK_PROCEDURE.md`.

### Reason For Changes

The SSH-over-Tailscale incident showed that future incidents need reusable workflows for evidence collection, confidence-scored root-cause analysis, safe recovery, permanent prevention, independent verification, and rollback.

### Future Improvement Opportunities

- Add executable evidence-collection scripts that produce structured JSON.
- Add parsers for Fail2ban status, nftables, iptables, and Tailscale output.
- Add policy checks that detect trusted private networks missing from Fail2ban `ignoreip`.
- Add an approval-gated remediation runner for safe Fail2ban unban and ignore-list changes.
- Add tests for generated incident reports to verify required sections are present.
