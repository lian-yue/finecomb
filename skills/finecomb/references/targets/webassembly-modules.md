# WebAssembly modules

Applies to: targets where you only have a `.wasm` module, whether it is used in the browser or as a server-side plugin. For how the host isolates the module, see [4.27](../specialties/4.27-interpreters-compilers-and-virtual-machines.md) and [4.32](../specialties/4.32-cross-language-boundaries-and-native-extensions.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| Imports and exports | Which host functions the module imports (files, network, processes, WASI capabilities) and what it exports; whether the capabilities the host gives go beyond what is needed |
| Source language and compiler | The `producers` custom section, the name section, debug information; modules compiled from C or C++ keep the memory bugs of the original language |
| **Flaws in linear memory** | Bounds on linear memory do not enforce the bounds of individual guest objects. A source-language memory bug can corrupt other objects or function-pointer indexes stored there. Inspect the actual compiler protections and host checks; do not infer either full guest memory safety or the absence of all mitigations from the Wasm format |
| Embedded secrets | Keys, addresses and license check logic in data segments; compiling to wasm does not mean others cannot see them |
| Resource limits | Maximum number of memory pages, table size; whether the host limits execution time or fuel (see [20](../dimensions/20-resource-bounds-and-backpressure.md)) |
| Integrity | Is the digest or signature verified before loading; which address is it loaded from |
| Host views and memory growth | Cached views, pointer lengths or shared-memory assumptions no longer match after growth or reentrant host calls; see [4.32](../specialties/4.32-cross-language-boundaries-and-native-extensions.md) |

Semantic references: [WebAssembly JavaScript interface](https://webassembly.github.io/spec/js-api/), [Wasmtime security boundaries](https://docs.wasmtime.dev/security.html).
