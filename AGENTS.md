# AGENTS.md

Rules for anyone, person or agent, who changes this repository. This file is the single source for maintenance rules; the READMEs only point here. `CLAUDE.md` is a symbolic link to this file.

## What this repository holds

- finecomb is an exhaustive code review and security audit checklist written in the open Agent Skills format. It is Markdown only: no scripts, binaries or executable content.
- `skills/finecomb/`: the skill. `SKILL.md` (English) and `SKILL.<language code>.md` (other languages, such as `SKILL.zh-CN.md`) hold the workflow, the index and the closing self-check; `references/` holds the details, in English only.
- `README.md` and `README.<language code>.md`: the overview in each language.
- `.claude-plugin/marketplace.json`: the Claude Code plugin marketplace manifest, also read by the skills CLI. Its plugin entry carries `category`, `tags` and `keywords` for search.
- `.claude-plugin/plugin.json`: the plugin manifest (name, display name, version, description, author, links, license, search `keywords`). It declares no components, because the marketplace entry is `strict: false` and lists the skill itself.
- `.codex-plugin/plugin.json`: the Codex and ChatGPT plugin manifest, with the listing fields in `interface` (category `Security`). `.agents/plugins/marketplace.json`: the Codex marketplace manifest, pointing at this repository.

## Versions and releases

