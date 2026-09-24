# finecomb

<a href="https://skills.sh/lian-yue/finecomb"><img alt="skills.sh" src="https://skills.sh/b/lian-yue/finecomb?style=for-the-badge" height="28"></a>

[中文说明](README.zh-CN.md)

finecomb is an exhaustive code review and security audit checklist. It is written in the open [Agent Skills](https://agentskills.io) format and is not tied to one agent or one way of installing:

- install it with one command through the skills CLI of [skills.sh](https://skills.sh);
- add it as a plugin marketplace in Claude Code;
- tell your agent "Install https://github.com/lian-yue/finecomb for me" and let it install itself;
- or copy the skill directory by hand.

It works with any agent that supports skills, such as Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot and OpenCode; see [Install](#install). It works for code in any language, and for repositories that mix several languages. You point your agent at one or more targets (directories, packages, repositories, files, changes); the skill tells it what to check, how to check it, what counts as a problem and how to report it.

The name comes from "going over something with a fine-tooth comb": checking every strand so nothing is missed.

## What it covers

- **Root-cause facets**: cross-domain questions distilled from the root causes of real vulnerabilities, asked of every object. Examples: is the validation complete; does a declared control actually take effect; what happens to a value after it is validated; do the sizing pass and the writing pass agree; are paired fields updated together; can the same object be finalized twice; can an old but validly signed version be accepted again. The facets were hit-tested against real vulnerabilities in nginx, the Linux kernel, OpenSSL, OpenSSH, glibc, mainstream frameworks and enterprise appliances, and on-chain and zero-knowledge incidents; every missed root cause was turned into a facet.
- **46 general dimensions**: dead code, duplication, API contracts, error handling, concurrency, resources, crash recovery, security, privacy, abuse and fraud resistance, performance, configuration, supply chain, tests, documentation, long-running behavior and more.
- **78 specialties by target type**: plugin and app platforms, workflow automation and low-code, multi-tenant platforms and user content hosting, remote access and device management, security tools, certificate authorities, DNS servers, telecom and SMS, end-to-end encrypted messaging, trading and exchanges, hardware designs, location and visibility of personal data, container runtimes, package registries, code hosting, online games, media streaming and DRM, backups, notebooks, biometrics and identity verification, health data, mail servers, sandboxes and broker processes, trusted execution environments and enclaves, industrial control and cyber-physical safety, search and retrieval, sync and offline clients, serverless and edge functions, networking, protocol parsing, cryptography, authentication, single sign-on and federated identity, databases, file systems, queues, schedulers, server middleware, SSRF, subprocesses, interpreters and VMs, proxies and tunnels, rule engines, system calls, local privileged components, kernel drivers and virtualization, mobile app components, code generators, cross-language boundaries, CI and infrastructure as code, publishable packages, client apps, firmware, data pipelines, LLMs and agents, payments and accounting, notifications and outbound messages, smart contracts, DeFi and oracles, cross-chain bridges, wallets and signing, account abstraction, zero-knowledge proofs, blockchain nodes and consensus, and more.
- **Language pitfall tables** for Go, Python, JavaScript/TypeScript, C/C++, Rust, Java/Kotlin, C#/.NET, PHP, Ruby, Shell, Swift/Objective-C and SQL, the contract languages Solidity/Vyper, Solana, Move, CosmWasm, Cairo, TON and zero-knowledge circuits, plus a method for building a table for any other language.
- **Historical vulnerability patterns**: generalized mechanisms distilled from the CWE Top 25, CISA's most exploited vulnerabilities and major incidents (Log4Shell, Heartbleed, the xz backdoor, MOVEit, Citrix Bleed, bridge and DeFi exploits and more), each with what to look for in a review.
- **Targets that are not source code**: when the target is a binary, an installer, a browser extension, firmware, a container image, a published package, a live URL, domain and mail configuration, a host, a Kubernetes cluster, a cloud account, a SaaS tenant, a database, logs and packet captures, an on-chain contract address, a design document, an agent configuration or a dependency manifest, what evidence is available, what to check first, what cannot be seen and what needs authorization.
- **Five per-object question lists** (public entry points, shared state, invariants, external side effects, background flows), a threat model, evidence levels, severities, a report format and tool options for each ecosystem.

By default the review is read-only. The skill looks for the target project's own rules first (for example `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `SECURITY.md`) and follows them; when there are none, it uses conservative defaults: no writes to the source tree and no installs; the network is used only for read-only lookups that check dependency versions for known security issues (only dependency names and versions are sent, never source code or secrets). When the target is a live service, an account, a tenant or an on-chain contract, access is passive and read-only by default; active scanning, login attempts and exploit verification need written authorization from the asset owner, test artifacts are cleaned up and obtained data is disposed of at the end, and no transactions are sent or signed on chain. When one check cannot be done because of authorization, the environment, tools or the reviewer's own rules, only that check is skipped, with the reason and what is missing stated at the top of the report; everything else is still done, and the review never quits or silently downgrades.

## Disclaimer and authorized use

finecomb is for security audits and code review of targets you own, maintain or are authorized to assess.

- **Authorization scope.** Static, read-only review of source code you own, maintain or may read, including public open-source repositories, needs no further authorization. Any active operation against live services, hosts, devices, cloud accounts, SaaS tenants or on-chain systems needs written authorization from the owner of the asset; without it, only public information is observed, passively. The skill states this scope at the top of `SKILL.md` and follows it; the details are in [Execution boundaries during review](skills/finecomb/references/scope.md#execution-boundaries-during-review).
- **Content.** Markdown only: no scripts, exploits, payloads or attack tools. Real vulnerabilities, including the well-known incidents named above, appear only as short root causes of vulnerabilities that are already public and fixed. There are no exploitation steps.
- **Disclaimer.** finecomb is provided as is, without warranty, under the Apache License 2.0. You are responsible for having authorization for every target and for following the laws and platform rules that apply to you; the authors are not responsible for use outside this scope. See [SECURITY.md](SECURITY.md).

## How coverage is validated

The root-cause facets and checkpoints were built by hit-testing against real vulnerabilities, not written from memory:

1. Find the real root cause of a vulnerability from its patch or official advisory.
2. Using only the skill's "question to ask" and "what counts as a problem" columns, never the example columns, decide whether a reviewer following the questions would reach that root cause.
3. When it would not, generalize the miss into a question that does not depend on the domain or language, add it, and retest on new samples that were not used to write the skill.

Three held-out rounds (208 vulnerabilities not used to write the skill; each sample records only the root cause at the patch or advisory level). "Hit" includes weak hits:

| Samples | Hit | Partial | Missed |
| --- | --- | --- | --- |
| Linux kernel, 2024–2026 (15) | 14 | 1 | 0 |
| nginx, 2009–2021 (8) | 8 | 0 | 0 |
| Apache httpd, HAProxy, Envoy, OpenSSL, 2024–2025 (8) | 8 | 0 | 0 |
| Web frameworks and business applications (10) | 9 | 1 | 0 |
| Libraries in memory-safe languages: Go, Rust, Java, npm, Python, .NET (10) | 8 | 2 | 0 |
| Browsers, script engines and desktop clients (10) | 8 | 2 | 0 |
| Open-source cloud-native components and build tools (10) | 6 | 4 | 0 |
| Cryptographic libraries, authentication protocols and identity systems (10) | 6 | 4 | 0 |
| Smart contracts, DeFi, bridges and zero-knowledge proofs (10) | 7 | 3 | 0 |
| AI agents and LLM applications (10) | 9 | 1 | 0 |
| Open-source databases, storage and messaging (10) | 6 | 4 | 0 |
| Round 3: retest of six new root-cause facets (12) | 11 | 1 | 0 |
| Round 3: retest of five new questions in existing facets (10) | 10 | 0 | 0 |
| Round 3: facets with few samples, part A (14) | 13 | 1 | 0 |
| Round 3: facets with few samples, part B (12) | 12 | 0 | 0 |
| Round 3: mobile apps and mobile frameworks (10) | 8 | 2 | 0 |
| Round 3: OS kernels other than Linux: FreeBSD, OpenBSD, XNU, Windows (10) | 9 | 1 | 0 |
| Round 3: authorization and business logic in web applications (9) | 7 | 2 | 0 |
| Round 3: denial of service and resource exhaustion (10) | 8 | 2 | 0 |
| Round 3: open-source embedded systems, bootloaders and stacks (10) | 8 | 2 | 0 |

No sample needed a new root-cause facet. Every partial result was a specific checkpoint missing under an existing facet, or a general question that only lived in one domain's specialty. All of them have been added; the round 3 fixes still await their retest on unused samples. A hit means the questions lead a reviewer to the root cause; it does not guarantee that following the checklist finds the bug in the code.

## Skills in this repository

| Skill | Path |
| --- | --- |
| `finecomb` | [skills/finecomb](skills/finecomb/SKILL.md) |

The entry file comes in English (`SKILL.md`) and Chinese (`SKILL.zh-CN.md`); the reference files are in English. The skill works with requests in any language: the agent writes the report in the language the user writes in.

## Install

### Install with an agent

Tell your coding agent (Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot and others):

```text
Install https://github.com/lian-yue/finecomb for me
```

An agent reading this should install it as follows:

1. **Pick the scope.** If the user does not say, install for the current user so every project can use it; if the user asks for the current project only, install into the project.
2. **With Node.js available, use the skills CLI.** Set `-a` to your own agent ID (such as `claude-code`, `codex`, `cursor`, `gemini-cli`, `github-copilot` or `opencode`); drop `-g` to install into the project:

   ```sh
   npx skills add https://github.com/lian-yue/finecomb --skill finecomb -a claude-code -g -y
   ```

3. **Without Node.js, install by hand.** Shallow-clone the repository into a temporary directory, copy the whole `skills/finecomb` directory into your skills directory, for example `~/.claude/skills/finecomb` for Claude Code (see the table below for other agents), then delete the temporary directory.
4. **Check and report.** Confirm the target directory contains `SKILL.md`, and tell the user where it was installed, whether the agent needs a restart, and how to use it, for example "review `./src` with finecomb" or "audit `https://github.com/<owner>/<repo>` for me".

The skill contains only Markdown files and no scripts; installing it runs none of its content.

### skills CLI

Install it with the [skills CLI](https://github.com/vercel-labs/skills) ([skills.sh](https://skills.sh)):

```sh
npx skills add lian-yue/finecomb --skill finecomb
```

Common options:

| Option | Effect |
| --- | --- |
| `-l`, `--list` | List the skills in the repository without installing |
| `-g`, `--global` | Install for your user instead of the current project |
| `-a`, `--agent <agent>` | Target a specific agent, for example `claude-code`, `codex`, `cursor`, `gemini-cli`, `github-copilot` or `opencode` |
| `--copy` | Copy the files instead of symlinking them into the agent directory |
| `-y`, `--yes` | Skip the prompts |

The source can also be the full URL `https://github.com/lian-yue/finecomb`, or the skill's directory, `https://github.com/lian-yue/finecomb/tree/main/skills/finecomb`; to install from a local copy, pass its path.

After installing:

```sh
npx skills list
```

```sh
npx skills update finecomb
```

```sh
npx skills remove finecomb
```

To use it once without installing (the skill is turned into a prompt for the agent):

```sh
npx skills use lian-yue/finecomb --skill finecomb --agent claude-code
```

The CLI puts the skill into each agent's skills directory, for example:

| Agent | Project | User |
| --- | --- | --- |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `.agents/skills/` | `~/.codex/skills/` |
| Cursor | `.agents/skills/` | `~/.cursor/skills/` |
| Gemini CLI | `.agents/skills/` | `~/.gemini/skills/` |
| GitHub Copilot | `.agents/skills/` | `~/.copilot/skills/` |

See the skills CLI documentation for the full list of agents.

### Claude Code plugin marketplace

The repository includes `.claude-plugin/marketplace.json`, so Claude Code can add it as a plugin marketplace:

```text
/plugin marketplace add lian-yue/finecomb
/plugin install finecomb@finecomb
```

### Agents that do not load skills automatically

The skill is only Markdown files, so any agent that can read files can use it. If your agent does not load skills automatically, put `skills/finecomb` where it can read it and say:

```text
Read skills/finecomb/SKILL.md and follow its workflow to audit <target>.
```

### Manual install

Copy the whole `skills/finecomb` directory into your agent's skills directory (for example `.claude/skills/` for Claude Code).

## Usage

Ask your agent in plain language, for example:

- "Review `./server` with finecomb."
- "Review `./server`, `./client` and this change with finecomb."
- "Audit `src/` with finecomb, excluding `vendor/`, `third_party/` and generated code."
- "Review this change with finecomb."
- "Use finecomb to check concurrency in `pkg/cache`."
- "Audit the contract at `0x…` on Ethereum mainnet with finecomb."
- "Audit `https://github.com/<owner>/<repo>` for me." (Any GitHub, GitLab or other Git repository URL works, including one that points at a branch, tag, subdirectory or merge request; the agent clones it read-only into a temporary directory, reviews it there, and states the commit it reviewed in the report.)

The agent resolves the target and exclusions, builds a factual baseline (languages, entry points, shared state, threat model), runs the question lists, root-cause facets, dimensions, specialties and language tables that apply, and writes a report where every finding has a location, trigger conditions, evidence, impact, a recommendation and an evidence level. Excluded code is still read when a call chain passes through it; it is only exempt from findings.

## Layout

```text
finecomb/
├── .claude-plugin/
│   ├── marketplace.json     Claude Code plugin marketplace manifest
│   └── plugin.json          plugin manifest: name, description, search keywords
├── AGENTS.md                maintenance rules (CLAUDE.md is a symbolic link to it)
├── CLAUDE.md -> AGENTS.md
├── LICENSE
├── README.md
├── README.zh-CN.md
├── SECURITY.md              authorization scope and disclaimer
└── skills/
    └── finecomb/            the skill (English)
        ├── SKILL.md         workflow, index, closing self-check
        ├── SKILL.zh-CN.md   the same in Chinese
        └── references/
            ├── scope.md         invocation, scope, hard boundaries, sharding
            ├── baseline.md      factual baseline, threat model, mechanism map
            ├── questions.md     five per-object question lists
            ├── facets/          root-cause facets: index.md plus one file per group
            ├── dimensions/      46 general dimensions: index.md plus one file per dimension
            ├── specialties/     78 specialties: index.md plus one file per specialty
            ├── history/         historical vulnerability patterns: index.md plus one file per group
            ├── targets/         targets that are not source code: index.md plus one file per target form
            ├── report.md        evidence levels, report format, fix discipline
            ├── tools.md         tools per ecosystem
            ├── languages.md     how to use the language tables
            └── lang-*.md        one table per language
```

`SKILL.md` stays short; the agent reads a reference file only when the current step needs it.

## Maintenance

Contributions are welcome. Two kinds are accepted: generalized coverage (a new or sharper root-cause facet, checkpoint, target type or language table, stated as a general question), and a missed vulnerability sample that is already public and already fixed (for example with a CVE, CNVD or CNNVD ID, or a closed GitHub or GitLab security advisory with a released fix). Open, unfixed or undisclosed vulnerabilities are not accepted.

All maintenance rules are in [AGENTS.md](AGENTS.md) (`CLAUDE.md` is a symbolic link to it): what contributions are accepted, which content is kept in several languages (only the READMEs and the skill's `SKILL.md` entry files; everything else is English only), how the languages are kept in step, how to add a language, the skill format, how hit tests are run and recorded, and the checks before committing. The hit-test records and samples live on the separate [`validation` branch](https://github.com/lian-yue/finecomb/tree/validation), so installing the skill never downloads them; the skill itself needs no samples and works offline.

## License

[Apache-2.0](LICENSE)

## Acknowledgements

Thanks to Anthropic's [Claude for Open Source](https://claude.com/contact-sales/claude-for-oss) program, which gives open-source maintainers Claude Max for free. finecomb was written, hit-tested and translated with it.
