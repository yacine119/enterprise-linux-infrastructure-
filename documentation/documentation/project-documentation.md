# Enterprise Linux Infrastructure Lab — Project Documentation

## 1. Overview

A two-server Linux infrastructure lab demonstrating Linux administration, cybersecurity controls, centralized logging, backup/recovery, security monitoring, and GRC risk management.

- **Rocky Linux — WEB01:** Web/Application server
- **Debian Linux — LOG01:** Central logging, management, and backup server
- **Virtualization:** VMware
- **Internal network:** `10.10.10.0/24`
- **Rocky:** `10.10.10.10`
- **Debian:** `10.10.10.20`

The project is an enterprise-style educational lab, not a production deployment.

## 2. Objectives

- Administer Linux servers and services.
- Apply least privilege and defense in depth.
- Harden SSH and restrict network exposure.
- Deploy and secure Nginx.
- Implement filesystem access control.
- Centralize security-relevant logs.
- Implement automated backup and recovery testing.
- Perform basic security monitoring.
- Identify risks, implement controls, test controls, collect evidence, and assess residual risk.

## 3. Architecture

```text
                         INTERNET
                            |
                           NAT
                            |
              +-------------+-------------+
              |                           |
              v                           v
      +---------------+           +---------------+
      | ROCKY LINUX   |           | DEBIAN LINUX  |
      | WEB01         |           | LOG01         |
      |               |           |               |
      | Nginx         |           | rsyslog       |
      | SSH           |           | Backup        |
      | Storage       |           | Management    |
      | firewalld     |           | UFW           |
      +-------+-------+           +-------^-------+
              |                           |
              | Centralized logs         |
              +-------------------------->|
              |                           |
              | Backup via SSH/rsync     |
              +-------------------------->|
```

## 4. Security Controls

### SSH

- `PermitRootLogin no`
- `AllowUsers webadmin`
- Configuration validation with `sshd -t`
- Authentication logging

### Firewall

Rocky uses `firewalld`; Debian uses UFW. Only required services are exposed. Debian UDP/514 is restricted to Rocky for centralized logging.

### Filesystem access

```text
/srv/data/
├── public/
└── private/
```

Groups `datausers` and `dataadmins` enforce access according to least privilege.

### Centralized logging

```text
Rocky Linux --rsyslog/UDP 514--> Debian Linux
```

The complete evidence chain is: configuration → listener → generated event → received event → stored log.

### Backup and recovery

```text
Rocky /srv/data/
        |
        | rsync over SSH
        v
Debian /srv/backup/rocky-web01/data/
```

Backup uses a dedicated `backup` account, an Ed25519 SSH key, `rsync`, a systemd service, and a systemd timer.

A file such as `/srv/data/public/restore2.txt` should appear on Debian as:

```text
/srv/backup/rocky-web01/data/public/restore2.txt
```

A recovery test should create data, back it up, delete the source, restore it, and verify the contents.

## 5. GRC Method

```text
Asset → Threat → Vulnerability → Risk → Impact
      → Control → Control Test → Evidence
      → Residual Risk → Risk Treatment
```

The project treats configuration alone as insufficient proof. Controls should be implemented, tested, supported by evidence, and evaluated for residual risk.

## 6. Representative Risk Register

| ID | Risk Scenario | Main Control | Treatment |
|---|---|---|---|
| R-01 | Unauthorized remote access | SSH hardening + firewall | Mitigate |
| R-02 | Unauthorized website modification | Least privilege + permissions | Mitigate |
| R-03 | Unauthorized data access | Groups + filesystem permissions | Mitigate |
| R-04 | Unauthorized network access | Firewall | Mitigate |
| R-05 | Exploitation of unpatched vulnerabilities | Patch management | Mitigate |
| R-06 | Loss of security evidence | Centralized logging | Mitigate |
| R-07 | Loss of security visibility | Monitoring | Mitigate |
| R-08 | Web/service unavailability | Monitoring + recovery | Mitigate |
| R-09 | Website/application data loss | Automated backup | Mitigate |
| R-10 | Backup/recovery failure | Restore testing | Mitigate |

## 7. Evidence Management

Evidence is organized by domain:

```text
evidence/
├── architecture/
├── patch-management/
├── iam/
├── ssh/
├── nginx/
├── storage/
├── firewall/
├── logging/
├── backup/
└── monitoring/
```

Naming convention:

```text
EVID-SSH-01-config.png
EVID-SSH-02-root-denied.png
EVID-FW-01-firewall.png
EVID-LOG-01-central-event.png
EVID-BACKUP-01-backup.png
EVID-BACKUP-02-restore.png
```

Each evidence item should identify the control, test, expected result, actual result, and risk addressed.

## 8. Project Limitations and Improvements

Possible future improvements include SIEM, IDS/IPS, file-integrity monitoring, vulnerability scanning, immutable/offline backups, backup encryption, TLS/HTTPS, network segmentation, centralized alerting, Windows/Active Directory integration, and automated compliance checks.

## 9. Final Outcome

The project demonstrates the transition from simply configuring infrastructure to assessing it as a risk analyst:

```text
Infrastructure → Controls → Testing → Evidence
             → Risk Assessment → Residual Risk → Treatment
```
