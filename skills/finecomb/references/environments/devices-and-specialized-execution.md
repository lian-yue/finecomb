# Devices and specialized execution

Applies to: the specialized contexts selected in [Appendix B](../environments.md). Read only the selected sections.

## Bare metal, RTOS and hardware designs

Can interrupts, reset domains, DMA or non-coherent memory change state outside ordinary threads? Are deadlines, watchdogs and power-loss behavior verified on the relevant hardware? RTL also needs clock-domain, reset and simulation-versus-synthesis checks.

Check [4.36](../specialties/4.36-embedded-firmware-and-real-time-constraints.md) and [4.66](../specialties/4.66-hardware-designs-and-hardware-security.md).

## GPUs and accelerators

Are launch success and operation completion distinguished? Do stream dependencies, barrier scope, host/device copies and buffer lifetimes hold for asynchronous work? Can quotas or device reset leave partial results or another user's data?

Check [A.33](../lang-gpu.md), [18](../dimensions/18-concurrency-and-memory-model.md), [19](../dimensions/19-lifecycle-and-resources.md) and [4.32](../specialties/4.32-cross-language-boundaries-and-native-extensions.md). Semantic reference: [CUDA asynchronous execution](https://docs.nvidia.com/cuda/cuda-programming-guide/02-basics/asynchronous-execution.html).

## CI runners and build hosts

Do profiles, executable search paths, writable caches and credentials differ from an interactive build? Can untrusted inputs affect a later privileged job on a reused worker?

Check [4.33](../specialties/4.33-build-scripts-ci-and-infrastructure-as-code.md) and [36](../dimensions/36-supply-chain-and-artifact-integrity.md).

## Batch jobs, notebooks and distributed workers

Do retries, partial checkpoints, execution order, reused sessions and worker counts change side effects or results? Are environment and random-state assumptions preserved when the job resumes elsewhere?

Check [4.37](../specialties/4.37-data-processing-batch-jobs-and-machine-learning.md) and [4.74](../specialties/4.74-notebooks-and-interactive-computing.md).

## Chain VMs and proof runtimes

Does the actual chain revision, metering, transaction rollback, asynchronous messaging and off-chain executor match the language table's assumptions? Is a local VM result distinguished from deployed chain state?

Check [contract languages and chain runtimes](../languages.md#contract-languages-and-chain-runtimes), [4.38](../specialties/4.38-smart-contracts-and-on-chain-interaction.md) and [4.44](../specialties/4.44-zero-knowledge-proofs-and-circuits.md).