- One version number, following semantic versioning, is kept the same in four places: `.claude-plugin/plugin.json`, `metadata.version` in `.claude-plugin/marketplace.json`, `.codex-plugin/plugin.json`, and `metadata.version` in the frontmatter of `SKILL.md`.
- Installed plugins update only when the version changes. After changes to the skill are pushed to `main`, raise the version in all four places in one commit, tag that commit `v<version>` on `main` of the published repository, push the tag, and publish a GitHub release for the tag with notes that list what changed.
- Tags and releases belong to the published repository only; the validation branch is not tagged.
- `LICENSE`: Apache-2.0.
- `SECURITY.md`: what finecomb is, the authorization scope and the disclaimer. It lists no contact address.
- Hit-test records and samples do **not** live on `main`. They live on the separate `validation` branch (see [Validation](#validation)), so installing the skill never downloads them.

## Size and network

- The skill is a generalized checklist, not a collection of samples. It grows by adding general facets and checkpoints, never by adding one entry per vulnerability, so it should stay at hundreds of Markdown files, around a thousand at most. Samples may grow without limit, but they never go into `skills/`.
- The skill must work offline. During an audit it uses the network only for read-only lookups of dependency versions and known vulnerabilities, plus fetching the target itself when the caller explicitly gives a repository URL. It never fetches its own content, samples or validation data.
- Keep every reference file small enough for any agent to read in one go: when a file grows past about 50 KB, split it into a directory with an `index.md` and one file per section, and link straight to the section files. `references/dimensions/`, `references/specialties/`, `references/history/`, `references/targets/` and `references/facets/` are split this way.

## Languages

- **Only two kinds of files are kept in several languages, each language in its own file:** the README (English in `README.md`, other languages in `README.<language code>.md`, such as `README.zh-CN.md`) and the skill's entry file (English in `skills/finecomb/SKILL.md`, other languages in `skills/finecomb/SKILL.<language code>.md`, such as `SKILL.zh-CN.md`). Today that is English and Chinese; more languages may come later. Do not merge languages into one file.
- **Everything else is English only,** with no copies in other languages. That includes everything under `skills/finecomb/references/`, this file, the validation records, `.claude-plugin/` and any file or directory added later. `SKILL.<language code>.md` links to the English references, and gives English names in brackets where it names a section or row.
- `SKILL.md` is the source; the other `SKILL.<language code>.md` files are its translations. Change every language in the same commit, and keep steps, index entries, self-check items and links in step. Keep the READMEs in every language in step in the same commit. The skill tells the agent to write the report in the user's language.
- Only `SKILL.md` carries the frontmatter; its `description` is in English and may add a short line in other languages so requests in those languages trigger the skill. For the same reason, the search `keywords` in `.claude-plugin/` and `.codex-plugin/` may include a few terms in other languages; everything else there stays English.
- Rows cited from other places use the exact first-column name of the target row, written as `[27](dimensions/27-security-and-trust-boundaries.md) ("Row name")`.
- **To add a language:** add a `README.<language code>.md` and a `skills/finecomb/SKILL.<language code>.md`, and add the new language to the language links at the top of every README and every `SKILL*.md`.

## Skill format

- Skills live at `skills/<name>/SKILL.md`, following the skills CLI discovery rules. The frontmatter has `name` (matching the directory name), `description`, `license` and `metadata`.
- `description` stays within 1024 characters. `SKILL.md` stays under 500 lines and its body under about 5,000 tokens, because agents load all of it on every activation; details go into `references/`.
- `SKILL.md` keeps a short index, one line per part. The full lists live, in groups, in the index file of each directory (`dimensions/index.md`, `specialties/index.md`, `facets/index.md`, `languages.md` and so on). A new dimension or specialty is added to the right group in its index file, and a new specialty also gets a row in the baseline's "mapping known attack mechanisms" table; neither goes into `SKILL.md`.
- The review options are defined in `references/scope.md` ("Review options"); the options table in every `SKILL*.md` is a summary of it and is kept in step with it.
- The skill is installed on its own, so every relative link and anchor must resolve inside `skills/finecomb/`. The skill does not link to the READMEs or the validation records.
- If the skill is renamed, update every manifest: `.claude-plugin/marketplace.json`, `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` and `.agents/plugins/marketplace.json`. When the coverage changes (new specialties, languages or domains), update the descriptions, `tags` and `keywords` in all of them so search stays accurate, and keep the keyword lists of the two `plugin.json` files the same.

## Content

- The root-cause facets (`references/facets/`) are the core. Coverage grows as general, domain-independent facets and checkpoints derived from real vulnerabilities, not as entries for a single CVE, product or algorithm.
- **Rely on the agent's own knowledge first.** A row names an area to check and the general question to ask there, briefly; the agent applies what it already knows about the specific platform, framework, protocol and its known issues. Add a new area in this brief form first. Write detailed checkpoints only for classes of root cause that agents are shown to miss, for example a class of public vulnerabilities that no general question reaches.
- **Every part says what it applies to.** Agents read a file, section or row only when the target has its subject, so each part must make that subject clear: a file or section through its title or an "Applies to" line, a row through its checkpoint name. Do not write the rule for loading only what applies in terms of particular languages, target types or examples; it holds for every part of the skill.
- The "what counts as a problem" and "question to ask" columns state general criteria. Real incidents go into `references/history/` or the facets' "real cases" column.
- Keep the rule "when one check is restricted, skip only that check; never quit or silently downgrade the whole review" at the top of `SKILL.md`.
- Write short sentences with common words. Use the real names of things, and the same word for the same thing everywhere.
- The repository holds no exploit code, payloads, attack tools or exploitation steps. Keep the authorization scope at the top of every `SKILL*.md`, the "Disclaimer and authorized use" section of every README and `SECURITY.md` consistent with the skill's execution boundaries (`references/scope.md`).

## Contributions

Anyone may help maintain finecomb. A contribution is accepted only if it is one of these two kinds:

1. **Generalized coverage.** A new or sharper root-cause facet, checkpoint, target type, domain table or language table, or better wording or links, stated as a general question that holds across products, versions and languages. It may use a public weakness classification (for example CWE, CAPEC or the OWASP lists) as its basis.
2. **A missed sample that is fully public.** A real vulnerability whose root cause the current skill does not reach (partial or missed under the rules in [Validation](#validation)), with the general fix it points to when possible.

A sample must be public and fixed:

- **Public:** it has an identifier or advisory from a public source, for example a CVE, CNVD or CNNVD ID, a GitHub or GitLab security advisory, a vendor advisory or a credible public post-mortem.
- **Fixed:** a fix has been released, as a published patch, a fixed release or a merged fix in a public repository.
- **Not accepted:** issues or advisories that are still open, vulnerabilities with no released fix, undisclosed or embargoed reports, and anything learned under a non-disclosure agreement.
- **Written as a root cause only:** one sentence at the patch or advisory level, with its sources. No exploitation steps, payloads or proof-of-concept code.

Skill changes go to `main` and follow the rest of this file. A sample's record and reproduction data go to the `validation` branch, in the format its README describes. Neither kind may add a row that names only one CVE, product or algorithm.

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
- Every manifest under `.claude-plugin/`, `.codex-plugin/` and `.agents/plugins/` is valid JSON; the Claude marketplace entry lists `skills/finecomb`; when the `claude` CLI is available, `claude plugin validate .` passes.
- The READMEs in every language have the same sections and install commands; every `SKILL*.md` has the same steps, options, index entries, self-check items and link targets.
