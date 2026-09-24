# 15 Algorithm and data structure correctness

| Checkpoint | What counts as a problem |
| --- | --- |
| Comparison functions | The comparison used for sorting is not a strict weak ordering (returns true both ways for equal elements, not transitive) → results out of order or even a crash |
| Overflow in comparisons | Subtraction used as the comparison result; subtracting two large numbers overflows → order reversed |
| Sort stability | A stable sort is needed but an unstable one is used; is the relative order of equal elements promised |
| Rounding direction | Up or down; can small values get stuck at 0 (a classic trap in ratio and quota calculations) |
| Modulo bias | Bias from taking random numbers modulo; using a mask as modulo requires the capacity to be a power of 2 |
| Hash usage | When the same hash is used for sharding in two places and both take the same bits → strongly correlated, the distribution degrades |
| Shuffling | Wrong range in the shuffle algorithm → uneven distribution |
| Binary search and ranges | Open vs. closed ranges, updates to the upper and lower bounds, termination conditions |
| **Map iteration order** | Relying on an iteration order that the language or container does not promise (some are random, some follow insertion order, some sort part of the keys; different containers in the same language also differ; see [Appendix A](../languages.md)) |
| **Serialization order stability** | Byte strings generated from unordered containers (config, query strings, checksum input) have an unstable order → unstable identity or checksum |
| Identity normalization | Are equivalent spellings of an input normalized to the same identity; can the normalization rule **wrongly merge inputs that are not equivalent** (such as lowercasing paths, see [27](27-security-and-trust-boundaries.md)) |
| **Completeness of equivalence checks** | Do equality comparisons, cache keys, dedup keys, memoization, state merging and pruning decisions include every field that affects the result and security; when two things that are not equivalent are judged equivalent, which check gets skipped and whose data gets returned |
| Set operations | Boundaries of dedup, intersection, union and difference; empty sets |
| Identifier generation | What guarantees uniqueness (number of random bits, clock and node number, sequence); can IDs repeat when the clock goes backward or when several instances run; can identifiers exposed to the outside be guessed or enumerated; do they leak the creation time or business volume |
| Probabilistic data structures | Are the error rate and capacity assumptions of structures such as Bloom filters, cardinality estimators and sampling written down; does the error rate get out of control beyond the designed capacity; what happens when a wrong answer lands on a security decision |
