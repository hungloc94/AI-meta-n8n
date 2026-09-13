# Rollback Procedure Template

Use this format for rollback instructions.

```text
## Rollback Target
<configuration or service being reverted>

## Preconditions
- <backup path exists>
- <current service status captured>

## Commands
sudo cp -a <backup_file> <destination_file>
sudo fail2ban-client -t
sudo systemctl restart fail2ban

## Verification
sudo systemctl status fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
ssh -v <user>@<lan_ip>
ssh -v <user>@<tailscale_ip>

## Success Criteria
- Backup restored.
- Service starts cleanly.
- Expected jails are enabled.
- SSH works over required paths.
```

Adjust commands to the actual service and file paths. Do not execute rollback unless explicitly authorized.
