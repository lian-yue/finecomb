# Hit-test records

This directory records, sample by sample, every vulnerability used in finecomb's hit tests: its ID, component, one-sentence root cause, the result at the time, the rows it hit or the gap it exposed, where that gap was later fixed, and source links. The main README's [How coverage is validated](https://github.com/lian-yue/finecomb#how-coverage-is-validated) section gives only the totals; the details live here so they can be looked up later.

These records live on the separate `validation` branch, never on `main`, so installing the skill never downloads them. The skill itself needs no samples and works offline; the samples only test whether the skill's questions lead to real root causes. Links to the skill point at the `main` branch: <https://github.com/lian-yue/finecomb>.

## Records

| File | Round | Samples | Notes |
| --- | --- | --- | --- |
| [Derivation round](derivation.md) | Samples used to find gaps while writing the skill | nginx, the Linux kernel, OpenSSL, OpenSSH, glibc, applications and enterprise appliances, on-chain and zero-knowledge incidents | Every missed root cause became a facet or checkpoint; hits on these samples only show the gaps were filled, not that the skill generalizes |
| [Held-out round 1](holdout-1.md) | 31 samples not used to write the skill, plus 14 retests | The Linux kernel, nginx 2009–2021, Apache httpd, HAProxy, Envoy, OpenSSL | Retests are listed separately and do not count toward the held-out totals |
| [Held-out round 2](holdout-2.md) | 80 samples not used to write the skill | Web frameworks, memory-safe language libraries, browsers and desktop clients, open-source cloud-native, cryptography and authentication, smart contracts, AI agents, open-source databases | One section per domain |
| [Held-out round 3](holdout-3.md) | 97 samples not used to write the skill | Four facet-targeted groups (new root-cause facets, new questions in existing facets, facets with few samples in two parts); mobile apps, OS kernels other than Linux, authorization and business logic in web applications, denial of service, open-source embedded systems | One section per group; the round 3 fixes still await their retest |

Reproduction data for every sample is in [samples/](samples/), one JSON object per line: the repository, the vulnerable and fixed refs, the fix commits, the files and functions involved, a suggested audit scope, advisories, on-chain addresses where relevant, and the root-cause facets (`classes`) the sample exercises.

## How results were judged

- For each sample, the real root cause was taken from the patch, the official advisory or a credible post-mortem, and written only at the patch level, with no exploitation details.
- Only the facets' "question to ask" column and the other tables' "checkpoint", "how to check" and "what counts as a problem" columns were used; no example or case column was used.
- **Hit**: a reviewer following one row would ask straight into this root cause. **Partial**: a row points in the right direction, but the reviewer has to take another step. **Missed**: no row leads there.
- Row names are the names at test time; if the skill later renamed or moved a row, check it against the "Where the gap was fixed" column and the current skill.

## Re-running the tests with any agent

finecomb is meant to work with any agent that can read files, not only Claude. There are two ways to test it. Both use the data in `samples/`.

### Mode A: question-level test

This checks the skill's content, not the agent. It is what the records in this directory measure.

1. Give the agent the skill files and one sample's `root_cause`.
2. Ask which rows lead a reviewer to that root cause, using only the facets' "question to ask" column and the other tables' "checkpoint", "how to check" and "what counts as a problem" columns.
3. Score hit, partial or missed with the rules above.

### Mode B: blind audit

This checks the agent together with the skill, on real code. It is the stronger test.

1. Clone `repo` into a scratch directory and check out `vulnerable_ref`. Only open-source samples, and on-chain samples with verified source, can run in this mode; `closed` samples stay in mode A.
2. Install finecomb for the agent under test, by any route in the main README.
3. Start a fresh session with web access turned off, so the agent cannot look up the advisory. Give only this prompt, with nothing about the CVE, the bug class or the file:

   ```text
   Audit <audit_scope> in this repository with finecomb.
   ```

4. Score the report against `root_cause`, `paths` and `symbols`:
   - **hit**: a reported issue has the same root cause at the same place, with a trigger that can reach it;
   - **partial**: the right place with a vague or different mechanism, or the right mechanism at the wrong place;
   - **missed**: neither.
5. Check precision. Run the same prompt on `fixed_ref`: the issue should no longer be reported as open. For the other issues the report lists, spot-check them and count how many are confirmed, unconfirmed or wrong.
6. Discard a run if the agent looked up the target's advisories, or if the audit stopped before covering `audit_scope`. Report the discard instead of scoring it.

Compare agents only on the same finecomb commit and the same samples.

### Recording runs

Record each run as one JSON line in `runs/<agent>-<model>-<date>.jsonl`:

```json
{"id": "CVE-2024-45336", "mode": "B", "agent": "…", "model": "…", "skill_commit": "…", "result": "hit", "location_found": "src/net/http/client.go", "fixed_ref_clean": true, "other_issues": {"confirmed": 2, "unconfirmed": 1, "wrong": 0}, "discarded": false, "notes": "…"}
```

Keep one file per agent, model and date; do not edit past runs.

### Measuring and fixing by class, not by CVE

- Report accuracy per root-cause facet: group results by the sample's `classes`, then count hit, partial and missed in each group. A weak facet shows up as a low rate across several samples, not as one missed CVE.
- Fix a miss at the facet level. Sharpen the facet's question, or add or widen the checkpoint it points to, so the whole class is covered. Never add a row that only names one CVE or product.
- A fix counts only after a retest. Rerun at least two other samples of the same class that were not used to make the fix.
- Classes with few samples are coverage gaps in the test set itself. Add samples to them before trusting their rate.

## When something was missed: how to trace it

1. In a checkout of this branch, search by ID, component or product name, for example `rg -F 'CVE-2025-1094' .`, to see whether it was tested and how it was judged.
2. If it was not tested, judge it yourself with the rules above: write the patch-level root cause in one sentence, find the facet in `skills/finecomb/references/facets/` on the `main` branch that would lead to it, then look for the specific checkpoint in the rows its "Detailed criteria" column points to.
3. Decide which layer is missing:
   - **Facet**: no root-cause facet leads to this root cause; add a domain-independent question to the matching file in `facets/`.
   - **Checkpoint**: the facet exists, but the rows it points to lack this specific checkpoint; add it to the matching dimension or specialty.
   - **Domain or language table**: it only holds for one kind of target or one language; add it to a specialty or a `lang-*.md` table.
   - **Wording or routing**: a row already says it, but too narrowly, in the wrong file, or the facet does not link to it.
4. Only vulnerabilities that are already public and already fixed are accepted as samples; see [Contributions](https://github.com/lian-yue/finecomb/blob/main/AGENTS.md#contributions) on `main`. After fixing it, add the sample to the matching record in the same format and say where the gap was fixed, add its reproduction data to `samples/`, and retest the class as described above. The records in this directory are kept in English only; the row names in them follow the English skill.
