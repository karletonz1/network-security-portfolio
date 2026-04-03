# Arista vEOS Switch Deployment Guide

This section documents the deployment and automation of the switch environment using Arista vEOS Switches for the North Star Infrastructure. These switches are used to simulate the aggregation and access layer both respectively labelled Spine and Leaf. MLAG will be deployed to ensure at least 2 links are connected between all switches, and MLAG links with LACP will be configured for the links connecting the Spines to the VyOS Routers.

### 1. Prerequisites & Node Specification

Image: Arista vEOS 4.35.3F

Resources (Per Node):
- RAM: 2048 MB 
- vCPUs: 1 
- Qemu Binary: x86_64 (v8.0.4)


### 2. Physical Topology

<img width="744" height="395" alt="image" src="https://github.com/user-attachments/assets/8a05820f-0eae-4fce-8946-54142ef2c797" />

### Interface Mapping

| Device A Device | Device A Hostname | Device A port | Device B Device | Device B Hostname | Device B Port |
|-----------------|-------------------|---------------|-----------------|-------------------|---------------|
| vEOS Spine Switch 01 | karlo-cn-spine-01 | eth1 | VyOS Core router 01 | karlo-cn-rtr-01 | eth1
| vEOS Spine Switch 01 | karlo-cn-spine-01 | eth2 | VyOS Core router 02 | karlo-cn-rtr-02 | eth2
| vEOS Spine Switch 01 | karlo-cn-spine-01 | eth3 | vEOS Spine Switch 02  | karlo-cn-spine-02  | eth3
| vEOS Spine Switch 01 | karlo-cn-spine-01 | eth4 | vEOS Spine Switch 02  | karlo-cn-spine-02  | eth4
| vEOS Spine Switch 01 | karlo-cn-spine-01 | eth5 | vEOS Leaf Switch 01  | karlo-cn-leaf-01 | eth5
| vEOS Spine Switch 01 | karlo-cn-spine-01 | eth6 | vEOS Leaf Switch 02  | karlo-cn-leaf-02 | eth6
| vEOS Leaf Switch 01 | karlo-cn-leaf-01 | eth5 | vEOS Spine Switch 01 | karlo-cn-spine-01 | eth5
| vEOS Leaf Switch 01 | karlo-cn-leaf-01 | eth6 | vEOS Spine Switch 02 | karlo-cn-spine-02 | eth6
| vEOS Leaf Switch 01 | karlo-cn-leaf-01 | eth12 | Linux Ansible Host | karlo-cn-ansible | eth0


| Device A Device | Device A Hostname | Device A port | Device B Device | Device B Hostname | Device B Port |
|-----------------|-------------------|---------------|-----------------|-------------------|---------------|
| vEOS Spine Switch 02 | karlo-cn-spine-02 | eth1 | VyOS Core router 02 | karlo-cn-rtr-02 | eth1 
| vEOS Spine Switch 02 | karlo-cn-spine-02 | eth2 | VyOS Core router 01 | karlo-cn-rtr-01 | eth2
| vEOS Spine Switch 02 | karlo-cn-spine-02 | eth3 | vEOS Spine Switch 01  | karlo-cn-spine-01 | eth3
| vEOS Spine Switch 02 | karlo-cn-spine-02 | eth4 | vEOS Spine Switch 01  | karlo-cn-spine-01  | eth4
| vEOS Spine Switch 02 | karlo-cn-spine-02 | eth5 | vEOS Leaf Switch 02  | karlo-cn-leaf-02 | eth5
| vEOS Spine Switch 02 | karlo-cn-spine-02 | eth6 | vEOS Leaf Switch 01  | karlo-cn-leaf-01 | eth6
| vEOS Leaf Switch 02 | karlo-cn-leaf-02 | eth5 | vEOS Spine Switch 02 | karlo-cn-spine-02 | eth5
| vEOS Leaf Switch 02 | karlo-cn-leaf-02 | eth6 | vEOS Spine Switch 01 | karlo-cn-spine-01 | eth6



