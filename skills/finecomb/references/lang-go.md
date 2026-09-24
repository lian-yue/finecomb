# A.1 Go

| Checkpoint | What counts as a problem |
| --- | --- |
| **nil interfaces** | A nil pointer stored in an interface makes the interface itself non-nil, so the caller always sees an error; returning a nil pointer of a concrete error type is the typical source |
| **Copying values that hold locks or atomics** | Copying a `sync.Mutex`, `sync.WaitGroup`, `atomic.*`, or a struct that contains them, after first use separates the protection from the data that is really shared (`go vet`'s copylocks catches some of these) |
| **64-bit atomic alignment** | On 32-bit platforms, atomic operations on bare `int64`/`uint64` fields require 8-byte alignment, otherwise they crash; avoid this with typed wrappers such as `atomic.Int64` |
| Method sets | Methods with pointer receivers are not in the method set of the value type, so interface assertions fail |
| Embedding promotion | Methods of an embedded type are accidentally promoted into this type's public API; when embedding concrete types such as `*net.TCPConn` or `*os.File`, `io.Copy` detects the promoted `ReadFrom`/`WriteTo` and bypasses the wrapper's `Read`/`Write`; when embedding only an interface, optional capabilities such as `CloseWrite` and `SyscallConn` may be lost |
| Closure capture | When the `go` version in `go.mod` is below 1.22, `for` loop variables are shared across iterations, so closures and goroutines read later values; from 1.22 each iteration creates a new variable |
| defer in loops | A `defer` registered inside a loop body runs only when the function ends, so resources are released late |
| context stored in fields | Storing `context.Context` in a struct mismatches lifetimes and makes cancellation propagation uncontrollable; the cancel function returned by `context.WithCancel` and similar is never called (`go vet`'s lostcancel) |
| Slice aliasing | `append` may reuse the underlying array, so data elsewhere gets changed; after truncating a slice for reuse, old references still point to the same array |
| Range copies | The value variable of `range` is a copy, so changing it has no effect; ranging over large structs by value costs copies |
| Strings and bytes | Unneeded copies when converting between `string` and `[]byte`; the original data is changed after a zero-copy conversion with `unsafe.String` or `unsafe.Slice`; `len` counts bytes, `range` iterates by rune, and invalid UTF-8 becomes U+FFFD |
| Unsafe operations | For every use of `unsafe`: are the memory layout assumptions it relies on written down; GC visibility; an address stored as `uintptr` does not keep the object alive |
| Reflection | The failure path of type assertions is not handled; used on hot paths; type information that could be cached is not cached |
| Generics | Constraints are too wide, so failures only show up at run time; zero-value semantics; behavior differs when the type parameter is a pointer versus a value |
| Package initialization and globals | The side effects, failures and execution order of `init` are not defined; ownership and concurrency contract of mutable globals are unclear |
| panic and recover | A panic in a goroutine without recover terminates the whole process; `recover` only works in a function called directly by defer; concurrent map reads and writes are an unrecoverable fatal error, not a panic |
| nil containers and channels | Writing to a nil map panics; sending or receiving on a nil channel blocks forever; closing a nil or already closed channel, or sending on a closed channel, panics |
| Non-comparable types | When an interface holds a slice, map or function, using it as a map key or comparing it with `==` panics at run time |
| goroutine leaks | After the caller returns on timeout, the child goroutine stays blocked on a send to an unbuffered channel and never exits |
| `io` contract | An `io.Reader` implementation may return `n > 0` and `err != nil` together, and the caller must handle those n bytes first; `Read` returning `0, nil` does not mean the end; `Write` must return an error when it returns `n < len(p)` |
| Error checks | Comparing wrapped errors with `==`; custom error types do not implement the needed `Unwrap`, `Is`, `As`; timeout errors do not satisfy `net.Error`'s `Timeout()` or `os.ErrDeadlineExceeded`, so callers cannot recognize them |
| HTTP | `resp.Body` is not closed; the Body is not fully read when the connection should be reused; `http.DefaultClient` has no timeout; changing default objects such as `http.DefaultTransport` is a process-wide side effect |
| Timers | When the `go` version in `go.mod` is below 1.23, a `time.Timer` or `time.Tick` that was not stopped is not collected before it fires; from 1.23 the semantics of timer channels also changed, so check against the version |
| `os.File` and descriptors | After calling `Fd()`, `SetDeadline` stops working on Unix; when an `*os.File` is collected, its finalizer closes the descriptor, and once the descriptor number is reused this may close a different file; after `os.NewFile` takes over a descriptor, the descriptor must not be closed separately |
| Thread binding | Changing thread-level system state such as the network namespace needs `runtime.LockOSThread`, otherwise the goroutine may be scheduled onto another thread |
| cgo | Passing Go pointers to C must follow the cgo pointer rules; memory allocated by C is not managed by the GC; threads and stacks when C callbacks enter Go |
| JSON | `encoding/json` matches field names case-insensitively; with duplicate keys the last one wins; unknown fields are ignored by default; numbers decoded into `any` are `float64` by default and lose precision above 2^53 (use `UseNumber` when needed) |
| Integers | Signed integer overflow wraps around silently; the width of `int` varies by platform |
| Container limits | Before Go 1.25, `GOMAXPROCS` is not aware of cgroup CPU limits; memory limits need `GOMEMLIMIT` |
| Workspaces and local overrides | `go.work` and `replace` make builds inside the repository pass, while the module is missing dependencies when used on its own; check with `GOWORK=off` |
| `//go:linkname` | References to internal standard library symbols may be refused at link time by newer toolchains (tightened from 1.23) |
| `iota` renumbering | Inserting a constant renumbers `iota`, and values that are persisted or passed between processes change with it |
