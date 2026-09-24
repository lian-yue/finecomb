# WebAssembly modules

Applies to: targets where you only have a `.wasm` module, whether it is used in the browser or as a server-side plugin. For how the host isolates the module, see [4.27](../specialties/4.27-interpreters-compilers-and-virtual-machines.md) and [4.32](../specialties/4.32-cross-language-boundaries-and-native-extensions.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| Imports and exports | Which host functions the module imports (files, network, processes, WASI capabilities) and what it exports; whether the capabilities the host gives go beyond what is needed |
| Source language and compiler | The `producers` custom section, the name section, debug information; modules compiled from C or C++ keep the memory bugs of the original language |
| **Flaws in linear memory** | Inside the module there is no stack protection, no address randomization and no guard pages; an out-of-bounds write can change other data in the module and indexes into the indirect call table. "Isolated from the host" does not mean the module's internal data is safe |
| Embedded secrets | Keys, addresses and license check logic in data segments; compiling to wasm does not mean others cannot see them |
| Resource limits | Maximum number of memory pages, table size; whether the host limits execution time or fuel (see [20](../dimensions/20-resource-bounds-and-backpressure.md)) |
| Integrity | Is the digest or signature verified before loading; which address is it loaded from |
