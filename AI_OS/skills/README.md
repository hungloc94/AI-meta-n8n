# AI_OS Skills

Reusable operational skills live under `AI_OS/skills/`.

## Existing Skills

- `dev-workflow/`: existing development workflow skill bundle.
- `investigate_fail2ban/`: forensic Fail2ban evidence collection.
- `investigate_ssh/`: forensic OpenSSH service and connection investigation.
- `investigate_tailscale/`: forensic Tailscale state and peer-path investigation.
- `verify_network/`: independent PASS/FAIL network and service verification.
- `rollback_fail2ban/`: controlled Fail2ban rollback workflow.
- `highlight_duplicate_rows/`: bôi màu dòng duplicate trên Google Sheet để review trực quan trước khi xóa.

## Skill File Contract

Each operational skill provides:

- `README.md`: purpose, scope, and shared templates.
- `PROMPT.md`: reusable operator prompt.
- `INPUT.md`: required inputs and default evidence commands.
- `OUTPUT.md`: required output/report structure.
- `CHECKLIST.md`: execution checklist.

## Shared Components

Common procedures are stored in `../templates/ops/` and referenced by each skill to avoid duplication.

Operational skills must follow:

- Evidence-first investigation.
- No system changes unless the skill explicitly supports recovery or rollback and the operator authorizes it.
- Explicit PASS/FAIL or confidence-scored conclusions.
- Commands and outputs recorded as incident evidence.
