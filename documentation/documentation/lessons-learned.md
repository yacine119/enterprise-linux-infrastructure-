# Lessons Learned

## 1. Technical Lessons

### Services must be tested

Installing a package does not prove that the service works correctly.

```text
Installed → Enabled → Running → Listening → Functional test
```

### SSH changes require caution

Keep an existing administrative session open when changing SSH or firewall settings. Validate before reload:

```bash
sudo sshd -t
systemctl status sshd
ss -tulpn
```

### Firewall configuration must be validated

A firewall rule should be compared with actual listening services and connectivity tests.

```text
Firewall configuration + Listening services + Connectivity test
```

### Centralized logging needs end-to-end testing

A running rsyslog service does not prove that remote logs are received.

```text
Rocky → forwarding → network → Debian listener → received event → stored log
```

A test event can be generated with:

```bash
logger "GRC-CENTRAL-LOG-TEST Rocky-WEB01"
```

## 2. Backup Troubleshooting Lesson

The backup service ran as the `backup` user while the destination was initially owned by `root:root`. Rsync therefore returned permission errors and exit code 23.

The troubleshooting chain was:

```text
systemd → script → SSH authentication → rsync → destination permissions
```

SSH itself was working. The failure was at the destination write stage.

The correction was:

```bash
sudo chown -R backup:backup /srv/backup/rocky-web01
```

This demonstrated that service-account permissions are part of secure automation design.

## 3. Backup Is Not Recovery

A successful file copy does not prove recoverability.

A stronger test is:

```text
Create test data
      ↓
Backup
      ↓
Verify backup
      ↓
Delete source
      ↓
Restore
      ↓
Verify contents
```

For example, `/srv/data/public/restore2.txt` on Rocky should appear as:

```text
/srv/backup/rocky-web01/data/public/restore2.txt
```

## 4. Risk Modeling Lessons

Keep these concepts separate:

- **Threat:** potential harmful event or actor.
- **Vulnerability:** weakness.
- **Risk:** consequence arising from a threat exploiting a vulnerability.
- **Control:** measure that reduces risk.
- **Evidence:** information showing that a control exists or operated as expected.

Example:

```text
Threat: Unauthorized remote access
Vulnerability: Weak SSH access control
Risk: Unauthorized compromise of the web server
Control: SSH hardening + firewall
Evidence: Configuration + test + logs
```

## 5. Event vs Risk

A failed SSH authentication is an event, not by itself a risk.

A stronger risk statement is:

> Unauthorized access to the web server through compromised or improperly controlled remote administration.

Similarly, excessive filesystem permissions are a vulnerability; the risk is unauthorized access to or modification of protected data.

## 6. Documentation vs Evidence

Documentation explains what was designed and implemented.

Evidence demonstrates what actually exists or happened.

Example:

> SSH root login is disabled.

Evidence should include configuration output plus an actual test and relevant authentication log.

## 7. Configuration Does Not Equal Effectiveness

```text
Configured ≠ Effective
```

A backup script may exist but fail because of permissions. A logging configuration may exist but not receive remote events. A firewall rule may exist but not protect the intended service.

Therefore the stronger model is:

```text
Configuration + Test + Expected result + Actual result
```

## 8. Separation of Roles

Separating web services from logging and backup improves resilience:

```text
Rocky → Web/Application
Debian → Logs + Backup
```

However, the second server must also be protected. Separation does not remove the need for access control, firewalling, credential protection, and monitoring.

## 9. Residual Risk

Controls reduce risk but rarely eliminate it completely.

For example, a firewall reduces unauthorized network access but allowed services can still be vulnerable. Backups reduce data-loss impact but a compromised backup server or credential can still affect recovery.

Therefore the correct conclusion is that controls **reduce** risk and residual risk must still be assessed.

## 10. Evidence Should Be Purposeful

Avoid collecting hundreds of unexplained screenshots. Each evidence item should answer:

1. What control does this prove?
2. What risk does it address?
3. What test was performed?
4. What was expected?
5. What actually happened?

Recommended naming:

```text
EVID-SSH-01-config.png
EVID-SSH-02-root-denied.png
EVID-FW-01-firewall.png
EVID-LOG-01-central-event.png
EVID-BACKUP-01-backup.png
EVID-BACKUP-02-restore.png
```

## 11. Production Improvements

Potential next steps include:

- Immutable/offline backups
- Backup encryption
- Stronger credential protection
- SIEM
- File Integrity Monitoring
- IDS/IPS
- Vulnerability scanning
- Centralized alerting
- HTTPS/TLS
- Network segmentation
- Automated compliance checks
- Formal incident response procedures

## 12. Final Learning Outcome

The main lesson is that cybersecurity is not only technical configuration. A risk-oriented process asks:

```text
What are we protecting?
        ↓
What can go wrong?
        ↓
Why can it happen?
        ↓
What control reduces the risk?
        ↓
Did we implement it?
        ↓
Did we test it?
        ↓
What evidence proves it?
        ↓
What risk remains?
        ↓
What should be done about it?
```

This is the transition from Linux administration toward cybersecurity and cyber risk analysis.
