# 6 Naming, comments and readability

| Checkpoint | What counts as a problem |
| --- | --- |
| Concept consistency | The same concept has different names in different places; the same name means different things in different places |
| Misleading names | A getter has side effects; a predicate changes state; the name promises more or less than the implementation does |
| Name does not match reality | The name describes past behavior (e.g. a field refreshed on every write but named "creation time") |
| Negative names | Double-negative booleans (`notDisabled`, `ignoreNoCheck`) |
| Units in names | A timeout does not say seconds or milliseconds; a size does not say bytes or item count |
| Abbreviations and spelling | Invented abbreviations; misspellings frozen into a public API |
| Scope and name length | Short names in long scopes; long names for one-line locals |
| Comments explain "why" | Comments only restate what the code does, not why it does it this way or why not another way |
| Missing comments on key constraints | Non-obvious preconditions, side effects, concurrency requirements, performance traits or failure semantics are not written down |
| Stale or misleading comments | Comments describe an old implementation; comments mention names that do not exist |
| Comment rules | Check comment language, structure and opening wording against the project rules; **for multilingual comments, check that every language is complete** (a common mistake: only one language mentions some return value) |
| Readable at a glance | Can someone unfamiliar with the code answer "under what conditions does it get here, and what happens on failure" from the code and comments alone |
