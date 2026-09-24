# Hit-test records

This directory records, sample by sample, every vulnerability used in finecomb's hit tests: its ID, component, one-sentence root cause, the result at the time, the rows it hit or the gap it exposed, where that gap was later fixed, and source links. The main README's [How coverage is validated](../README.md#how-coverage-is-validated) section gives only the totals; the details live here so they can be looked up later.

This directory is not part of the skill and is not installed with it.

## Records

| File | Round | Samples | Notes |
| --- | --- | --- | --- |
| [Derivation round](derivation.md) | Samples used to find gaps while writing the skill | nginx, the Linux kernel, OpenSSL, OpenSSH, glibc, applications and enterprise appliances, on-chain and zero-knowledge incidents | Every missed root cause became a facet or checkpoint; hits on these samples only show the gaps were filled, not that the skill generalizes |
| [Held-out round 1](holdout-1.md) | 31 samples not used to write the skill, plus 14 retests | The Linux kernel, nginx 2009–2021, Apache httpd, HAProxy, Envoy, OpenSSL | Retests are listed separately and do not count toward the held-out totals |
| [Held-out round 2](holdout-2.md) | 80 samples not used to write the skill | Web frameworks, memory-safe language libraries, browsers and desktop clients, open-source cloud-native, cryptography and authentication, smart contracts, AI agents, open-source databases | One section per domain |

## How results were judged

- For each sample, the real root cause was taken from the patch, the official advisory or a credible post-mortem, and written only at the patch level, with no exploitation details.
- Only the facets' "question to ask" column and the other tables' "checkpoint", "how to check" and "what counts as a problem" columns were used; no example or case column was used.
- **Hit**: a reviewer following one row would ask straight into this root cause. **Partial**: a row points in the right direction, but the reviewer has to take another step. **Missed**: no row leads there.
- Row names are the names at test time; if the skill later renamed or moved a row, check it against the "Where the gap was fixed" column and the current skill.

## When something was missed: how to trace it

1. Search this directory by ID, component or product name, for example `rg -F 'CVE-2025-1094' validation/`, to see whether it was tested and how it was judged.
2. If it was not tested, judge it yourself with the rules above: write the patch-level root cause in one sentence, find the facet in `skills/finecomb/references/facets.md` that would lead to it, then look for the specific checkpoint in the rows its "Detailed criteria" column points to.
3. Decide which layer is missing:
   - **Facet**: no root-cause facet leads to this root cause; add a domain-independent question to `facets.md`.
   - **Checkpoint**: the facet exists, but the rows it points to lack this specific checkpoint; add it to the matching dimension or specialty.
   - **Domain or language table**: it only holds for one kind of target or one language; add it to a specialty or a `lang-*.md` table.
   - **Wording or routing**: a row already says it, but too narrowly, in the wrong file, or the facet does not link to it.
4. After fixing it, add the sample to the matching record in the same format and say where the gap was fixed. The records in this directory are kept in English only; the row names in them follow the English skill.
