# Network devices and firewall configuration

Applies to: configuration exports or rule sets of routers, switches, firewalls, load balancers and VPN gateways.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Management plane exposure** | Telnet, HTTP or SNMP open to the internet or to user network segments; management access not restricted by source address; management interfaces open to the internet, which is where many gateway vulnerabilities of recent years came in |
| Passwords and keys | How local passwords are stored (for example, Cisco type 7 is reversible and type 5 is MD5; type 8 or 9 should be used); SNMP still on v1 or v2c with community strings `public` or `private`; strength of pre-shared keys |
| **Overly broad rules** | `any any` allow rules; rules shadowed by earlier rules, or rules that never match; temporary rules kept for a long time; no restriction on outbound traffic (see [4.29](../specialties/4.29-rule-and-policy-matching.md)) |
| Authentication and accounting | Is centralized authentication used (TACACS+, RADIUS); emergency local accounts; command accounting |
| Encryption protocols | SSH version and algorithms; for VPNs, the IKE version and cipher suites, and whether certificates or pre-shared keys are used |
| Routing and layer 2 | Are routing protocols authenticated; are unused ports shut down; DHCP snooping, ARP inspection |
| Logs and time | Where logs are sent; is NTP configured and authenticated |
| Firmware version | Compare the version against vendor security advisories (check per [36](../dimensions/36-supply-chain-and-artifact-integrity.md) ("Known vulnerabilities")); note in the conclusion that it is inferred from the version |
