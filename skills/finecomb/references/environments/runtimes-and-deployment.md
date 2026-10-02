# Runtimes and deployment environments

Applies to: the runtime and deployment contexts selected in [Appendix B](../environments.md). Read only the selected sections.

## JavaScript server runtimes and edge hosts

Are module resolution, conditional exports, permissions, built-in APIs and native add-ons checked for the actual host? Does a compatibility API preserve the required timeout, streaming and cancellation behavior?

Check [A.3](../lang-javascript.md) and [4.55](../specialties/4.55-serverless-and-edge-functions.md).

## JVM and CLR

Do reflection, dynamic loading, serialization metadata and generated proxies survive trimming or ahead-of-time compilation? Do class or assembly load contexts split type identity or shared state?

Check [A.6](../lang-jvm.md), [A.7](../lang-dotnet.md) and [39](../dimensions/39-generated-artifacts-and-toolchain.md).

## Interpreter workers and embedded runtimes

Are request-local globals reset in persistent workers? Do subinterpreters, free-threaded builds, alternative implementations or native extensions invalidate serialization and resource-lifetime assumptions?

Check [16](../dimensions/16-language-and-runtime-pitfalls.md), [4.27](../specialties/4.27-interpreters-compilers-and-virtual-machines.md) and [4.32](../specialties/4.32-cross-language-boundaries-and-native-extensions.md).

## Containers and Kubernetes

Are quotas, user namespaces, PID 1, read-only mounts, service identity and termination grace periods effective in the deployed configuration? Do projected file updates leave cached handles using old secrets or policy?

Check [34](../dimensions/34-runtime-environment-and-deployment-contract.md), [Kubernetes evidence](../targets/kubernetes-clusters.md) and [4.68](../specialties/4.68-container-runtimes-and-isolation.md).

## Serverless and snapshot-based hosts

Does warm reuse, freeze/thaw or restoring a snapshot retain caller state, expired connections, leases or duplicated random-generator state? Is unfinished work still owned after the invocation ends?

Check [4.55](../specialties/4.55-serverless-and-edge-functions.md) and [25](../dimensions/25-crash-and-recovery.md).

## VMs and translated architectures

Does migration or CPU translation change available instructions, memory ordering, timing or device behavior? Is evidence from emulation being used to claim native concurrency or device correctness?

Check [38](../dimensions/38-portability-and-build-context.md) and [4.48](../specialties/4.48-kernel-drivers-device-emulation-and-virtualization.md).

## WebAssembly and WASI hosts

Are host imports and preopened resources bounded? After memory growth or a host callback, are borrowed views still valid? Are host isolation, guest object bounds and host-call resource accounting checked separately?

Check [4.27](../specialties/4.27-interpreters-compilers-and-virtual-machines.md), [4.32](../specialties/4.32-cross-language-boundaries-and-native-extensions.md) and [Wasm artifacts](../targets/webassembly-modules.md). Semantic references: [JavaScript embedding and memory growth](https://webassembly.github.io/spec/js-api/), [Wasmtime security boundaries](https://docs.wasmtime.dev/security.html).

## Kernels, drivers and verified bytecode

Which execution context permits blocking, allocation and memory access? Does verifier acceptance cover the actual helpers, maps, JIT and attachment mode? Can unprivileged input reach a more privileged path through shared buffers, callbacks or changed object lifetimes?

Check [4.48](../specialties/4.48-kernel-drivers-device-emulation-and-virtualization.md), [4.27](../specialties/4.27-interpreters-compilers-and-virtual-machines.md) and [18](../dimensions/18-concurrency-and-memory-model.md).

## Enclaves and trusted execution environments

What remains controlled by the host despite enclave isolation: scheduling, I/O, shared buffers, time and persisted state? Are attestation, sealing, rollback and side-channel assumptions established for the selected hardware and runtime?

Check [4.51](../specialties/4.51-trusted-execution-environments-and-enclaves.md).
