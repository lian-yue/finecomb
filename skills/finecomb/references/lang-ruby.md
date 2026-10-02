# A.9 Ruby

| Checkpoint | What counts as a problem |
| --- | --- |
| **Deserialization** | `Marshal.load`; unsafe YAML loading (from Psych 4, `YAML.load` is safe by default and `unsafe_load` is not) |
| Dynamic calls | The method name or class name passed to `send`, `public_send` or `constantize` comes from external input |
| Command execution | `Kernel#open` runs a command when its argument starts with a vertical bar (the pipe character); building strings for backticks or `system` by concatenation |
| **Regex anchors** | `^` and `$` match the start and end of a line; to validate the whole string use `\A` and `\z` |
| Mass assignment | Request parameters are assigned directly to a model (see "automatic field binding" in [28](dimensions/28-authorization-and-access-control.md)) |
| Monkey patching | Reopening core classes changes behavior for the whole process |
| Global VM lock | Like Python's global interpreter lock, it does not make compound operations atomic |
| Random numbers | `rand` and `Random` are not cryptographic random sources; use `SecureRandom` |
| Hash defaults and keys | A mutable `Hash.new` default is shared across missing keys; string and symbol keys or an indifferent-access wrapper change which field is selected |
| Blocks, Proc and lambda | Non-local `return` or `break`, argument handling or invoking a saved block after its defining method returns violates the caller's control-flow contract |
| Truth and missing values | Zero and empty strings are treated as false; `\|\|=` overwrites an intentionally false value; a missing key and a stored nil require different handling |
| Fibers and execution context | Code confuses fiber-local and thread-local state or assumes scheduler-based I/O gives parallelism or cancellation guarantees it does not provide |

Semantic references: [Ruby Hash](https://docs.ruby-lang.org/en/master/Hash.html), [Proc and lambda](https://docs.ruby-lang.org/en/master/Proc.html). Confirm the deployed implementation and version.
