# Mini Cybersecurity Homelab

A personal homelab built entirely on existing hardware: a MacBook running VMware Fusion Pro and a Windows laptop running VMware Workstation Pro. The environment is connected using Tailscale to work around NAT constraints and simulate a multi-network setup.

This lab is designed on a **student budget**, focusing on learning the full attack–detect cycle while developing and testing a custom monitoring stack.

---

## Goals

- Practice offensive techniques in a controlled environment (Kali, Metasploitable)
- Build and tune a detection stack (pfSense + Suricata, Network Monitoring Project)
- Deploy and attack a Windows Server 2019 Active Directory lab
- Understand the full lifecycle: reconnaissance → exploitation → detection → response

---

## Hardware

| Machine | Role |
|---|---|
| MacBook (VMware Fusion Pro) | pfSense, Kali Linux, Metasploitable, Monitoring Stack (Ubuntu) |
| Windows Laptop (VMware Workstation Pro) | Windows Server 2019 (Domain Controller), Windows 10 client |

> The two machines are connected via Tailscale, enabling cross-host communication without modifying local network infrastructure.

## Monitoring Stack
The detection layer runs [Network Monitoring System](https://github.com/ml448/network-monitoring-dash) — a custom FastAPI + InfluxDB + Grafana pipeline deployed via Docker Compose on the Ubuntu Server VM.

---

## Network
| Segment | Subnet | Purpose |
|---|---|---|
| LAN (vmnet2) | 192.168.94.0/24 | Kali, Monitoring VM |
| DMZ (vmnet3) | 192.168.95.0/24 | Metasploitable (isolated target) |
| Tailscale | 100.x.x.x | Cross-host connectivity |

---
## VMs
| VM | RAM | Network | Role |
|---|---|---|---|
| pfSense | 1GB | NAT, vmnet2, vmnet3 | Firewall, NAT, DHCP, Suricata IDS |
| Kali | 3.5GB | vmnet2 | Attacker |
| Ubuntu Server | 2GB | vmnet2, bridged | Monitoring stack |
| Metasploitable | 512MB | vmnet4 | Vulnerable target |

---

## Design Approach

- **Budget-conscious**: No dedicated servers or enterprise hardware  
- **Modular**: VMs run only when needed to conserve resources  
- **Realistic constraints**: NAT, segmentation, and limited compute mirror real-world environments  
- **Offense + Defense**: Attacks are used to validate and improve detection logic  

---

## Phases

1. Infrastructure setup  
2. Active Directory lab deployment  
3. Attack scenario development  
4. Defense & detection tuning  
5. Documentation and portfolio build-out  

---

## Status

🔧 In progress
