# Published packages

Applies to: npm packages, Python wheels and sdists, JARs, gems, crates, NuGet packages, Go modules and so on, obtained from a registry or download page. What you review is what users actually install, not the repository. For release checks of the library itself, see [4.34](../specialties/4.34-publishable-libraries-sdks-and-packages.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Package matches source** | Files in the package do not match the source at the corresponding tag or commit: extra build scripts, compressed test data, precompiled files, or altered content. The xz utils backdoor was only in the released tarball and could not be seen in the repository |
| Source commit | Does the package record its source commit (`.cargo_vcs_info.json` for crates, `gitHead` in npm registry metadata, the commit in the provenance); does that commit exist in the repository; was the working tree clean at packaging time |
| **Execution at install and build time** | npm `preinstall`, `install` and `postinstall`; sdist `setup.py` and build backends; gem native extensions; crate `build.rs` and procedural macros; NuGet `.props` and `.targets`. What they run, whether they access the network, whether they read environment variables |
| Precompiled content | Can the native libraries and minified or obfuscated scripts in the package be rebuilt from source; those that cannot are checked per [compiled artifacts and binaries](compiled-artifacts-and-binaries.md) |
| Signatures and provenance | Do npm provenance, PyPI digital attestations, NuGet author signatures and Maven Central signature files exist, and do they point to the expected repository and workflow (see [signatures and provenance](signatures-and-provenance.md)) |
| Publishers and version history | Recent changes of maintainers or publishing accounts; versions pulled soon after release; unusual jumps in version numbers; same-named packages on other registries |
| Differences between versions | Network access, processes, file writes and decode-then-execute code newly added between adjacent versions |
| Metadata | Do the repository address, license and declared dependencies match the source; a repository address that points to someone else's project to borrow its stars and reputation |
