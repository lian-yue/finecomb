# A.28 Zig

Applies to: Zig programs and C interoperability. Confirm the compiler version and build mode; language and standard-library contracts must match the pinned toolchain.

| Checkpoint | What counts as a problem |
| --- | --- |
| Safety modes | Production disables runtime safety checks that the code relies on to validate external input |
| Error unions and optionals | `catch unreachable`, unchecked unwrapping or discarded errors turn ordinary failures into traps or illegal behavior |
| Allocator ownership | Memory is freed with the wrong allocator; arena reset invalidates retained slices; allocation failures leave partial state |
| Deferred cleanup | `defer` frees a returned value or `errdefer` misses a failing acquisition; ownership transfer is inconsistent across exits |
| Slices and pointers | Reallocation invalidates aliases; a sentinel, alignment or pointer cast promises a condition not established by the caller |
| Integer operations | Truncating or wrapping operations are used where checked sizes are required; release behavior invalidates an overflow guard |
| C ABI and compile time | Native declarations disagree on layout or lifetime; compile-time branches and target options leave deployed paths unreviewed |

Semantic reference: [Zig language reference](https://ziglang.org/documentation/master/) (select documentation for the target version). Also use [38](dimensions/38-portability-and-build-context.md) and [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md).
