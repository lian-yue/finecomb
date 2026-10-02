# A.27 Perl

Applies to: Perl programs, modules and XS extensions.

| Checkpoint | What counts as a problem |
| --- | --- |
| Scalar and list context | A call returns a count or another context-dependent value instead of the collection the caller expects |
| Truth and definedness | The string `"0"`, an empty string and `undef` are conflated, losing valid input or accepting missing data |
| Implicit state | `$_`, regex captures, localized variables or package globals are overwritten by nested calls before use |
| Input and commands | Two-argument `open`, diamond input, string-form commands or string `eval` interpret external data as operations |
| Taint handling | Taint mode is treated as complete authorization; permissive captures mark unsafe data as trusted without checking its meaning |
| Aliases and structures | A `foreach` alias changes its source; autovivification or shallow references mutate supposedly read-only state |
| Text and regex | Byte strings and decoded characters mix; match modifiers or state alter validation; external patterns cause excessive work |
| Modules and XS | Module search paths select caller-controlled code; native extensions break lifetime or memory-safety assumptions |

Semantic references: [Perl security](https://perldoc.perl.org/perlsec), [Perl traps](https://perldoc.perl.org/perltrap). For decoding and native calls also use [4.22](specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md) and [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md).
