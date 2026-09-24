# A.8 PHP

| Checkpoint | What counts as a problem |
| --- | --- |
| **Loose comparison** | With `==`, two numeric strings are compared as numbers (for example `"0e123" == "0e456"` is true); `in_array` and `array_search` are loose by default; compare secrets with `hash_equals` |
| Type juggling | What a function does when it receives an unexpected type such as an array differs by version (older versions often return `null` with a warning, while PHP 8 mostly throws `TypeError` instead), which may bypass validation |
| **Deserialization** | `unserialize` on external data triggers gadgets through magic methods; metadata deserialization in the `phar://` wrapper (check against the version) |
| File inclusion | Paths for `include` and `require` come from external input; the remote inclusion setting |
| Variable overwriting | `extract()` and variable variables `$$` overwrite existing variables |
| Integers | Overflow turns the value into a float automatically and loses precision |
| Uploads | Uploaded files land in an executable directory; the file is judged only by its extension or the type the client declares |
