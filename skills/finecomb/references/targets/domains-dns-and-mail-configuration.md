# Domains, DNS and mail configuration

Applies to: targets given only as a domain name, or given as DNS zone files or mail service configuration. For subdomain takeover, see [live services](live-services.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| **SPF** | The record is not unique or has syntax errors; it ends with `+all` or `?all`; it needs more than 10 DNS lookups, after which the whole record fails; it `include`s a shared sending platform, so other customers on that platform can also send mail as your domain |
| **DMARC** | No record; the policy stays at `p=none` for a long time; the subdomain policy; whether the report addresses are under your control; alignment mode; check for deprecated tags against the current standard (the new version of the standard removed `pct`) |
| DKIM | Keys in use are shorter than 1024 bits (2048 bits recommended); still signing with `rsa-sha1`; old selectors and test keys have not been removed |
| Domains that do not send mail | Domains and subdomains that do not send mail do not declare `v=spf1 -all` and `p=reject`, which makes them easy to spoof |
| Transport encryption | Is the MTA-STS policy set to `enforce`, and do the MX entries in the policy match the actual ones; TLS reporting; DANE when DNSSEC is present |
| **DNSSEC** | Not signed; the DS and DNSKEY chain is broken; outdated algorithms; signatures about to expire; the zone contents can be walked when NSEC is used |
| **CAA** | No restriction on which authorities can issue certificates; `issuewild` for wildcard certificates; the `iodef` notification address |
| Delegation and registration | Name servers point to a hosting provider that is no longer in use (name server takeover); the domain is about to expire; no registrar lock |
| Zone transfer and open resolvers | Authoritative servers allow zone transfers from any source; recursive resolution offered to the outside, which can be used for amplification attacks. Both need active queries, so handle them according to the authorization |
| Leaks in records | Internal information and verification tokens in TXT records; internal addresses in resolved records |
