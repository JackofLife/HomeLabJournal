🛠️ Incident Report: Diagnosing a Complete Homelab Network Outage

Scenario: Returned to the local environment after working remotely. Devices successfully connected to the local Wi-Fi access point, but reported "Connected, No Internet." Physical hardware status lights indicated normal operation.
Objective: Systematically isolate the point of failure across the VPN, VLANs, physical switch, firewall, and hypervisor to restore internet connectivity.

Phase 1: Initial Triage & VPN Isolation

- Action: Disabled the remote access VPN (Tailscale) to eliminate potential routing loops or MagicDNS conflicts while physically on-premise.
- Diagnostic: Ran ipconfig on the local machine.
- Result: Discovered the wireless adapter was pulling an APIPA address (169.254.x.x). Ping tests to the default gateway (<FIREWALL_IP>) failed.
- Deduction: The laptop was successfully authenticating to the Wi-Fi, but DHCP broadcasts were not reaching the firewall, or the firewall was down.

Phase 2: Layer 1 & Layer 2 Bypass (Switch Verification)

- Action: Performed a standard power cycle of the Wireless Access Point (WAP) and managed network switch. After reboot, Wi-Fi still pulled an APIPA.
- Action: Bypassed the WAP entirely by hardwiring the laptop directly into the managed switch via Ethernet.
- Result: The Ethernet adapter successfully pulled a valid, routable IP address in the expected <PERSONAL_VLAN_SUBNET>.
- Deduction: The managed switch and the firewall's DHCP server were actually online and functioning!

Phase 3: VLAN Isolation & The Physical Bypass

- Action: Attempted to access the firewall web GUI (<FIREWALL_IP>) via the hardwired switch connection, but the connection timed out.
- Deduction: The switch port was strictly tagged for the Personal VLAN, and standard firewall rules enforce strict VLAN isolation, blocking access to the Management interface.
- Action: Bypassed the switch entirely. Disconnected the primary LAN cable from the switch and hardwired the laptop directly into the hypervisor's dedicated LAN NIC.
- Action: Configured a static "Backstage Pass" IP (<STATIC_MGMT_IP>) on the laptop to force it onto the Management subnet.
- Result: Connection to the firewall GUI still timed out, indicating the core router itself was inaccessible.

Phase 4: Hypervisor Triage & Root Cause Analysis

- Action: Pivoted from targeting the virtualized firewall to targeting the bare-metal hypervisor (Proxmox) web GUI (<HYPERVISOR_IP>).
- Result: The hypervisor GUI loaded instantly.
- Action: Checked the console for the virtualized firewall (OPNsense).
- Result: The VM state showed as "Guest not running." Checked the bare-metal host uptime and noted it was only a few hours.
- Root Cause: A brief environmental power flicker caused the physical server to reboot. While the bare-metal hypervisor booted back up successfully, the firewall VM was not configured to start automatically, leaving the network without a router.

Resolution & Hardening

1. Immediate Fix: Manually started the firewall VM. Full network routing and internet access were restored within 60 seconds.
2. Permanent Hardening: Edited the hypervisor configurations for the firewall and other core infrastructure VMs, enabling the "Start at boot" flag to ensure automatic disaster recovery in the event of future power losses.
Takeaway: When troubleshooting complex segmented networks, never assume the core is dead just because you can't reach it. By methodically bypassing the WAP, the Switch, and the VLAN tagging, I was able to pinpoint the exact failure state without disrupting the rest of the working architecture.
