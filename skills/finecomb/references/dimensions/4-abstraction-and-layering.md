# 4 Abstraction and layering

| Checkpoint | What counts as a problem |
| --- | --- |
| Wrapper layers | A layer that only renames and adds no behavior |
| Number of callers of an abstraction | An interface or abstraction with only one caller and no need for substitution, test doubles or dependency inversion |
| Interface width | Interface methods that nobody calls |
| Configuration vs. per-call policy | Long-lived configuration mixed with per-call policy; for concrete parameter misuse see [10](10-apis-and-contracts.md) and [11](11-usability-and-misuse-resistance.md) |
| Ownership of responsibility | Logic that serves only one object is split out into free-floating functions; the object's own state/lifecycle is not expressed by its own methods |
| Layer ownership | Business policy pushed down into a general-purpose layer; specific business, deployment or product policy shows up in a general-purpose layer |
| Layer responsibilities | When there are two layers (such as a handle layer / storage layer), does one layer overstep or re-implement the other's responsibility |
| File organization | Main-flow objects do not have their own files; vague catch-all files exist |

> Dependency direction, dependency cycles and layer inversion are checked in [5 Coupling and blast radius](5-coupling-and-blast-radius.md); the size of functions and files is checked in [7 Complexity and maintainability](7-complexity-and-maintainability.md).
