# Network-Security-Portfolio
Hands-on network and security lab portfolio fully run in GNS3

## Overview

This is a network and security lab designed to demonstrate skills in infrastructure security, endpoint monitoring, SIEM correlation, backup and recovery, and vulnerability assessment. The lab is cross-platform, including Windows and Linux servers, clients, and virtual network devices all virtualized within GNS3.

**Key objectives:**
- Implement firewall, routing, VLAN segmentation, and DMZ isolation  
- Deploy endpoint monitoring and real-time detection using Wazuh  
- Centralize logs and build dashboards using Splunk Free  
- Perform vulnerability scanning with Nessus Essentials  
- Simulate attack scenarios using Kali Linux  
- Demonstrate backup and disaster recovery with Veeam

## Lab topology
Logical Diagram Overview

## Components & Tools

| Component | Role / Purpose |
|-----------|----------------|
| **OPNsense Firewall** | Network edge security, NAT, ACLs, VPN |
| **VyOS Router** | VLAN routing, OSPF |
| **Windows Server DC1/DC2** | Active Directory, DNS, DHCP, domain redundancy |
| **Windows & Linux Clients** | Cross-platform endpoint monitoring (Wazuh agents installed) |
| **DMZ Servers (Debian + IIS)** | Web services in DMZ, monitored endpoints |
| **Kali Linux VM** | Attack simulation & penetration testing |
| **Splunk Free** | Centralized SIEM dashboards and log correlation |
| **Wazuh Server + Agents** | Endpoint detection and alerting |
| **Veeam Backup Server** | Backup and disaster recovery with secondary repository |
| **Nessus VM** | Vulnerability scanning of internal and DMZ servers |

## Lab Scenarios

1. **Firewall / DMZ Isolation**
   - Simulate attacks from Kali Linux → blocked by OPNsense → logged in Splunk/Wazuh

2. **Endpoint Monitoring**
   - File changes, failed logins, rootkits detected via Wazuh → correlated in Splunk dashboards

3. **Routing / Resilience**
   - OSPF route changes; simulate link failure → verify traffic reroutes correctly

4. **Backup & Recovery**
   - Delete a critical file → restore via Veeam backup  
   - Demonstrate secondary repository copy

5. **Vulnerability Management**
   - Nessus scans DMZ and internal servers → highlight vulnerabilities

6. **Attack Simulation**
   - Kali attacks → firewall blocks, Wazuh detects, Splunk correlates

## Skills Demonstrated
- Network security & routing (OPNsense, VyOS, VLANs, OSPF)  
- Endpoint security (Wazuh agent deployment, cross-platform monitoring)  
- SIEM dashboards and log correlation (Splunk Free)  
- Backup and disaster recovery procedures (Veeam Community Edition)  
- Vulnerability assessment & proactive risk management (Nessus Essentials)  
- Attack simulation & SOC-style analysis (Kali Linux)

## Repository Structure
**Portfolio structure**

- 01_Topology | Network and physical topology diagrams
- 02_Servers_Clients | Windows Server 2022/Debian Linux server and virtual machine configs, Windows 10 & Rocky Linux client configs
- 03_Firewall_Router | OPNsense firewall policy and rules & VyOS router configurations.
- 04_Security | Splunk dashboards, Wazuh configs, Kali scenario, Nessus configs
- 05_Backups | Veeam backup configurations and proofs.
- 06_Scenarios | Lab scenario documentation and screenshots

## Considerations
- **VM Environment:** The entire lab is virtualized via GNS3. Due to difficulties in licensing virtual switches and routers, a compromise was decided when selecting appropriate network options needed to simulate the lab environment.
- **Windows & Debian Servers:** Windows Server 2022 evaluation used; includes DNS, DHCP, IIS and Active Directory. Debian (DMZ Web server) and Rocky Linux (desktop / test clients) used for cross-platform examples.
- **SIEM:** Splunk Free dashboards used for visualization; alerting simulated manually.
- **Endpoint Monitoring:** Wazuh agents installed on all Windows and Linux servers/clients; logs forwarded to central Splunk server.
- **Vulnerability Scanning:** Nessus Essentials VM used for periodic scans. A hard limit of five IP addresses is enforced using Nessus Essentials.
- **Backups:** Veeam Backup Server handles backups for Windows and Linux servers; secondary repository simulated to another VM within the lab.
- **Firewall, Routing, and Switching:** OPNsense is the chosen firewall appliance which handles NAT, ACLs, VPN etc; VyOS router handles l3 funtions such as VLANs and OSPF routing etc. Native l2 switch within GN3 was chosen but this limits the ability to configure LACP between the two distribution switches. A configuration document will instead be used to reference what would have been configured if l3 switches were used.
- **Attack Simulation:** Kali Linux VM used for penetration testing scenarios targeting DMZ, internal hosts, using common attack methods to test the effectiveness of the security systems implemented.
- **Topology Notes:** DMZ isolated, firewall at network edge; internal VLANs routed via VyOS; VPN simulated from an external client to DMZ server.

## How to Explore
1. Start with the Topology - Open `01_Topology` to understand the network layout.
2. Review Server & Client Configurations - `02_Servers_Clients`/ contain screenshots, configuration files, and notes.
3. Review Firewall Policies & Router Configurations - `03_Firewall_Router`/ contain screenshots, configuration files, and policy rules.
4. Inspect Security Tools - `04_Security`/ has Splunk dashboards, Wazuh agent configs, Kali scenarios, and Nessus reports.
5. Check Backup Configurations - `05_Backups`/ contains Veeam backup jobs, secondary repository simulation, and recovery examples.
6. Read Lab Scenarios - `06_Scenarios`/ includes step-by-step use cases demonstrating attacks, monitoring, and behaviours that were observed as well as remediation (if required).

## Contact
- GitHub: https://github.com/karletonz1
- LinkedIn: https://www.linkedin.com/in/karloc