### 2. Logical Topology

<img width="1163" height="653" alt="image" src="https://github.com/user-attachments/assets/7e6cf077-2704-439c-998b-88ce8f16adba" />

### IP Address Allocation (Distribution Switch Layer-OVS)
| Hostname | GNS3 Port | Logical Interface |Allowed Vlan | Role | Link type| 
|----------|-----------|-------------------|-------------|------|----------|
| karlo-cn-spine-01| eth1 | Bond0 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router | Trunk (LACP)
| karlo-cn-spine-01| eth2 | Bond0 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router | Trunk (LACP)
| karlo-cn-spine-01| eth5 | - | 10,11,20,21,30,40,50,60,666 | Downlink to Access Switch 01 | Trunk
| karlo-cn-spine-01| eth6 | - | 10,11,20,21,30,40,50,60,666 | Downlink to Access Switch 02 | Trunk
| karlo-cn-spine-01| eth14 | Bond1 | 10,11,20,21,30,40,50,60,666 | Peer to Distro Switch 2 | Trunk (LACP)
| karlo-cn-spine-01| eth15 | Bond1 | 10,11,20,21,30,40,50,60,666 | Peer to Distro Switch 2 | Trunk (LACP)
| karlo-cn-spine-02| eth1 | Bond0 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router | Trunk (LACP)
| karlo-cn-spine-02| eth2 | Bond0 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router | Trunk (LACP)
| karlo-cn-spine-02| eth5 | - | 10,11,20,21,30,40,50,60,666 | Downlink to Access Switch 01 | Trunk
| karlo-cn-spine-02| eth6 | - | 10,11,20,21,30,40,50,60,666 | Downlink to Access Switch 02 | Trunk
| karlo-cn-spine-02| eth14 | Bond1 | 10,11,20,21,30,40,50,60,666 | Peer to Distro Switch 1 | Trunk (LACP)
| karlo-cn-spine-02| eth15 | Bond1 | 10,11,20,21,30,40,50,60,666 | Peer to Distro Switch 1 | Trunk (LACP)

### IP Address Allocation (Access Switch Layer-OVS)
| Hostname | GNS3 Port | Logical Interface |Allowed Vlan | Role | Link type| 
|----------|-----------|-------------------|-------------|------|----------|
| karlo-cn-access-01| eth0 | - | 10,11,20,21,30,40,50,60,666 | Uplink to Distro Switch 01 | Trunk
| karlo-cn-access-01| eth1 | - | 10,11,20,21,30,40,50,60,666 | Uplink to Distro Switch 02 | Trunk
| karlo-cn-access-01| eth15 | - | All | Ansible uplink to Access Switch 01 | Trunk
| karlo-cn-access-02| eth0 | - | 10,11,20,21,30,40,50,60,666 | Uplink to Distro Switch 01 | Trunk
| karlo-cn-access-02| eth1 | - | 10,11,20,21,30,40,50,60,666 | Uplink to Distro Switch 02 | Trunk


### 4. High Availability & Routing Logic

  Design:
  This lab uses MLAG at layer 2 and VRRP at layer 3 in order to eliminate single points of failure. By using MLAG at Layer 2 and VRRP at Layer 3, the design achieves an active-active network state. This provides a resilient Virtual IP gateway for all end-host subnets, which allows for fail-over to the redunant links seamlessly.

  Link Aggregation:  
  Karlo-cn-spine-01/02 use a cross topology with eth1 and eth2 connecting to rtr-01/rtr-02 eth1 and eth 2 respectively. MLAG is configured on the spines and LACP is configured on the routers.  



### 5. Automation Workflow

Step 1: Manual Bootstrap: Minimum configuration required via CLI to allow Ansible to reach the devices before pushing the remaining configuration via automation.  

⚠️For all switches, run ***#zerotouch cancel*** to stop Arista ZTP and enter manual configuration mode. This will trigger an immediate switch reload.  

