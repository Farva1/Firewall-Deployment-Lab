# Firewall Deployment Lab (pfSense + Ubuntu LAN/DMZ)

**Author:** Umme Farva  
**Date:** 2025-11-24  

## Overview
This project demonstrates a complete firewall deployment using **pfSense** inside a virtualized environment.  
It simulates a small enterprise network with three segments:

- **WAN** – VirtualBox NAT (pfSense obtains: 10.0.2.15)
- **LAN** – Internal secure network (192.168.10.0/24)
- **DMZ** – Public-facing network (192.168.20.0/24)

The goal is to enforce real-world security controls including:
- LAN → Internet/DMZ allowed  
- DMZ → LAN blocked (segmentation)  
- WAN → DMZ port forward (8080 → 80)  
- Capturing and analyzing firewall logs  
- Testing segmentation using `ping`, `curl`, and `nmap`

---

## Network Architecture
![Network Topology](./diagrams/network-topology.png)

### **Interface IP Summary**
| Interface | Network | IP Address | Purpose |
|----------|----------|------------|---------|
| WAN (em0) | VirtualBox NAT | **10.0.2.15** | Internet access + external NAT |
| LAN (em1) | Internal Network | **192.168.10.1/24** | Workstation network |
| DMZ (em2) | Internal Network | **192.168.20.1/24** | Public-facing server |

---

## Setup Summary

### **pfSense Configuration**
- Installed on VirtualBox with 3 NICs
- **LAN IP:** 192.168.10.1/24  
- **DMZ IP:** 192.168.20.1/24  
- **WAN IP (DHCP/NAT):** 10.0.2.15  
- **LAN DHCP:** 192.168.10.100–200  
- **DMZ DHCP:** 192.168.20.100–200  

### **DMZ Web Server (Ubuntu Server)**
- Receives IP via DHCP: `192.168.20.x`  
- Serves webpage using:

python3 -m http.server 80

markdown
Copy code

### **Port Forward**
WAN port **8080** → DMZ port **80**  
Used to expose the DMZ web server externally.

---

## Key Artifacts Included in This Repository

- **/configs/pfsense-backup.xml**  
  Full pfSense configuration backup.

- **/screenshots/**  
  Contains all required screenshots:
  - Dashboard  
  - LAN/DMZ interfaces  
  - DHCP pages  
  - Firewall rules  
  - NAT rule  
  - Logs (DMZ→LAN blocks)  
  - Python server  
  - Browser access  
  - nmap & curl results  

- **/notes/setup-steps.md**  
  Contains installation, interface setup, DHCP, firewall rule configuration steps.

- **/notes/testing-notes.md**  
  Contains all validation and segmentation test output.

- **/extras/nmap-results.txt**  
  WAN port-scanning results:
  - Command used:
    ```
    nmap -p 8080 10.0.2.15
    ```
  - Port 8080 = OPEN → forwarded to DMZ server.

---

## What I Configured

###  Network Segmentation  
- LAN network isolated from DMZ  
- DMZ isolated from LAN (zero trust)

### Firewall Rules  
- **Allow LAN → ANY**
- **Block DMZ → LAN**
- **Allow DMZ → Internet**
- **WAN → DMZ port forward (8080→80)**

### NAT  
- Configured using port forwarding  
- pfSense auto-created matching WAN firewall rule

### DHCP  
- Separate DHCP scopes for LAN & DMZ  
- Ensures proper client IP allocation

### Logging & Monitoring  
- Verified DMZ→LAN blocked packets in pfSense logs  
- Captured screenshots for project evidence

---

## Validation Tests Performed

### 1. LAN → DMZ Access (Allowed)
curl http://192.168.20.x
Result: "Hello from DMZ"

shell
Copy code

### 2️. DMZ → LAN Access (Blocked)
ping 192.168.10.x → 100% packet loss
curl http://192.168.10.x → blocked

shell
Copy code

### 3️ WAN → DMZ Access (Port Forward)
umme-farva@ubuntu-lan:~$ nmap -p 8080 10.0.2.15
PORT STATE SERVICE
8080/tcp open http-proxy

kotlin
Copy code

Browser external access:
http://10.0.2.15:8080 → "Hello from DMZ"

yaml
Copy code

---

## Lessons Learned

- Firewall **rule ordering is critical** (block rules must be placed above allow rules).  
- pfSense’s segmentation effectively prevents DMZ lateral movement into LAN.  
- NAT + port forwarding works seamlessly with VirtualBox NAT.  
- Packet capture and logs are essential to validate real firewall behavior.  
- pfSense backups enable quick re-deployment.

---

## Future Enhancements

- Add **Suricata IDS/IPS** on WAN or DMZ  
- Forward pfSense logs to **Splunk or Microsoft Sentinel**  
- Use HTTPS with Let's Encrypt for the DMZ server  
- Add VLANs instead of multiple NICs for scalability  

---

## Author
**Umme Farva**  
Cybersecurity Analyst  