# finecomb

[中文说明](README.zh-CN.md)

finecomb is an exhaustive code review and security audit checklist, packaged as an [Agent Skill](https://agentskills.io). It works for code in any language, and for repositories that mix several languages. You point your agent at one or more targets (directories, packages, repositories, files, changes); the skill tells it what to check, how to check it, what counts as a problem and how to report it.

The name comes from "going over something with a fine-tooth comb": checking every strand so nothing is missed.

## What it covers

- **Root-cause facets**: cross-domain questions distilled from the root causes of real vulnerabilities, asked of every object. Examples: is the validation complete; does a declared control actually take effect; what happens to a value after it is validated; do the sizing pass and the writing pass agree; are paired fields updated together; can the same object be finalized twice; can an old but validly signed version be accepted again. The facets were hit-tested against real vulnerabilities in nginx, the Linux kernel, OpenSSL, OpenSSH, glibc, mainstream frameworks and enterprise appliances, and on-chain and zero-knowledge incidents; every missed root cause was turned into a facet.
- **45 general dimensions**: dead code, duplication, API contracts, error handling, concurrency, resources, crash recovery, security, privacy, performance, configuration, supply chain, tests, documentation, long-running behavior and more.
- **50 specialties by target type**: networking, protocol parsing, cryptography, authentication, single sign-on and federated identity, databases, file systems, queues, schedulers, server middleware, SSRF, subprocesses, interpreters and VMs, proxies and tunnels, rule engines, system calls, local privileged components, kernel drivers and virtualization, mobile app components, code generators, cross-language boundaries, CI and infrastructure as code, publishable packages, client apps, firmware, data pipelines, LLMs and agents, payments and accounting, notifications and outbound messages, smart contracts, DeFi and oracles, cross-chain bridges, wallets and signing, account abstraction, zero-knowledge proofs, blockchain nodes and consensus, and more.
- **Language pitfall tables** for Go, Python, JavaScript/TypeScript, C/C++, Rust, Java/Kotlin, C#/.NET, PHP, Ruby, Shell, Swift/Objective-C and SQL, the contract languages Solidity/Vyper, Solana, Move, CosmWasm, Cairo, TON and zero-knowledge circuits, plus a method for building a table for any other language.
- **Historical vulnerability patterns**: generalized mechanisms distilled from the CWE Top 25, CISA's most exploited vulnerabilities and major incidents (Log4Shell, Heartbleed, the xz backdoor, MOVEit, Citrix Bleed, bridge and DeFi exploits and more), each with what to look for in a review.
- **Targets that are not source code**: when the target is a binary, an installer, a browser extension, firmware, a container image, a published package, a live URL, domain and mail configuration, a host, a Kubernetes cluster, a cloud account, a SaaS tenant, a database, logs and packet captures, an on-chain contract address, a design document, an agent configuration or a dependency manifest, what evidence is available, what to check first, what cannot be seen and what needs authorization.
- **Five per-object question lists** (public entry points, shared state, invariants, external side effects, background flows), a threat model, evidence levels, severities, a report format and tool options for each ecosystem.

By default the review is read-only. The skill looks for the target project's own rules first (for example `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `SECURITY.md`) and follows them; when there are none, it uses conservative defaults: no writes to the source tree and no installs; the network is used only for read-only lookups that check dependency versions for known security issues (only dependency names and versions are sent, never source code or secrets). When the target is a live service, an account, a tenant or an on-chain contract, access is passive and read-only by default; active scanning, login attempts and exploit verification need written authorization from the asset owner, test artifacts are cleaned up and obtained data is disposed of at the end, and no transactions are sent or signed on chain.

## How coverage is validated

The root-cause facets and checkpoints were built by hit-testing against real vulnerabilities, not written from memory:

1. Find the real root cause of a vulnerability from its patch or official advisory.
2. Using only the skill's "question to ask" and "what counts as a problem" columns, never the example columns, decide whether a reviewer following the questions would reach that root cause.
3. When it would not, generalize the miss into a question that does not depend on the domain or language, add it, and retest on new samples that were not used to write the skill.

The latest held-out round (31 vulnerabilities not used to write the skill):

| Samples | Hit | Partial | Missed |
| --- | --- | --- | --- |
| Linux kernel, 2024–2026 (15) | 14 | 1 | 0 |
| nginx, 2009–2021 (8) | 8 | 0 | 0 |
| Apache httpd, HAProxy, Envoy, OpenSSL, 2024–2025 (8) | 8 | 0 | 0 |

The one partial result has since become new questions (implicit checks lost when a call is replaced, and where an external offset lands). A hit means the questions lead a reviewer to the root cause; it does not guarantee that following the checklist finds the bug in the code.

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
- "Audit the contract at `0x…` on Ethereum mainnet with finecomb."

The agent resolves the target and exclusions, builds a factual baseline (languages, entry points, shared state, threat model), runs the question lists, root-cause facets, dimensions, specialties and language tables that apply, and writes a report where every finding has a location, trigger conditions, evidence, impact, a recommendation and an evidence level. Excluded code is still read when a call chain passes through it; it is only exempt from findings.

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
    │       ├── facets.md        root-cause facets
    │       ├── dimensions.md    45 general dimensions
    │       ├── specialties.md   50 specialties
    │       ├── history.md       historical vulnerability patterns
    │       ├── targets.md       targets that are not source code
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
