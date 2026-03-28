# VyOS Core Router Deployment Guide

This section documents the deployment and automation of the Core Routing layer for the Karlo-CN Infrastructure. These routers handle the L3 boundary between the internal distribution layer and the security perimeter. 

Prerequisites & Design Rationale
---

**Software & Nodes**

- VyOS 1.5-rolling:

- GNS3 Nodes: 2x VyOS Routers

**Core Architecture**

- The Router level utilizes a 2-port LACP (802.3ad) Bond between KARLO-CN-CORE-RTR-01 and KARLO-CN-CORE-RTR-02 using eth8 and eth9 for the LAG link

- To eliminate a single point of failure for the default gateway, this lab will use VRRP using a single Virtual IP to be the gateway for all network devices. 
