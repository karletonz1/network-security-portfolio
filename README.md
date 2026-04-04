# Project North Star | Development Sandbox & Iteration
Active development repository containing multiple changes, troubleshooting steps, and technical design pivots as the North Star lab develops and improves over time.

For the final production release, see `north-star-prd.`

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
| **Arista vEOS Switch** | MLAG, Spine and Leaf redundancy |
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
- **Firewall, Routing, and Switching:** OPNsense is the chosen firewall appliance which handles NAT, ACLs, VPN etc; VyOS router handles l3 funtions such as OSFP routing and VRRP etc. Arista vEOS switches provides an impressive amount of real world capabilities and has been chosen to fill the spine and leaf roles in the lab. A main advantage of this switch is the ability to implement MLAG for true active-active redundancy.
- **Attack Simulation:** Kali Linux VM used for penetration testing scenarios targeting DMZ, internal hosts, using common attack methods to test the effectiveness of the security systems implemented.
- **Topology Notes:** DMZ isolated, firewall at network edge; internal VLANs routed via VyOS; VPN simulated from an external client to DMZ server.

## How to Explore
1. Start with the Topology - Open `01_Topology` to understand the network layout.
2. Review Server & Client Configurations - `02_Servers_Clients`/ contain screenshots, configuration files, and notes.
3. Review Firewall Policies & Router Configurations - `03_Firewall_Router`/ contain screenshots, configuration files, and policy rules.
4. Inspect Security Tools - `04_Security`/ has Splunk dashboards, Wazuh agent configs, Kali scenarios, and Nessus reports.
5. Check Backup Configurations - `05_Backups`/ contains Veeam backup jobs, secondary repository simulation, and recovery examples.
6. Read Lab Scenarios - `06_Scenarios`/ includes step-by-step use cases demonstrating attacks, monitoring, and behaviours that were observed as well as remediation (if required).

## Project Evolution | A Journey of Discovery

### In the beginning (Phase 1)
The initial choices made for the lab revolved heavily around the resources available to deploy the lab in GNS3 with minimal cost. Some roadblocks included some vendors requiring payment for the use of official QCOW2 files needed for GNS3 appliances and other hurdles were free options but did not have the full functionality needed for the lab. It was also important to be able to deploy this network by practicing automation, and Ansible was chosen for this purpose. A mix of manual configuration (bootstrap) was still required to get the initial networking up and running before fully configuring the network devices via Ansible Playbooks.

The phase 1 plan was to deploy the infrastructure network, which included switches, routers, FWs and an Ansible host. The following were chosen:  
- Open vSwitches
- VyOS Routers
- GNS3 Network Automation (Ansible host)
- OPNsense Firewall

This phase had several challenges to overcome but it also had some valuable wins:
1.  I chose to configure the Open vSwitches first to simulate the Ansible node communication through a real network, but it was discovered that Ansible was unable to speak to these devices via SSH. Several troubleshooting steps were done including trying community 'fixed' versions of OVS. I found that no version of OVS I tried had the SSH module installed.  
2.  A decision to pivot to Rest-API was made and to use Extreme Exos switches instead. However, it was discovered that these switches were defaulting to HTTP despite troubleshooting, and I could not get Ansible to speak to the switches. A final pivot to use Arista vEOS switches was made and these switches succesfully communicated with my Ansible node.  
- Numerous topology and IP table changes were made during this phase to accomodate the pivots needed to find applications that would work in GNS3. These updates also allowed for improvements to the structure of my Github repository to reflect best practices and showcase security considerations in the initial design.  
3. The Ansible host did not have all the prerequisites to allow for automation via Rest-API which required updating via the internet.
4. The various pivots allowed me to refine the development of creating standardized Ansible directory structures, inventory management, and also allowed me to practice verification and troubleshooting commands.  

With the managment network stabilized, phase 2 focuses on deploying the full switch configurations via automation and moving from a single-homed design to a dual-home design using MLAG and LACP. 

### Into the Automation Unknown (Phase 2)
This phase was testing the concepts of Ansible automation on the Leaf switches first before expanding the configuring to the Spines.

Challenges and wins:
1. Since only a bootstrap configuration was done to the switches, single links were connected with no redundancy links to due MSTP running on default. This meant that as the playbook was run, the configuration was essentially cutting off Ansible's network access as it deployed the final configuration.  
- An OOBM switch was needed to simulate enterprise environments where management and production networks are separated. Only one OOBM switch is used for simplicity in this lab, but it represents what would be done in environments like data centres for example.
- The connection eth12 from Ansible to leaf-01 had to be moved to a dedicated OOBM switch, and the bootstrap configurations needed to be updated to move the management IP addresses to the management port. Links from the OOBM switch to all network devices were run to their respective management ports and segmented the management traffic by using a dedicated VRF management instance.  
- Proof of concept was achieed after the removal of the pre-existing connections via data ports, and successfully running the playbooks using the OOBM connections via a GNS3 Ethernet switch. Multiple tests using the new lean bootstrap configuration was done.
2. Numerous changes to physical and logical topologies were done to reflect this new addition to the lab.
3. Blood, sweat, and tears were shed whilst trying to create the playbooks needed to deploy the leaf and spine switches. I encountered instances where unknown precedences were happening within the switches causing it to retain certain unwanted commands which broke the MLAG configuration. Ad-hoc solutions like adding 'no' commands within an ansible task to remove the unwanted code, were needed to achieve 100% automated deployment up until the end of phase 2. The end of phase 2 indicates successful deployment of the full spine and leaf configurations, confirmed MLAG peer status, and port-channel active status from the leafs to their respective spines, all through the use of Ansible.

## Contact
- GitHub: https://github.com/karletonz1
- LinkedIn: https://www.linkedin.com/in/karloc
