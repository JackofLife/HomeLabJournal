---
date: 2026-04-20
author: Austin Silva
topic: VPN, Zero-Trust, Remote Access
---

# Remote Access via Tailscale Subnet Routing

## Objective
Establish secure, zero-trust remote access to the internal network without exposing public-facing ports on the firewall or requiring VPN clients to be installed on every internal virtual machine.

## Implementation: OPNsense Subnet Router
Instead of installing Tailscale on individual VMs (Proxmox, TrueNAS, etc.), Tailscale was deployed directly at the network edge on the OPNsense firewall.

1. **Plugin Installation:** Installed `os-tailscale` via the OPNsense package manager.
2. **Headless Authentication:** Authenticated the firewall to the Tailnet silently using a Pre-Authentication Key, bypassing standard interactive SSO loops.
3. **Subnet Advertisement:** Configured the OPNsense node to advertise the internal RFC 1918 subnets (`10.[VLAN_MGMT].0/24`, `10.[VLAN_PERS].0/24`, etc.) to the Tailscale overlay network.
4. **MagicDNS Handoff:** Integrated Tailscale MagicDNS to natively resolve `*.example.home.arpa` domain names remotely.

## Security Considerations
* By operating as a Subnet Router, internal nodes remain entirely unaware of the VPN overlay. They do not receive `100.X.X.X` Tailscale addresses, maintaining internal network purity.
* Resolves "Overlapping Subnet" conflicts (common on public Wi-Fi networks utilizing default `10.0.0.0/24` scopes) by forcefully tunneling the designated home lab requests through the WireGuard-backed Tailscale interface rather than local Captive Portals.