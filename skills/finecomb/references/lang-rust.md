# A.5 Rust

| Checkpoint | What counts as a problem |
| --- | --- |
| **unsafe** | An `unsafe` block that breaks aliasing, initialization or lifetime rules causes undefined behavior, even when the code that calls it is safe code; whether the preconditions of each `unsafe` block are written down; whether hand-written `Send` and `Sync` implementations really hold |
| Integers | Overflow panics in debug builds and wraps around by default in release builds (unless `overflow-checks` is enabled); `as` casts truncate, and float-to-integer casts saturate |
| Panic sources | `unwrap`, `expect`, out-of-bounds indexing, division by zero and double borrows of a `RefCell` that public input can trigger; `Drop` does not run with `panic = "abort"`; panics crossing the FFI boundary (depending on version and ABI, the result is an abort or undefined behavior) |
| Locks | Handling of a poisoned `Mutex`; `Mutex` is not reentrant; holding a `std::sync::Mutex` guard across `.await` |
| async | Calling blocking operations in async functions stalls the executor; cancellation safety: a future dropped at an `.await` leaves half-finished state (such as the branches of `select!`); dropping a tokio `JoinHandle` does not cancel the task |
| Resources | `mem::forget` and `Rc` reference cycles leak, and both count as safe code; the order in which `Drop` runs |
| Build-time execution | `build.rs` and procedural macros run arbitrary code at build time; feature unification unexpectedly turns on some features of dependencies |
| Deserialization | `serde` ignores unknown fields by default (use `deny_unknown_fields` when needed); ambiguous matching of `untagged` enums |
| Hashing | The standard `HashMap` uses a collision-resistant hash by default; after switching to a faster non-randomized hash, untrusted keys lose that protection |
