# GRC Methodology

## 1. Purpose

This document defines the methodology used to assess cybersecurity risks and controls in the Linux infrastructure lab.

## 2. Risk Assessment Model

```text
ASSET
  ↓
THREAT
  ↓
VULNERABILITY
  ↓
RISK SCENARIO
  ↓
IMPACT
  ↓
CONTROL
  ↓
CONTROL TEST
  ↓
EVIDENCE
  ↓
RESIDUAL RISK
  ↓
RISK TREATMENT
```

## 3. Assets

Examples include Rocky WEB01, `/srv/data`, Nginx, SSH administration, Debian LOG01, centralized logs, backup data, and SSH credentials/keys.

## 4. Threats

Examples include unauthorized remote access, malware, exploitation of vulnerabilities, unauthorized modification, data deletion, credential compromise, backup destruction, service failure, and denial of service.

## 5. Vulnerabilities

Examples include weak SSH configuration, excessive filesystem permissions, unpatched software, unnecessary exposed services, insufficient backup protection, inadequate monitoring, and weak credential protection.

A vulnerability should only be recorded when there is a reasonable basis for identifying it.

## 6. Risk Scenario

A risk scenario combines the asset, threat, and vulnerability into a meaningful consequence.

**Example:**

- Asset: Rocky WEB01
- Threat: Unauthorized remote access
- Vulnerability: Weak or excessive SSH access
- Risk: Unauthorized access could compromise the web server.

A failed SSH login is an **event**, not by itself a risk. Excessive filesystem permissions are a **vulnerability**, not the risk itself.

## 7. Impact

Assess consequences across confidentiality, integrity, availability, and operational impact.

## 8. Risk Scoring

A simple model is:

```text
Risk Score = Likelihood × Impact
```

Likelihood and impact can each use a 1–5 scale. Organizational thresholds should be defined before labels such as Low, Medium, or High are assigned.

## 9. Inherent and Residual Risk

**Inherent risk** is assessed before considering controls.

**Residual risk** is the risk remaining after controls are implemented and evaluated.

```text
Control implemented ≠ Risk eliminated
```

## 10. Control Types

### Preventive

- Firewall
- SSH restrictions
- Least privilege
- Filesystem permissions
- Patch management

### Detective

- Centralized logging
- Log analysis
- Security monitoring

### Corrective / Recovery

- Backup
- Restore procedures
- Incident response

## 11. Control Testing

```text
Control → Test → Expected Result → Actual Result → Evidence → Conclusion
```

Example SSH test:

```bash
ssh root@10.10.10.10
```

Expected result: authentication denied when direct root login is disabled.

## 12. Evidence

Three useful categories are:

1. **Configuration evidence** — proves the control exists.
2. **Test evidence** — proves the control was tested.
3. **Result evidence** — shows what actually happened.

The strongest evidence connects all three.

## 13. Risk Treatment

Four standard options are used:

- **Mitigate** — reduce likelihood or impact.
- **Avoid** — stop the activity creating the risk.
- **Transfer** — shift some consequences to another party.
- **Accept** — formally acknowledge risk within the organization's tolerance.

## 14. Control-to-Risk Mapping

| Control | Risk |
|---|---|
| SSH hardening | Unauthorized remote access |
| Firewall | Unauthorized network access |
| Least privilege | Unauthorized modification/access |
| Filesystem permissions | Unauthorized data access |
| Patch management | Exploitation of known vulnerabilities |
| Centralized logging | Loss of security visibility/evidence |
| Backup | Data loss |
| Restore testing | Recovery failure |

## 15. Backup Risk Example

**Asset:** Backup system — DEBIAN-LOG01

**Threat:** Attackers gain administrative control and delete, encrypt, or alter backup data.

**Vulnerability:** Access to stored credentials and recovery data.

**Risk:** Compromise or loss of backup data prevents recovery after an incident.

**Impact:** Data loss, extended downtime, and loss of recovery capability.

**Control:** Automated Rocky-to-Debian backup using rsync over SSH with key-based authentication and a dedicated backup account.

**Evidence:** Backup configuration, SSH key, systemd service/timer, backup execution, destination files, and restore test.

**Residual risk:** Compromise of the backup server or credentials could still affect recovery data.

**Treatment:** Mitigate.

## 16. Risk Register Fields

Recommended fields:

```text
Risk ID
Date
Asset
Threat
Vulnerability
Risk Scenario
Impact
Likelihood
Inherent Risk
Controls
Control Test
Evidence
Residual Risk
Treatment
Status
Owner
```

## 17. Analyst Perspective

The administrator asks: **How do I configure SSH securely?**

The security engineer asks: **How do I technically harden SSH?**

The risk analyst asks: **What risk does remote administration create, what controls are required, are they effective, what evidence supports that conclusion, and what residual risk remains?**

The project is designed to practice this third perspective while maintaining the technical understanding needed to evaluate controls.
