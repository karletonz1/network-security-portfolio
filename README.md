# Project North Star | Development Sandbox & Iteration
Active development repository in real-time with multiple document changes and technical design pivots as the North Star lab develops and improves over time.

For the final production release, see `north-star-prd.`

## Overview

This is a lab designed to demonstrate skills in infrastructure deployment, security best practices, endpoint monitoring, deployment and configuration of SIEM for centralized logging, backup and recovery, and vulnerability assessment. The lab is cross-platform, including Windows and Linux servers, clients, and virtual network devices all virtualized within GNS3.

**Key objectives:**
- Implement firewall, routing, VLAN segmentation, and DMZ isolation  
- Deploy endpoint monitoring and real-time detection using Wazuh  
- Centralize logs and build dashboards using Splunk Free  
- Perform vulnerability scanning with Nessus Essentials  
- Simulate attack scenarios using Kali Linux  
- Demonstrate backup and disaster recovery with Veeam

## Lab topology
Logical Diagram Overview

## Lab Devices & Applications

**Network & Edge**
- 2x VyOS Routers
- 2x OPNsense Firewalls
- 4x Arista vEOS Switches
- GNS3 Ethernet Switch

**Compute and Services**
- 3x Proxmox Servers (DC1/2, and IIS)
- Windows 10 Client
- Linux Client
- Veeam Server (Backups)

**Security and Monitoring**
- Kali Linux Security Node
- Wazuh
- Tenable
- Splunk

Automation
- Ansible Node

## Skills Demonstrated
:white_check_mark: Network infrastructure and security deployment (OPNsense, VyOS, vEOS)  
:white_check_mark: Endpoint installation and security monitoring (Windows, Linux, Wazuh)  
:white_check_mark: Central SIEM deployment and management (Splunk Free)  
:white_check_mark: Backup services implemented, and recovery demonstrated (Veeam Community Edition)  
:white_check_mark: Vulnerability assessment on endpoints (Nessus Essentials)  
:white_check_mark: Conduct Attack simulations & analyse logs (Kali Linux)  
:white_check_mark: Server provisioning, configuration and management (Windows 2022, Debian)  
:white_check_mark: Complete Lab Virtualization via GNS3  
:white_check_mark: GitHub repository storage and best practices (GitHub Web)  
:white_check_mark: Out-of-Band Management (OOBM) and Management VRF implementation (GNS3 Switch)  

| Skills demonstrated | Description |
|-----------|----------------|
| **Firewall, Routing, and Switching** | Firewall policies, VRRP, VLANs, OSPF, MLAG active-active redundancy, and LACP are done on OPNsense firewalls, VyOS routers, and Arista vEOS switches
| **Endpoint Monitoring** | Wazuh agents are installed on all Windows and Linux servers/clients and logs are forwarded to the central Splunk server
| **SIEM** | Splunk Free dashboards used for visualization
| **Backups** | Veeam Backup Server handles backups for Windows and Linux servers. A secondary repository is simulated using another VM within the lab 
| **Vulnerability Scanning** | Nessus Essentials is used for periodic scans of targeted endpoints. There is a hard limit of five IP addresses that is enforced in Nessus Essentials
| **Attack Simulation** | Kali Linux is used for penetration testing scenarios such as targeting DMZ and internal hosts to test the effectiveness of the security systems and monitoring used in the lab
| **Windows & Debian Servers** | Windows Server 2022 evaluation used; includes DNS, DHCP, IIS and Active Directory. Debian (DMZ Web server) and Rocky Linux (desktop / test clients) is used for cross-platform examples
| **VM Environment** | The entire lab is virtualized via GNS3
| **GitHub** | This is where the lab documentation is stored, which includes section guides, device configurations, and Ansible files used in the GNS3 lab. This includes implementing best practices around managing passwords using Ansible Vault.
| **OOBM Simulation** | The use of Ansible required the simulation of an OOBM network that would persist throughout the deployment phase.

## Lab Scenarios

| Scenario | Scenario Action | Success Criteria |
|-----------|----------------|--------------|
| **Simulated Attack** | Simulate attacks from Kali Linux either to a server or client | Blocked by the firewall, Wazuh detects, and the logs are sent to Splunk 
| **Endpoint Monitoring** | Conduct file changes or simulate failed login attempts | Detected by Wazuh and recorded in Splunk
| **Routing / Resilience** | Simulate link failure to test the MLAG resiliency | Verify packets are not dropped due to the use of redundant links
| **Backup & Recovery** | Delete a file on a client to simulate a lost critical file | Recover the file back to the client via Veeam restore  
| **Vulnerability Management** | Nessus scans servers and clients | Successful scan and capture of vulnerabilities 

## Repository Structure
**Portfolio structure**
- 01_Topology | Physical and logical topology diagrams
- 02_Servers_Clients | Windows Server 2022/Debian Linux server and virtual machine configs
- 03_Firewall_Router | OPNsense firewall policy and rules & VyOS router configurations
- 04_Security | Splunk dashboards, Wazuh configs, Kali scenario, Nessus configs
- 05_Backups | Veeam backup configurations and proofs
- 06_Scenarios | Lab scenario documentation and screenshots

