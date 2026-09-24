# Compiled artifacts and binaries

Applies to: executables, dynamic libraries, plugins and drivers. For installers, see [desktop installers and desktop apps](desktop-installers-and-desktop-apps.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| Identity and origin | Can the digest, version, build number and signature be checked; who is the signer; can the same artifact be built reproducibly from the source code |
| **Hardening options** | Address randomization (PIE), non-executable stack and data (NX), read-only relocations (RELRO), stack protection, `_FORTIFY_SOURCE`, control-flow protection; on Linux, full read-only relocations (`BIND_NOW`), executable stack (`GNU_STACK`) and CET properties; on Windows, ASLR (including high entropy), DEP, CFG, `/GS`, SafeSEH (32-bit) and CET shadow stack compatibility; on macOS, the hardened runtime and entitlements |
| **Dangerous entitlements** | A macOS release build carries `get-task-allow` (it can be attached to for debugging and injection), `disable-library-validation`, `allow-dyld-environment-variables` or `allow-unsigned-executable-memory` |
| **Embedded secrets and addresses** | Keys, passwords, tokens, internal addresses and debug switches in strings, resources and configuration sections |
| Linked libraries and versions | Statically or dynamically linked libraries and their versions, and whether they have known vulnerabilities (check per [36](../dimensions/36-supply-chain-and-artifact-integrity.md)) |
| Embedded build information | Go programs and Rust programs built with `cargo auditable` embed the full list of dependencies and versions, so you can look up vulnerabilities by version directly, which is more reliable than fingerprinting |
| Debug leftovers | Unstripped symbols, debug interfaces, test backdoors, hidden commands |
| Dangerous calls | Command execution, unsafe string functions and dynamic loading in the import table |
| **Library search paths** | ELF RPATH or RUNPATH contains relative paths or writable directories; Mach-O `LC_RPATH` entries and weak links point to libraries that do not exist; the Windows search order finds a same-named DLL in a writable directory first |
| Updates and loading | Auto-update and plugin loading (see [4.35](../specialties/4.35-client-apps-extensions-and-auto-update.md), [4.32](../specialties/4.32-cross-language-boundaries-and-native-extensions.md)) |
| Licenses and notices | License obligations of the third-party components bundled in |
