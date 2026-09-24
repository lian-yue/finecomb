# AGENTS.md

Rules for anyone, person or agent, who changes this repository. This file is the single source for maintenance rules; the READMEs only point here. `CLAUDE.md` is a symbolic link to this file.

## What this repository holds

- finecomb is an exhaustive code review and security audit checklist written in the open Agent Skills format. It is Markdown only: no scripts, binaries or executable content.
- `skills/finecomb/`: the skill. `SKILL.md` (English) and `SKILL.<language code>.md` (other languages, such as `SKILL.zh-CN.md`) hold the workflow, the index and the closing self-check; `references/` holds the details, in English only.
- `README.md` and `README.<language code>.md`: the overview in each language.
- `.claude-plugin/marketplace.json`: the Claude Code plugin marketplace manifest, also read by the skills CLI.
- `LICENSE`: Apache-2.0.
- `SECURITY.md`: the security policy, the intended use, and the contact (finecomb@lianyue.com) for reporting problems in finecomb or its misuse.
- Hit-test records and samples do **not** live on `main`. They live on the separate `validation` branch (see [Validation](#validation)), so installing the skill never downloads them.

## Size and network

- The skill is a generalized checklist, not a collection of samples. It grows by adding general facets and checkpoints, never by adding one entry per vulnerability, so it should stay at hundreds of Markdown files, around a thousand at most. Samples may grow without limit, but they never go into `skills/`.
- The skill must work offline. During an audit it uses the network only for read-only lookups of dependency versions and known vulnerabilities, plus fetching the target itself when the caller explicitly gives a repository URL. It never fetches its own content, samples or validation data.
- Keep every reference file small enough for any agent to read in one go: when a file grows past about 50 KB, split it into a directory with an `index.md` and one file per section, and link straight to the section files. `references/dimensions/`, `references/specialties/`, `references/history/` and `references/targets/` are split this way.

## Languages

- **Only two kinds of files are kept in several languages, each language in its own file:** the README (English in `README.md`, other languages in `README.<language code>.md`, such as `README.zh-CN.md`) and the skill's entry file (English in `skills/finecomb/SKILL.md`, other languages in `skills/finecomb/SKILL.<language code>.md`, such as `SKILL.zh-CN.md`). Today that is English and Chinese; more languages may come later. Do not merge languages into one file.
- **Everything else is English only,** with no copies in other languages. That includes everything under `skills/finecomb/references/`, this file, the validation records, `.claude-plugin/` and any file or directory added later. `SKILL.<language code>.md` links to the English references, and gives English names in brackets where it names a section or row.
- `SKILL.md` is the source; the other `SKILL.<language code>.md` files are its translations. Change every language in the same commit, and keep steps, index entries, self-check items and links in step. Keep the READMEs in every language in step in the same commit. The skill tells the agent to write the report in the user's language.
- Only `SKILL.md` carries the frontmatter; its `description` is in English and may add a short line in other languages so requests in those languages trigger the skill.
- Rows cited from other places use the exact first-column name of the target row, written as `[27](dimensions/27-security-and-trust-boundaries.md) ("Row name")`.
- **To add a language:** add a `README.<language code>.md` and a `skills/finecomb/SKILL.<language code>.md`, and add the new language to the language links at the top of every README and every `SKILL*.md`.

## Skill format

- Skills live at `skills/<name>/SKILL.md`, following the skills CLI discovery rules. The frontmatter has `name` (matching the directory name), `description`, `license` and `metadata`.
- `description` stays within 1024 characters. `SKILL.md` stays under 500 lines; details go into `references/`.
- The skill is installed on its own, so every relative link and anchor must resolve inside `skills/finecomb/`. The skill does not link to the READMEs or the validation records.
- If the skill is renamed, update `.claude-plugin/marketplace.json`.

## Content

- The root-cause facets (`references/facets.md`) are the core. Coverage grows as general, domain-independent facets and checkpoints derived from real vulnerabilities, not as entries for a single CVE, product or algorithm.
- The "what counts as a problem" and "question to ask" columns state general criteria. Real incidents go into `references/history/` or the facets' "real cases" column.
- Keep the rule "when one check is restricted, skip only that check; never quit or silently downgrade the whole review" at the top of `SKILL.md`.
- Write short sentences with common words. Use the real names of things, and the same word for the same thing everywhere.
- The repository holds no exploit code, payloads, attack tools or exploitation steps. Keep the "Security and authorized use" section of every README and `SECURITY.md` consistent with the skill's execution boundaries (`references/scope.md`).

## Validation

- Coverage is checked by hit tests. For each vulnerability, take the real root cause from the patch, the official advisory or a credible post-mortem; judge it using only the facets' "question to ask" column and the other tables' "checkpoint", "how to check" and "what counts as a problem" columns, never the case columns.
- **Hit**: a row asks straight into the root cause. **Partial**: a row only points in the right direction. **Missed**: no row leads there.
- Turn every partial or missed result into a general question or checkpoint at the facet level, never a row for one CVE or product. A fix counts only after at least two other samples of the same class, not used to make the fix, are retested.
- The skill must work with any agent that can read files, not only Claude. Blind audits with other agents (mode B in the validation README) are how that is measured.
- The records live on the orphan `validation` branch of the published repository (<https://github.com/lian-yue/finecomb/tree/validation>), never on `main`. On that branch: `README.md` (method, formats, how to re-run and trace), the per-round records, `samples/*.jsonl` (reproduction data) and `runs/*.jsonl` (blind-audit results). Record every sample there in English, including where each gap was fixed and its reproduction data. When a new held-out round is added, update the totals under "How coverage is validated" in every README on `main`.
- Links from `main` to the records use the branch URL; links from the records to the skill use URLs of `main`, because the two branches never share a checkout.
- Write only patch-level root causes. No exploitation steps, payloads or undisclosed details.

## Checks before committing

- Every relative link and anchor resolves; every cited row name exists in its target section.
- No Chinese characters are left in files that must be English.
- The `description` is within 1024 characters; `SKILL.md` is under 500 lines.
- `.claude-plugin/marketplace.json` is valid JSON and lists `skills/finecomb`.
- The READMEs in every language have the same sections and install commands; every `SKILL*.md` has the same steps, index entries, self-check items and link targets.
