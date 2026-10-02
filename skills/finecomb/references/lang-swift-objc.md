# A.11 Swift and Objective-C

| Checkpoint | What counts as a problem |
| --- | --- |
| Force unwrapping | Crashes from `!`, `try!` and `as!` that public input can trigger |
| Integer overflow | Swift arithmetic overflow terminates the process at run time (only operators such as `&+` wrap around), so external input can use it to cause a crash |
| Reference cycles | An owner retains a closure that strongly retains the owner, so neither is released; an `unowned` reference is used after its lifetime guarantee no longer holds |
| actor reentrancy | An actor method can be reentered at an `await`, so the state before and after the suspension is inconsistent (see [18](dimensions/18-concurrency-and-memory-model.md)) |
| Main thread | A UI or actor-isolated operation runs outside its required executor; missing annotations alone are not proof of an isolation violation |
| Archive decoding | Unarchiving external data in a way that does not require `NSSecureCoding` |
| Objective-C | A message to `nil` is mistaken for completed work or for setting an error output; format strings come from outside; key-value coding keys come from external input |
| Tasks and cancellation | An unstructured task outlives its owner; cancellation is assumed to stop work without cooperation; a detached task relies on inherited actor or task-local context |
| Continuations | Callback, timeout and cancellation paths can resume the same continuation twice, or leave it without any resume path |
| Sendability and native boundaries | `@unchecked Sendable`, unsafe pointers or legacy interop suppress checks without establishing safe sharing and lifetime guarantees |

Concurrency semantics: [The Swift Programming Language](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/). Also check [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md) for ARC and native ownership.
