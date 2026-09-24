# A.11 Swift and Objective-C

| Checkpoint | What counts as a problem |
| --- | --- |
| Force unwrapping | Crashes from `!`, `try!` and `as!` that public input can trigger |
| Integer overflow | Swift arithmetic overflow terminates the process at run time (only operators such as `&+` wrap around), so external input can use it to cause a crash |
| Reference cycles | Closures that hold a strong reference to `self` cause leaks and need `[weak self]` or similar |
| actor reentrancy | An actor method can be reentered at an `await`, so the state before and after the suspension is inconsistent (see [18](dimensions.md#18-concurrency-and-memory-model)) |
| Main thread | UI updates are not on the main thread; `@MainActor` annotations are incomplete |
| Archive decoding | Unarchiving external data in a way that does not require `NSSecureCoding` |
| Objective-C | Messages sent to `nil` silently return zero values; format strings come from outside; key-value coding keys come from external input |
