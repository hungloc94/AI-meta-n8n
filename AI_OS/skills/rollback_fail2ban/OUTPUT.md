# Outputs

Produce a rollback report with:

- Authorization status.
- Backup file used.
- Active file restored.
- Diff reviewed before rollback.
- Commands executed.
- Validation result.
- Services restarted.
- Verification result.
- Rollback success or failure.

## Required Result Format

```text
Rollback Executed: yes|no
Authorization Evidence: <who/when or not authorized>
Files Restored:
- <path or none>
Services Restarted:
- <service or none>
Verification Result: PASS|FAIL|not run
```
