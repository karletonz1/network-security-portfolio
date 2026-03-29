# Open vSwitch Deployment Guide

This section documents the deployment and automation of the Distribution Switch layer using Open vSwitches for the North Star Infrastructure. These switches are used to simulate the aggregation layer, but it is mainly used as a layer 2 tool for the distribution and access layers. There are mLAG limitations of these switches within GN3 which necessitated a design choice of redundant pathing across all layers, with a single HA link between the routers but running VRRP in a ROAS topology.

### 1. Prerequisites & Node Specification

Image: Open vSwitch 3.1.0

Resources (Per Node):

- RAM: 1024 MB 

- vCPUs: 1 

- Qemu Binary: x86_64 (v8.0.4)

- Disk: 8 GB (Thin-provisioned).
    
[Additional information of the devices can be found here](https://github.com/karletonz1/karlo-cn-ent-lab/blob/main/03-network-devices/open-vswitch/ansible/hosts)

Management Access: SSH enabled with dedicated ansible service account.

### 2. Physical Topology

<img width="877" height="410" alt="image" src="https://github.com/user-attachments/assets/697b6470-c3bc-45db-a3dc-f6f96c2252f2" />


### Interface Mapping

| Device A Device | Device A Hostname | Device A port | Device B Device | Device B Hostname | Device B Port |
|-----------------|-------------------|---------------|-----------------|-------------------|---------------|
| OVS Distribution Switch 01 | karlo-cn-ds-01 | eth1 | VyOS Core router 01 | karlo-cn-rtr-01 | eth1
| OVS Distribution Switch 01 | karlo-cn-ds-01 | eth2 | VyOS Core router 01 | karlo-cn-rtr-01 | eth2
| OVS Distribution Switch 01 | karlo-cn-ds-01 | eth5 | OVS Access Switch 01 | karlo-cn-access-01 | eth0
| OVS Distribution Switch 01 | karlo-cn-ds-01 | eth6 | OVS Access Switch 02 | karlo-cn-access-02 | eth0
| OVS Distribution Switch 01 | karlo-cn-ds-01 | eth14 | OVS Distribution Switch 02 | karlo-cn-ds-02 | eth14
| OVS Distribution Switch 01 | karlo-cn-ds-01 | eth15 | OVS Distribution Switch 02 | karlo-cn-ds-02 | eth15
| OVS Access Switch 01 | karlo-cn-access-01 | eth0 | OVS Distribution Switch 01 | karlo-cn-ds01 | eth5
| OVS Access Switch 01 | karlo-cn-access-01 | eth1 | OVS Distribution Switch 02 | karlo-cn-ds02 | eth5
| OVS Access Switch 01 | karlo-cn-access-01 | eth15 | Linux Ansible Host | karlo-cn-ansible | eth15


| Device A Device | Device A Hostname | Device A port | Device B Device | Device B Hostname | Device B Port |
|-----------------|-------------------|---------------|-----------------|-------------------|---------------|
| OVS Distribution Switch 02 | karlo-cn-ds-02 | eth1 | VyOS Core router 02 | karlo-cn-rtr-02 | eth1 
| OVS Distribution Switch 02 | karlo-cn-ds-02 | eth2 | VyOS Core router 02 | karlo-cn-rtr-02 | eth2
| OVS Distribution Switch 02 | karlo-cn-ds-02 | eth5 | OVS Access Switch 01 | karlo-cn-access-01 | eth1
| OVS Distribution Switch 02 | karlo-cn-ds-02 | eth6 | OVS Access Switch 02 | karlo-cn-access-02 | eth1
| OVS Distribution Switch 02 | karlo-cn-ds-02 | eth14 | OVS Distribution Switch 01 | karlo-cn-ds-01 | eth14
| OVS Distribution Switch 02 | karlo-cn-ds-02 | eth15 | OVS Distribution Switch 01 | karlo-cn-ds-01 | eth15
| OVS Access Switch 02 | karlo-cn-access-02 | eth0 | OVS Distribution Switch 01 | karlo-cn-ds01 | eth6
| OVS Access Switch 02 | karlo-cn-access-02 | eth1 | OVS Distribution Switch 02 | karlo-cn-ds02 | eth6



### 2. Logical Topology

<img width="1163" height="653" alt="image" src="https://github.com/user-attachments/assets/7e6cf077-2704-439c-998b-88ce8f16adba" />

### IP Address Allocation (Distribution Switch Layer-OVS)
| Hostname | GNS3 Port | Logical Interface |Allowed Vlan | Role | Link type| 
|----------|-----------|-------------------|-------------|------|----------|
| karlo-cn-ds-01| eth1 | Bond0 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router | Trunk (LACP)
| karlo-cn-ds-01| eth2 | Bond0 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router | Trunk (LACP)
| karlo-cn-ds-01| eth5 | - | 10,11,20,21,30,40,50,60,666 | Downlink to Access Switch 01 | Trunk
| karlo-cn-ds-01| eth6 | - | 10,11,20,21,30,40,50,60,666 | Downlink to Access Switch 02 | Trunk
| karlo-cn-ds-01| eth14 | Bond1 | 10,11,20,21,30,40,50,60,666 | Peer to Distro Switch 2 | Trunk (LACP)
| karlo-cn-ds-01| eth15 | Bond1 | 10,11,20,21,30,40,50,60,666 | Peer to Distro Switch 2 | Trunk (LACP)
| karlo-cn-ds-02| eth1 | Bond0 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router | Trunk (LACP)
| karlo-cn-ds-02| eth2 | Bond0 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router | Trunk (LACP)
| karlo-cn-ds-02| eth5 | - | 10,11,20,21,30,40,50,60,666 | Downlink to Access Switch 01 | Trunk
| karlo-cn-ds-02| eth6 | - | 10,11,20,21,30,40,50,60,666 | Downlink to Access Switch 02 | Trunk
| karlo-cn-ds-02| eth14 | Bond1 | 10,11,20,21,30,40,50,60,666 | Peer to Distro Switch 1 | Trunk (LACP)
| karlo-cn-ds-02| eth15 | Bond1 | 10,11,20,21,30,40,50,60,666 | Peer to Distro Switch 1 | Trunk (LACP)

### IP Address Allocation (Access Switch Layer-OVS)
| Hostname | GNS3 Port | Logical Interface |Allowed Vlan | Role | Link type| 
|----------|-----------|-------------------|-------------|------|----------|
| karlo-cn-access-01| eth0 | - | 10,11,20,21,30,40,50,60,666 | Uplink to Distro Switch 01 | Trunk
| karlo-cn-access-01| eth1 | - | 10,11,20,21,30,40,50,60,666 | Uplink to Distro Switch 02 | Trunk
| karlo-cn-access-01| eth15 | - | All | Ansible uplink to Access Switch 01 | Trunk
| karlo-cn-access-02| eth0 | - | 10,11,20,21,30,40,50,60,666 | Uplink to Distro Switch 01 | Trunk
| karlo-cn-access-02| eth1 | - | 10,11,20,21,30,40,50,60,666 | Uplink to Distro Switch 02 | Trunk


### 4. High Availability & Routing Logic

  Gateway Redundancy:  
  This lab uses VRRP at the core routers.

  Link Aggregation:  
  The Router level utilizes LACP between karlo-cn-rtr-01/02 and karlo-cn-ds-01/02 using eth1 and eth2 for the LAG link. Trunk ports are configured for eth1/2 at the distribution switch end, and sub-interfaces are configured at the core router end. 

Trunking:  
Trunk ports will be configured for eth1 and eth2. Vlan configuration for all vlans in the lab will also be configured.


### 5. Automation Workflow

Step 1: Manual Bootstrap: Minimum configuration required via CLI to allow Ansible to reach the devices before pushing the remaining configuration via automation.

**karlo-cn-ds-01 Bootstrap Config**
```text
# Ensure a clean OVS node to prevent errors about bridge br0 already existing
ovs-vsctl del-br br0

# Create bridge and enable STP
ovs-vsctl add-br br0
ovs-vsctl set bridge br0 stp_enable=true

# Add eth5 (The downlink to Access-01) for initial bootstrap reachability
ovs-vsctl add-port br0 eth5

# Add the first peer link
ovs-vsctl add-port br0 eth14

# Add the second peer link
ovs-vsctl add-port br0 eth15

# Assign Management IP address
ip addr flush dev eth5
ip addr add 10.0.10.104/24 dev br0
ip link set br0 up

# Set Gateway to the Router Management VIP
ip route add default via 10.0.10.254
```
**karlo-cn-ds-02 Bootstrap Config**
```text
# Ensure a clean OVS node to prevent errors about bridge br0 already existing
ovs-vsctl del-br br0

# Create bridge and enable STP
ovs-vsctl add-br br0
ovs-vsctl set bridge br0 stp_enable=true

# Add eth6 (The downlink to Access-02) for initial bootstrap reachability
ovs-vsctl add-port br0 eth6

# Add the first peer link
ovs-vsctl add-port br0 eth14

# Add the second peer link
ovs-vsctl add-port br0 eth15

# Assign Management IP address
ip addr flush dev eth6
ip addr add 10.0.10.105/24 dev br0
ip link set br0 up

# Set Gateway to the Router Management VIP
ip route add default via 10.0.10.254
```
**karlo-cn-access-01 Bootstrap Config**
```text
# Ensure a clean OVS node to prevent errors about bridge br0 already existing
ovs-vsctl del-br br0

# Create bridge and enable STP
ovs-vsctl add-br br0
ovs-vsctl set bridge br0 stp_enable=true

# Add eth15 (The link to Ansible node)
ovs-vsctl add-port br0 eth15

# Add the uplinks to reach the Distro Layer
ovs-vsctl add-port br0 eth0  # Connection to ds-01 
ovs-vsctl add-port br0 eth1  # Connection to ds-02

# Assign Management IP address
ip addr flush dev eth15
ip addr add 10.0.10.106/24 dev br0
ip link set br0 up

# Set Gateway to the Router Management VIP
ip route add default via 10.0.10.254
```
**karlo-cn-access-02 Bootstrap Config**
```text
# Ensure a clean OVS node to prevent errors about bridge br0 already existing
ovs-vsctl del-br br0

# Create bridge and enable STP
ovs-vsctl add-br br0
ovs-vsctl set bridge br0 stp_enable=true

# Add the uplinks to reach the Distro Layer
ovs-vsctl add-port br0 eth0  # Connection to ds-01
ovs-vsctl add-port br0 eth1  # Connection to ds-02

# Assign Management IP address
ip addr flush dev eth1
ip addr add 10.0.10.107/24 dev br0
ip link set br0 up

# Set Gateway to the Router Management VIP
ip route add default via 10.0.10.254
```

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
Problem:  
Even with the Sudo command removed from the bootstrap configuration, a brand new OVS node would still say the it cannot create a bridge because it already exists.  

<img width="690" height="274" alt="image" src="https://github.com/user-attachments/assets/1f3a0c0b-69e7-4230-949d-ed60bed77451" />

Solution:  
I needed to find the command that would essentially reset the device and add this command as the first step for the bootstrap configs. The command was ```ovs-vsctl del-br br0```

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

### 8. Verification Commands

Useful CLI commands for validating the state of this layer:
- ```ovs-appctl bond/show``` - shows exactly which physical interfaces are active in the bond and LACP negotiation status
- ```ovs-vsctl list port``` - Confirms that your ports are correctly assigned to the intended VLAN trunks
- ```ovs-appctl stp/show``` - Shows the STP state of every port
- ```ovs-appctl fdb/show br0``` - Displays the MAC table
- ```ip addr show br0``` - You should see this to verify bootstrap configuration success <img width="639" height="116" alt="image" src="https://github.com/user-attachments/assets/163ad77d-d621-4714-91b8-201225e732be" />


