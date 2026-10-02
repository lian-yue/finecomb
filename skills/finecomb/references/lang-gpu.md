# A.33 GPU kernels and shading languages

Applies to: CUDA, OpenCL, GLSL, HLSL, Metal and WGSL kernels or shaders. Check the specific language, device features, compiler mode and host API. C-derived syntax does not imply a CPU execution or memory model.

| Checkpoint | What counts as a problem |
| --- | --- |
| Dispatch geometry | Work-item indexes, strides, partial groups or specialization constants address outside the bound resources |
| Collective control flow | Required participants do not reach the same barrier or subgroup operation; an early return or divergent branch breaks its uniformity contract |
| Address spaces and memory scope | Local, shared, device and host memory are confused; atomics or barriers cover a narrower memory or execution scope than the communication requires |
| Subgroups | Code assumes a fixed width, a full active mask or implicit lockstep execution that the target does not promise |
| Host layout | Vector alignment, matrix layout, padding and buffer stride disagree with the host's declarations |
| Initialization and aliasing | Partially written outputs, overlapping resources or reused shared memory expose stale values; robustness features are assumed to validate logical object bounds |
| Arithmetic modes | Reduced precision, contraction or handling of non-finite and subnormal values violates the application's numeric contract |
| Completion and resource lifetime | Submission is treated as completion; buffers are reused before dependent work finishes; device loss leaves partial output accepted as valid |

Semantic references: [WGSL](https://www.w3.org/TR/WGSL/), [CUDA programming guide](https://docs.nvidia.com/cuda/cuda-programming-guide/index.html). Use [execution environments](environments.md) for host lifecycle and [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md) for host/device boundaries.
