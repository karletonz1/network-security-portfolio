# VyOS Core Router Deployment Guide

This section documents the deployment and automation of the Core Routing layer for the North Star Infrastructure. These routers handle the L3 boundary between the internal distribution layer and the security perimeter. 

### 1. Prerequisites & Node Specification

Image: VyOS 1.5-rolling (vyos-2026.03.27-0030-rolling-generic-amd64)

Resources (Per Node):

  - RAM: 2048 MB

  - vCPUs: 4

  - Qemu Binary: x86_64 (v8.0.4)
    
[Additional information of the devices can be found here](https://github.com/karletonz1/karlo-cn-ent-lab/blob/main/03-network-devices/vyos-router/ansible/hosts)

Management Access: SSH enabled with dedicated ansible service account.

### 2. Physical Topology

### Interface Mapping

<img width="556" height="170" alt="image" src="https://github.com/user-attachments/assets/e9377e3f-2453-46b7-834e-978f9d6fd589" />  



| Device A Device | Device A Hostname | Device A port | Device B Device | Device B Hostname | Device B Port |
|-----------------|-------------------|---------------|-----------------|-------------------|---------------|
| VyOS Core router 01 | karlo-cn-rtr-01 | eth0 | VyOS Core router 02 | karlo-cn-rtr-02 | eth0
| VyOS Core router 01 | karlo-cn-rtr-01 | eth1 | OVS Distribution switch 01 | karlo-cn-ds-01 | eth1
| VyOS Core router 01 | karlo-cn-rtr-01 | eth2 | OVS Distribution switch 01 | karlo-cn-ds-01 | eth2


| Device A Device | Device A Hostname | Device A port | Device B Device | Device B Hostname | Device B Port |
|-----------------|-------------------|---------------|-----------------|-------------------|---------------|
| VyOS Core router 02 | karlo-cn-rtr-02 | eth0 | VyOS Core router 01 | karlo-cn-rtr-01 | eth0
| VyOS Core router 02 | karlo-cn-rtr-02 | eth1 | OVS Distribution switch 02 | karlo-cn-ds-02 | eth1
| VyOS Core router 02 | karlo-cn-rtr-02 | eth2 | OVS Distribution switch 02 | karlo-cn-ds-02 | eth2

### 3. Logical Topology
<img width="925" height="336" alt="image" src="https://github.com/user-attachments/assets/cf55f0cd-82b0-4f55-9f14-db4f79c4c323" />


### IP Address Allocation (Core Router Layer-VyOS)
| Hostname | GNS3 Port | IP Address | Vlan ID | Subnet | Gateway (VIP) | MTU | Role | Link type| 
|----------|------|------------|---------|--------|---------|-----|------|----------|
| karlo-cn-rtr-01| eth0 | 10.0.70.1 | 70 | 10.0.70.0/30 | PTP | 1500 | Dedicated HA link | Trunk
| karlo-cn-rtr-01| eth1/2:Bond0.10 | 10.0.10.1 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Gateway for Infra-MGT Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.11 | 10.0.11.1 | 11 | 10.0.11.0/24 | 10.0.11.254 | 1500 | Gateway for SVR-MGT Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.20 | 10.0.20.1 | 20 | 10.0.20.0/24 | 10.0.20.254 | 1500 | Gateway for WIN-CLIENTS Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.21 | 10.0.21.1 | 21 | 10.0.21.0/24 | 10.0.21.254 | 1500 | Gateway for LIN-CLIENTS Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.30 | 10.0.30.1 | 30 | 10.0.30.0/24 | 10.0.30.254 | 1500 | Gateway for SEC-APPS Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.40 | 10.0.40.1 | 40 | 10.0.40.0/24 | 10.0.40.254 | 1500 | Gateway for DMZ Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.50 | 10.0.50.1 | 50 | 10.0.50.0/24 | 10.0.50.254 | 1500 | Gateway for PRD-SVR Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.60 | 10.0.60.1 | 60 | 10.0.60.0/24 | 10.0.60.254 | 1500 | Gateway for BACKUPS Subnet| Sub-interface
| karlo-cn-rtr-02| eth0 | 10.0.70.2 | 70 | 10.0.70.0/30 | PTP | 1500 | Dedicated HA link  | Trunk
| karlo-cn-rtr-02| eth1/2:Bond0.10 | 10.0.10.2| 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Gateway for Infra-MGT Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.11 | 10.0.11.2| 11 | 10.0.11.0/24 | 10.0.11.254 | 1500 | Gateway for SVR-MGT Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.20 | 10.0.20.2| 20 | 10.0.20.0/24 | 10.0.20.254 | 1500 | Gateway for WIN-CLIENTS Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.21 | 10.0.21.2| 21 | 10.0.21.0/24 | 10.0.21.254 | 1500 | Gateway for LIN-CLIENTS Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.30 | 10.0.30.2| 30 | 10.0.30.0/24 | 10.0.30.254 | 1500 | Gateway for SEC-APPS Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.40 | 10.0.40.2| 40 | 10.0.40.0/24 | 10.0.40.254 | 1500 | Gateway for DMZ Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.50 | 10.0.50.2| 50 | 10.0.50.0/24 | 10.0.50.254 | 1500 | Gateway for PRD-SVR Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.60 | 10.0.60.2| 60 | 10.0.60.0/24 | 10.0.60.254 | 1500 | Gateway for BACKUPS Subnet| Sub-interface






### 4. High Availability & Routing Logic

  Gateway Redundancy:  
  This lab uses VRRP at the core routers via sub-interfaces.

  Link Aggregation:  
  The Router level utilizes LACP between karlo-cn-rtr-01/02 and karlo-cn-ds-01/02 using eth1 and eth2 for the LAG link.

  Dynamic Routing:  
  OSPF is used in this lab for dynamic learning and distribution of routes.

  High Availablity:  
  A HA heartbeak link is configured between the core routers via a /30 subnet.


### 5. Automation Workflow

Step 1: Manual Bootstrap: Minimum configuration required via CLI to allow Ansible to reach the devices before pushing the remaining configuration via automation. Since it is a ROAS design, this temporary configuration will allow ansible to push the playbook to configure the router and move the 10.0.10.x IP addresses to the bond0.10 sub-interface.

**karlo-cn-rtr-01 Config**
```text
configure

# Use eth1 for initial bootstrap
# We must tag this for VLAN 10 (INFRA-MGMT) to reach the Ansible node
set interfaces ethernet eth1 vif 10 address 10.0.10.102/24

# Enable SSH for Ansible access
set service ssh port 22

# Create a dedicated user called Ansible for the playbooks
set system login user ansible authentication plaintext-password ansible
set system login user ansible level admin

commit
save
exit
```
**karlo-cn-rtr-02 Config**
```text
configure

# Use eth1 for initial bootstrap
# We must tag this for VLAN 10 (INFRA-MGMT) to reach the Ansible node
set interfaces ethernet eth1 vif 10 address 10.0.10.103/24

# Enable SSH for Ansible access
set service ssh port 22

# Create a dedicated user called Ansible for the playbooks
set system login user ansible authentication plaintext-password ansible
set system login user ansible level admin

commit
save
exit
```
**Ansible Orchestration**  
State-based configuration to manage:  
  ✅ Basic configuration such as system hostnames, compliance banners  
  ✅ Control plane hardening and RBAC (Individual admin accounts)  
  ✅ SNMP and Syslog configuration  
  ✅ NTP/DNS Settings for local synchronization.  
  ✅ Routing (OSPF)   
  ✅ High Availability via VRRP and LACP configuration.  
  
[Refer here for configuration](https://github.com/karletonz1/karlo-cn-ent-lab/blob/main/03-network-devices/vyos-router/ansible/base-config.yml)

### 6. Deployment Hurdles & Pivots

ℹ️**Hurdle 1:**  The Physical Connectivity Problem
During the initial build i had initially wanted to start with the VyOS routers and to configure them via Anisble at the same time. A physical constraint was identified where the Ansible node possessed only a single eth0 interface.

The Problem:  
The Automation node lacked direct physical connectivity to both Routers' management interfaces at the same time.

The Solution:  
The deployment order was pivoted to prioritize the OVS device configuration first. Establishing the Network Management VLAN on the switches first ensured a path for Ansible to reach the routers in order to configure them both at the same time.

ℹ️**Hurdle 2:**  The Logical Connectivity Problem
The Problem: Without a dedicated OOB port, bootstrapping a router through a trunked switch created a chicken-before-the-egg problem.

The Solution: A temporary bootstrap configration was needed to bypass the challenges of virtualizing this network. By manually tagging VLAN 10 on a single physical member of the future bond (eth1), a temporary management link, through the switch network, would be established. This allows Ansible to reach the node and perform the transition to the final configuration.

### 7. Security & Compliance Hardening

**Banner/MOTD:**  
Mandatory legal warning for unauthorized access.

**Management ACLs:**  
Restrict SSH and SNMP traffic so that only the Ansible node's IP can connect. The end goal is to simulate a dedicated windows machine to simulate an authorized Administrator/Engineer's workstation being the only device able to manage into the routers.

**SSH Hardening:**  
✅ Disable Telnet.  
✅ Disable SSH Version 1.  
✅ Set a Timeout Policy: Automatically kick idle sessions after 5 or 10 minutes.  

**Administrative Security:**  
✅ Individual Accounts: Every admin has their own username.  
✅ Role-Based Access Control (RBAC):  Admins: Full config rights  |  Operators/Auditors: "Show" commands only.  

**External Authentication:**  
An authentication order will be configured:  
✅ RADIUS as primary (running on the Domain Controllers)  
✅ local as fallback.  

**Logging & Accountability**  
✅ NTP  
✅ Syslog configuration  
✅ SNMPv3 only

[Refer here for configuration](https://github.com/karletonz1/karlo-cn-ent-lab/blob/main/03-network-devices/vyos-router/ansible/base-config.yml)

### 8. Verification Commands

Useful CLI commands for validating the state of this layer:
- show vrrp summary - Verify Master/Backup status. karlo-cn-rtr-01 should be master and karlo-cn-rtr-02 should be backup.
- show interfaces bonding - Check LACP member health.
- show ip route - Confirm OSPF adjacency with distribution layer.
