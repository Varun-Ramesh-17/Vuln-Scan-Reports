# Aggressive Scan Analysis — scanme.nmap.org

**Assessor:** Varun Ramesh  
**Date:** 2026-06-27  
**Tool:** Nmap 7.98 (`-A` aggressive scan)  
**Target:** scanme.nmap.org (45.33.32.156)  
**Command Used:**
```
nmap -A scanme.nmap.org -oN scan_scanme_aggressive.txt
```

---

## Comparison with Previous Scan

| Finding | Scan 1 (`-sV -O -sC`) | Scan 2 (`-A`) |
|---|---|---|
| Port 22 SSH | ✓ OpenSSH 6.6.1p1 | ✓ Confirmed |
| Port 80 HTTP | ✓ Apache 2.4.7 | ✓ Confirmed |
| Port 9929 | ✓ nping-echo | ✓ Confirmed |
| Port 31337 | ✓ tcpwrapped | ✓ Confirmed |
| Port 25 SMTP | Not detected | ✓ **Filtered** — new finding |
| Traceroute | Not captured | ✓ **15 hops mapped** — new finding |

---

## New Findings — Aggressive Scan

### Port 25 (SMTP) — Filtered

The aggressive scan revealed port 25 (SMTP) in a **filtered** state — meaning a firewall is actively blocking the port rather than the service simply not running.

**What filtered means vs closed:**
- **Closed** — port is accessible, no service listening. Nmap gets a RST response.
- **Filtered** — a firewall/ACL is dropping or rejecting packets. Nmap gets no response or ICMP unreachable.

**Why this matters:**
A filtered SMTP port tells us the host has firewall rules in place — but the filter itself is detectable. An attacker learns: firewall exists, mail service may be present behind it, and the filtering behaviour reveals packet handling logic.

**MITRE ATT&CK:** T1046 — Network Service Discovery

---

### Traceroute — 15 Hops to Target

The aggressive scan captured the full network path:

```
HOP  RTT        ADDRESS
1    0.63 ms    172.23.192.1          ← Local WSL gateway
2    1.78 ms    192.168.1.1           ← Home router
4    6.66 ms    keralavisionisp (103.139.65.5)  ← ISP (Kerala Vision)
7    81.67 ms   Telstra Global (210.57.30.86)   ← International backbone
8    71.10 ms   Telstra Global (202.84.207.181)
9    236.52 ms  Equinix core (202.84.141.153)   ← Equinix data centre
15   235.71 ms  scanme.nmap.org (45.33.32.156)  ← Target (US)
```

**Analysis:**
- Total latency: ~235ms (India → US West Coast)
- Traffic routes: Kerala Vision ISP → Telstra Global backbone → Equinix (likely Dallas or Fremont) → Linode/Akamai (scanme.nmap.org is hosted on Linode)
- Hops 3, 5-6, 11-14 did not respond (ICMP TTL exceeded filtered) — common on enterprise backbone routers

**Why traceroute matters in pentesting:**
- Reveals ISP and backbone providers — useful for understanding traffic interception risk
- Identifies filtering points (non-responding hops) — potential security controls
- Confirms geographic routing — relevant for data sovereignty assessments

**MITRE ATT&CK:** T1590.004 — Gather Victim Network Information: Network Topology

---

## Confirmed Attack Surface Summary

After both scans, the confirmed external attack surface for this host is:

| Port | Service | Version | Risk |
|---|---|---|---|
| 22/tcp | SSH | OpenSSH 6.6.1p1 | 🔴 Critical — outdated, CVE-exposed |
| 80/tcp | HTTP | Apache 2.4.7 | 🔴 Critical — outdated, RCE CVEs |
| 25/tcp | SMTP | Unknown (filtered) | 🟡 Medium — firewall present, service unknown |
| 9929/tcp | nping-echo | Nping | 🟡 Medium — unnecessary exposure |
| 31337/tcp | Unknown | tcpwrapped | 🟢 Low — connection terminated immediately |

---

## Key Takeaway — Methodology Note

Running two scan types on the same target revealed different information:

- `-sV -O -sC` — better for service fingerprinting and script-based detection
- `-A` — better for OS detection, traceroute, and catching filtered ports

**Best practice:** Always run multiple scan profiles against a target. A single scan type will miss findings.

---

## Raw Scan Output

See: [`../scans/scan_scanme_aggressive.txt`](../scans/scan_scanme_aggressive.txt)

---

*Both scans conducted on scanme.nmap.org — authorised public scanning target provided by the Nmap Project.*
