# Dependency manifests, lock files and SBOMs

Applies to: targets that only have dependency manifests, lock files, a software bill of materials (SBOM) or a Vulnerability Exploitability eXchange (VEX) document.

| Checkpoint | What counts as a problem |
| --- | --- |
| Known vulnerabilities and fix versions | Check per [36](../dimensions/36-supply-chain-and-artifact-integrity.md) ("Known vulnerabilities", "Security fix versions") |
| Support lifetime and maintenance status | Is the component past its support lifetime, or no longer maintained |
| Licenses | Are the licenses compatible with the way it is distributed |
| Private package names | Has someone else already registered a private package name on a public registry (dependency confusion) |
| Integrity | Does the lock file include checksums; do the manifest and the lock file agree |
| **Manifest matches artifact** | Whether the SBOM was generated from source code, from the build process or from the final artifact; compared with the artifact actually released, it misses statically linked libraries, copied-in code and components downloaded at runtime |
| Minimum elements | Are the component name, version, producer, unique identifier (purl or CPE), dependency relationships, component hash, license, generating tool and time of generation, and the manifest's author and timestamp all present; components without a unique identifier cannot be checked for vulnerabilities |
| Valid format | Does it pass CycloneDX or SPDX format validation; when a version or purl is written wrong, scanners silently skip that component |
| **VEX conclusions are justified** | Do entries marked "not affected" state a justification (component not present, vulnerable code not present, not in the execution path, attacker cannot control the input, inline mitigation already in place), and does the justification match the code and configuration; who issued it and when; has it gone stale after the artifact was upgraded |
| Signed manifests | Are the SBOM and VEX signed, or bound to the artifact digest as an attestation |
