# Inputs

Required:

- Active Fail2ban configuration path.
- Backup configuration path.
- Target jail name, for example `sshd`.
- Trusted networks and test IPs.

Optional:

- Previous permanent-fix report.
- Reason rollback is being considered.
- Approved maintenance window.

## Default Commands

```bash
sudo test -e <backup_file>
sudo test -e <active_file>
diff -u <backup_file> <active_file>
sudo cp -a <backup_file> <active_file>
sudo fail2ban-client -t
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

The restore and restart commands require explicit operator approval.
