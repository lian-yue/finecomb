# 36 Supply chain and artifact integrity

| Checkpoint | What counts as a problem |
| --- | --- |
| Trusted sources | Are the dependency sources trustworthy; are there impostor packages with look-alike names; is anything pulled from unofficial channels |
| **Dependency confusion** | Internal package names can also be resolved from public registries; is the build locked to use only internal sources |
| Locks and checksums | Is the lock file committed; is there checksum verification; can dependencies be swapped silently |
| **Known vulnerabilities** | Judge whether the target is affected from the actually locked versions, transitive dependencies, fork patches, build conditions, reachable paths and vendor advisories; record the tool and the date of the vulnerability database; a package-name match is not the same as an exploitable vulnerability, and an empty result is not proof of safety. By default, query public vulnerability databases and official advisories online (allowed scope in [the target project's hard boundaries](../scope.md#the-target-projects-hard-boundaries)); without network access, state the date of the offline database and what could not be checked |
| **Security fix versions** | The locked version is behind a security fix already released upstream; check the registry and release notes for the lowest version with the fix, the latest supported version, and whether upgrading crosses a breaking change |
| Maintenance status | Is the dependency abandoned, unmaintained for a long time, run by a single maintainer, or recently handed over to someone new |
| **Support lifetime** | Are the language runtime, operating system, base image or framework past their official support period, so they no longer get security fixes |
| **Nonexistent or newly registered package names** | A dependency's package name does not exist in the registry, was registered only recently, or has unusually low downloads; AI-generated code often brings in made-up package names, and attackers register those names first |
| **Binaries and archives in the repository** | Executables, prebuilt libraries, archives and test binaries committed to the repository: where do they come from, and can they be rebuilt from source; can build scripts read and execute their contents |
| Extra files shipped with dependencies | Can examples, tests, debug pages or scripts shipped inside dependency packages be reached from outside after they are deployed along with it |
| Reproducible builds | Does the same input produce the same artifact; are timestamps, absolute paths or build machine information embedded |
| Version traceability | Where does the version marker in the artifact come from; can it be matched to the source |
| Signing and distribution | Are artifacts signed; are the distribution channels trustworthy |
| **What a signature covers** | Which bytes and files the signature and integrity checks actually cover; parts that are loaded or executed at runtime but are outside that coverage (file headers, appended sections, resources, snapshots, config, plugin directories) |
| **Names re-registered after they lapse** | After external resources that dependencies, build scripts, docs or DNS refer to by name (domains, subdomains, storage buckets, package names, repository paths, the domains of maintainer email addresses) are deleted, renamed or expire, can someone else register and take them over |
| Code execution at build time | Can the build run arbitrary code from dependencies (install scripts, generation steps, plugins) |
| Build and release permissions | Can untrusted commits, dependency scripts or workflow inputs read release credentials, write artifacts or deploy; are build jobs, runners and release identities isolated from each other |
| Build cache and artifact substitution | Can untrusted jobs poison caches or intermediate outputs that trusted jobs use; is the identity and digest checked in the final signature verification bound to the artifact actually deployed; can tags or download URLs be swapped |
| Component inventory | Can the runtime, base images, dependencies and plugins be traced from the actual artifact; does the existing software bill of materials (SBOM) match the artifact; do dynamic downloads bypass locking and integrity checks |
| Licenses | Are dependency licenses compatible with the way this project is distributed; are the notice files complete |
