# A.32 Regular expressions

Applies to: regular expressions embedded in code, configuration or generated rules. Record the engine, version, flags and calling API, not only the pattern text.

| Checkpoint | What counts as a problem |
| --- | --- |
| Match extent | A search or line match is used where the entire value must match; final-newline behavior and multiline flags admit extra input |
| Character semantics | Unicode properties, case folding, locale, byte mode or character classes accept a different alphabet than the downstream operation |
| Pattern construction | Literal external text becomes regex syntax; quoting is applied before another layer decodes escapes; fragments change capture numbering or grouping |
| Replacement language | Pattern escaping is reused for replacement text, where capture references and special substitutions follow different rules |
| Match state and iteration | A reused matcher keeps a cursor or captures from another operation; an empty match fails to advance or splits the input at an invalid boundary |
| Failure versus no match | Compile errors, invalid encoding, budget exhaustion or runtime failures are converted into ordinary no-match results with a different allow/deny outcome |
| Cost and engine substitution | Compile, match and substitution work are unbounded, or a replacement engine changes syntax and match semantics; use [20](dimensions/20-resource-bounds-and-backpressure.md) for cost analysis |

Semantic references: [PCRE2 pattern semantics](https://www.pcre.org/current/doc/html/pcre2pattern.html), [PCRE2 API and replacement semantics](https://www.pcre.org/current/doc/html/pcre2api.html). These illustrate mechanisms, not guarantees for other engines; confirm the target engine's contract.
