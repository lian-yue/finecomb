# Live services

Applies to: targets given only as an address, domain name or URL. This is a black-box review; conclusions only cover what can be observed from outside.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Authorization and scope** | Are target ownership, the allowed addresses and time windows, and the allowed operations (passive observation only, or active testing allowed) confirmed in writing; without confirmation, only observe passively |
| Exposed surface | Open ports and services, subdomains, host names in certificate transparency logs, public management interfaces and debug interfaces |
| Transport layer | Protocol versions, certificate chain and validity period |
| Responses and headers | Security response headers (including HSTS), cookie attributes; response headers and error pages that leak versions and internal information |
| Publicly exposed sensitive files | Version control directories, environment variable files, backup files, source maps, directory listings |
| Public API descriptions | Whether `openapi.json`, `swagger.json`, GraphQL introspection and configuration under `/.well-known/` are exposed; once you have them, check per [API specification files](api-specification-files.md) |
| Cross-origin access | A plain request with an external `Origin` header is enough to see whether any origin is reflected with credentials allowed (see [4.14](../specialties/4.14-server-request-handling-and-middleware.md)) |
| Frontend resources | Third-party scripts referenced by the pages and their integrity checks; check bundled scripts per [frontend bundles and source maps](frontend-bundles-and-source-maps.md) |
| **Dangling DNS and subdomain takeover** | DNS records point to cloud resources, storage buckets or third-party services that have been released; once someone else claims them, they can serve content under your domain |
| Domains and mail | Check per [domains, DNS and mail configuration](domains-dns-and-mail-configuration.md) |
| Security contact | Is there a `/.well-known/security.txt`; has its `Expires` date passed |
| Versions and known vulnerabilities | Known vulnerabilities for the products and versions that can be identified; note in the conclusion that they are inferred from versions |
| Active testing | Login attempts, scanning, fuzzing and vulnerability verification are done only within the authorized scope; no denial of service, and no reading of real user data |

When you need to map to OWASP ASVS, what outside observation alone can verify is mainly some of the requirements in version 5.0 chapters V3 (web frontend security), V12 (secure communication) and V13 (configuration); with a test account, add some of the requirements in V6 (authentication), V7 (sessions) and V8 (authorization). The other chapters need source code or internal evidence; record them as "Not checked" in the coverage record.
