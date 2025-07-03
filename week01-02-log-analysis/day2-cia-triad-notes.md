# Day 2 – CIA Triad & Security+ Refresh

## CIA Triad Overview

### 1. Confidentiality
- Controls: encryption (AES), permissions, MFA
- Tools: BitLocker, TLS, VPNs
- Key Threats: data leakage, unauthorized access

### 2. Integrity
- Controls: checksums, digital signatures, hashing (SHA256)
- Tools: Tripwire, file integrity monitors
- Key Threats: tampering, MITM, data corruption

### 3. Availability
- Controls: redundancy, failover, UPS, load balancing
- Tools: HAProxy, backups, DDoS mitigation (Cloudflare, WAF)
- Key Threats: DDoS attacks, system crashes, sabotage

---

## Security+ Review Notes – Logs & Attacks

### Logs to Monitor:
- **Authentication Logs**: Detect brute force, credential misuse
- **Process Creation Logs (4688)**: Catch suspicious PowerShell usage or LOLBins
- **System Logs**: Service crashes, unauthorized config changes

### Key Attack Types Reviewed:
- **Brute Force**: multiple failed logins from same IP
- **Phishing**: malicious links in emails or payload delivery
- **Privilege Escalation**: suspicious access attempts, event ID 4672
- **Malware Behavior**: processes spawning unexpected child processes (e.g., Word > PowerShell)
- **SQL Injection**: web server logs showing suspicious input patterns ) (e.g. **' OR 1=1 --**), often targeting login forms
- **Buffer Overflow**: app crash logs, unusual memory access attempts, or service instability; often a sign of exploit testing

Security+ reinforced how these logs and patterns are essential to **detect early indicators of compromise (IOCs)** as a SOC Analyst.

---

## Takeaways

- CIA Triad ties directly into Blue Team responsibilities
- Security+ concepts are now easier to understand with hands-on context
- These principles will help guide my log review and triage decisions

