# Network Design & Standards (LLD)

This document defines the logical architecture, IP addressing scheme, and operational standards for the Karlo-CN Infrastructure. All devices within the lab will adhere to these specifications.

## 1. IP Address Management (IPAM)

Subnets

| Vlan ID | Vlan Name | Subnet |  
|---------|-----------|--------|
| 10 | Network Management | 10.0.10.0/24 
| 11 | Server Management | 10.0.11.0/24 
| 20 | Windows Clients | 10.0.20.0/24
| 21 | Linux Clients | 10.0.21.0/24
| 30 | Security Applications | 10.0.30.0/24
| 40 | DMZ | 10.0.40.0/24
| 50 | Production Servers | 10.0.50.0/24 
| 60 | Backups | 10.0.60.0/24
| 70 | HA link | 10.0.70.0/30
| 700 | MLAG peer | 10.0.70.4/30
| 666 | Black Hole Sun | 

ℹ️ **VLAN 666:** Used as a "Blackhole" Native VLAN for all trunk ports**

IP address Allocation (VRRP Gateways)

| Vlan ID | Vlan Name | Subnet | VRRP IP (GW) | MTU | Primary Nodes |  
|---------|-----------|--------|--------------|-----|---------------|
| 10 | INFRA-MGMT	| 10.0.10.0/24 | 	10.0.10.254 |	1500 |	VyOS, OPNsense, OVS, Ansible
| 11 | SRV-MGMT |	10.0.11.0/24 |	10.0.11.254 |	1500 |	Proxmox Host, Veeam, Domain Controllers 
| 20 |	WIN-CLIENTS |	10.0.20.0/24 |	10.0.20.254 |	1500 |	Windows 10 Workstations 
| 21 |	LIN-CLIENTS |	10.0.21.0/24 |	10.0.21.254 |	1500 |	Ubuntu Workstations 
| 30 | 	SEC-APPS |	10.0.30.0/24 |	10.0.30.254 |	1500 |	Wazuh Manager, Splunk, Tenable, Kali Linux
| 40 |	DMZ |	10.0.40.0/24 |	10.0.40.254 |	1500 |	Apache & IIS Web Servers
| 50 | 	PRD-SVRS |	10.0.50.0/24 |	10.0.50.254	| 1500 |	Simulated App Servers (Proxmox)
| 60 | BACKUPS | 10.0.60.0/24 | 10.0.60.254 | 1500 | Veeam Backups
| 666 |	NATIVE |	NONE |	N/A |	1500 |	Unused Ports (Security Blackhole)

ℹ️ GNS3 Virtualization  
While physical hardware often includes a dedicated management port, the GNS3 environment utilizes standard Ethernet interfaces. To replicate production environments, VLAN 10 has been designated as the Management VLAN. All administrative traffic (SSH, SNMP, Ansible) is logically isolated into this VLAN, to separate management traffic from production traffic.

### IP Address Allocation (Firewall Layer-OPNSENSE)
| Hostname | GNS3 Port | IP Address | Vlan ID | Subnet | Gateway | MTU | Role | Link type| 
|----------|------|------------|---------|--------|---------|-----|------|----------|
| karlo-cn-fw-01| eth0 | 10.0.10.10 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Management Port | Access
| karlo-cn-fw-02| eth0 | 10.0.10.20 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Management Port | Access

