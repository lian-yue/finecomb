# A.2 Python

| Checkpoint | What counts as a problem |
| --- | --- |
| Mutable default arguments | The default value of `def f(x=[])` is created only once and shared across calls |
| Late-binding closures | A `lambda` or inner function created in a loop reads the variable only when it is called, so all of them read the last value |
| Aliasing and shallow copies | The three rows of `[[0] * 3] * 3` are the same list; `copy.copy` copies only one level |
| Identity and equality | Comparing numbers or strings with `is` relies on implementation details such as small-integer caching and string interning |
| Modifying during iteration | Adding or removing keys while iterating over a dict raises `RuntimeError`; removing elements while iterating over a list skips elements |
| **Catching exceptions too broadly** | A bare `except:` or `except BaseException` swallows `KeyboardInterrupt`, `SystemExit` and `asyncio.CancelledError`; `except Exception: pass` silently swallows errors; a `return` in `finally` swallows the exception; a context manager's `__exit__` returning a truthy value swallows the exception |
| Global interpreter lock | The GIL does not make compound operations such as `x += 1` or check-then-write atomic; in the free-threaded build (optional from 3.13), code and C extensions that rely on the serialization the GIL implies may break |
| asyncio | Calling a blocking function in a coroutine stalls the whole event loop; if the return value of `asyncio.create_task` is not kept, the task may be collected while it runs; when one task in `gather` fails, the other tasks keep running; catching `CancelledError` without raising it again breaks cancellation |
| **Deserialization is execution** | `pickle`, `marshal`, `shelve`, `yaml.load` with an unsafe Loader, and `torch.load` (unrestricted by default before 2.6, and in older versions even `weights_only=True` can be bypassed; check against the version actually pinned) can all execute code (see [4.22](specialties.md#422-subprocesses-dynamic-execution-and-decoder-side-effects)); use `yaml.safe_load` for YAML |
| Dynamic execution and commands | `eval`, `exec`, `subprocess` with `shell=True`, `os.system`, `os.popen` |
| Imports and paths | Side effects at import time; circular imports get a half-initialized module; the script's directory comes first in `sys.path`, so a file with the same name can hijack the standard library or a dependency |
| Arbitrary-precision integers | They do not overflow, but external input can build huge integers that exhaust the CPU; newer versions limit by default the number of digits when converting between integers and decimal strings (4300 digits), and going over it raises `ValueError` |
| Money and precision | Using `float` for money; the context precision and rounding mode of `Decimal` are not set explicitly |
| Text and encoding | Mixing `str` and `bytes`; `open()` without `encoding` depends on the locale settings |
| Type annotations | Annotations are not checked at run time; external input needs explicit validation |
| Iteration order | `dict` keeps insertion order (guaranteed by the language from 3.7); `set` does not, and string hashing is randomized per process, so the iteration order of a `set` may differ between runs of the same code |
| Equality and hashing | A class defines `__eq__` but not `__hash__`, so its instances become unhashable; mutable objects used as keys |
| Recursion | The default recursion limit is about 1000, and deeply nested external input triggers `RecursionError` |
| Paths | `os.path.join` drops all earlier parts when it meets an absolute component, which leads to path traversal; entry paths when unpacking with `tarfile` and `zipfile` (check the `filter` parameter of `tarfile` and its default against the version); `os.path.realpath` is not strict by default: when a path does not exist, forms a loop or is too long, it raises no error and returns only a partly resolved result, so containment checks must use strict mode and check again at the point of use |
| Temporary files | `tempfile.mktemp` has a race; use `mkstemp` or `NamedTemporaryFile` |
| Network defaults | Clients such as `requests` may wait forever when no `timeout` is passed |
| Random numbers | `random` is not a cryptographic random source; use `secrets` |
| Time | Mixing `datetime` values without a time zone and values with a time zone; `datetime.utcnow()` returns a value without a time zone (deprecated from 3.12) |
| Multiprocessing | Using the `fork` start method in a multithreaded process may deadlock; the default start method changes with platform and version (see "fork and threads" in [18](dimensions.md#18-concurrency-and-memory-model)) |
| Releasing resources | Relying on `__del__` or reference counting to release files and connections promptly; on other implementations (such as PyPy) they are no longer released promptly |
| Assertions | `python -O` removes `assert`, so validation done with it stops working in optimized mode |
| Installing dependencies | Installing source packages runs their build scripts; `--extra-index-url` may let internal package names resolve from the public index (dependency confusion) |
