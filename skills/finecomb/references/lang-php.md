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
| Random numbers | `rand`, `mt_rand` and `uniqid` are not cryptographic random sources; use `random_bytes` or `random_int` |
| Array keys and operators | Key coercion merges distinct identifiers; array union and merge operations use different precedence or renumber numeric keys, changing a validated configuration |
| Retained references | A by-reference `foreach` variable still aliases the final element after the loop; later assignment corrupts the original array |
| Missing and false-like values | `isset`, `empty`, null coalescing and explicit key lookup conflate absent, null, false, zero and `"0"` where the contract distinguishes them |
| Persistent workers and fibers | Code assumes request-end cleanup resets globals, locale or credentials; suspension lets another operation change shared state before resumption |

Semantic references: [PHP arrays](https://www.php.net/manual/en/language.types.array.php), [foreach references](https://www.php.net/manual/en/control-structures.foreach.php). Select the deployed worker model in [Appendix B](environments.md).
