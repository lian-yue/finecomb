# A.4 C and C++

| Checkpoint | What counts as a problem |
| --- | --- |
| **Undefined behavior** | Signed overflow, out-of-bounds access, use after free, double free, reading uninitialized values, null pointer dereference, strict aliasing violations, shift amounts not smaller than the bit width, data races; the compiler may remove overflow checks based on this (such as `a + b < a` written for signed numbers) |
| **Buffers and strings** | A copy or formatting operation can exceed its destination; `strncpy` is assumed to terminate a truncated string; `snprintf`'s required length is mistaken for bytes actually written; capacity leaves out the NUL. An API name alone does not establish a reachable overflow |
| Integers | Implicit conversion when comparing signed and unsigned values; `size_t` subtraction underflow; multiplication overflow in allocation sizes (`malloc(n * size)`); `long` has different widths on LP64 and LLP64 platforms; whether `char` is signed is implementation-defined |
| Memory ownership | Who frees the memory is not written down; when `p = realloc(p, n)` fails, the original pointer is lost; error paths miss frees |
| Format strings | Passing an external string directly as the format argument of `printf` |
| errno | Meaningful only when the call failed, and overwritten by later calls; calls interrupted by signals return `EINTR` |
| Signals | A callback actually runs in asynchronous signal context but invokes functions that are not async-signal-safe, such as allocation or formatted I/O; do not transfer this restriction to ordinary event-loop or worker-thread timer callbacks |
| Process entry assumptions | Assuming `argc` is at least 1 and that `argv[0]` exists and can be trusted; environment variables may be duplicated, malformed or carefully crafted by the caller (PwnKit came from out-of-bounds reads and writes when `argc` was 0) |
| Threads | Functions that are not thread-safe (`strtok`, `localtime`, `rand`, `getenv` running concurrently with `setenv`); `volatile` provides no synchronization |
| Wiping secrets | Clearing a key with `memset` may be optimized away; use `explicit_bzero`, `memset_s` or the platform equivalent |
| Check then use | The file is replaced after the `access()` check and before `open()` |
| Macros | Arguments evaluated more than once; missing parentheses change precedence |
| Operator precedence | Precedence mistakes in length, offset and mask expressions, not only in macros: `a - b + c` where `a - (b + c)` was meant; `x & MASK == 0` parses as `x & (MASK == 0)`; `a << n + 1` parses as `a << (n + 1)` |
| C++ lifetimes | Iterators invalidated after the container changes; returning a reference to a local variable; `string_view` or references pointing to temporary objects; lambdas that capture by reference outlive the captured objects |
| C++ object model | Base class destructor is not virtual where deletion through it is required; object slicing; rule of three/five; state of moved-from objects; exceptions escape a `noexcept` destructor or a destructor running during exception unwinding and cause termination; `shared_ptr` reference cycles leak |
| Static initialization order | The initialization order of static objects across translation units is unspecified |
| Build hardening | Whether warnings are fully enabled and treated as errors; whether stack protection, `_FORTIFY_SOURCE`, PIE, RELRO and similar are enabled (see [34](dimensions/34-runtime-environment-and-deployment-contract.md), [36](dimensions/36-supply-chain-and-artifact-integrity.md)) |
| ABI | Struct layout, alignment, packing, byte order; C++ ABI across different compilers or standard library versions |
| Exceptions crossing boundaries | An exception or non-local jump crosses a boundary whose ABI or resource contract does not support it; required cleanup is skipped or an exception escapes a non-throwing boundary |
| Random numbers | `rand`, `std::mt19937` and similar are not cryptographic random sources, and seeding them from the time makes them even easier to guess; for keys, tokens and seed phrases, use the operating system's random source (`getrandom`, `arc4random`, `BCryptGenRandom`) or a cryptography library |
| Coroutine frames | References or captured objects expire while a coroutine is suspended; a completion callback resumes an already destroyed frame; frame destruction and cancellation have no single owner |

Coroutine semantics: [C++ await expressions](https://eel.is/c++draft/expr.await).
