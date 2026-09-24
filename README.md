# finecomb

[中文说明](README.zh-CN.md)

finecomb is an exhaustive code review and security audit checklist, packaged as an [Agent Skill](https://agentskills.io). It works for code in any language, and for repositories that mix several languages. You point your agent at one or more targets (directories, packages, repositories, files, changes); the skill tells it what to check, how to check it, what counts as a problem and how to report it.

The name comes from "going over something with a fine-tooth comb": checking every strand so nothing is missed.

## What it covers

- **45 general dimensions**: dead code, duplication, API contracts, error handling, concurrency, resources, crash recovery, security, privacy, performance, configuration, supply chain, tests, documentation, long-running behavior and more.
- **38 specialties by target type**: networking, protocol parsing, cryptography, authentication, databases, file systems, queues, schedulers, server middleware, SSRF, subprocesses, interpreters and VMs, proxies and tunnels, rule engines, system calls, code generators, cross-language boundaries, CI and infrastructure as code, publishable packages, client apps, firmware, data pipelines, smart contracts and more.
- **Language pitfall tables** for Go, Python, JavaScript/TypeScript, C/C++, Rust, Java/Kotlin, C#/.NET, PHP, Ruby, Shell, Swift/Objective-C and SQL, plus a method for building a table for any other language.
- **Five per-object question lists** (public entry points, shared state, invariants, external side effects, background flows), a threat model, evidence levels, severities, a report format and tool options for each ecosystem.

By default the review is read-only. The skill looks for the target project's own rules first (for example `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `SECURITY.md`) and follows them; when there are none, it uses conservative defaults: no writes to the source tree and no installs; the network is used only for read-only lookups that check dependency versions for known security issues (only dependency names and versions are sent, never source code or secrets).

## Skills in this repository

| Skill | Language | Path |
| --- | --- | --- |
| `finecomb` | English | [skills/finecomb](skills/finecomb/SKILL.md) |
| `finecomb-zh` | Chinese | [skills/finecomb-zh](skills/finecomb-zh/SKILL.md) |

The two skills have the same content. Install only one: both respond to the same kind of request.

## Install

With the [skills CLI](https://skills.sh):

```sh
npx skills add lian-yue/finecomb --skill finecomb
```

For the Chinese version, use `--skill finecomb-zh`. Add `-g` to install for your user instead of the current project, or `-a <agent>` to target a specific agent. To install from a local copy, pass its path instead of `lian-yue/finecomb`.

You can also copy `skills/finecomb` by hand into your agent's skills directory (for example `.claude/skills/` for Claude Code).

## Usage

Ask your agent in plain language, for example:

- "Review `./server` with finecomb."
- "Review `./server`, `./client` and this change with finecomb."
- "Audit `src/` with finecomb, excluding `vendor/`, `third_party/` and generated code."
- "Review this change with finecomb."
- "Use finecomb to check concurrency in `pkg/cache`."

The agent resolves the target and exclusions, builds a factual baseline (languages, entry points, shared state, threat model), runs the question lists, dimensions, specialties and language tables that apply, and writes a report where every finding has a location, trigger conditions, evidence, impact, a recommendation and an evidence level. Excluded code is still read when a call chain passes through it; it is only exempt from findings.

## Layout

```text
finecomb/
├── LICENSE
├── README.md
├── README.zh-CN.md
└── skills/
    ├── finecomb/            English skill
    │   ├── SKILL.md         workflow, index, closing self-check
    │   └── references/
    │       ├── scope.md         invocation, scope, hard boundaries, sharding
    │       ├── baseline.md      factual baseline, threat model, mechanism map
    │       ├── questions.md     five per-object question lists
    │       ├── dimensions.md    45 general dimensions
    │       ├── specialties.md   38 specialties
    │       ├── report.md        evidence levels, report format, fix discipline
    │       ├── tools.md         tools per ecosystem
    │       ├── languages.md     how to use the language tables
    │       └── lang-*.md        one table per language
    └── finecomb-zh/         Chinese skill, same layout
```

`SKILL.md` stays short; the agent reads a reference file only when the current step needs it.

## Maintenance

- The Chinese skill (`skills/finecomb-zh`) is the source. The English skill is its translation. Change both in the same commit and keep headings, tables and row counts in step.
- Keep `SKILL.md` under 500 lines and move detail into `references/`.
- Relative links and anchors must resolve inside each skill directory, because each skill can be installed on its own.

## License

[Apache-2.0](LICENSE)
