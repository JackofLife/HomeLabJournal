---
date: 2026-04-12
author: <YOUR_NAME>
topic: Networking, Architecture, IPAM
---

# Network Segmentation & IP Architecture

## Overview
To ensure a secure, enterprise-grade lab environment, the network is strictly segmented into distinct VLANs. This isolates vulnerable testing environments from daily personal use and core management infrastructure. 

The domain architecture utilizes the RFC 8375 standard `.home.arpa` Top Level Domain to prevent mDNS (Bonjour/Avahi) broadcast conflicts on the local network.

## IP Schema & Subnet Allocation

| Network Group | Subnet / VLAN | Subdomain Layout | Purpose / Example FQDN |
| :--- | :--- | :--- | :--- |
| **Management** | `10.[VLAN_MGMT].0/24` | `*.example.home.arpa` | Core infrastructure (`proxmox.example.home.arpa`) |
| **Personal** | `10.[VLAN_PERS].0/24` | `*.home.example.home.arpa` | Daily drivers, trusted PCs, and mobile devices |
| **Guest** | `10.[VLAN_GUEST].0/24` | `*.guest.example.home.arpa` | Untrusted visitor devices |
| **IoT** | `10.[VLAN_IOT].0/24` | `*.iot.example.home.arpa` | Smart TVs, Cameras (Isolated internet access only) |
| **Security Lab** | `10.[VLAN_SEC].0/24` | `*.lab.example.home.arpa` | Detonation/Testing (Wazuh, Kali Linux) |
| **K8s / VMs** | `10.[VLAN_K8S].0/24` | `*.k8s.example.home.arpa` | Production workloads and Kubernetes nodes |

## Routing & Access Policies
* Inter-VLAN routing is handled by a virtualized OPNsense instance.
* Default firewall rules enforce strict isolation (Deny All) between the Security Lab, IoT, and Personal networks.
* The Management network is strictly accessible only via hardwired management ports or authenticated VPN tunnels.