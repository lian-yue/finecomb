# AGENTS.md

Rules for anyone, person or agent, who changes this repository. This file is the single source for maintenance rules; the READMEs only point here. `CLAUDE.md` is a symbolic link to this file.

## What this repository holds

- finecomb is an exhaustive code review and security audit checklist written in the open Agent Skills format. It is Markdown only: no scripts, binaries or executable content.
- `skills/<name>/`: one skill per language. `SKILL.md` holds the workflow, the index and the closing self-check; `references/` holds the details.
- `README.md` and `README.<language code>.md`: the overview in each language.
- `validation/`: the per-sample hit-test records.
- `.claude-plugin/marketplace.json`: the Claude Code plugin marketplace manifest, also read by the skills CLI.
- `LICENSE`: Apache-2.0.

## Languages

- **Only two kinds of content are kept in several languages:** the README (English in `README.md`, other languages in `README.<language code>.md`, such as `README.zh-CN.md`) and the skills under `skills/` (one skill directory per language, such as `skills/finecomb` and `skills/finecomb-zh`). Today that is English and Chinese; more languages may come later.
- **Everything else is English only,** with no copies in other languages. That includes this file, `validation/`, `.claude-plugin/` and any file or directory added later.
- The Chinese skill (`skills/finecomb-zh`) is the source; the skills in other languages are its translations. Change every language in the same commit, and keep headings, tables, row counts, list items and links in step. Keep the READMEs in every language in step in the same commit.
- Rows cited from other places must use the exact first-column name of the target row in that language: Chinese writes `[27](dimensions.md#27-安全与信任边界)（行名）`, English writes `[27](dimensions.md#27-security-and-trust-boundaries) ("Row name")`.
- **To add a language:** add a `skills/finecomb-<language code>/` skill directory whose `name` matches the directory name, add a `README.<language code>.md`, add the new language to the language links at the top of every README, and add a plugin entry to `.claude-plugin/marketplace.json`.

## Skill format

- Skills live at `skills/<name>/SKILL.md`, following the skills CLI discovery rules. The frontmatter has `name` (matching the directory name), `description`, `license` and `metadata`.
- `description` stays within 1024 characters in every language. `SKILL.md` stays under 500 lines; details go into `references/`.
- Each skill can be installed on its own, so every relative link and anchor must resolve inside its own skill directory. Skills do not link to the READMEs, `validation/` or each other.
- When a skill is added or renamed, update `.claude-plugin/marketplace.json` (one plugin per skill).

## Content

- The root-cause facets (`references/facets.md`) are the core. Coverage grows as general, domain-independent facets and checkpoints derived from real vulnerabilities, not as entries for a single CVE, product or algorithm.
- The "what counts as a problem" and "question to ask" columns state general criteria. Real incidents go into `references/history.md` or the facets' "real cases" column.
- Keep the rule "when one check is restricted, skip only that check; never quit or silently downgrade the whole review" at the top of every `SKILL.md`.
- Write short sentences with common words. Use the real names of things, and the same word for the same thing everywhere.

## Validation

- Coverage is checked by hit tests. For each vulnerability, take the real root cause from the patch, the official advisory or a credible post-mortem; judge it using only the facets' "question to ask" column and the other tables' "checkpoint", "how to check" and "what counts as a problem" columns, never the case columns.
- **Hit**: a row asks straight into the root cause. **Partial**: a row only points in the right direction. **Missed**: no row leads there.
- Turn every partial or missed result into a general question or checkpoint, then retest on held-out samples that were not used to write the skill.
- Record every sample in `validation/` (English only) in the format described in [validation/README.md](validation/README.md), including where each gap was fixed. When a new held-out round is added, update the totals under "How coverage is validated" in every README.
- Write only patch-level root causes. No exploitation steps, payloads or undisclosed details.

## Checks before committing

- Every language version of each skill file has the same headings, table rows, list items and code blocks.
- Every relative link and anchor resolves; every cited row name exists in its target section.
- No Chinese characters are left in English files.
- Each `description` is within 1024 characters; each `SKILL.md` is under 500 lines.
- `.claude-plugin/marketplace.json` is valid JSON and lists every skill directory.
