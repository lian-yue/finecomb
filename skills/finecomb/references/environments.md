# Appendix B: Execution environments

Applies to: the environments that build or run the target, including embedded runtimes and deployed artifacts. A language alone does not identify its execution contract.

## Establish the environment matrix

For each supported combination, record the language and implementation version, compiler and build mode, CPU architecture and ABI, OS, runtime host, deployment form and effective identity. Keep declared support, build evidence and execution evidence separate. Select only combinations supported or actually used by the target; this is not a request to test every possible combination.

Use the common questions in [16](dimensions/16-language-and-runtime-pitfalls.md), [34](dimensions/34-runtime-environment-and-deployment-contract.md) and [38](dimensions/38-portability-and-build-context.md). The index below selects detail sections; read only the sections that apply, in a batch with the other selected material. Several sections can apply to one component. Missing access or hardware leaves that combination unverified; a local build or simulator does not establish production behavior.

## Hosts and client environments

- [Unix services](environments/hosts-and-clients.md#unix-services)
- [Windows services and applications](environments/hosts-and-clients.md#windows-services-and-applications)
- [macOS applications and helpers](environments/hosts-and-clients.md#macos-applications-and-helpers)
- [Android and iOS applications](environments/hosts-and-clients.md#android-and-ios-applications)
- [Browser pages and embedded WebViews](environments/hosts-and-clients.md#browser-pages-and-embedded-webviews)
- [Workers and service workers](environments/hosts-and-clients.md#workers-and-service-workers)

## Runtime and deployment environments

- [JavaScript server runtimes and edge hosts](environments/runtimes-and-deployment.md#javascript-server-runtimes-and-edge-hosts)
- [JVM and CLR](environments/runtimes-and-deployment.md#jvm-and-clr)
- [Interpreter workers and embedded runtimes](environments/runtimes-and-deployment.md#interpreter-workers-and-embedded-runtimes)
- [Containers and Kubernetes](environments/runtimes-and-deployment.md#containers-and-kubernetes)
- [Serverless and snapshot-based hosts](environments/runtimes-and-deployment.md#serverless-and-snapshot-based-hosts)
- [VMs and translated architectures](environments/runtimes-and-deployment.md#vms-and-translated-architectures)
- [WebAssembly and WASI hosts](environments/runtimes-and-deployment.md#webassembly-and-wasi-hosts)
- [Kernels, drivers and verified bytecode](environments/runtimes-and-deployment.md#kernels-drivers-and-verified-bytecode)
- [Enclaves and trusted execution environments](environments/runtimes-and-deployment.md#enclaves-and-trusted-execution-environments)

## Devices and specialized execution

- [Bare metal, RTOS and hardware designs](environments/devices-and-specialized-execution.md#bare-metal-rtos-and-hardware-designs)
- [GPUs and accelerators](environments/devices-and-specialized-execution.md#gpus-and-accelerators)
- [CI runners and build hosts](environments/devices-and-specialized-execution.md#ci-runners-and-build-hosts)
- [Batch jobs, notebooks and distributed workers](environments/devices-and-specialized-execution.md#batch-jobs-notebooks-and-distributed-workers)
- [Chain VMs and proof runtimes](environments/devices-and-specialized-execution.md#chain-vms-and-proof-runtimes)

## Other environments

For an unlisted host, map its identity and capability model, scheduling and cancellation, resource limits, storage and network semantics, restart behavior and build modes to [34](dimensions/34-runtime-environment-and-deployment-contract.md) and [38](dimensions/38-portability-and-build-context.md). Record the target-version documentation used and the evidence missing. A row's absence does not mean the environment is out of scope, and an unavailable runtime does not make its code dead.
