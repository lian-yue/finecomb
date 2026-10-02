# A.24 Scala and Clojure

Applies to: Scala and Clojure code. Apply the shared runtime rows of [A.6](lang-jvm.md) for JVM deployments; select the actual [environment](environments.md) for JavaScript or native backends. Language-specific rows apply only to the named language.

| Checkpoint | What counts as a problem |
| --- | --- |
| Lazy collections | A sequence or view is consumed after its resource closes, or its evaluation happens outside the intended error or authority scope |
| Scala by-name parameters | Repeated parameter use repeats a side effect or observes different state; unused parameters silently skip required work |
| Scala partial operations | Non-exhaustive matches, partial functions or unchecked `Option.get` fail on reachable inputs |
| Scala Futures | Starting a Future is confused with describing deferred work; blocking starves its execution context; failure or timeout leaves other work running |
| Clojure retries | `swap!` callbacks or STM transactions perform external effects that can run again after a conflict |
| Clojure mutable values | A persistent collection or transactional reference contains mutable Java objects whose changes bypass the intended isolation |
| Clojure reading and evaluation | A general code reader, tagged-reader callback or `eval` handles untrusted data with unintended authority |
| Dynamic bindings and interop | Work moved to another executor loses required context; Java nulls or exceptions bypass the language-level result contract |

Semantic references: [Scala by-name parameters](https://docs.scala-lang.org/tour/by-name-parameters.html), [Clojure transactions](https://clojure.org/reference/refs). Apply [16](dimensions/16-language-and-runtime-pitfalls.md) to the target's versions and libraries as well.
