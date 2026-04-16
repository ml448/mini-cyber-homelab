# small-homelab

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
| MacBook (VMware Fusion Pro) | pfSense, Kali Linux, Metasploitable, Monitoring (Ubuntu) |
| Windows Laptop (VMware Workstation Pro) | Windows Server 2019 (Domain Controller), Windows 10 client |

> The two machines are connected via Tailscale, enabling cross-host communication without modifying local network infrastructure.

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