### IP Address Allocation (Core Router Layer-VyOS)
| Hostname | GNS3 Port | IP Address | Vlan ID | Subnet | Gateway (VIP) | MTU | Role | Link type| 
|----------|------|------------|---------|--------|---------|-----|------|----------|
| karlo-cn-rtr-01| eth8/9:bond1.70| 10.0.70.1 | 70 | 10.0.70.0/30 | - | 1500 | Master HA link | PTP
| karlo-cn-rtr-01| eth1/2:Bond0.10 | 10.0.10.1 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Gateway for Infra-MGT Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.11 | 10.0.11.1 | 11 | 10.0.11.0/24 | 10.0.11.254 | 1500 | Gateway for SRV-MGT Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.20 | 10.0.20.1 | 20 | 10.0.20.0/24 | 10.0.20.254 | 1500 | Gateway for WIN-CLIENTS Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.21 | 10.0.21.1 | 21 | 10.0.21.0/24 | 10.0.21.254 | 1500 | Gateway for LIN-CLIENTS Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.30 | 10.0.30.1 | 30 | 10.0.30.0/24 | 10.0.30.254 | 1500 | Gateway for SEC-APPS Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.40 | 10.0.40.1 | 40 | 10.0.40.0/24 | 10.0.40.254 | 1500 | Gateway for DMZ Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.50 | 10.0.50.1 | 50 | 10.0.50.0/24 | 10.0.50.254 | 1500 | Gateway for PRD-SRVS Subnet| Sub-interface
| karlo-cn-rtr-01| eth1/2:Bond0.60 | 10.0.60.1 | 60 | 10.0.60.0/24 | 10.0.60.254 | 1500 | Gateway for BACKUPS Subnet| Sub-interface
| karlo-cn-rtr-02| eth8/9:bond1.70 | 10.0.70.2 | 70 | 10.0.70.0/30 | - | 1500 | Secondary HA link  | PTP
| karlo-cn-rtr-02| eth1/2:Bond0.10 | 10.0.10.2| 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Gateway for Infra-MGT Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.11 | 10.0.11.2| 11 | 10.0.11.0/24 | 10.0.11.254 | 1500 | Gateway for SRV-MGT Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.20 | 10.0.20.2| 20 | 10.0.20.0/24 | 10.0.20.254 | 1500 | Gateway for WIN-CLIENTS Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.21 | 10.0.21.2| 21 | 10.0.21.0/24 | 10.0.21.254 | 1500 | Gateway for LIN-CLIENTS Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.30 | 10.0.30.2| 30 | 10.0.30.0/24 | 10.0.30.254 | 1500 | Gateway for SEC-APPS Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.40 | 10.0.40.2| 40 | 10.0.40.0/24 | 10.0.40.254 | 1500 | Gateway for DMZ Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.50 | 10.0.50.2| 50 | 10.0.50.0/24 | 10.0.50.254 | 1500 | Gateway for PRD-SRVS Subnet| Sub-interface
| karlo-cn-rtr-02| eth1/2:Bond0.60 | 10.0.60.2| 60 | 10.0.60.0/24 | 10.0.60.254 | 1500 | Gateway for BACKUPS Subnet| Sub-interface

### IP Address Allocation (Spine Switch Layer-Arista vEOS)
| Hostname | GNS3 Port | Logical Interface |Allowed Vlan | Role | Link type| 
|----------|-----------|-------------------|-------------|------|----------|
| karlo-cn-spine-01| eth1 | Port-channel 1 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router 01 | Trunk 
| karlo-cn-spine-01| eth2 | Port-channel 2 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router 02 | Trunk 
| karlo-cn-spine-01| eth5 | Port-channel 10 | 10,11,20,21,30,40,50,60,666 | Downlink to Leaf Switch 01 | Trunk
| karlo-cn-spine-01| eth6 | Port-channel 20 | 10,11,20,21,30,40,50,60,666 | Downlink to Leaf Switch 02 | Trunk
| karlo-cn-spine-01| eth3 | Port-channel 70 | 10,11,20,21,30,40,50,60,700 | MLAG Primary peer link | Trunk 
| karlo-cn-spine-01| eth4 | Port-channel 70 | 10,11,20,21,30,40,50,60,700 | MLAG Primary peer link | Trunk 
| karlo-cn-spine-02| eth1 | Port-channel 2 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router 02 | Trunk 
| karlo-cn-spine-02| eth2 | Port-channel 1 | 10,11,20,21,30,40,50,60,666 | Uplink to Core Router 01 | Trunk 
| karlo-cn-spine-02| eth3 | Port-channel 70 | 10,11,20,21,30,40,50,60,700 | MLAG Secondary peer link | Trunk 
| karlo-cn-spine-02| eth4 | Port-channel 70 | 10,11,20,21,30,40,50,60,700 | MLAG Secondary peer link | Trunk
| karlo-cn-spine-02| eth5 | Port-channel 20 | 10,11,20,21,30,40,50,60,666 | Downlink to Leaf Switch 02 | Trunk
| karlo-cn-spine-02| eth6 | Port-channel 10 | 10,11,20,21,30,40,50,60,666 | Downlink to Leaf Switch 01 | Trunk


