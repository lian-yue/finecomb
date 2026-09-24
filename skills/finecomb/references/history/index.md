# Historical vulnerability patterns

This file turns high-severity, widely exploited vulnerabilities from the past into general mechanisms. Use it during a review to "go over the code once more with history's lessons in mind" and avoid repeating old mistakes.

How to use it:

- After [establishing the factual baseline](../baseline.md), pick the groups that match the mechanisms the target uses and compare row by row. The "Where it lands" column points to the dimension or specialty that holds the detailed criteria.
- Each row is a generalized **mechanism**. The historical cases only help you recognize it; they are not a complete list. The same mechanism still holds in another language, framework or product.
- Matching a pattern does not mean a vulnerability exists. You still need to check reachable paths and blocking points per [Threat model and attack chains](../baseline.md#threat-model-and-attack-chains), and record the result with an [evidence level](../report.md#part-v-evidence-levels-and-report-format).
- Many historical vulnerabilities come from **combinations**: each step alone looks like "just a small issue". When you find one pattern, look in the same group and nearby groups for the other half that can be chained with it.
- Incomplete fixes, fixes that are later bypassed, and regressions are common (see the last row of [14 Supply chain, build and release](14-supply-chain-build-and-release.md)).

- [1 Expressions, templates and lookup languages](1-expressions-templates-and-lookup-languages.md)
- [2 Deserialization and object graphs](2-deserialization-and-object-graphs.md)
- [3 Commands, file names and environments assembled later](3-commands-file-names-and-environments-assembled-later.md)
- [4 Query injection](4-query-injection.md)
- [5 Parsing and normalization differentials](5-parsing-and-normalization-differentials.md)
- [6 Authentication, sessions and identity](6-authentication-sessions-and-identity.md)
- [7 Authorization and privilege escalation](7-authorization-and-privilege-escalation.md)
- [8 Memory safety](8-memory-safety.md)
- [9 Resource exhaustion and protocol-level denial of service](9-resource-exhaustion-and-protocol-level-denial-of-service.md)
- [10 Cryptography and randomness](10-cryptography-and-randomness.md)
- [11 SSRF, internal trust and exposed services](11-ssrf-internal-trust-and-exposed-services.md)
- [12 Files, archives and decoder side effects](12-files-archives-and-decoder-side-effects.md)
- [13 Containers, kernels and privilege](13-containers-kernels-and-privilege.md)
- [14 Supply chain, build and release](14-supply-chain-build-and-release.md)
- [15 Secrets, debug interfaces and information leaks](15-secrets-debug-interfaces-and-information-leaks.md)
- [16 Business logic and concurrency](16-business-logic-and-concurrency.md)
- [17 Clients, browsers and sandboxes](17-clients-browsers-and-sandboxes.md)
- [18 Smart contracts and on-chain](18-smart-contracts-and-on-chain.md)
- [19 Blockchain protocols and zero-knowledge proofs](19-blockchain-protocols-and-zero-knowledge-proofs.md)

## Sources

The material below was used to derive the patterns above. For any specific vulnerability, its own official advisory is the authority.

- [CWE Top 25 (2024)](https://cwe.mitre.org/top25/archive/2024/2024_top25_list.html)
- [CISA AA24-317A: The most routinely exploited vulnerabilities of 2023](https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-317a)
- [OWASP API Security Top 10 (2023)](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
- [OWASP Smart Contract Top 10](https://scs.owasp.org/sctop10/)
- [Okta advisory on bcrypt truncation](https://trust.okta.com/security-advisories/okta-ad-ldap-delegated-authentication-username/)
- [Microsoft's investigation into the Storm-0558 key acquisition](https://www.microsoft.com/en-us/msrc/blog/2023/09/results-of-major-technical-investigations-for-storm-0558-key-acquisition)
- [Zcash Orchard vulnerability advisory (GHSA-ww9q-8r59-xv46 / CVE-2026-54496)](https://osv.dev/vulnerability/GHSA-ww9q-8r59-xv46)
- [Trail of Bits: the Frozen Heart disclosure series](https://blog.trailofbits.com/2022/04/13/part-1-coordinated-disclosure-of-vulnerabilities-affecting-girault-bulletproofs-and-plonk/)
- [Bitcoin Core: CVE-2018-17144 disclosure](https://bitcoincore.org/en/2018/09/20/notice/)
- [Monero: disclosure of a major bug in CryptoNote-based currencies (2017)](https://www.getmonero.org/2017/05/17/disclosure-of-a-major-bug-in-cryptonote-based-currencies.html)
