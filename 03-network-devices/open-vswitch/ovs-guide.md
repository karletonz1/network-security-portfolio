# Open vSwitch Deployment Guide

This section documents the deployment and automation of the Distribution Switch layer using Open vSwitches for the North Star Infrastructure. These switches are use to simulate the aggregation layer, but it is mainly used as a layer 2 tool for the distribution and access layers. There are mLAG limitations of these switches within GN3 which necessitated a design choice of redundant pathing across all layers, with a single HA link between the routers but running VRRP in a ROAS topology.

### 1. Prerequisites & Node Specification

Image: Open vSwitch 3.1.0

Resources (Per Node):

- RAM: 1024 MB 

- vCPUs: 1 

- Qemu Binary: x86_64 (v8.0.4)

- Disk: 8 GB (Thin-provisioned).
    
[Additional information of the devices can be found here](https://github.com/karletonz1/karlo-cn-ent-lab/blob/main/03-network-devices/open-vswitch/ansible/hosts)

Management Access: SSH enabled with dedicated ansible service account.

### 2. Network Topology & Interface Mapping

**karlo-cn-ds-01** 
| Device A port | Device B Device | Device B Hostname | Device B Port |
|----------------------|-----------------|-------------------|---------------|
| eth1 | VyOS Core router 01 | karlo-cn-rtr-01 | port 1
| eth2 | VyOS Core router 01 | karlo-cn-rtr-01 | port 2
| eth14 | OVS Distribution Switch 02 | karlo-cn-ds-02 | port 14
| eth15 | OVS Distribution Switch 02 | karlo-cn-ds-02 | port 15


**karlo-cn-ds-02** 
| Device A port | Device B Device | Device B Hostname | Device B Port |
|----------------------|-----------------|-------------------|---------------|
| eth1 |	VyOS Core router 02 | karlo-cn-rtr-02 | port 1 
| eth2 |	VyOS Core router 02 | karlo-cn-rtr-02 | port 2
| eth14 | OVS Distribution Switch 01 | karlo-cn-ds-01 | port 14
| eth15 | OVS Distribution Switch 01 | karlo-cn-ds-01 | port 15


### 3. High Availability & Routing Logic

  Gateway Redundancy:  
  This lab uses VRRP at the core routers.

  Link Aggregation:  
  The Router level utilizes LACP between karlo-cn-rtr-01/02 and karlo-cn-ds-01/02 using eth1 and eth2 for the LAG link. Trunk ports are configured for eth1/2 at the distribution switch end, and sub-interfaces are configured at the core router end. 

  Dynamic Routing:  
  OSPF is used in this lab for dynamic learning and distribution of routes.


### 5. Automation Workflow

Step 1: Manual Bootstrap: Minimum configuration required via CLI to allow Ansible to reach the devices before pushing the remaining configuration via automation.

**karlo-cn-rtr-01 Config**
```text
# 1. Create the logical bridge
sudo ovs-vsctl add-br br0

# 2. Enable STP immediately
sudo ovs-vsctl set bridge br0 stp_enable=true

# 3. Transition the physical port to the bridge
sudo ovs-vsctl add-port br0 eth0

# 4. Move Management IP to the Bridge
sudo ip addr flush dev eth0
sudo ip addr add 10.0.10.150/24 dev br0
sudo ip link set br0 up

# 5. Set route to VyOS VIP and point to Ansible Node
sudo ip route add default via 10.0.10.254
```
**karlo-cn-rtr-02 Config**
```text
# 1. Create the logical bridge
sudo ovs-vsctl add-br br0

# 2. Enable STP immediately
sudo ovs-vsctl set bridge br0 stp_enable=true

# 3. Transition the physical port to the bridge
sudo ovs-vsctl add-port br0 eth0

# 4. Move Management IP to the Bridge
sudo ip addr flush dev eth0
sudo ip addr add 10.0.10.250/24 dev br0
sudo ip link set br0 up

# 5. Set route to VyOS VIP and point to Ansible Node
sudo ip route add default via 10.0.10.254
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

The Hurdle: 
This meant a single router could not "bond" two links across two different switches.

The Solution:  
This necessitated a High-Availability Router-on-a-Stick (ROAS) design. Redundancy is handled at Layer 3 (VRRP) rather than Layer 2 (mLAG), ensuring that if a distribution switch fails, the entire routing path shifts to the secondary node.

ℹ️**Hurdle 2:Control Plane Isolation**  
The Hurdle:  
Relying on the switching fabric for router heartbeats introduced the risk of "Split-Brain" scenarios during high broadcast traffic.

The Solution:  
A dedicated Point-to-Point HA Link was established between karlo-cn-rtr-01 and rtr-02 using a non-routable /30 subnet (169.254.255.0/30). This isolates the "Keep-Alive" traffic from the production data plane. The update of the IP table with the inclusion of the new subnet and ip addresses for the routers was needed.


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
- sudo ovs-appctl bond/show - shows exactly which physical interfaces are active in the bond and LACP negotiation status
- sudo ovs-vsctl list port - Confirms that your ports are correctly assigned to the intended VLAN trunks
- sudo ovs-appctl stp/show - Shows the STP state of every port
- sudo ovs-appctl fdb/show br0 - Displays the MAC table
