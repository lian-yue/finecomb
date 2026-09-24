# A.4 C and C++

| Checkpoint | What counts as a problem |
| --- | --- |
| **Undefined behavior** | Signed overflow, out-of-bounds access, use after free, double free, reading uninitialized values, null pointer dereference, strict aliasing violations, shift amounts not smaller than the bit width, data races; the compiler may remove overflow checks based on this (such as `a + b < a` written for signed numbers) |
| **Buffers and strings** | `strcpy`, `strcat`, `sprintf`, `gets`; `strncpy` does not guarantee NUL termination; `snprintf` returns the length it would have written, and truncation must be detected from that value; sizes that leave out the one byte for the NUL |
| Integers | Implicit conversion when comparing signed and unsigned values; `size_t` subtraction underflow; multiplication overflow in allocation sizes (`malloc(n * size)`); `long` has different widths on LP64 and LLP64 platforms; whether `char` is signed is implementation-defined |
| Memory ownership | Who frees the memory is not written down; when `p = realloc(p, n)` fails, the original pointer is lost; error paths miss frees |
| Format strings | Passing an external string directly as the format argument of `printf` |
| errno | Meaningful only when the call failed, and overwritten by later calls; calls interrupted by signals return `EINTR` |
| Signals | Calling functions that are not async-signal-safe (such as `malloc`, `printf`) in signal handlers; doing cleanup or writing logs in timeout or signal callbacks is just as dangerous |
| Process entry assumptions | Assuming `argc` is at least 1 and that `argv[0]` exists and can be trusted; environment variables may be duplicated, malformed or carefully crafted by the caller (PwnKit came from out-of-bounds reads and writes when `argc` was 0) |
| Threads | Functions that are not thread-safe (`strtok`, `localtime`, `rand`, `getenv` running concurrently with `setenv`); `volatile` provides no synchronization |
| Wiping secrets | Clearing a key with `memset` may be optimized away; use `explicit_bzero`, `memset_s` or the platform equivalent |
| Check then use | The file is replaced after the `access()` check and before `open()` |
| Macros | Arguments evaluated more than once; missing parentheses change precedence |
| C++ lifetimes | Iterators invalidated after the container changes; returning a reference to a local variable; `string_view` or references pointing to temporary objects; lambdas that capture by reference outlive the captured objects |
| C++ object model | Base class destructor is not virtual; object slicing; rule of three/five; state of moved-from objects; a destructor throwing an exception causes termination; `shared_ptr` reference cycles leak |
| Static initialization order | The initialization order of static objects across translation units is unspecified |
| Build hardening | Whether warnings are fully enabled and treated as errors; whether stack protection, `_FORTIFY_SOURCE`, PIE, RELRO and similar are enabled (see [34](dimensions.md#34-runtime-environment-and-deployment-contract), [36](dimensions.md#36-supply-chain-and-artifact-integrity)) |
| ABI | Struct layout, alignment, packing, byte order; C++ ABI across different compilers or standard library versions |
| Exceptions crossing boundaries | C++ exceptions passing through C interfaces or callbacks |
