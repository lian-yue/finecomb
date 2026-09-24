# Hosts and operating systems

Applies to: servers, virtual machines and workstations logged into with a read-only account, or their configuration exports and baseline check results. Hosts that run a container runtime are also checked per this section.

| Checkpoint | What counts as a problem |
| --- | --- |
| Patches and support lifetime | The system and kernel versions are past their support lifetime; security updates have not been installed |
| Remote login | SSH allows root login or password login, or uses outdated algorithms; Remote Desktop without Network Level Authentication; management ports open to the outside |
| **Local privilege escalation paths** | Passwordless sudo rules; programs with SUID; service files, scheduled job scripts and `PATH` directories that ordinary users can write; on Windows, unquoted service paths, writable service binaries and `AlwaysInstallElevated` |
| Accounts and passwords | Empty passwords, shared accounts, accounts unused for a long time; outdated password hash algorithms (such as `$1$` in `/etc/shadow`) |
| Listening services | Services that listen on all addresses but do not need to be external; unused services still running |
| Host firewall and kernel parameters | The firewall is not enabled or allows everything; kernel parameters such as address randomization, forwarding and source address validation |
| Mandatory access control and encryption | SELinux or AppArmor turned off; disks not encrypted; secure boot not enabled |
| File permissions and integrity | World-writable files and directories; overly broad permissions on private keys and credential files; system files whose checksums differ from what the package manager recorded |
| Logs, auditing and time | Audit rules, log forwarding and retention; time synchronization |
| **Container runtime** | The Docker remote API without authentication; members of the `docker` group are equivalent to root; daemon configuration (user namespaces, inter-container communication) |
