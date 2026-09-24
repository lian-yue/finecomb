# Targets that are not source code

A review target is not always source code. This file explains, for targets that are compiled artifacts, installers, extensions, images, published packages, live addresses, hosts, clusters, account and tenant state, configuration, data, logs, contract addresses, documents or agent configurations, what evidence you can get, what to check first, and what you cannot see. The detailed criteria for each checkpoint are still in the [general dimensions](dimensions.md) and [specialties](specialties.md); this file only covers the evidence-gathering methods and checkpoints specific to these targets. Still go through the [root-cause facets](facets.md) for every object.

## General principles

- **First ask whether it can be swapped for source code.** When source code, symbols, build configuration, an SBOM or design documents are available, prefer that stronger evidence; for on-chain contracts, prefer source code verified on a block explorer. When .NET and JVM bytecode, packaged Python, source maps or Electron asar archives can be restored to something close to source code, review the restored code as source code, and state how it was restored and what is missing.
- **Pin down the target's identity.** Record the artifact's digest (such as SHA-256), version and signature; for live targets, record the address and access time; for on-chain targets, record the chain ID, the contract address and the block height at read time; for accounts, clusters and tenants, record their identifiers and the identity used to read them. Put these in the "Conclusion and scope" part of the report.
- **Preserve evidence.** When you receive an artifact, export or data, first compute its digest and record the source, who delivered it, how it was obtained and when (with time zone; using UTC throughout is recommended). Keep the original read-only and do the analysis on a copy. For the raw output behind each conclusion (responses, screenshots, command output, packet captures), also record its digest, time and storage location.
- **Conclusions expire.** The state of live services, accounts, clusters and DNS can change at any time, so a conclusion only holds for the moment of reading. The report states the read time; when rechecking, read again instead of reusing old conclusions.
- **Evidence-gathering methods have levels.** Static inspection of artifacts is allowed by default; running artifacts is done only in an isolated environment; active probing, login attempts and fuzzing against live targets, and any on-chain transaction, need written authorization (see [execution boundaries during review](scope.md#execution-boundaries-during-review)). When reverse engineering is restricted by a license agreement or local law, confirm first.
- **Record dimensions you cannot see as they are.** Without source code, the code-shape dimensions ([1](dimensions.md#1-dead-code-and-reachability) to [7](dimensions.md#7-complexity-and-maintainability)) usually do not apply or can only be partially checked, and internal logic can only be inferred from behavior. In the coverage record, write "Partially checked" or "Not checked" with the reason; do not write it as "Checked, none found".
- **Keep strong and weak evidence apart.** Known vulnerabilities inferred from version numbers and logic inferred from decompilation are usually at evidence level "Unverified", unless there is a reproducible observation.

## Quick reference

| Target form | Key dimensions and specialties | Usually not visible |
| --- | --- | --- |
| Compiled artifacts and binaries | [16](dimensions.md#16-language-and-runtime-pitfalls), [27](dimensions.md#27-security-and-trust-boundaries), [36](dimensions.md#36-supply-chain-and-artifact-integrity), [38](dimensions.md#38-portability-and-build-context), [4.4](specialties.md#44-cryptography-and-credentials), [4.30](specialties.md#430-os-interfaces-system-calls-and-descriptors), [4.32](specialties.md#432-cross-language-boundaries-and-native-extensions), [4.35](specialties.md#435-client-apps-extensions-and-auto-update); for privileged services and setuid programs see [4.47](specialties.md#447-local-privileged-components-and-local-privilege-escalation), for drivers see [4.48](specialties.md#448-kernel-drivers-device-emulation-and-virtualization) | Naming, comments, tests, design intent; build conditions and deployment configuration |
| Desktop installers and desktop apps | [4.35](specialties.md#435-client-apps-extensions-and-auto-update), [4.47](specialties.md#447-local-privileged-components-and-local-privilege-escalation), [36](dimensions.md#36-supply-chain-and-artifact-integrity), [28](dimensions.md#28-authorization-and-access-control), [34](dimensions.md#34-runtime-environment-and-deployment-contract), [4.8](specialties.md#48-file-systems-and-paths) | The server side; components downloaded after installation; behavior that only appears on specific OS versions |
| Mobile app packages | [4.49](specialties.md#449-mobile-app-components-and-inter-process-communication), [4.35](specialties.md#435-client-apps-extensions-and-auto-update), [4.4](specialties.md#44-cryptography-and-credentials), [4.5](specialties.md#45-authentication-sessions-and-tokens), [4.1](specialties.md#41-networking-and-connections), [30](dimensions.md#30-privacy-data-governance-and-compliance), [36](dimensions.md#36-supply-chain-and-artifact-integrity), [4.18](specialties.md#418-user-interfaces-and-accessibility) | Server-side logic and authorization; configuration and code delivered at runtime; the main binary encrypted by the app store (when not decrypted) |
| Browser extensions | [4.35](specialties.md#435-client-apps-extensions-and-auto-update), [4.18](specialties.md#418-user-interfaces-and-accessibility), [30](dimensions.md#30-privacy-data-governance-and-compliance), [4.4](specialties.md#44-cryptography-and-credentials) | What the backend does with the data after receiving it; configuration delivered at runtime |
| Firmware and device images | [4.36](specialties.md#436-embedded-firmware-and-real-time-constraints), [27](dimensions.md#27-security-and-trust-boundaries), [36](dimensions.md#36-supply-chain-and-artifact-integrity), [4.4](specialties.md#44-cryptography-and-credentials), [4.14](specialties.md#414-server-request-handling-and-middleware) | Whether fuses, read protection and secure boot are enabled on the physical device; the boot ROM; companion apps and the cloud |
| Device communication and IoT protocols | [4.1](specialties.md#41-networking-and-connections), [4.2](specialties.md#42-protocols-and-frame-parsing), [4.4](specialties.md#44-cryptography-and-credentials), [4.36](specialties.md#436-embedded-firmware-and-real-time-constraints) | The device's internal implementation; pairing and network joining that were not captured |
| Container images | [34](dimensions.md#34-runtime-environment-and-deployment-contract), [36](dimensions.md#36-supply-chain-and-artifact-integrity), [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code) | Orchestration configuration at runtime |
| Published packages | [36](dimensions.md#36-supply-chain-and-artifact-integrity), [4.34](specialties.md#434-publishable-libraries-sdks-and-packages), [4.22](specialties.md#422-subprocesses-dynamic-execution-and-decoder-side-effects), [35](dimensions.md#35-dependencies) | Distribution channels other than the registry; content downloaded only after installation |
| Signatures and provenance | [36](dimensions.md#36-supply-chain-and-artifact-integrity), [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code), [4.4](specialties.md#44-cryptography-and-credentials) | Whether the build platform is really isolated internally; whether the signer itself has been compromised |
| Dependency manifests, lock files and SBOMs | [35](dimensions.md#35-dependencies), [36](dimensions.md#36-supply-chain-and-artifact-integrity) | Whether the dependencies are actually called; whether the manifest matches the artifact actually released |
| Live services | [27](dimensions.md#27-security-and-trust-boundaries), [34](dimensions.md#34-runtime-environment-and-deployment-contract), [4.1](specialties.md#41-networking-and-connections), [4.14](specialties.md#414-server-request-handling-and-middleware), [4.5](specialties.md#45-authentication-sessions-and-tokens), [4.18](specialties.md#418-user-interfaces-and-accessibility) | Internal implementation, the data layer, unexposed interfaces; features behind login when there is no test account; the origin server behind a CDN or protection layer |
| Frontend bundles and source maps | [4.18](specialties.md#418-user-interfaces-and-accessibility), [4.4](specialties.md#44-cryptography-and-credentials), [36](dimensions.md#36-supply-chain-and-artifact-integrity), [28](dimensions.md#28-authorization-and-access-control) | Server-side logic and authorization |
| API specification files | [10](dimensions.md#10-apis-and-contracts), [28](dimensions.md#28-authorization-and-access-control), [4.14](specialties.md#414-server-request-handling-and-middleware), [20](dimensions.md#20-resource-bounds-and-backpressure) | Whether the authentication and authorization written in the spec are actually enforced |
| Domains, DNS and mail configuration | [4.40](specialties.md#440-notifications-and-outbound-messages), [4.1](specialties.md#41-networking-and-connections), [34](dimensions.md#34-runtime-environment-and-deployment-contract) | DKIM selectors (they cannot be listed from outside; you need the headers of a sample email); the internal DNS view; inbound filtering at the mail gateway |
| Hosts and operating systems | [34](dimensions.md#34-runtime-environment-and-deployment-contract), [28](dimensions.md#28-authorization-and-access-control), [32](dimensions.md#32-configuration-and-defaults), [4.47](specialties.md#447-local-privileged-components-and-local-privilege-escalation), [4.23](specialties.md#423-security-audit-logs), [36](dimensions.md#36-supply-chain-and-artifact-integrity) | Application logic; files a read-only account cannot read; changes after the check |
| Cloud accounts and infrastructure state | [28](dimensions.md#28-authorization-and-access-control), [34](dimensions.md#34-runtime-environment-and-deployment-contract), [4.4](specialties.md#44-cryptography-and-credentials), [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code), [4.23](specialties.md#423-security-audit-logs) | Application code; host internals; organization-level policies, other accounts and regions that a read-only identity cannot see |
| Kubernetes clusters | [28](dimensions.md#28-authorization-and-access-control), [34](dimensions.md#34-runtime-environment-and-deployment-contract), [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code), [4.23](specialties.md#423-security-audit-logs), [36](dimensions.md#36-supply-chain-and-artifact-integrity) | Application code; configuration of a managed control plane; node state when you have no node access |
| Configuration, deployment manifests and infrastructure code | [32](dimensions.md#32-configuration-and-defaults), [34](dimensions.md#34-runtime-environment-and-deployment-contract), [28](dimensions.md#28-authorization-and-access-control), [4.4](specialties.md#44-cryptography-and-credentials), [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code) | How the program interprets the configuration; runtime overrides (environment variables, manual changes in the console) |
| Network devices and firewall configuration | [4.29](specialties.md#429-rule-and-policy-matching), [28](dimensions.md#28-authorization-and-access-control), [4.1](specialties.md#41-networking-and-connections), [4.23](specialties.md#423-security-audit-logs) | Changes after the export; device firmware internals |
| SaaS tenants and third-party integrations | [28](dimensions.md#28-authorization-and-access-control), [4.5](specialties.md#45-authentication-sessions-and-tokens), [4.46](specialties.md#446-federated-identity-and-single-sign-on), [30](dimensions.md#30-privacy-data-governance-and-compliance), [4.23](specialties.md#423-security-audit-logs), [4.24](specialties.md#424-webhooks-and-external-events) | The platform's own implementation; how third-party apps store the tokens they receive |
| Agent configurations and MCP servers | [4.26](specialties.md#426-llms-and-tool-calling), [36](dimensions.md#36-supply-chain-and-artifact-integrity), [4.4](specialties.md#44-cryptography-and-credentials), [4.14](specialties.md#414-server-request-handling-and-middleware) | The server's implementation code; how the model runtime actually picks tools |
| Databases and datasets | [30](dimensions.md#30-privacy-data-governance-and-compliance), [23](dimensions.md#23-transactions-atomicity-and-consistency), [24](dimensions.md#24-encoding-and-persistent-formats), [28](dimensions.md#28-authorization-and-access-control), [4.5](specialties.md#45-authentication-sessions-and-tokens), [4.7](specialties.md#47-databases-and-queries) | The code paths that read and write this data; the database's own configuration when you only have an export file; changes after the export |
| Logs, traffic and runtime data | [30](dimensions.md#30-privacy-data-governance-and-compliance), [33](dimensions.md#33-observability-and-diagnosability), [4.23](specialties.md#423-security-audit-logs), [4.1](specialties.md#41-networking-and-connections), [4.4](specialties.md#44-cryptography-and-credentials) | The code that produces this data; events that were not logged or were rotated out and dropped; the content of encrypted traffic |
| Incident reports and postmortems | [4.23](specialties.md#423-security-audit-logs), [33](dimensions.md#33-observability-and-diagnosability), [22](dimensions.md#22-fault-isolation-and-degradation), [historical vulnerability patterns](history.md) | Facts the report does not state; the original evidence |
| On-chain contract addresses | [4.38](specialties.md#438-smart-contracts-and-on-chain-interaction), [4.41](specialties.md#441-defi-economics-and-oracles) to [4.43](specialties.md#443-wallets-signing-and-off-chain-components); for contracts that verify zero-knowledge proofs, also see [4.44](specialties.md#444-zero-knowledge-proofs-and-circuits); for smart contract wallets and EIP-7702 delegation, also see [4.50](specialties.md#450-account-abstraction-and-smart-contract-wallets); pick from [A.13](lang-solidity.md) to [A.19](lang-ton.md) by chain | The source-level intent of unverified contracts; off-chain components |
| Design, specification and architecture documents | The threat model in [Part I factual baseline](baseline.md), [Part II](questions.md), [root-cause facets](facets.md), [8](dimensions.md#8-functional-correctness-and-fitness-for-requirements), [9](dimensions.md#9-business-logic-and-flow-integrity) | Whether the implementation matches the design |
| Office documents and PDFs | [4.22](specialties.md#422-subprocesses-dynamic-execution-and-decoder-side-effects), [4.3](specialties.md#43-decoding-untrusted-external-data), [30](dimensions.md#30-privacy-data-governance-and-compliance) | The version and security settings of the software that opens it |
| WebAssembly modules | [4.27](specialties.md#427-interpreters-compilers-and-virtual-machines), [4.32](specialties.md#432-cross-language-boundaries-and-native-extensions), [20](dimensions.md#20-resource-bounds-and-backpressure), [16](dimensions.md#16-language-and-runtime-pitfalls) | Which capabilities the host actually grants |
| Model files and datasets | [4.37](specialties.md#437-data-processing-batch-jobs-and-machine-learning), [4.26](specialties.md#426-llms-and-tool-calling), [4.22](specialties.md#422-subprocesses-dynamic-execution-and-decoder-side-effects) | The training process |
| Game clients and anti-cheat | [4.35](specialties.md#435-client-apps-extensions-and-auto-update), [4.2](specialties.md#42-protocols-and-frame-parsing), [4.39](specialties.md#439-payments-accounting-and-billing), [4.30](specialties.md#430-os-interfaces-system-calls-and-descriptors) | Server-side validation logic |

## Compiled artifacts and binaries

Applies to: executables, dynamic libraries, plugins and drivers. For installers, see [desktop installers and desktop apps](#desktop-installers-and-desktop-apps).

| Checkpoint | What counts as a problem |
| --- | --- |
| Identity and origin | Can the digest, version, build number and signature be checked; who is the signer; can the same artifact be built reproducibly from the source code |
| **Hardening options** | Address randomization (PIE), non-executable stack and data (NX), read-only relocations (RELRO), stack protection, `_FORTIFY_SOURCE`, control-flow protection; on Linux, full read-only relocations (`BIND_NOW`), executable stack (`GNU_STACK`) and CET properties; on Windows, ASLR (including high entropy), DEP, CFG, `/GS`, SafeSEH (32-bit) and CET shadow stack compatibility; on macOS, the hardened runtime and entitlements |
| **Dangerous entitlements** | A macOS release build carries `get-task-allow` (it can be attached to for debugging and injection), `disable-library-validation`, `allow-dyld-environment-variables` or `allow-unsigned-executable-memory` |
| **Embedded secrets and addresses** | Keys, passwords, tokens, internal addresses and debug switches in strings, resources and configuration sections |
| Linked libraries and versions | Statically or dynamically linked libraries and their versions, and whether they have known vulnerabilities (check per [36](dimensions.md#36-supply-chain-and-artifact-integrity)) |
| Embedded build information | Go programs and Rust programs built with `cargo auditable` embed the full list of dependencies and versions, so you can look up vulnerabilities by version directly, which is more reliable than fingerprinting |
| Debug leftovers | Unstripped symbols, debug interfaces, test backdoors, hidden commands |
| Dangerous calls | Command execution, unsafe string functions and dynamic loading in the import table |
| **Library search paths** | ELF RPATH or RUNPATH contains relative paths or writable directories; Mach-O `LC_RPATH` entries and weak links point to libraries that do not exist; the Windows search order finds a same-named DLL in a writable directory first |
| Updates and loading | Auto-update and plugin loading (see [4.35](specialties.md#435-client-apps-extensions-and-auto-update), [4.32](specialties.md#432-cross-language-boundaries-and-native-extensions)) |
| Licenses and notices | License obligations of the third-party components bundled in |

## Desktop installers and desktop apps

Applies to: Windows MSI and EXE installers, macOS DMG and pkg, Linux deb, rpm, AppImage, Snap and Flatpak, and the desktop apps they install (including frameworks with embedded web content such as Electron). Check the executables inside separately per [compiled artifacts and binaries](#compiled-artifacts-and-binaries); for the update mechanism, see [4.35](specialties.md#435-client-apps-extensions-and-auto-update).

| Checkpoint | What counts as a problem |
| --- | --- |
| Signing and notarization | Windows Authenticode signatures and timestamps; macOS developer signing, notarization and stapling; repository signatures for Linux packages; whether every executable in the package is signed; whether the signing certificate has expired or been revoked |
| **Install scripts and custom actions** | MSI custom actions, pkg `preinstall` and `postinstall`, and deb and rpm maintainer scripts usually run with the highest privileges: what they execute, what they download, and whether it is verified |
| **Privilege escalation through repair and uninstall** | MSI files are cached in a system directory, and ordinary users can trigger a repair; can the actions run as SYSTEM during the repair be hijacked (referencing files that do not exist, writing to directories ordinary users can write, popping up a command window) |
| Install location and permissions | A program run by a high-privilege service is installed in a directory ordinary users can write; Windows service paths without quotes |
| **Library search paths** | The installer loads a same-named DLL from the download directory; macOS `@rpath` or Linux RPATH points to a writable directory |
| **Changes to the system** | Installing root certificates (when the private key ships with the installer, anyone can issue trusted certificates); changing hosts, proxy or firewall rules; installing drivers, services or scheduled tasks; whether they remain after uninstall |
| Sandbox and permission declarations | Snap `classic` or `devmode` confinement; `--filesystem=host` in Flatpak `finish-args`, and session bus access to `org.freedesktop.Flatpak` (which can escape the sandbox); the macOS App Sandbox and hardened runtime entitlements |
| Embedded web frameworks | Electron's Node integration, context isolation and fuse settings (`RunAsNode`, asar integrity checks, loading only from asar); source code and secrets in asar archives |
| Bundled runtimes | Versions and support lifetime of the bundled Java, Python, Node and OpenSSL (see [36](dimensions.md#36-supply-chain-and-artifact-integrity)) |

## Mobile app packages

Applies to: installation packages such as APK, AAB and IPA. The full criteria for components and inter-process communication are in [4.49](specialties.md#449-mobile-app-components-and-inter-process-communication); this section only lists what to check first when you have the package.

| Checkpoint | What counts as a problem |
| --- | --- |
| Manifest and configuration | Debuggable, backups allowed, cleartext traffic allowed, network security configuration, iOS ATS exceptions |
| **Exported components and deep links** | Exported activities, services, broadcast receivers and content providers; parameter validation for URL schemes and universal links (see [4.35](specialties.md#435-client-apps-extensions-and-auto-update)) |
| **Embedded secrets** | API keys, client secrets, private service addresses; secrets that belong on the server placed in the client |
| Certificate validation and pinning | Whether certificate validation is skipped; whether certificate pinning is done, and what happens when the pinned certificate expires or is rotated |
| Local storage | Where tokens and personal data are stored (the system keychain or Keystore, or plaintext files, preferences or databases); screenshots and the clipboard |
| **Local authentication** | Biometrics that only return a boolean before letting the user in can be bypassed by hooking; the result should be bound to a key in the system keystore (Android `CryptoObject`, iOS keychain access control) |
| Platform interaction | Mutable `PendingIntent`; forwarding a received Intent as is; overly broad `FileProvider` path configuration (such as `root-path`); sensitive screens without protection against overlays |
| Target OS version | A low `targetSdkVersion` keeps old insecure defaults (below 24, user-installed certificates are trusted; below 31, components with intent filters are exported by default); a low minimum version lacks system protections |
| Embedded web content | JavaScript interfaces, file access and mixed content exposed by WebView |
| **Cloud backend configuration** | Firebase, object storage and identity pool configuration in the package; whether database rules or storage buckets are publicly readable or writable. Verifying this means sending requests to the service, so handle it according to the authorization |
| Dynamic loading | Whether code or script bundles downloaded and loaded at runtime (hot updates) have their signatures verified |
| Native libraries | Check the `.so` files and frameworks in the package for hardening and versions per [compiled artifacts and binaries](#compiled-artifacts-and-binaries) |
| Third-party SDKs | What data analytics, advertising and crash-reporting SDKs collect and send out (see [30](dimensions.md#30-privacy-data-governance-and-compliance)) |
| Privacy declarations | Whether the iOS privacy manifest (`PrivacyInfo.xcprivacy`), the store's data safety declaration and what third-party SDKs actually collect agree with each other |
| Signing and obfuscation | Signing scheme and certificate; whether security depends on "secrecy" from obfuscation |
| Forced updates | After a vulnerability is found, can old versions be disabled or forced to upgrade |

## Browser extensions

Applies to: extension packages (CRX, XPI, ZIP) for browsers such as Chrome, Edge, Firefox and Safari, or the version listed in a store. For the code level, see [4.35](specialties.md#435-client-apps-extensions-and-auto-update).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Permissions and site scope** | `permissions`, `host_permissions` and content script `matches` go beyond what the features need; high privileges such as `<all_urls>`, `cookies`, `webRequest`, `debugger`, `scripting` and `nativeMessaging` |
| **Remote code** | Downloading and running remote scripts, `eval`, running remote configuration as code; store policies forbid this, but sideloaded and old versions may still have it |
| **Communication with web pages** | Which websites `externally_connectable` allows to send messages; whether messages received by `onMessageExternal` and `postMessage` messages received by content scripts are checked for origin and structure; whether a web page can drive the extension's privileged interfaces |
| Resources web pages can access | Pages exposed by `web_accessible_resources` can be embedded by web pages (clickjacking), or used to detect which extensions the user has installed |
| Content security policy | The content security policy of extension pages is relaxed |
| Secrets in local storage | Tokens stored in `storage.local`, or in `storage.sync`, which syncs to the account; whether the web page context can read them |
| Native program communication | `allowed_origins` in the native messaging host manifest; whether the host program validates input |
| **Updates and publishing accounts** | A custom `update_url`; whether the store publishing account has multi-factor authentication (MFA) enabled and who can publish. In late 2024, several extensions pushed malicious updates after their developers were phished |
| Data sent out | Where collected browsing history, page content and form data are sent (see [30](dimensions.md#30-privacy-data-governance-and-compliance)) |

## Firmware and device images

| Checkpoint | What counts as a problem |
| --- | --- |
| Extraction and file system | Can it be unpacked; services, startup scripts and configuration in the file system |
| **Default credentials and shared keys** | Default account passwords; all devices share the same private key, certificate or SSH host key |
| Outdated components | Versions and known vulnerabilities of built-in open-source components (such as busybox, OpenSSL, web servers) |
| **Management interface** | Command injection, path traversal and unauthenticated interfaces in the device's web management pages; this is the most common kind of flaw behind mass exploitation of network devices |
| **Remote services enabled by default** | Whether remote management services such as Telnet, UPnP and TR-069 are on by default, and whether they listen on the WAN port |
| **Secure boot chain** | Whether the bootloader verifies the signatures of the kernel and file system; U-Boot environment variables are writable, and the boot countdown can be interrupted to get a command line; the verification logic is visible in the image, but whether the chip's fuses are blown has to be checked on the physical device |
| Firmware download and encryption | Whether the firmware download address uses HTTPS and the download is verified; encryption is not signing, and firmware that is only encrypted, without signature verification, can still be replaced |
| Update mechanism | Signature verification and anti-downgrade for update packages (see [4.36](specialties.md#436-embedded-firmware-and-real-time-constraints)) |
| Program hardening | Programs in firmware usually lack PIE and stack protection; check per [compiled artifacts and binaries](#compiled-artifacts-and-binaries) |
| Cloud connection credentials | Whether the keys, certificates and messaging service accounts a device uses to connect to the cloud are unique per device (see [device communication and IoT protocols](#device-communication-and-iot-protocols)) |
| Debug interfaces | Whether serial ports, JTAG and debug services are turned off in production builds |
| Limits of emulation | The interface running under emulation differs from the real device; parts that fail to run under emulation can only be looked at statically; note this in the conclusions |

## Device communication and IoT protocols

Applies to: when you only have the device or gateway, captured wireless and messaging traffic (BLE, Zigbee, MQTT, CoAP and so on), or the configuration of these protocols. For firmware, see [firmware and device images](#firmware-and-device-images).

| Checkpoint | What counts as a problem |
| --- | --- |
| **BLE pairing** | With legacy pairing (LE Legacy Pairing), capturing the pairing is enough to compute the key; "Just Works" pairing does not protect against man-in-the-middle attacks; sensitive characteristics can be read or written without requiring encryption and authentication |
| BLE privacy | The device uses a fixed address instead of a resolvable random address, so it can be tracked |
| **Zigbee network joining** | Still accepts the public default Trust Center link key (`ZigBeeAlliance09`); install codes are not used; the join window stays open too long; the network key is sent encrypted with a publicly known key |
| **MQTT authentication and authorization** | Anonymous connections are allowed; the service is only on plaintext port 1883; topic permissions are not isolated per device, so any client can subscribe to `#`; retained messages contain secrets; a client ID collision can kick another device offline |
| Provisioning | Whether the hotspot used for first-time provisioning is open; whether the Wi-Fi password is sent in plaintext |
| Device identity | Whether each device has its own certificate or key, or they all share one (see the firmware checkpoint "Default credentials and shared keys") |
| Replay and freshness | Can commands such as unlock or on/off be recorded and replayed |
| Over-the-air updates | Whether update packages are signed and verified over the wireless link (see [4.36](specialties.md#436-embedded-firmware-and-real-time-constraints)) |
| Amplification and exposure | UDP-based services such as CoAP and SSDP can be used for reflection amplification when they are exposed externally |

## Container images

| Checkpoint | What counts as a problem |
| --- | --- |
| **Secrets in layer history** | A secret added in one layer and deleted in a later layer still remains in the image; secrets in build arguments and environment variables |
| Base image | The base image's source, whether it is pinned to a digest, whether it is past its support lifetime |
| OS package and dependency vulnerabilities | Known vulnerabilities in the OS packages and language dependencies in the image |
| Runtime identity | Whether the default user is root; files with SUID; unneeded tools (shell, package manager, compiler) |
| Entry point and ports | Entry command, exposed ports, health checks |
| Signatures and provenance | Can the image signature and build provenance be verified; is the signer the expected one (see [signatures and provenance](#signatures-and-provenance)) |

## Published packages

Applies to: npm packages, Python wheels and sdists, JARs, gems, crates, NuGet packages, Go modules and so on, obtained from a registry or download page. What you review is what users actually install, not the repository. For release checks of the library itself, see [4.34](specialties.md#434-publishable-libraries-sdks-and-packages).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Package matches source** | Files in the package do not match the source at the corresponding tag or commit: extra build scripts, compressed test data, precompiled files, or altered content. The xz utils backdoor was only in the released tarball and could not be seen in the repository |
| Source commit | Does the package record its source commit (`.cargo_vcs_info.json` for crates, `gitHead` in npm registry metadata, the commit in the provenance); does that commit exist in the repository; was the working tree clean at packaging time |
| **Execution at install and build time** | npm `preinstall`, `install` and `postinstall`; sdist `setup.py` and build backends; gem native extensions; crate `build.rs` and procedural macros; NuGet `.props` and `.targets`. What they run, whether they access the network, whether they read environment variables |
| Precompiled content | Can the native libraries and minified or obfuscated scripts in the package be rebuilt from source; those that cannot are checked per [compiled artifacts and binaries](#compiled-artifacts-and-binaries) |
| Signatures and provenance | Do npm provenance, PyPI digital attestations, NuGet author signatures and Maven Central signature files exist, and do they point to the expected repository and workflow (see [signatures and provenance](#signatures-and-provenance)) |
| Publishers and version history | Recent changes of maintainers or publishing accounts; versions pulled soon after release; unusual jumps in version numbers; same-named packages on other registries |
| Differences between versions | Network access, processes, file writes and decode-then-execute code newly added between adjacent versions |
| Metadata | Do the repository address, license and declared dependencies match the source; a repository address that points to someone else's project to borrow its stars and reputation |

## Signatures and provenance

Applies to: any artifact that has a signature, provenance, an in-toto attestation or a transparency log entry, including binaries, installers, images, language packages and firmware.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Verification bound to identity** | Only checks that "there is a valid signature", without checking that the signer is the expected one (certificate subject, OIDC issuer, key fingerprint). When keyless signing does not restrict the identity, anyone can produce a "valid" signature |
| **Attestation matches the artifact** | The subject digest in the attestation differs from the digest of the artifact in hand; the two are matched only by file name or tag |
| **Source and builder** | The source repository, branch or tag, build workflow or build platform in the attestation differs from what is expected; builds from a forked repository, an unprotected branch or a local machine |
| Build level | The claimed SLSA build level has no evidence: L1 only requires provenance to exist; L2 requires it to be signed by a hosted build platform; L3 also requires builds to be isolated from each other and the signing key to be out of reach of build steps |
| Transparency log | Are signatures and attestations recorded in a transparency log (such as Rekor); is the inclusion proof included for offline verification |
| Certificates and time | Was the certificate valid at signing time; is there a trusted timestamp; how are old signatures handled after key rotation and revocation |
| Build inputs | Are the build inputs listed in the attestation (dependencies, base images, tools) pinned to digests |
| in-toto layout | When there is a layout, do the performer of each step and the chain of artifacts between steps all pass verification; who signs the layout itself |

## Dependency manifests, lock files and SBOMs

Applies to: targets that only have dependency manifests, lock files, a software bill of materials (SBOM) or a Vulnerability Exploitability eXchange (VEX) document.

| Checkpoint | What counts as a problem |
| --- | --- |
| Known vulnerabilities and fix versions | Check per [36](dimensions.md#36-supply-chain-and-artifact-integrity) ("Known vulnerabilities", "Security fix versions") |
| Support lifetime and maintenance status | Is the component past its support lifetime, or no longer maintained |
| Licenses | Are the licenses compatible with the way it is distributed |
| Private package names | Has someone else already registered a private package name on a public registry (dependency confusion) |
| Integrity | Does the lock file include checksums; do the manifest and the lock file agree |
| **Manifest matches artifact** | Whether the SBOM was generated from source code, from the build process or from the final artifact; compared with the artifact actually released, it misses statically linked libraries, copied-in code and components downloaded at runtime |
| Minimum elements | Are the component name, version, producer, unique identifier (purl or CPE), dependency relationships, component hash, license, generating tool and time of generation, and the manifest's author and timestamp all present; components without a unique identifier cannot be checked for vulnerabilities |
| Valid format | Does it pass CycloneDX or SPDX format validation; when a version or purl is written wrong, scanners silently skip that component |
| **VEX conclusions are justified** | Do entries marked "not affected" state a justification (component not present, vulnerable code not present, not in the execution path, attacker cannot control the input, inline mitigation already in place), and does the justification match the code and configuration; who issued it and when; has it gone stale after the artifact was upgraded |
| Signed manifests | Are the SBOM and VEX signed, or bound to the artifact digest as an attestation |

## Live services

Applies to: targets given only as an address, domain name or URL. This is a black-box review; conclusions only cover what can be observed from outside.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Authorization and scope** | Are target ownership, the allowed addresses and time windows, and the allowed operations (passive observation only, or active testing allowed) confirmed in writing; without confirmation, only observe passively |
| Exposed surface | Open ports and services, subdomains, host names in certificate transparency logs, public management interfaces and debug interfaces |
| Transport layer | Protocol versions, certificate chain and validity period |
| Responses and headers | Security response headers (including HSTS), cookie attributes; response headers and error pages that leak versions and internal information |
| Publicly exposed sensitive files | Version control directories, environment variable files, backup files, source maps, directory listings |
| Public API descriptions | Whether `openapi.json`, `swagger.json`, GraphQL introspection and configuration under `/.well-known/` are exposed; once you have them, check per [API specification files](#api-specification-files) |
| Cross-origin access | A plain request with an external `Origin` header is enough to see whether any origin is reflected with credentials allowed (see [4.14](specialties.md#414-server-request-handling-and-middleware)) |
| Frontend resources | Third-party scripts referenced by the pages and their integrity checks; check bundled scripts per [frontend bundles and source maps](#frontend-bundles-and-source-maps) |
| **Dangling DNS and subdomain takeover** | DNS records point to cloud resources, storage buckets or third-party services that have been released; once someone else claims them, they can serve content under your domain |
| Domains and mail | Check per [domains, DNS and mail configuration](#domains-dns-and-mail-configuration) |
| Security contact | Is there a `/.well-known/security.txt`; has its `Expires` date passed |
| Versions and known vulnerabilities | Known vulnerabilities for the products and versions that can be identified; note in the conclusion that they are inferred from versions |
| Active testing | Login attempts, scanning, fuzzing and vulnerability verification are done only within the authorized scope; no denial of service, and no reading of real user data |

When you need to map to OWASP ASVS, what outside observation alone can verify is mainly some of the requirements in version 5.0 chapters V3 (web frontend security), V12 (secure communication) and V13 (configuration); with a test account, add some of the requirements in V6 (authentication), V7 (sessions) and V8 (authorization). The other chapters need source code or internal evidence; record them as "Not checked" in the coverage record.

## Frontend bundles and source maps

Applies to: when you only have the built web frontend (bundled scripts, styles, HTML), source maps, or the live assets of a single-page application.

| Checkpoint | What counts as a problem |
| --- | --- |
| Restoring source | When `sourcesContent` in the source map can restore the original source, review it as source code; what is restored is only the frontend |
| **Secrets in the bundle** | Keys, private API addresses and server-side keys for third-party services built into the frontend; the frontend should only contain identifiers that can be public |
| Hidden routes and features | Admin page routes, feature flags, calls to unreleased APIs; they reveal the list of server APIs, and authorization still has to be checked on the server (see [28](dimensions.md#28-authorization-and-access-control)) |
| Client-side authorization | Permissions controlled only by hiding buttons or routes |
| Third-party scripts | Source and integrity checks of external scripts (see [4.18](specialties.md#418-user-interfaces-and-accessibility)) |
| Dangerous DOM writes | Places such as `innerHTML`, `dangerouslySetInnerHTML` and `eval` that receive external data (see [4.18](specialties.md#418-user-interfaces-and-accessibility)) |
| Dependency versions | Libraries bundled in and their versions (check per [36](dimensions.md#36-supply-chain-and-artifact-integrity)); bundling often strips version information, so note that conclusions from fingerprinting are inferences |
| Debug leftovers | Debug logs, test accounts and mock APIs in the production bundle |

## API specification files

Applies to: targets that only have API descriptions such as OpenAPI, AsyncAPI, GraphQL schemas, protobuf or gRPC definitions, or Postman collections. For whether the spec matches the implementation, see [43](dimensions.md#43-documentation-consistency).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Authentication coverage** | Which operations declare no security requirement; a global requirement removed by a single operation with `security: []`, or an array that contains an empty object `{}`, which makes authentication optional; the authentication method itself is unsuitable (such as an API key in query parameters) |
| **Object identifiers** | Object IDs in paths and parameters can be enumerated (such as auto-increment integers); these operations all need object-level authorization (see [28](dimensions.md#28-authorization-and-access-control)), and the spec itself cannot prove whether it is done |
| Input constraints | Strings, arrays and numbers without `maxLength`, `maxItems`, `maximum` or `pattern`; objects that allow extra fields (`additionalProperties`), which may allow mass assignment; no limits on upload size and type |
| Output exposure | Response models that contain password hashes, tokens, internal IDs or personal information; error response models that carry internal information |
| Server addresses | `http://` addresses, internal addresses and test environment addresses in `servers` |
| Callback addresses | Addresses in `callbacks`, `webhooks` or parameters that the server will send requests to (see [4.21](specialties.md#421-outbound-requests-and-server-side-request-forgery), [4.24](specialties.md#424-webhooks-and-external-events)) |
| Pagination and batching | List endpoints with no page size limit; batch endpoints with no limit on how many items one call can process (see [20](dimensions.md#20-resource-bounds-and-backpressure)) |
| Versions and deprecation | Operations marked deprecated but still in the spec; differences between the specs of different versions (for shadow APIs, see [4.14](specialties.md#414-server-request-handling-and-middleware)) |
| Secrets in examples | Real tokens and accounts in `example` fields and Postman environment variables |
| GraphQL and gRPC | Query depth and complexity limits cannot be seen in the spec, so mark them "Unverified"; authorization of mutations; whether gRPC reflection is enabled |

## Domains, DNS and mail configuration

Applies to: targets given only as a domain name, or given as DNS zone files or mail service configuration. For subdomain takeover, see [live services](#live-services).

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

## Hosts and operating systems

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

## Cloud accounts and infrastructure state

Applies to: cloud accounts or platform configuration viewed with a read-only identity. For clusters, also see [Kubernetes clusters](#kubernetes-clusters).

| Checkpoint | What counts as a problem |
| --- | --- |
| Identity and permissions | Overly broad roles and policies, keys unused for a long time, administrators without MFA, cross-account trust |
| Public exposure | Public storage buckets, snapshots and images; security groups and load balancers open to the whole internet |
| **Instance metadata service** | Is the version that requires a session token enforced (such as AWS IMDSv2); once a workload is hit by SSRF, can it get cloud credentials (see [4.21](specialties.md#421-outbound-requests-and-server-side-request-forgery)) |
| **Federated identity trust conditions** | Do roles that trust external identities (such as OIDC tokens from CI) restrict the repository, branch and audience; when only the issuer is trusted and the subject is not restricted, other people's workflows can also assume the role |
| Cross-account trust | Do cross-account roles given to third parties require an external ID, to prevent misuse (confused deputy) |
| Root and emergency accounts | Does the root account have access keys; does it have MFA; has it been used recently |
| Organization-level guardrails | Organization policies, bans on public sharing, region restrictions; when a read-only identity cannot see higher-level policies, record them as "Not checked" |
| Secrets in managed services | Secrets in the environment variables of function and container services; permissions of function execution roles; public function URLs |
| Logs and monitoring | Is audit logging enabled, how long is it retained, can it be tampered with |
| Encryption and keys | Encryption at rest, key rotation, who can use the keys |
| Drift from the declared state | Drift between the actual state and the infrastructure code (see [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code)) |

## Kubernetes clusters

Applies to: Kubernetes clusters viewed with a read-only identity, whether self-hosted or managed. The control plane of a managed cluster is run by the vendor, so the related baseline items are recorded as not applicable, and that vendor's CIS benchmark is used instead. When you only have manifest files, check per [configuration, deployment manifests and infrastructure code](#configuration-deployment-manifests-and-infrastructure-code).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Overly broad RBAC** | Subjects bound to `cluster-admin`; wildcard verbs and resources; `escalate`, `bind` and `impersonate`; subjects that can create Pods, read Secrets or run `exec` effectively hold the node or its credentials |
| **Unauthenticated entry points** | The API server allows anonymous access; the kubelet is unauthenticated or its read-only port is open; the dashboard or etcd is reachable from outside or does not use certificate authentication |
| Workload isolation | The Pod security level of each namespace (privileged, baseline, restricted); privileged containers, host directory mounts, host network or process namespaces, extra capabilities, running as root |
| **Workloads submitted by tenants** | When the platform runs Pods, jobs or containers for tenants, are spec fields let through only by an allowlist (run-as UID, shared process namespace, volumes, capabilities, host network); is an allowed UID the same as that of a platform component such as a service mesh sidecar, which lets the workload bypass that component's traffic rules; can the workload share a process namespace with a sidecar the platform injects, and read that sidecar's token and configuration |
| Service account tokens | Tokens are auto-mounted by default; long-lived token Secrets; workloads use the default service account |
| **Network isolation** | No default-deny network policy; the network plugin in use does not enforce network policies, so written policies have no effect; Pods can reach the cloud metadata service |
| Secret protection | Encryption at rest is not enabled for Secrets; secrets placed in ConfigMaps or environment variables; who can list Secrets |
| Admission and images | Is there an admission policy; are images pinned to digests, only from trusted registries, and signature-verified |
| Audit logs | Is the API server audit policy enabled; its log level and retention |
| Versions and support lifetime | Are the cluster and node versions still within the support period of upstream or the managed service vendor |

## Configuration, deployment manifests and infrastructure code

Applies to: targets that only have configuration files, deployment manifests, Helm charts, Kustomize, infrastructure code, or Terraform state and plan files. Check per [32](dimensions.md#32-configuration-and-defaults), [34](dimensions.md#34-runtime-environment-and-deployment-contract) and [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code).

| Checkpoint | What counts as a problem |
| --- | --- |
| Secrets | Plaintext credentials in configuration |
| Exposure and privileges | Listen addresses, public ports, privileges, host mounts |
| Defaults and environment differences | Differences from the production environment; dangerous defaults that are not overridden |
| **Check after rendering** | Helm charts and Kustomize overlays must be rendered with the actual values files before checking; looking only at templates and defaults misses conditional branches, and the defaults themselves are often insecure |
| Chart origin | Does the chart repository use HTTPS; is there a provenance file (`.prov`) or signature, and has it been verified; are the dependent subcharts version-locked |
| Install hooks and cluster reads | Permissions of Helm hook jobs; `lookup` in templates reads Secrets from the cluster and returns empty during rendering, so the rendered output differs from the actual install |
| **State and plan files** | Terraform state files and plan files store all resource attributes in plaintext, including database passwords and generated private keys; `sensitive` only affects display |
| State backend | Is state storage encrypted; are versioning and locking enabled; who can read and write it; is the storage bucket public |
| Freshness of state | State is only a snapshot from the last apply or refresh and may already differ from the real environment (see [cloud accounts](#cloud-accounts-and-infrastructure-state) ("Drift from the declared state")) |

## Network devices and firewall configuration

Applies to: configuration exports or rule sets of routers, switches, firewalls, load balancers and VPN gateways.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Management plane exposure** | Telnet, HTTP or SNMP open to the internet or to user network segments; management access not restricted by source address; management interfaces open to the internet, which is where many gateway vulnerabilities of recent years came in |
| Passwords and keys | How local passwords are stored (for example, Cisco type 7 is reversible and type 5 is MD5; type 8 or 9 should be used); SNMP still on v1 or v2c with community strings `public` or `private`; strength of pre-shared keys |
| **Overly broad rules** | `any any` allow rules; rules shadowed by earlier rules, or rules that never match; temporary rules kept for a long time; no restriction on outbound traffic (see [4.29](specialties.md#429-rule-and-policy-matching)) |
| Authentication and accounting | Is centralized authentication used (TACACS+, RADIUS); emergency local accounts; command accounting |
| Encryption protocols | SSH version and algorithms; for VPNs, the IKE version and cipher suites, and whether certificates or pre-shared keys are used |
| Routing and layer 2 | Are routing protocols authenticated; are unused ports shut down; DHCP snooping, ARP inspection |
| Logs and time | Where logs are sent; is NTP configured and authenticated |
| Firmware version | Compare the version against vendor security advisories (check per [36](dimensions.md#36-supply-chain-and-artifact-integrity) ("Known vulnerabilities")); note in the conclusion that it is inferred from the version |

## SaaS tenants and third-party integrations

Applies to: when you are given only a read-only administrator identity for a SaaS platform such as an office suite, code hosting or customer management, or when you need to review third-party apps and integrations connected to these platforms.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Third-party app authorization** | The permission scopes held by authorized OAuth apps and integrations (read and write all mail, all files, all repositories); whether ordinary users can grant authorization on their own; whether the publisher is verified; long-unused authorizations that have not been revoked |
| **Long-lived tokens and keys** | Personal access tokens, API keys and service account keys with no expiry or not rotated for a long time; integration-only accounts with too many permissions |
| **Login and MFA** | Is single sign-on enforced; is MFA enforced for administrators and all users; are local accounts that bypass single sign-on and legacy authentication protocols (such as IMAP and POP with basic authentication) turned off |
| Administrators | The number of super administrators; how emergency accounts are kept and monitored; whether test tenants or old tenants are connected to production |
| External sharing | "Anyone with the link" sharing; guests and external collaborators; public repositories, boards and forms |
| Data exfiltration channels | Rules that auto-forward to external mailboxes; destinations of webhooks and data export connectors (see [4.24](specialties.md#424-webhooks-and-external-events)) |
| Audit logs | Are they enabled, and how long are they kept; are key events not recorded because the license tier is too low |
| Offboarding and revocation | Are departed employees' accounts, tokens and authorizations revoked together with the identity source |
| Code hosting platforms | Organization default permissions; branch protection or rulesets; permissions of the default CI token; deploy keys; secret scanning and push protection (see [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code)) |

## Agent configurations and MCP servers

Applies to: when you only have an agent's system prompt, rule files (such as `AGENTS.md`, `CLAUDE.md`, `.cursor/rules/`), skill and tool lists, MCP client configuration (such as `.mcp.json`, `.vscode/mcp.json`, `claude_desktop_config.json`) or the tools declared by an MCP server, and are not reviewing their implementation code. For the code level, see [4.26](specialties.md#426-llms-and-tool-calling).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Secrets in configuration** | Tokens and keys hard-coded in `env`, `headers` or command arguments; configuration files committed to the repository or synced to the cloud |
| **Server source and pinning** | Servers launched with `npx -y`, `uvx` or `docker run` without a pinned version or digest, so every start may run different code; whether the package name impersonates another; who operates the remote server |
| **Tool description poisoning** | Instructions for the model hidden in tool names, descriptions or parameter descriptions; hidden Unicode characters (zero-width, bidirectional controls, tag characters); descriptions quietly changed after the user approved them |
| Name shadowing | Several servers provide tools with the same or similar names; one server's description changes how another tool is used |
| **Permissions and auto-approval** | The list of tools that need no confirmation, allowed command wildcards (for example, allowing any shell), the range of readable and writable directories; whether high-risk tools such as running commands, writing files, sending messages and making payments still require human confirmation |
| **Dangerous combinations** | When the same session can read private data, touch untrusted content and also send data out, external content can carry the data out; judge by combinations of tools, not by single tools only |
| Remote server authentication | Does the remote server require authentication; does the server pass the client's token through unchanged to downstream services; are OAuth scopes minimal |
| Local server exposure | Does a local HTTP server bind only to the loopback address, check `Host` and `Origin` and require authentication; otherwise a web page can call it through DNS rebinding (same-origin requests may carry no `Origin`, so the defense against rebinding relies mainly on checking `Host`) |
| Security carried by prompts | Security rules written only in the prompt and not enforced at the tool execution layer; keys and internal addresses in the prompt (see [4.26](specialties.md#426-llms-and-tool-calling)) |
| Writable context | Who can write long-term memory, retrieval stores, rule files and skill directories; can rule files submitted by others change the agent's behavior |
| Call records | Are tool calls and approvals recorded; can you trace who approved what |

## Databases and datasets

Applies to: data exports, read-only connections or data files.

| Checkpoint | What counts as a problem |
| --- | --- |
| Access permissions | Permissions of accounts and roles; who can read sensitive tables |
| Sensitive data inventory | Which tables and fields hold personal information, credentials and payment data; whether they are encrypted or masked |
| **How passwords and tokens are stored** | The password hash format in the user table (`$argon2id$`, `$2b$`, or unsalted MD5 or SHA-1 hex strings); tokens, API keys and security question answers stored in plaintext (criteria in [4.5](specialties.md#45-authentication-sessions-and-tokens)) |
| Database configuration | Listen address, transport encryption, default or empty-password accounts, authentication not enabled, audit switches; check this when you have a read-only connection, and record it as "Not checked" when you only have an export file |
| Tenant isolation | Is multi-tenant data isolated by tenant column and row-level security; views and functions that run with definer privileges |
| Constraints and integrity | Do the expected unique, foreign key, not-null and check constraints exist; does the data already contain records that break business rules |
| Backup and retention | Are backups encrypted, and are they regularly verified by restoring; is expired data cleaned up |
| The export file itself | Where this export is stored, whether it is encrypted, who can get it; treat it as sensitive data too |

## Logs, traffic and runtime data

Applies to: log exports, packet captures (pcap), HAR files, monitoring data and so on.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Evidence integrity** | Compute the digest on receipt, and record the source, time range and time zone; analyze on a copy; record where the packet capture was taken |
| Coverage and gaps | Are there gaps in the timeline (rotation, sampling, drops, clock jumps); conclusions for gap periods can only be "cannot be determined" |
| Sensitive data | Personal information, tokens and passwords in log and traffic samples (see [30](dimensions.md#30-privacy-data-governance-and-compliance), [4.4](specialties.md#44-cryptography-and-credentials)) |
| **Session material** | HAR files and debug packet captures carry cookies and tokens and are credentials in themselves. In 2023, HAR files were stolen from a customer support system, which led to session hijacking |
| Plaintext protocols and credentials | HTTP basic authentication, FTP, Telnet, plaintext SMTP login; unencrypted traffic between internal services |
| Transport parameters | The negotiated TLS version and cipher suite; the certificate chain (under TLS 1.3 certificates are encrypted and cannot be seen in captures); whether the client keeps connecting after a certificate error |
| Unusual outbound connections | Unexpected outbound destinations; callbacks at fixed intervals; very long random subdomains in DNS queries |
| Completeness of audit events | Are all expected security events recorded (see [4.23](specialties.md#423-security-audit-logs)) |
| Log injection | Forged line breaks and fields, forged events (see [4.23](specialties.md#423-security-audit-logs)) |
| Signs of attack | Obvious probing, exploitation attempts, unusual authentication; when an intrusion is suspected, report it through the project's incident response process; this checklist does not replace forensics |

## Incident reports and postmortems

Applies to: targets where you only have incident reports, postmortem documents, tickets or timelines, without the original evidence. All conclusions are "Unverified" by default unless original evidence is attached.

| Checkpoint | What counts as a problem |
| --- | --- |
| Complete timeline | Are the times of the first intrusion, detection, containment and recovery all present, with sources and time zones noted; how was the dwell time determined |
| **Root cause separated from trigger** | Does it state the root cause, or only the trigger; why the protections did not stop it and why monitoring did not catch it (detection gaps); classify it per the [root-cause facets](facets.md) |
| Basis for the impact scope | What logs back "no data exfiltration found", and does log retention cover the whole dwell time; when logs are missing, the conclusion should be "cannot be determined" |
| **Complete remediation** | Are all leaked or possibly leaked credentials, tokens and keys rotated; are persistence mechanisms (backdoor accounts, scheduled tasks, OAuth authorizations, SSH public keys) removed; are hosts from the same batch and similar systems all fixed |
| Checking for the same issue elsewhere | Does the same mechanism also exist in other systems, repositories and environments (compare with [historical vulnerability patterns](history.md)) |
| Improvement items | Does each item have an owner, a deadline and a way to verify it; items that only say "raise awareness" do not count |
| Notification obligations | Are the notification deadlines required by applicable regulations and contracts met; leave legal judgment to the legal team (see [30](dimensions.md#30-privacy-data-governance-and-compliance) ("Source of requirements")) |
| Evidence preservation | Are the original logs, images and memory preserved with their digests recorded, and can they still be re-examined now |

## On-chain contract addresses

Applies to: targets given only as a contract address (or protocol name). Only do read-only on-chain queries and local fork simulation; do not send transactions, do not sign anything, and do not touch private keys.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Source matches bytecode** | Is there verified source code on the block explorer; do the compiler version, optimization settings and constructor arguments match the on-chain bytecode; without verified source code, you can only decompile, and the conclusion notes that it is based on bytecode |
| **Proxy and implementation** | Is it a proxy contract (read from standard slots such as EIP-1967); the current implementation contract address; whether the implementation contract itself has been initialized |
| Initialized version | Does the initialized version number recorded by an upgradeable contract match what the current implementation requires; are variables added in an upgrade still zero |
| **Privilege holders** | The actual addresses of the owner, administrators, upgraders, pausers and members of each role; whether they are externally owned accounts, multisigs or timelocks; multisig thresholds and signers; timelock delays; reconstruct all members from role grant and revoke events, focusing on deployer and developer addresses |
| Pending changes | Operations that are queued but not yet executed in timelock queues, multisig pending queues and governance proposals; decode the call data of each one |
| Upgrade and parameter history | Past upgrade, parameter change and privilege transfer events; whether there have been unusual recent changes |
| Current parameters | Are current values such as fees, caps, oracle addresses and allowlists reasonable, and do they match the documentation |
| Assets and approvals | Assets held by the contract; token approvals the contract has given to others; the size of the approvals users have given to the contract |
| Contracts it depends on | The addresses of the oracles, routers, tokens and other protocols it calls, and their own privileges and upgrade state |
| Multi-chain deployments | Are the addresses, code versions, parameters and privileges of the same protocol consistent across chains |
| **Cross-chain messaging configuration** | The actual configuration of a cross-chain application on the messaging layer: the number of verifiers and the threshold for each path, whether the default configuration is inherited, whether peer addresses are correct; bridge limits and pause privileges |
| Account types | Whether related addresses are externally owned accounts that have delegated code through EIP-7702; module and paymaster configuration of smart contract wallets |
| Transaction history | Unusual calls, failed transactions; preparatory moves that suggest an attack (small test transactions, interactions with newly deployed contracts) |

When source code is available, also review the code per [4.38](specialties.md#438-smart-contracts-and-on-chain-interaction), [4.41](specialties.md#441-defi-economics-and-oracles) to [4.43](specialties.md#443-wallets-signing-and-off-chain-components) and the matching language tables; for contracts that verify zero-knowledge proofs, also see [4.44](specialties.md#444-zero-knowledge-proofs-and-circuits); for smart contract wallets and delegation, see [4.50](specialties.md#450-account-abstraction-and-smart-contract-wallets).

## Design, specification and architecture documents

Applies to: targets that have no code yet, or when only the design is reviewed.

| Checkpoint | What counts as a problem |
| --- | --- |
| Threat modeling | Are data flows, trust boundaries, assets and attackers drawn clearly (see [threat model and attack chains](baseline.md#threat-model-and-attack-chains)) |
| Failure and abuse scenarios | Do the requirements include error paths, abuse scenarios and limits |
| Turning dimensions into questions | Use the [root-cause facets](facets.md), the 45 dimensions and the relevant specialties as questions about the design; note that the conclusions are at the design level and the implementation still needs to be checked |
| Verifiability | Can the key security and correctness requirements be tested and audited |

## Office documents and PDFs

Applies to: when files such as Word, Excel, PowerPoint, RTF and PDF files are themselves the review target, for example files published externally, report templates generated by a product, or suspicious files received. For server-side code that parses these files, see [4.3](specialties.md#43-decoding-untrusted-external-data) and [4.22](specialties.md#422-subprocesses-dynamic-execution-and-decoder-side-effects).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Macros and auto-execution** | VBA macros, Excel 4.0 (XLM) macros; auto-run entry points (`AutoOpen`, `Workbook_Open`); calling external commands, downloading files |
| **External references** | Remote templates (templates in relationship files that point to external addresses), external OLE objects, DDE fields, protocol handler links (such as `ms-msdt:`) |
| Embedded objects | Embedded executables, OLE packages, other documents |
| PDF actions | `/OpenAction`, `/AA`, `/JavaScript`, `/Launch`, `/EmbeddedFile`, `/URI`, XFA forms |
| **Hidden content and metadata** | Author, machine name, local paths, revision history, comments, hidden worksheets, rows and columns, the original images of cropped pictures; old versions kept in PDF incremental updates; "redaction" that only draws a black box, with the text still underneath |
| PDF signatures | Does the signature cover the whole file; does an incremental update appended after signing change what is displayed |

## WebAssembly modules

Applies to: targets where you only have a `.wasm` module, whether it is used in the browser or as a server-side plugin. For how the host isolates the module, see [4.27](specialties.md#427-interpreters-compilers-and-virtual-machines) and [4.32](specialties.md#432-cross-language-boundaries-and-native-extensions).

| Checkpoint | What counts as a problem |
| --- | --- |
| Imports and exports | Which host functions the module imports (files, network, processes, WASI capabilities) and what it exports; whether the capabilities the host gives go beyond what is needed |
| Source language and compiler | The `producers` custom section, the name section, debug information; modules compiled from C or C++ keep the memory bugs of the original language |
| **Flaws in linear memory** | Inside the module there is no stack protection, no address randomization and no guard pages; an out-of-bounds write can change other data in the module and indexes into the indirect call table. "Isolated from the host" does not mean the module's internal data is safe |
| Embedded secrets | Keys, addresses and license check logic in data segments; compiling to wasm does not mean others cannot see them |
| Resource limits | Maximum number of memory pages, table size; whether the host limits execution time or fuel (see [20](dimensions.md#20-resource-bounds-and-backpressure)) |
| Integrity | Is the digest or signature verified before loading; which address is it loaded from |

## Model files and datasets

| Checkpoint | What counts as a problem |
| --- | --- |
| File format and loading | Does the model file use a format that runs code at load time (such as pickle-based formats); a different format is not automatically safe: does the loader import and run code based on configuration, class names, function names, templates or a remote-code switch in the file (see [4.22](specialties.md#422-subprocesses-dynamic-execution-and-decoder-side-effects)); are a safer format and safer loading options available |
| Source and integrity | Can the source, digest and license be verified |
| Data issues | Personal information in the data; sources that may be poisoned |
| Claims versus measurement | Are the capabilities and limits claimed in the model description backed by evaluations |

## Game clients and anti-cheat

Applies to: game clients, anti-cheat components and their drivers, and online game protocols that can only be observed from the client. Do this on a test server or private server, not on live servers: testing there affects other players and violates the terms of service.

| Checkpoint | What counts as a problem |
| --- | --- |
| **The client has the final say** | Position, damage, currency, items and cooldowns are computed and reported by the client, and the server trusts them directly |
| Protocol | Messages have no integrity protection and no sequence numbers; they can be replayed, sped up or reordered; the server does not check rates and physical limits |
| In-app purchases and rewards | Receipts are not verified with the platform on the server side; reward delivery is not idempotent (see [4.39](specialties.md#439-payments-accounting-and-billing)) |
| **Anti-cheat drivers** | Control interfaces exposed by the kernel driver do not check the caller; a signed driver is taken by others as a "bring your own vulnerable driver" to disable security software (ransomware abused mhyprot2.sys in 2022); whether it remains after uninstall |
| Privacy and permissions | The scope of the process, file and hardware information collected by anti-cheat, and where it is sent (see [30](dimensions.md#30-privacy-data-governance-and-compliance)) |
| Anti-debugging and obfuscation | They only raise the cost and are not a security boundary; they are no reason to skip server-side validation |
| Mods and scripts | The execution capabilities granted when loading user mods, scripts and maps (see [4.27](specialties.md#427-interpreters-compilers-and-virtual-machines)) |
