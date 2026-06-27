# Vulnerability Assessment Report — scanme.nmap.org

**Assessor:** Varun Ramesh  
**Date:** 2026-06-27  
**Tool:** Nmap 7.98  
**Target:** scanme.nmap.org (45.33.32.156)  
**Scope:** Authorised public target — nmap.org provides this host explicitly for scanning practice  
**Command Used:**
```
nmap -sV -O -sC --open scanme.nmap.org -oN scan_scanme.txt
```

---

## Executive Summary

4 open TCP ports identified on target host `45.33.32.156`. The most significant finding is **OpenSSH 6.6.1p1** — an outdated SSH version with multiple known CVEs, including remote code execution and information disclosure vulnerabilities. The Apache HTTP server version (2.4.7) is similarly outdated and carries critical CVEs. Both services represent meaningful attack surface on a production host.

| Severity | Count |
|---|---|
| Critical | 1 |
| High | 2 |
| Medium | 1 |
| Informational | 1 |

---

## Host Information

| Field | Value |
|---|---|
| IP Address | 45.33.32.156 |
| Hostname | scanme.nmap.org |
| OS (detected) | Linux 4.19 – 5.15 (97% confidence) |
| Network Distance | 15 hops |
| Scan Duration | 34.61 seconds |
| Open Ports Found | 4 (22, 80, 9929, 31337) |

---

## Findings

---

### [CRITICAL] CVE-2016-0777 / CVE-2016-0778 — OpenSSH Information Leak & Buffer Overflow

**Port:** 22/tcp  
**Service:** OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13  
**CVSS Score:** 8.0 (High) / escalated to Critical in context due to version age and exposure  

**Description:**  
The detected OpenSSH version (6.6.1p1) is significantly outdated. OpenSSH versions before 7.1p2 are vulnerable to CVE-2016-0777, which allows a malicious SSH server to trigger an information leak of private keys from a connecting client via the roaming feature. CVE-2016-0778 is a related buffer overflow in the same roaming code.

Current OpenSSH stable: **9.x**. This version is approximately **12 major releases behind.**

**Evidence from scan:**
```
22/tcp open ssh OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 ac:00:a0:1a:82:ff:cc:55:99:dc:67:2b:34:97:6b:75 (DSA)
|   2048 20:3d:2d:44:62:2a:b0:5a:9d:b5:b3:05:14:c2:a6:b2 (RSA)
```

**Additional concern:** DSA 1024-bit key detected. DSA-1024 is considered cryptographically weak by modern standards and should not be in use.

**MITRE ATT&CK Mapping:**  
- T1110 — Brute Force (SSH exposed to internet)  
- T1552.004 — Unsecured Credentials: Private Keys  

**Remediation:**  
1. Upgrade OpenSSH to current stable (9.x minimum)  
2. Disable DSA host keys; use ED25519 or RSA-4096 only  
3. Restrict SSH access by IP allowlist or move behind VPN  
4. Enforce key-based authentication only; disable password auth  

---

### [HIGH] CVE-2017-7679 / CVE-2017-7668 — Apache HTTP Server 2.4.7 Multiple Vulnerabilities

**Port:** 80/tcp  
**Service:** Apache httpd 2.4.7 (Ubuntu)  
**CVSS Score:** 9.8 (Critical) for CVE-2017-7679  

**Description:**  
Apache 2.4.7 is severely outdated. Current stable is Apache 2.4.62+. This version is affected by numerous CVEs including:

| CVE | Description | CVSS |
|---|---|---|
| CVE-2017-7679 | mod_mime buffer overread — remote code execution possible | 9.8 |
| CVE-2017-7668 | ap_find_token buffer overread — DoS / info leak | 9.8 |
| CVE-2017-3169 | mod_ssl null pointer dereference | 7.5 |
| CVE-2017-3167 | Authentication bypass via third-party modules | 9.1 |

**Evidence from scan:**
```
80/tcp open http Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Go ahead and ScanMe!
```

**Note:** Server version is disclosed in HTTP headers (`Server: Apache/2.4.7`). Version disclosure alone is an informational finding — attackers can directly target known CVEs for this exact version.

**MITRE ATT&CK Mapping:**  
- T1190 — Exploit Public-Facing Application  
- T1082 — System Information Discovery (version disclosure)  

**Remediation:**  
1. Upgrade Apache to 2.4.62 or latest stable immediately  
2. Suppress version disclosure: set `ServerTokens Prod` and `ServerSignature Off` in Apache config  
3. Review and disable unused modules (mod_mime, mod_ssl if not needed)  

---

### [MEDIUM] Port 9929 — Nping Echo Service Exposed

**Port:** 9929/tcp  
**Service:** nping-echo  
**CVSS Score:** 5.3  

**Description:**  
Nping echo service is running and publicly accessible. While not directly exploitable in most scenarios, an exposed echo/probe service can be used for network reconnaissance, amplification in DDoS scenarios, and reveals additional information about the host's network tooling.

**MITRE ATT&CK Mapping:**  
- T1046 — Network Service Discovery  

**Remediation:**  
Restrict port 9929 to localhost or remove from public-facing interface if not intentionally exposed.

---

### [INFORMATIONAL] Port 31337 — tcpwrapped

**Port:** 31337/tcp  
**Service:** tcpwrapped  

**Description:**  
Port 31337 is open but the service immediately terminates the connection (tcpwrapped). The port number itself is notable — 31337 ("eleet") is historically associated with backdoors and hacking tools (Back Orifice, etc.). On this host it is intentional (part of the scanme setup), but on any other host this port open would be an immediate escalation trigger.

**MITRE ATT&CK Mapping:**  
- T1049 — System Network Connections Discovery  

**Remediation:**  
Flag for investigation on any non-intentional host. On this target: informational only.

---

## OS Fingerprint Analysis

Nmap OS detection returned:

```
Aggressive OS guesses: Linux 4.19 - 5.15 (97%), Linux 4.15 (93%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

**Assessment:** Host is almost certainly running Linux kernel in the 4.x–5.x range. Combined with Ubuntu-tagged service banners, this is likely Ubuntu 14.04 LTS (Trusty) — which reached end-of-life in April 2019 and no longer receives security patches.

An EOL Ubuntu install running EOL versions of SSH and Apache represents compounding risk — vulnerabilities discovered after EOL will never be patched by the vendor.

---

## MITRE ATT&CK Summary

| Technique | ID | Relevant Finding |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Apache 2.4.7 RCE CVEs |
| Brute Force | T1110 | SSH exposed on port 22 |
| Unsecured Credentials: Private Keys | T1552.004 | OpenSSH CVE-2016-0777 |
| Network Service Discovery | T1046 | Port 9929 nping-echo exposed |
| System Information Discovery | T1082 | Apache version disclosure in headers |

---

## Remediation Priority

| Priority | Action | Timeframe |
|---|---|---|
| 1 — Immediate | Upgrade OpenSSH to 9.x | Within 24 hours |
| 1 — Immediate | Upgrade Apache to 2.4.62+ | Within 24 hours |
| 2 — Short term | Suppress Apache version disclosure | Within 7 days |
| 2 — Short term | Restrict SSH to key-based auth only | Within 7 days |
| 3 — Medium term | Restrict port 9929 to localhost | Within 30 days |
| 3 — Medium term | Evaluate OS upgrade from EOL Ubuntu | Within 30 days |

---

## Raw Scan Output

See: [`../scans/scan_scanme.txt`](../scans/scan_scanme.txt)

---

*Assessment conducted on scanme.nmap.org — a host provided by the Nmap Project for authorised scanning practice. See https://nmap.org/book/legal-issues.html*
