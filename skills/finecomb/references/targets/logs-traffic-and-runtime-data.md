# Logs, traffic and runtime data

Applies to: log exports, packet captures (pcap), HAR files, monitoring data and so on.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Evidence integrity** | Compute the digest on receipt, and record the source, time range and time zone; analyze on a copy; record where the packet capture was taken |
| Coverage and gaps | Are there gaps in the timeline (rotation, sampling, drops, clock jumps); conclusions for gap periods can only be "cannot be determined" |
| Sensitive data | Personal information, tokens and passwords in log and traffic samples (see [30](../dimensions/30-privacy-data-governance-and-compliance.md), [4.4](../specialties/4.4-cryptography-and-credentials.md)) |
| **Session material** | HAR files and debug packet captures carry cookies and tokens and are credentials in themselves. In 2023, HAR files were stolen from a customer support system, which led to session hijacking |
| Plaintext protocols and credentials | HTTP basic authentication, FTP, Telnet, plaintext SMTP login; unencrypted traffic between internal services |
| Transport parameters | The negotiated TLS version and cipher suite; the certificate chain (under TLS 1.3 certificates are encrypted and cannot be seen in captures); whether the client keeps connecting after a certificate error |
| Unusual outbound connections | Unexpected outbound destinations; callbacks at fixed intervals; very long random subdomains in DNS queries |
| Completeness of audit events | Are all expected security events recorded (see [4.23](../specialties/4.23-security-audit-logs.md)) |
| Log injection | Forged line breaks and fields, forged events (see [4.23](../specialties/4.23-security-audit-logs.md)) |
| Signs of attack | Obvious probing, exploitation attempts, unusual authentication; when an intrusion is suspected, report it through the project's incident response process; this checklist does not replace forensics |
