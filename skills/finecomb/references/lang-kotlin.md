# A.30 Kotlin

Applies to: Kotlin on JVM, Native, JavaScript and WebAssembly. Select the actual source sets and compiler backend; JVM deployments also use the shared runtime checks in [A.6](lang-jvm.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| Null safety at boundaries | Java platform types, external declarations, `!!` or uninitialized `lateinit` fields allow unchecked values to reach code that assumes non-null input |
| Coroutine ownership | Cancellation is swallowed as an ordinary exception; an unstructured coroutine outlives its owner; a blocking bridge stalls the dispatcher needed for progress |
| Flows and channels | Recollecting a cold flow repeats side effects; buffering, cancellation or failed delivery loses an element whose resource ownership was already transferred |
| Shared values | A read-only collection interface is treated as proof that the underlying object cannot change; a data-class copy still shares mutable members |
| Multiplatform implementations | An `actual` implementation weakens the validation, error, numeric or lifetime contract of the common `expect` declaration |
| Native memory and interop | Shared-heap access is assumed synchronized; native callbacks retain unpinned memory or scoped allocations; GC and ARC ownership leave cycles or release resources too late |
| JavaScript interop | `dynamic` disables checks relied on by common code; external objects follow JavaScript coercion, prototype or numeric rules |
| Wasm and exported entry points | Imports, generated adapters or dead-code removal change the reachable interface or host capabilities; native and JVM tests are treated as evidence for this backend |

Semantic references: [Native memory management](https://kotlinlang.org/docs/native-memory-manager.html), [dynamic types](https://kotlinlang.org/docs/dynamic-type.html), [flows](https://kotlinlang.org/docs/coroutines-flow.html). Check the target-version interop contracts with [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md).
