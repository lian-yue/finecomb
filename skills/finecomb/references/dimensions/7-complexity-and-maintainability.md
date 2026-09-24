# 7 Complexity and maintainability

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| Branch complexity | Complexity metric tools | A single function has so many branches that a person cannot read it in one pass or enumerate its tests |
| Nesting depth | Metrics or manual reading | Deeply nested conditions and loops; can early returns flatten them |
| Function length | Metrics | One function does too many things |
| File length and cohesion | Metrics + manual reading | One file carries several unrelated responsibilities |
| Number of exits | Manual reading | Too many scattered exits, making it hard to check that cleanup is complete on each exit |
| Cognitive load | Manual reading | How many contexts must be held in mind at once to understand a passage |
| Automated constraints | Check the checks and configuration the project already requires | Agreed checks are missing or broken; having no automated complexity gate is not a defect by itself |

> Parameter count, interface width and the number of abstraction layers are checked in [4 Abstraction and layering](4-abstraction-and-layering.md) and [10 APIs and contracts](10-apis-and-contracts.md).
