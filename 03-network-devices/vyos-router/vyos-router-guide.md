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

### 2. Network Topology & Interface Mapping

**karlo-cn-rtr-01** 
| Device A port | Device B Device | Device B Hostname | Device B Port |
|----------------------|-----------------|-------------------|---------------|
| eth0 |	VyOS Core router 02 | karlo-cn-rtr-02 | port 0
| eth1 |  OVS Distribution switch 02 | karlo-cn-ds-02 | port 1
| eth2 |	OVS Distribution switch 02 | karlo-cn-ds-02 | port 2


**karlo-cn-rtr-02** 
| Device A port | Device B Device | Device B Hostname | Device B Port |
|----------------------|-----------------|-------------------|---------------|
| eth0 |	VyOS Core router 01 | karlo-cn-rtr-01 | port 0
| eth1 |	OVS Distribution switch 01 | karlo-cn-ds-01 | port 1 
| eth2 |	OVS Distribution switch 01 | karlo-cn-ds-01 | port 2


### 3. High Availability & Routing Logic

  Gateway Redundancy:  
  This lab uses VRRP at the core routers via sub-interfaces.

  Link Aggregation:  
  The Router level utilizes LACP between karlo-cn-rtr-01/02 and karlo-cn-ds-01/02 using eth1 and eth2 for the LAG link.

  Dynamic Routing:  
  OSPF is used in this lab for dynamic learning and distribution of routes.

  High Availablity:  
  A HA heartbeak link is configured between the core routers via a /30 subnet.


### 5. Automation Workflow

Step 1: Manual Bootstrap: Minimum configuration required via CLI to allow Ansible to reach the devices before pushing the remaining configuration via automation.

**karlo-cn-rtr-01 Config**
```text
configure

# Set the management IP (eth0 is the link to karlo-cn-ds-01)
set interfaces ethernet eth0 address 10.0.10.1/24 

# Enable SSH for Ansible access
set service ssh port 22

# Create a dedicated 'ansible' user for the playbooks
set system login user ansible authentication plaintext-password 'ansible'
set system login user ansible level admin

commit
save
exit
```
**karlo-cn-rtr-02 Config**
```text
configure

# Set the management IP (eth0 is the link to karlo-cn-ds-02)
set interfaces ethernet eth0 address 10.0.10.2/24 

# Enable SSH for Ansible access
set service ssh port 22

# Create a dedicated 'ansible' user for the playbooks
set system login user ansible authentication plaintext-password 'ansible'
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

ℹ️**Hurdle 1:**  
During the initial build i had initially wanted to start with the VyOS routers and to configure them via Anisble at the same time. A physical constraint was identified where the Ansible node possessed only a single eth0 interface.

The Problem:  
The Automation node lacked direct physical connectivity to both Routers' management interfaces at the same time.

The Solution:  
The deployment order was pivoted to prioritize the Open vSwitch (OVS) device configuration first. Establishing the Network Management VLAN (VLAN 10) on the switches first ensured a stable "Control Plane" path for Ansible to reach the VyOS routers in order to configure them at the same time.

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
