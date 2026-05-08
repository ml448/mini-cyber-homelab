# Homelab Setup Notes

## Infrastructure (Week 1)
Created multiple isolated networks:
- vmnet2: 192.168.94.0/24 (LAN - Kali, Monitoring VM)
- vmnet4: 192.168.95.0/24 (DMZ - Metasploitable)
- pfSense: 3 NICs (NAT/WAN, vmnet2/LAN, vmnet3/DMZ)
- Moved Metasploitable to DMZ so traffic can route through pfSense

## VMs
- pfSense: 1GB RAM, 1 core, FreeBSD, legacy BIOS
- Kali: 3.5GB RAM, vmnet2 only
- Ubuntu Server (Monitoring VM): 2GB RAM, vmnet2 + bridged
- Metasploitable 2: 512MB RAM, vmnet4 only

## Issues & Fixes
- pfSense webConfigurator kept crashing: bumped RAM 512MB -> 1GB
- Ubuntu Server DNS dropping: set static DNS in systemd-resolved
- Kali-to-Metasploitable traffic bypassed pfSense: created DMZ segment
- filterlog hostname parsing: pfSense omits hostname, fixed syslog parser
- filterlog CSV unreadable: added parse_filterlog() for human-readable output

## Validated
- Nmap scan from Kali -> Metasploitable routes through pfSense
- pfSense forwards syslog to Monitoring VM's port
- Grafana syslog dashboard shows filterlog entries
- 23 open ports detected on Metasploitable
