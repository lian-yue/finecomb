# 38 Portability and build context

| Checkpoint | What counts as a problem |
| --- | --- |
| Paths | Hard-coded separators; functions for file system paths and URL paths mixed up |
| File system differences | Case sensitivity, permission models, atomic replace guarantees, maximum file name length, link support, file name normalization forms → one logical name may map to two files |
| Word size | Relying on platform integers being 64-bit; **overflow branches that were unreachable become reachable on 32-bit platforms**; for atomic alignment see [16](16-language-and-runtime-pitfalls.md) |
| Byte order and alignment | Relying on a specific byte order; assumptions about struct layout |
| System call and network stack differences | Differences across platforms in socket options, port reuse, routing and interface enumeration, and name resolution paths |
| Build conditions and native code | Build tags, conditional compilation, platform-specific files and optional features select different entry points; tool-based analysis is assumed to cover configurations it did not load. Record static review, compilation and execution separately for each relevant configuration |
| Cross-compilation | Does it build for every platform it claims to support |
| Platform-specific code | Do not judge it dead or rewrite it just because the current environment cannot verify it; when the task really involves that behavior, change it within the authorization and state which platforms were not verified |
| **Standalone build context** | Repo-level workspaces and local overrides (such as Go workspaces and `replace`, npm/pnpm/yarn workspace and link, pip editable installs, Cargo `[patch]` and path dependencies, CMake subdirectories) hide what is missing when a unit is built alone; can units that will be published or referenced from outside build and test outside the workspace |
| Minimum runtime version | The declared minimum language, runtime or compiler version is lower than what the syntax, standard library or semantics the code actually uses require; or the declared version keeps new semantics from taking effect |
| Runtime and backend matrix | The same source targets several interpreters, native code, JavaScript or WebAssembly, but numeric semantics, available APIs and isolation guarantees are checked for only one; record combinations using [Appendix B](../environments.md) |
| Optimization and metadata removal | AOT, trimming, minification, link-time optimization or fast-math removes metadata, entry points or numerical guarantees required by reflection, dynamic registration, serialization or validation |
| Build evidence versus runtime evidence | Successful cross-compilation or execution under emulation is treated as proof of native system calls, weak-memory behavior, device access or permissions on every claimed platform |
| Build host versus target | A generator, configure probe or build script uses the build host's widths, byte order, headers, paths or available features to produce code for a different target |
| Native platform requirements | Compiler CPU features, system-library ABI or dynamically loaded symbols exceed the deployed platform's guarantees; feature detection or fallback does not cover the unsupported path |
