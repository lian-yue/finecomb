# Root-cause facets

Root-cause facets are questions distilled from the root causes of real vulnerabilities, independent of domain and language. The same facet shows up again and again in kernels, web servers, cryptography libraries, smart contracts and business systems, only in a different shape. The [general dimensions](../dimensions/index.md) are organized by quality attribute and the [specialties](../specialties/index.md) by target type, and either one can let a cross-domain root cause fall through the gaps. This file pulls the root causes out on their own: **go through them for every object**.

How to use:

- After the [per-object questions](../questions.md), ask every facet in turn for each entry point, validation point, piece of shared state, buffer, parsing loop, state transition and external interaction. When a facet does not apply to an object, write down the reason in the coverage record.
- The "Detailed criteria" column of each facet points to the specific check rows. This file only gives the question to ask and why it is easy to miss.
- "Real cases" only show what the facet looks like. They are not a checklist; the same facet holds in other domains too.
- Group the issues you find by facet. This also works as a reverse check: if a facet was never seriously asked during the whole review, that is a coverage gap.
- This checklist has been tested for hits against real vulnerabilities (nginx, the Linux kernel, OpenSSL, OpenSSH, glibc, major frameworks and enterprise appliances, on-chain and zero-knowledge proof incidents), and every root cause it missed has been turned into a facet here. Check new vulnerabilities the same way: if no facet matches, a facet needs to be added.

The facets are grouped into seven files:

- [Validation and trust](validation-and-trust.md)
- [Representation and transformation](representation-and-transformation.md)
- [Memory, lengths and parsing](memory-lengths-and-parsing.md)
- [State, timing and paths](state-timing-and-paths.md)
- [Consistency and completeness](consistency-and-completeness.md)
- [Resources, externals and observability](resources-externals-and-observability.md)
- [Process](process.md)