## How to Explore
1. Start with the Topology  
   Open `01_Topology` to understand the lab layout.
   
2. Review Server & Client Configurations  
   `02_Servers_Clients`/ This contains configuration files and notes about the various servers and clients.
   
3. Review Firewall Policies & Router Configurations  
   `03_Firewall_Router`/ This contains screenshots of firewall policy rules as well as router configuration files.
   
4. Inspect Security Tools  
   `04_Security`/ This contains Splunk dashboards and Wazuh Agent deployment screenshots as well as Nessus reports.
   
5. Check Backup Configurations  
   `05_Backups`/ This contains screenshots of Veeam configuration and screenshots of successful backups.
   
6. Read Lab Scenarios  
   `06_Scenarios`/ This includes the scenarios as listed above. The scenario pages will follow a format as follows:  
   - What was the scenario  
   - What was done to simulate the scenario  
   - What was the result of the scenario  

## Project Evolution | A Journey of Discovery

### In the beginning (Phase 1)
The initial choices made for the lab revolved heavily around the resources available to deploy the lab in GNS3 with minimal cost. Some roadblocks were finding that some vendors required payment for using their official QCOW2 files needed for GNS3 appliances, and other hurdles were finding free alternatives but they did not have the full functionality needed to achieve the lab objectives. It was also important to be able to deploy this network by practicing automation, and Ansible was chosen for this purpose. A mix of manual configuration (bootstrap) was still required to get the initial networking up and running before fully configuring the network devices via Ansible Playbooks.

The phase 1 plan was to deploy the infrastructure network starting with the distribution and access layer switches, as well as the Ansible node (GNS3 Automation Network Node).

This phase had several challenges to overcome but it also had some valuable wins:
1.  I initially chose to configure the lab using Open vSwitches (OVS), but I discovered that Ansible was unable to speak to the OVS via SSH. I tried numerous troubleshooting steps like installing necessary dependencies and modular automation content on the Ansible node and also trying community 'fixed' versions of OVS. I found that no version of OVS I tried had the SSH connection plugin installed.
   
2.  A decision to pivot to REST-API was made and to use Extreme EXOS switches instead. However, I found that these switches were defaulting to HTTP despite configuring them to only use HTTPS, and I could not get Ansible to speak to the switches. A final pivot to use Arista vEOS switches was made and these switches successfully communicated with my Ansible node.  
- Numerous physical and logical topology and IP addressing schema changes were made during this phase to reflect the various options tested to find a switch the worked. These updates also allowed for improvements to the structure of my GitHub repository to reflect best practices and to also consider and implement an active-active architecture through the use of MLAGs.  

3. The Ansible host did not have all the prerequisites to allow for automation via Rest-API which required updating via the internet.  

4. The various pivots allowed me to refine the development of creating standardized Ansible directory structures, inventory management, and also allowed me to practice verification and troubleshooting commands.  

With the network switches confirmed, phase 2 focused on deploying the full switch configurations via automation and moving from a single-homed design to a dual-homed design using MLAG and LACP. 

### Into the Automation Unknown (Phase 2)
This phase was testing the concepts of Ansible automation on the Leaf switches first before expanding the configuration to the Spines.

Challenges and wins:
1. Since only a bootstrap configuration was done to the switches, single links were connected with no redundancy links due to MSTP running by default. This meant that as the playbook was run, the configuration was essentially cutting off Ansible's network access as it deployed the final configuration.  
- An OOBM switch was needed to solve this problem and to simulate enterprise environments where management and production networks are separated. Only one OOBM switch is used for simplicity in this lab, but it represented what would be done in environments like a data centre.
- A connection from Ansible was attached to the dedicated OOBM switch, and the bootstrap configurations needed to be updated to move the management IP address to the management port. Links from the OOBM switch to all network devices were run to their respective management ports and management traffic was segmented by using a dedicated VRF management instance.  
- Proof of concept was achieved after the removal of the pre-existing connections via data ports, and successfully running the playbooks using the OOBM connections via a GNS3 Ethernet switch. Multiple tests using the new lean bootstrap configuration was done.

2. Numerous changes to physical and logical topologies were done to reflect this new addition to the lab.
   
3. Blood, sweat, and tears were shed whilst trying to learn and create the playbooks needed to deploy the Arista leaf and spine switches. I encountered instances where unknown precedences were happening within the switches causing it to retain certain unwanted commands which broke the MLAG configuration. Ad-hoc solutions like adding 'no' commands within an Ansible task to remove the unwanted code, were needed to achieve 100% automated deployment up until the end of phase 2. The end of phase 2 indicates successful deployment of the full spine and leaf configurations, confirmed MLAG peer status, and port-channel active status from the leafs to their respective spines, all through the use of Ansible.

## Contact
- GitHub: https://github.com/karletonz1
- LinkedIn: https://www.linkedin.com/in/karloc
