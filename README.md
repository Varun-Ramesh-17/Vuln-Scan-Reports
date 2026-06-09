# 🕵️ Vulnerability Scan Reports

> Nmap-based network enumeration, CVE identification, CVSS severity rating, and structured remediation reports — across 20+ lab hosts.

---

## Overview

This project documents my vulnerability assessment methodology across a simulated enterprise lab environment. Each assessment follows a full cycle: scoping → enumeration → CVE identification → severity rating → remediation recommendations.

**Key results:**
- 20+ lab hosts assessed
- 30+ CVEs identified and documented
- All findings prioritised by CVSS score and exploitability
- Reports structured in pentest-standard format

---

## Assessment Methodology

```
1. SCOPING      → Define target hosts, assessment boundaries
2. ENUMERATION  → Nmap port scan, service detection, OS fingerprinting
3. CVE MAPPING  → Match open services/versions to known CVEs
4. SEVERITY     → Score using CVSS v3.1, contextualise for environment
5. REMEDIATION  → Patch, harden, or mitigate each finding
6. REPORTING    → Structured report with executive summary + technical detail
```

---

## Nmap Scan Reference

### Full Enumeration Scan (used per host)
```bash
# Service version + OS detection + default scripts
nmap -sV -O -sC -p- --open <target_ip> -oN scan_output.txt

# Targeted aggressive scan on discovered ports
nmap -A -p <open_ports> <target_ip>

# UDP scan for additional services
nmap -sU --top-ports 100 <target_ip>
```

### Output Format Used
All scans saved in Normal (`-oN`), Grepable (`-oG`), and XML (`-oX`) format for cross-referencing.

---

## Sample CVE Findings

| CVE | Service | CVSS Score | Severity | Status |
|---|---|---|---|---|
| CVE-2017-0144 | SMBv1 (EternalBlue) | 9.8 | Critical | Patched in lab |
| CVE-2021-34527 | Windows Print Spooler | 8.8 | High | Service disabled |
| CVE-2019-0708 | RDP BlueKeep | 9.8 | Critical | Patch applied |
| CVE-2020-1472 | Netlogon (Zerologon) | 10.0 | Critical | Patch applied |
| CVE-2018-11776 | Apache Struts | 9.8 | Critical | Version updated |
| CVE-2021-44228 | Log4Shell | 10.0 | Critical | Config remediated |

---

## Sample Report Structure

Each host report follows this structure:

```
VULNERABILITY ASSESSMENT REPORT
================================
Target:         <hostname / IP>
Assessment Date: <date>
Assessor:       Varun Ramesh

EXECUTIVE SUMMARY
-----------------
X vulnerabilities found: Y Critical, Z High, N Medium

FINDINGS
--------
[CRITICAL] CVE-XXXX-XXXX — <Service Name>
  Description: ...
  CVSS Score: 9.8
  Affected Version: ...
  Proof of Concept: <nmap output / banner grab>
  Remediation: Patch to version X.X.X or disable service

REMEDIATION SUMMARY
-------------------
Priority 1 (Patch immediately): ...
Priority 2 (Patch within 30 days): ...
Priority 3 (Mitigate or accept risk): ...
```

---

## MITRE ATT&CK Mapping

| CVE | Technique | Tactic |
|---|---|---|
| EternalBlue (CVE-2017-0144) | T1210 Exploitation of Remote Services | Lateral Movement |
| BlueKeep (CVE-2019-0708) | T1210 Exploitation of Remote Services | Initial Access |
| Zerologon (CVE-2020-1472) | T1558 Steal or Forge Kerberos Tickets | Credential Access |
| Log4Shell (CVE-2021-44228) | T1190 Exploit Public-Facing Application | Initial Access |

---

## Files in This Repo

```
Vuln-Scan-Reports/
├── README.md                    ← This file
├── methodology.md               ← Full assessment methodology
├── reports/
│   ├── host-01-report.md
│   ├── host-02-report.md
│   └── ...
├── scans/
│   ├── host-01-nmap.txt
│   └── ...
├── cve-reference-table.md       ← All CVEs found, CVSS scores, remediation
└── mitre-mapping.md
```

---

## Tools Used

- **Nmap** — port scanning, service detection, OS fingerprinting
- **NVD / CVE Database** — CVE research and CVSS scoring
- **MITRE ATT&CK** — technique mapping
- **Kali Linux** — assessment environment

---

*Part of my cybersecurity portfolio. See also: [Network-Threat-Lab](../Network-Threat-Lab) · [SOC-Playbooks](../SOC-Playbooks)*