### IP Address Allocation (Access Switch Layer-Arista vEOS)
| Hostname | GNS3 Port | Logical Interface |Allowed Vlan | Role | Link type| 
|----------|-----------|-------------------|-------------|------|----------|
| karlo-cn-leaf-01| eth5 | Port-channel 10 | 10,11,20,21,30,40,50,60,666 | Uplink to Spine Switch 01 | Trunk
| karlo-cn-leaf-01| eth6 | Port-channel 10 | 10,11,20,21,30,40,50,60,666 | Uplink to Spine Switch 02 | Trunk
| karlo-cn-leaf-01| eth12 | - | 10 | Downlink to karlo-cn-ansible | Access
| karlo-cn-leaf-02| eth5 | Port-channel 20 | 10,11,20,21,30,40,50,60,666 | Uplink to Spine Switch 02 | Trunk
| karlo-cn-leaf-02| eth6 | Port-channel 20 | 10,11,20,21,30,40,50,60,666 | Uplink to Spine Switch 01 | Trunk

### IP Address Allocation (Servers-Proxmox)
| Hostname | GNS3 Port | IP Address | Vlan ID | Subnet | Gateway | MTU | Role | Link type|
|----------|------|------------|---------|--------|---------|-----|------|----------|
| karlo-cn-esx-01| eth0/vmk0 | 10.0.50.1 | 50 | 10.0.50.0/24 | 10.0.50.254 | 1500 | Uplink to Access Switch 01 | Access
| karlo-cn-esx-02| eth0/vmk0 | 10.0.50.2 | 50 | 10.0.50.0/24 | 10.0.50.254 | 1500 | Uplink to Access Switch 02 | Access

### IP Address Allocation (Clients)
| Hostname | GNS3 Port | IP Address | Vlan ID | Subnet | Gateway | MTU | Role | Link type| 
|----------|------|------------|---------|--------|---------|-----|------|----------|
| karlo-cn-ansible| eth0 | 10.0.10.253 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Uplink to Leaf Switch 01 | Access

### IP Address Allocation (IP Management-All Devices)
| Hostname | IP Address | Vlan ID | Subnet | Gateway | MTU | Role |
|----------|------------|---------|--------|---------|-----|------|
| karlo-cn-fw-01 | 10.0.10.100 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | In-band Management 
| karlo-cn-fw-02 | 10.0.10.101 | 10 | 10.0.10.0/24  | 10.0.10.254 | 1500 | In-band Management 
| karlo-cn-rtr-01 | 10.0.10.102 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | In-band Management 
| karlo-cn-rtr-02 | 10.0.10.103 | 10 | 10.0.10.0/24  | 10.0.10.254 | 1500 | In-band Management 
| karlo-cn-spine-01 | 10.0.10.104 | 10 | 10.0.10.0/24  | 10.0.10.254 | 1500 | In-band Management 
| karlo-cn-spine-02 | 10.0.10.105 | 10 | 10.0.10.0/24  | 10.0.10.254 | 1500 | In-band Management 
| karlo-cn-leaf-01 | 10.0.10.106 | 10 | 10.0.10.0/24   | 10.0.10.254 | 1500 | In-band Management 
| karlo-cn-leaf-02 | 10.0.10.107 | 10 | 10.0.10.0/24   | 10.0.10.254 | 1500 | In-band Management 
| karlo-cn-ansible | 10.0.10.253 | 10 | 10.0.10.0/24  | 10.0.10.254 | 1500 | In-band Management 



Service & Port Map
| Source |	Destination |	Protocol/Port |	Service |
|--------|--------------|---------------|---------|
| ANY |	DMZ|  (50) |	TCP 80, 443 |	Web Traffic (External)
| ALL SUBNETS |	SEC-APPS (30) |	UDP 1514, 1515 |	Wazuh Agent Logs
| ALL SUBNETS |	SEC-APPS (30) |	TCP 9997 |	Splunk Data Ingest
| NET-MGMT (10) |	ALL SUBNETS |	TCP 22 |	Ansible SSH Management



## 3. High Availability Logic (VRRP)

Redundancy is managed via VRRP on top of the LACP Bond (bond0).

- VRID Strategy: The VRRP ID must match the VLAN ID (e.g., VLAN 20 = VRID 20).

- Preemption: Enabled on RTR-01 (Priority 200) to ensure it resumes Master status after a reboot.

- Hello Interval: 1 Second.



## 4. MTU & Performance Policy

Standard MTU (1500):

- Applied to all Management (OOB) interfaces.

- Applied to all WAN/Internet-facing interfaces.

- Applied to the Inter-Router LACP Backbone.

- Applied to Servers connections.  
ℹ️ In a production environment, there would be specific instances where an MTU higher than 1500 would be needed to accomodate jumbo frames. Specific storage devices have not been applied to this lab.

