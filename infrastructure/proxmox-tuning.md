---
date: 2026-04-22
author: Austin Silva
topic: Virtualization, ZFS, High Availability
---

# Hypervisor Tuning & Resource Management

## OPNsense Resource Allocation
The OPNsense firewall is virtualized on top of a bare-metal Proxmox hypervisor. 

**Specifications:**
* 4 vCPUs
* 4GB RAM

### ZFS ARC Memory Utilization
Initial hypervisor monitoring flagged OPNsense memory utilization at a constant ~100%. Upon deeper system-level investigation via the FreeBSD `top` command, it was determined this was a benign artifact of the **ZFS ARC (Adaptive Replacement Cache)**.

FreeBSD intentionally consumes unallocated RAM to use as an ultra-fast read cache (Wired ARC memory). This memory remains fluid; ZFS will instantly release cached memory back to the system if OPNsense experiences a state-table spike (e.g., heavy connections or large file transfers). 

*Conclusion:* The allocated 4GB is well within safe thresholds for routing multiple VLANs, passing gigabit internal traffic, and managing Tailscale cryptography. RAM will only need to be scaled up if full Intrusion Detection/Prevention (IDS/IPS) like Suricata is deployed in the future.

### High-Availability Safeguards
Following a temporary power anomaly that resulted in a network outage, the hypervisor configuration was hardened. Core infrastructure VMs (OPNsense, internal DNS) have the **Start at boot** parameter enabled at the hypervisor level. This ensures that if the bare-metal server loses power, the core network gateway will automatically boot and establish routing without human intervention.