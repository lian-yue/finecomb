# 8 Functional correctness and fitness for requirements

> The earlier dimensions ask "is the code well written". This one asks the most basic question: **does it do what it should, does it do all of it, and does it do it right.**

| Checkpoint | What counts as a problem |
| --- | --- |
| Requirement coverage | Is every claimed capability implemented; is anything "in the docs but not in the code" |
| Functional suitability | Is what was built actually needed; was anything built that nobody wants |
| Core correctness | Are the main-path results right; have they been compared with the authoritative definition, a reference implementation or known test vectors |
| **Source of the spec** | What is the behavior based on — a spec, a standard, upstream docs, or "it looks like it should be this way"; are the places with no basis marked |
| Following external standards | When implementing a standard / protocol / format, are all required parts done; are the choices on optional parts written down; **are deviations from the standard recorded with reasons** |
| Boundary semantics | Is the behavior on empty input, a single element and extreme values what the requirement wants (not just "does not crash") |
| Implicit premises | Do the idempotency, reentrancy, ordering and identity-comparison semantics that the calling scenario or upper-layer contract actually relies on hold; requirements that cannot be derived must not be made up |
| Paired capabilities | Are the create and delete, acquire and release pairs the contract requires complete; judge objects released by an external owner, or explicitly append-only, by their actual duties |
| Acceptance method | Does each claimed capability have a matching way to verify it; what proves it works |
| **Completeness of verification** | Do validators, checkers, type checks, policy engines, signature or proof verification and circuit constraints pin down the "allowed inputs" exactly; during review, ask the reverse: "what other inputs can also pass". Correct inputs passing does not mean wrong inputs are rejected |
