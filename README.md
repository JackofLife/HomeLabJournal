# HomeLabJournal

OpSec & Documentation Standards
To prevent the leakage of sensitive network topology and infrastructure data, this repository strictly adheres to industry-standard sanitization practices:

Domains: All internal domains are masked using the RFC 2606 example placeholder combined with the RFC 8375 local TLD (e.g., *.example.home.arpa).

Subnets: Internal RFC 1918 private subnets are masked using bracketed variables for the defining octets to demonstrate architecture without revealing exact routing (e.g., 10.[VLAN_ID].X.0/24).

Specific Hosts: Target IP addresses are represented by descriptive variables (e.g., <FIREWALL_IP>, <PROXMOX_IP>).

VPN Overlays: Provider-specific CGNAT IP ranges are generalized (e.g., 100.X.X.X).
