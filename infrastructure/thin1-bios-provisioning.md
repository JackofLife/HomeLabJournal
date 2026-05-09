---
date: 2026-05-09
author: Austin Silva
topic: Hardware, BIOS, Out-of-Band Management, Proxmox
---

# `thin1` Hardware Provisioning & BIOS Configuration

## Overview
This document outlines the standardized bare-metal provisioning process for the `thin1` node (and future thin clients) before the Proxmox OS is installed. The goal is to establish rock-solid remote management, ensure high availability after power events, and enable hardware passthrough for virtualization.

## Intel Active Management Technology (AMT/MEBx) Setup
To provide Out-of-Band (OOB) management (Remote KVM) completely independent of the operating system, Intel MEBx was configured at Ring -3.

### Configuration Steps:
1. **Network Settings:**
   * **DHCP Mode:** Enabled (Relies on OPNsense firewall for static IP assignment via MAC address reservation).
   * **Domain Name:** Set to the internal routing domain `example.home.arpa`.
   * **Shared/Dedicated FQDN:** Shared (Shares the physical NIC and IP address with the Proxmox host OS).
   * **Dynamic DNS Update:** Disabled.
2. **Security Hardening:**
   * Changed default AMT password to a strong, complex credential.
   * "User Consent" set to 'None' to allow unattended remote KVM access.
3. **Verification:**
   * Successfully tested remote login via `http://<THIN1_IP>:16992` from the Management VLAN.

## UEFI / BIOS Configuration Checklist
The following parameters were modified from factory defaults to optimize the hardware for a 24/7 hypervisor workload.

### ✅ Applied Configurations
* **Update BIOS/UEFI:** Flashed to the latest vendor firmware to ensure security patches and compatibility.
* **Security:** Set a strong BIOS Admin Password (required replacing the physical motherboard jumper to enable password enforcement).
* **Virtualization:** Enabled **Intel VT-d** (IOMMU) for Directed I/O, allowing PCI passthrough of network cards or USB controllers directly to VMs.
* **Boot Speed:** **Fastboot Disabled** (Ensures hardware initializes fully and cleanly during warm reboots).
* **Power Management:**
  * **After Power Loss:** Set to **Power On** (Ensures the node automatically recovers after an outage).
  * **Wake on LAN (WoL):** **Enabled** (Allows remote power-on magic packets via the network).
  * **S5 Maximum Power Savings:** **Disabled** (Crucial: Keeps the NIC active during shutdown states so WoL and Intel AMT remain reachable).
* **Replication:** Backed up final BIOS settings to a USB drive to quickly clone this profile to future thin client nodes.

### ❌ Skipped or N/A Configurations
* *Onboard Devices (Audio, Serial, Parallel):* Skipped disabling these; leaving them active does not currently impact our resource allocation.
* *Boot Order:* Left default; manual boot override was used for the initial Proxmox USB install.
* *Fan Speed Control:* N/A on this specific hardware revision.
* *SATA Mode (AHCI):* N/A (System utilizes NVMe storage).
* *Halt on POST Errors:* N/A (Option not present in this UEFI version).