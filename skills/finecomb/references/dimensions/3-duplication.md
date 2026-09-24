# 3 Duplication

| Checkpoint | How to check |
| --- | --- |
| Line-by-line duplicate functions | Compare the implementations of the same concept (e.g. a background job and a foreground entry point each wrote an identical scan) |
| Copy-pasted inline logic | A piece of logic exists as its own function and is also copied by hand somewhere else (e.g. a function hand-copies the whole body of another function that already exists) |
| The same check scattered in many places | The same fail-fast check is written once in each of three entry points |
| Synonymous entry points | Two public entry points only forward to each other, and their docs use the same wording |
| Parallel types | Two structures express the same thing |
| Duplicate constants | The same magic number is written separately in many places |
| Cross-module duplication | Another module in the project already has an equivalent capability (see the comparison method in [Review method](../scope.md#review-method)) |
| Documentation duplication | The same fact is written separately in several documents and comments; change one and the others drift — see [43](43-documentation-consistency.md) |
