# 38 Portability and build context

| Checkpoint | What counts as a problem |
| --- | --- |
| Paths | Hard-coded separators; functions for file system paths and URL paths mixed up |
| File system differences | Case sensitivity, permission models, atomic replace guarantees, maximum file name length, link support, file name normalization forms → one logical name may map to two files |
| Word size | Relying on platform integers being 64-bit; **overflow branches that were unreachable become reachable on 32-bit platforms**; for atomic alignment see [16](16-language-and-runtime-pitfalls.md) |
| Byte order and alignment | Relying on a specific byte order; assumptions about struct layout |
| System call and network stack differences | Differences across platforms in socket options, port reuse, routing and interface enumeration, and name resolution paths |
| Build conditions and native code | Build tags, conditional compilation macros, platform-suffixed files, optional dependencies and feature switches (such as optional install groups and compile features) give the same source several build results; static analysis only covers the current platform and the default conditions, so **entry points in other build contexts must be stated separately as not covered** |
| Cross-compilation | Does it build for every platform it claims to support |
| Platform-specific code | Do not judge it dead or rewrite it just because the current environment cannot verify it; when the task really involves that behavior, change it within the authorization and state which platforms were not verified |
| **Standalone build context** | Repo-level workspaces and local overrides (such as Go workspaces and `replace`, npm/pnpm/yarn workspace and link, pip editable installs, Cargo `[patch]` and path dependencies, CMake subdirectories) hide what is missing when a unit is built alone; can units that will be published or referenced from outside build and test outside the workspace |
| Minimum runtime version | The declared minimum language, runtime or compiler version is lower than what the syntax, standard library or semantics the code actually uses require; or the declared version keeps new semantics from taking effect |
