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
| 666 | Black Hole Sun | 

IP address Allocation (VRRP)

| Vlan ID | Vlan Name | Subnet | VRRP IP (GW) | MTU | Primary Nodes |  
|---------|-----------|--------|--------------|-----|---------------|
| 10| NET-MGMT	| 10.0.10.0/24 | 	10.0.10.1 |	1500 |	VyOS, OPNsense, OVS, Ansible Node 
| 11 | SRV-MGMT |	10.0.11.0/24 |	10.0.11.1 |	1500 |	Hypervisor Host, Veeam, VM Consoles 
| 20 |	WIN-CLIENTS |	10.0.20.0/24 |	10.0.20.1 |	1500 |	Windows 10/11 Workstations 
| 21 |	LIN-CLIENTS |	10.0.21.0/24 |	10.0.21.1 |	1500 |	Ubuntu/Fedora Workstations 
| 30 | 	SEC-APPS |	10.0.30.0/24 |	10.0.30.1 |	1500 |	Wazuh Manager, Splunk Indexer 
| 40 |	DMZ |	10.0.40.0/24 |	10.0.40.1 |	1500 |	Web Servers, External-Facing Apps
| 50 | 	PRD-SVRS |	10.0.50.0/24 |	10.0.50.1	| 1500 |	App Servers, DB Servers, File Shares
| 666 |	NATIVE |	NONE |	N/A |	1500 |	Unused Ports (Security Blackhole)



**Note on VLAN 666: Used as a "Blackhole" Native VLAN for all trunk ports to prevent VLAN Hopping attacks.**

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

- Applied to Server and Storage segments to reduce CPU overhead during high-volume transfers. 
Note: in a production environment, there would be specific instances where an MTU higher than 1500 would be needed to accomodate jumbo frames. This rationale has not been applied to this lab.