**karlo-cn-leaf-01 Bootstrap Config**  
```text
enable
config

! Management User (This cannot be pasted - recommended manually configuring user)
username leafadmin privilege 15 secret "{{ vault_leaf_admin_pass }}"
enable password "{{ vault_leaf_enable_pass }}"

! Enable eAPI
management api http-commands
   protocol https
   no protocol HTTP
   no shutdown
   vrf management
   no shutdown
   exit

! Create the management VRF
vrf instance management

! Configure the physical management port
interface Management1
   description OOBM-TO-ANSIBLE
   vrf management
   ip address 10.0.10.106/24
   no shutdown
exit

copy run start
```

**karlo-cn-leaf-02 Bootstrap Config**  

```text
enable
config

! Management User (This cannot be pasted - recommended manually configuring user)
username leafadmin privilege 15 secret "{{ vault_leaf_admin_pass }}"
enable password "{{ vault_leaf_enable_pass }}"

! Enable eAPI
management api http-commands
   protocol https
   no protocol HTTP
   no shutdown
   vrf management
   no shutdown
   exit

! Create the management VRF
vrf instance management

! Configure the physical management port
interface Management1
   description OOBM-TO-ANSIBLE
   vrf management
   ip address 10.0.10.107/24
   no shutdown
exit

copy run start
```
**karlo-cn-spine-01 Bootstrap Config**  

```text
enable
config

! Management User (This cannot be pasted - recommended manually configuring user)
username spineadmin privilege 15 secret "{{ vault_spine_admin_pass }}"
enable password "{{ vault_spine_enable_pass }}"

! Enable eAPI
management api http-commands
   protocol https
   no protocol http
   no shutdown
   vrf management
   no shutdown
   exit

! Create the management VRF
vrf instance management

! Configure the physical management port
interface Management1
   description OOBM-TO-ANSIBLE
   vrf management
   ip address 10.0.10.104/24
   no shutdown
exit

copy run start
```  
**karlo-cn-spine-02 Bootstrap Config**  

```text
enable
config

! Management User (This cannot be pasted - recommended manually configuring user)
username spineadmin privilege 15 secret "{{ vault_spine_admin_pass }}"
enable password "{{ vault_spine_enable_pass }}"

! Enable eAPI
management api http-commands
   protocol https
   no protocol http
   no shutdown
   vrf management
   no shutdown
   exit

! Create the management isolation container
vrf instance management

! Configure the physical management port
interface Management1
   description OOBM-TO-ANSIBLE
   vrf management
   ip address 10.0.10.105/24
   no shutdown
exit

copy run start
```
**karlo-cn-ansible Bootstrap Config**
```text
# 1. Assign the IP and bring the interface up
ifconfig eth0 10.0.10.253 netmask 255.255.255.0 up

# 2. Add the default gateway
route add default gw 10.0.10.254
```
ℹ️ To check if ansible was successful for the leaf configuration only, you should see something like this:  
<img width="863" height="850" alt="image" src="https://github.com/user-attachments/assets/79b5f5ad-4e55-449b-a9af-8ec519163441" />


**Ansible Orchestration**  
State-based configuration to manage:  
  ✅ Basic configuration such as system hostnames, compliance banners  
  ✅ Control plane hardening and RBAC  
  ✅ SNMP and Syslog configuration  
  ✅ NTP/DNS Settings  
  ✅ LACP configuration  
  ✅ VLAN and Trunking configuration  
  ✅ RSTP configuration  
  
