# A.25 Haskell and OCaml

Applies to: Haskell and OCaml applications and native bindings. Haskell's default laziness must not be assumed for OCaml; apply each row only to mechanisms actually used.

| Checkpoint | What counts as a problem |
| --- | --- |
| Partial functions and matches | Missing constructors, empty collections or failed parsing raise exceptions on accepted inputs |
| Haskell lazy evaluation | Deferred computation raises errors after the handler returns, retains unbounded data or reads a resource after it closes |
| Haskell cancellation | Asynchronous exceptions interrupt acquisition or ownership transfer; excessive masking prevents shutdown; cleanup loses the original failure |
| Haskell effects and STM | Unsafe escape hatches undermine purity assumptions; effects outside transactional state are treated as rolled back or executed once |
| OCaml domains | Mutable values are shared across domains without synchronization; code assumes an earlier single-runtime-lock model |
| OCaml effects and cleanup | Suspended or abandoned computations retain resources; continuation handling violates the runtime's resumption contract |
| Representation and FFI | Machine integer width, unboxed values, GC roots or callbacks disagree with the native side's layout and lifetime |

Semantic references: [Haskell expressions and evaluation](https://www.haskell.org/onlinereport/haskell2010/haskellch3.html), [OCaml parallel programming](https://ocaml.org/manual/5.3/parallelism.html), [OCaml effect handlers](https://ocaml.org/manual/5.3/effects.html). Check the target compiler and concurrency library for cancellation and effect semantics; native boundaries also use [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md).
