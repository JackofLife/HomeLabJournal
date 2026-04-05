---
date: 2026-04-05
author: <Austin Silva>
topic: Infrastructure, Hardware, Resource Allocation
---

# Home Lab Hardware Sizing & Storage Architecture

## Overview
The core virtualization environment runs on a bare-metal HP ProLiant MicroServer Gen10 Plus. The hardware was specifically selected and upgraded to support enterprise-grade Cybersecurity simulation (SIEM, malware detonation) alongside standard infrastructure services (NAS, routing, media).

## Hardware Specifications
* **CPU:** Intel Xeon E-2246G (6-Core / 12-Thread) @ 3.6GHz Base / 4.8GHz Turbo. 
  * *Note:* Features Intel QuickSync (integrated GPU) which offloads media transcoding, preserving CPU cycles for SIEM ingestion and VM compute.
* **RAM:** 64GB ECC (Error-Correcting Code) Registered RDIMM. 
  * *Allocation Strategy:* Provides sufficient overhead for ZFS caching, SIEM indexers (~16GB), NAS (~16GB), and multiple containerized workloads.

## Storage Tiering Architecture
To prevent IOPS (Input/Output Operations Per Second) bottlenecks—the most common failure point when running a SIEM and NAS on the same host—storage is segmented into Hot and Cold tiers.

* **Hot Tier (Speed / Application Layer):** 240GB Enterprise SSD
  * Dedicated to Proxmox OS, VM boot drives, and SIEM Hot/Warm index buckets. Ensures instantaneous search queries and rapid container startup.
* **Cold Tier (Capacity / Storage Layer):** 2x 6TB WD Red HDDs
  * Dedicated to TrueNAS/storage pools. Used exclusively for static media, backups, and long-term SIEM cold archive storage.