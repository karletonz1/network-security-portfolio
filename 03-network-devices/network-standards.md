# Network Design & Standards (LLD)

This document defines the logical architecture, IP addressing scheme, and operational standards for the Karlo-CN Infrastructure. All network devices (VyOS, OPNsense, OVS) will adhere to these specifications.

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
| 666 | Black Hole Sun | 

ℹ️ **VLAN 666:** Used as a "Blackhole" Native VLAN for all trunk ports**

IP address Allocation (VRRP Gateways)

| Vlan ID | Vlan Name | Subnet | VRRP IP (GW) | MTU | Primary Nodes |  
|---------|-----------|--------|--------------|-----|---------------|
| 10| NET-MGMT	| 10.0.10.0/24 | 	10.0.10.254 |	1500 |	VyOS, OPNsense, OVS, Ansible Node, ESXi Host
| 11 | SRV-MGMT |	10.0.11.0/24 |	10.0.11.254 |	1500 |	Hypervisor Host, Veeam, VM Consoles, Domain Controllers 
| 20 |	WIN-CLIENTS |	10.0.20.0/24 |	10.0.20.254 |	1500 |	Windows 10 Workstations 
| 21 |	LIN-CLIENTS |	10.0.21.0/24 |	10.0.21.254 |	1500 |	Ubuntu Workstations 
| 30 | 	SEC-APPS |	10.0.30.0/24 |	10.0.30.254 |	1500 |	Wazuh Manager, Splunk, Tenable, Kali Linux
| 40 |	DMZ |	10.0.40.0/24 |	10.0.40.254 |	1500 |	Apache & IIS Web Servers
| 50 | 	PRD-SVRS |	10.0.50.0/24 |	10.0.50.254	| 1500 |	Simulated App Servers
| 60 | BACKUPS | 10.0.60.0/24 | 10.0.60.254 | 1500 | Veeam Backups
| 666 |	NATIVE |	NONE |	N/A |	1500 |	Unused Ports (Security Blackhole)

This design utilizes RFC 5798 (VRRPv3). The .1 address in each subnet is a Virtual IP shared by a redundant pair of VyOS routers to ensure high availability for all gateway services.

IP Address Allocation (Devices)
| Hostname | Port | IP Address | Vlan ID | Subnet | Gateway | MTU | Role |
|----------|------|------------|---------|--------|---------|-----|------|
| karlo-cn-rtr-01| eth0 | 10.0.10.1 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | OOBM Management |
| karlo-cn-rtr-02| eth0 | 10.0.10.2 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | OOBM Management |
| karlo-cn-ds-01| eth0 | 10.0.10.150 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Management Port |
| karlo-cn-ds-02| eth0 | 10.0.10.250 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Management Port |
| karlo-cn-fw-01| eth0 | 10.0.10.100 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Management Port |
| karlo-cn-fw-02| eth0 | 10.0.10.200 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Management Port |
| karlo-cn-esx-01| eth0/vmk0 | 10.0.10.3 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Management Port |
| karlo-cn-esx-02| eth0/vmk0 | 10.0.10.4 | 10 | 10.0.10.0/24 | 10.0.10.254 | 1500 | Management Port |

ℹ️**OOBM (Out-of-Band Management):**  
Dedicated logical interface on the VyOS routers used exclusively for administrative traffic (SSH, SNMP, Syslog). This path is isolated from the data-plane LACP bonds to ensure          reachability during network contention.

ℹ️**Management Port:**  
Physical management interfaces on the OPNsense firewalls and Open vSwitch (OVS) nodes. These ports provide access to the WebGUI and Control Plane, isolated from the production transit and client VLANs.

ℹ️ **Notation:**  
The Port/Interface column displays the physical GNS3 port followed by the logical OS interface (e.g., eth0/vmk0). This ensures alignment between the virtual lab topology and the internal hypervisor configuration.


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