[Refer here for configuration](https://github.com/karletonz1/karlo-cn-ent-lab/blob/main/03-network-devices/open-vswitch/ansible/base-config.yml)

### 6. Deployment Hurdles & Pivots

ℹ️**Hurdle 1:The "Virtual MLAG" Constraint**  
Unlike physical enterprise switches (Cisco Nexus/Arista), the Open vSwitch (OVS) appliance in GNS3 does not natively support Multi-chassis Link Aggregation (mLAG).

The Problem: 
This meant a single router could not "bond" two links across two different switches.

The Solution:  
This necessitated a High-Availability Router-on-a-Stick (ROAS) design. Redundancy is handled at Layer 3 (VRRP) rather than Layer 2 (mLAG), ensuring that if a distribution switch fails, the entire routing path shifts to the secondary node.

ℹ️**Hurdle 2:Control Plane Isolation**  
The Problem:  
Relying on the distro switches for router heartbeats introduced the risk of split-brain scenarios into the topology.

The Solution:  
The decision was made to create a dedicated point-to-point HA Link between rtr-01 and rtr-02 using a /30 subnet (10.0.70.0/30). This keeps the keep-alive traffic separate from production. The update of the IP table with the inclusion of the new subnet and ip addresses for the routers was needed.

ℹ️**Hurdle 3: Multi-Tiered Redundancy Pathing**  
The Problem:  
Ensuring high availability from the Client to the Core without creating logical loops.

The Solution:  
I implemented a three-tier hierarchical design. Redundancy is achieved at Layer 2 via LACP Bonds between switches and at Layer 3 via VRRP Gateways on the VyOS Core. Explicitly tagging all infrastructure traffic and blackholing the native VLAN, created a hardened environment suitable for automated deployment.

ℹ️**Hurdle 4: Management configuration for initial boostrap  
The Problem:  
Using eth15 on Access Switch 01 wa chosen as the entry point of the Ansible node. You would typically configure the interface as an access port with the specific VLAN for that device however, if I were to tag eth15 with vlan 10 now it would begin to tag that traffic which would need configuration at the router level to process this traffic.

The solution:  
Adding the management interface and eth15 without a specific tag would allow me to treat this traffic as native or untagged, which would bypass the need now to have a device capable or routing tagged traffic. 

ℹ️**Hurdle 5: Permission Assumptions  
Problem:  
I assumed tunning sudo commands was always required, but on a native root terminal it resulted in command not found and too many errors.  

<img width="317" height="181" alt="image" src="https://github.com/user-attachments/assets/4023a10f-0ade-4cfb-8dd7-05d402aa183e" />

Solution:  
I stripped sudo prefixes from the initial bootstrap config to match the environment of the GNS3 OVS appliance to prevent the errors from occuring.

ℹ️**Hurdle 6: OVS/GNS3 Nuances  
The Problem:  
Even with the Sudo command removed from the bootstrap configuration, a brand new OVS node would still say that it cannot create a bridge because it already exists.  

<img width="690" height="274" alt="image" src="https://github.com/user-attachments/assets/1f3a0c0b-69e7-4230-949d-ed60bed77451" />

The Solution:  
I needed to find the command that would essentially reset the device and add this command as the first step for the bootstrap configs. The command was ```ovs-vsctl del-br br0```

ℹ️**Hurdle 7: Ansible Host Bootstrapping  
The Problem:   
I discovered that ping commands were not working from the Ansible host. After some research i discovered that configuration was needed on the Linux host similar to configuring an ipv4 address on a Windows machine if you're not using DHCP.

The Solution:  
I verified the commands needed to be added to the boostrap configuration for the Linux host running Ansible.

### 7. Security & Compliance Hardening

**Banner/MOTD:**  
Mandatory legal warning for unauthorized access.

**Management ACLs:**  
Restrict SSH and SNMP traffic so that only the Ansible node's IP can connect. The end goal is to simulate a dedicated windows machine to simulate an authorized Administrator/Engineer's workstation being the only device able to manage into the routers.

**SSH Hardening:**  

**Administrative Security:**  
✅ Individual Accounts: Linux Sudo Groups

**External Authentication:**  
An authentication order will be configured via Pluggable Authentication Modules:  
✅ RADIUS as primary (running on the Domain Controllers)  
✅ local as fallback.  

**Logging & Accountability**  
✅ NTP  
✅ Syslog configuration  
✅ SNMPv3 only

[Refer here for configuration](https://github.com/karletonz1/karlo-cn-ent-lab/blob/main/03-network-devices/open-vswitch/ansible/base-config.yml)


