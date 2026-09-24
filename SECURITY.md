# Security policy and disclaimer

## What finecomb is

finecomb is a checklist for security audits and code review, written as Markdown files. It contains no executable code, exploits, payloads or attack tools, and installing it runs nothing.

Real vulnerabilities appear only as short root causes at the patch or advisory level, and only for vulnerabilities that are already public and fixed. They are there so that reviewers can recognize and prevent the same class of flaw. The repository does not describe how to exploit them.

## Authorization scope

Use finecomb only on targets you own, maintain or are authorized to assess.

- **No further authorization needed:** static, read-only review of source code you own, maintain or have the right to read, including public open-source repositories, and read-only lookups of dependency versions and known vulnerabilities.
- **Written authorization from the owner of the asset needed:** any active operation against live services, hosts, devices, networks, cloud accounts, SaaS tenants or on-chain systems, such as scanning, login attempts, fuzzing or exploit verification. The authorization states the scope, the time window and the allowed operations. Without it, only passive, low-rate observation of public information is allowed.
- **Never:** acting outside the authorization, logging in with credentials found during a review, cracking password hashes, reading unrelated users' data, social engineering or phishing without explicit authorization, or signing or sending on-chain transactions.
- **Findings in software you do not own:** report them to the affected project through its own security process, and do not publish them before a fix is available.

The skill follows these limits itself; see [Execution boundaries during review](skills/finecomb/references/scope.md#execution-boundaries-during-review).

## Disclaimer

- finecomb is provided "as is", without warranty of any kind, under the [Apache License 2.0](LICENSE). It does not guarantee that every defect is found, and a review that uses it is not a certification.
- You are responsible for obtaining the authorization each target needs and for following the laws, contracts and platform rules that apply to you, including the testing policies of cloud and SaaS providers.
- The authors and contributors are not responsible for any use of finecomb outside the authorization scope above, or for any damage caused by such use.

## Privacy

finecomb collects, stores and sends no data. It is a set of Markdown instructions that your own agent reads and follows in your own environment; it has no server, no telemetry and no account. What leaves your machine during a review depends on the agent and platform you use. The skill tells the agent to send out only dependency names and versions for known-vulnerability lookups, and to fetch a remote repository only when you give its URL; see [Execution boundaries during review](skills/finecomb/references/scope.md#execution-boundaries-during-review).

## Supported versions

Only the latest commit on the `main` branch is maintained.
