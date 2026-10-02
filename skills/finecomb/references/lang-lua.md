# A.21 Lua and LuaJIT

Applies to: Lua code and embedded Lua runtimes. Check the actual dialect; LuaJIT and Lua releases do not share every numeric, library or cleanup rule.

| Checkpoint | What counts as a problem |
| --- | --- |
| Truth and absence | Zero and empty strings are treated as false; assigning `nil` removes a required table entry |
| Tables and sequences | `#` is treated as a count for a table with holes; iteration order is relied on; `ipairs` silently stops before later entries |
| Multiple results | Assignment, parentheses or a non-final expression drops an error or other return values |
| Metamethods | Indexing, comparison or conversion invokes user code that changes validated state or escapes the intended capabilities |
| Protected calls | `pcall` or coroutine resume status is ignored; an error object is mistaken for a normal result |
| Numeric representation | Integer width, floating precision or LuaJIT cdata conversions change sizes, identifiers or arithmetic checks |
| Host access | `load`, module searchers, native modules, debug facilities or LuaJIT FFI expose authority outside the permitted environment |
| Resources and coroutines | Yield, error or abandoned coroutines leave host resources open; cleanup depends on features unavailable in the selected runtime |

Semantic reference: [Lua reference manual](https://www.lua.org/manual/5.4/manual.html). For host isolation use [4.27](specialties/4.27-interpreters-compilers-and-virtual-machines.md) and [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md); confirm LuaJIT differences against the target's pinned implementation.
