# A.9 Ruby

| Checkpoint | What counts as a problem |
| --- | --- |
| **Deserialization** | `Marshal.load`; unsafe YAML loading (from Psych 4, `YAML.load` is safe by default and `unsafe_load` is not) |
| Dynamic calls | The method name or class name passed to `send`, `public_send` or `constantize` comes from external input |
| Command execution | `Kernel#open` runs a command when its argument starts with a vertical bar (the pipe character); building strings for backticks or `system` by concatenation |
| **Regex anchors** | `^` and `$` match the start and end of a line; to validate the whole string use `\A` and `\z` |
| Mass assignment | Request parameters are assigned directly to a model (see "automatic field binding" in [28](dimensions.md#28-authorization-and-access-control)) |
| Monkey patching | Reopening core classes changes behavior for the whole process |
| Global VM lock | Like Python's global interpreter lock, it does not make compound operations atomic |
