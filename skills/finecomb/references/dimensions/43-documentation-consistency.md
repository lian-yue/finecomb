# 43 Documentation consistency

| Checkpoint | How to check |
| --- | --- |
| Do the documents exist | Per the project rules, do the directories that should have an overview document (README) / flow document / test matrix have them |
| Entry point mapping | Per the authorization, use an existing targeted tool or check by hand; entry points in scope should have no missing, duplicate or extra mappings |
| Flow accuracy | **A passing tool does not mean the flow is drawn right**: for each entry point, compare it with the source by hand — how parameters affect branches, whether every exit is drawn, whether the source and pass-through rules of dynamic errors are written down |
| Invariants | Does each one state its owner, the conditions under which it holds, what it guarantees, the result of breaking it, and the related flows |
| Stale facts | The docs mention things that were deleted, never implemented or renamed |
| Single authoritative location | Is the same fact in two places that both only point elsewhere, so it is never really written down anywhere |
| Are contracts written down | Concurrency contracts, caller obligations, consistency promises, failure semantics, resource ownership, security assumptions — are these, the things most in need of writing down, actually written |
| Example code | Do the examples in the docs still compile/run; do they use entry points that were deleted |
| Test matrix | Do the case names in the matrix and in the code match in both directions |
| Links and renderability | Do relative links, anchors and cross-references point to files and headings that exist; can diagram syntax (such as mermaid) be parsed and rendered |
| Interface description files | Do interface descriptions such as OpenAPI, proto and GraphQL schemas match the implementation: paths, fields, required fields, types, error codes, authentication requirements |

> The quality of the comments themselves is checked in [6 Naming, comments and readability](6-naming-comments-and-readability.md).
