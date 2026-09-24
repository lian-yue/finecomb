# 5 Coupling and blast radius

> Most "fix one place, break another" issues are not inside a single function but in **the relationships between things**.

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Call graph | List the target's callers and callees; who in the project references it | Treated as "unused" when it actually has callers; the blast radius of a change is not worked out |
| **Multiple copies of the same fact** | List every place the same data lives: memory, disk, counters, summary structures, indexes, remote | No clear time or owner for bringing the copies back in line; the inconsistency window is not defined or documented |
| Linked invariants | Look for paired state where "if A changes, B must change too" | Paths exist that change only half — **especially error paths and early returns** |
| Implicit coupling | Dependencies built through global variables, file system layout, naming conventions or execution order | The dependency is not expressed in code; changing one place silently breaks another |
| Consistency of two-way mappings | Two ways of locating the same object (such as "derive an index from a name" and "build a path from a name") | The two algorithms disagree → written at A, looked up at B |
| Dependency direction | Does a lower layer know about an upper layer; are there cycles | Layer inversion, dependency cycles |
| Initialization order | Package/module-level initialization, global registries, singletons, default-value wiring | Order dependencies not written down; I/O or things that can fail done during initialization |
| Cross-module assumptions | The assumptions this target makes about other modules' behavior | The assumption is not guaranteed by the other module's public contract |
| **Implicit duties of replaced calls** | For changes that swap one call for another (a lower-level primitive or another library function, inlining, splitting or merging helpers), list each check, lock, zeroing, count and error conversion the old call did as a side effect | The new code drops one of them, even though no line of checking was deleted |
| **Readers of a moved or redefined field** | For changes that move a value to another field or slot, or change what a field means, list every place that reads the old field or slot | When a value moves to another field or slot, or a field changes meaning, not every reader of the old place is updated |
| Blast radius of a change | For each change, list: whose behavior changed, whose docs must change, whose tests must change | Callers, docs or the test matrix missed |
