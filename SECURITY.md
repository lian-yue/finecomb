# Security policy

## What this repository contains

finecomb is a code review and security audit checklist, written as Markdown files. It contains no executable code, exploits, payloads or attack tools, and installing it runs nothing.

Real vulnerabilities appear only as short root causes at the patch or advisory level. They are taken from public fixes and advisories, and they are there so that reviewers can recognize and prevent the same class of flaw. The repository does not describe how to exploit them.

## Intended use

finecomb is for defensive work: reviewing code and assets you own or are authorized to assess, security research, teaching, and security testing that the owner of the asset has authorized.

Do not use it to scan, access, exploit, disrupt or change any system you are not authorized to test. The skill itself keeps a review read-only by default and limits what an agent may do against live targets; see [execution boundaries during review](skills/finecomb/references/scope.md#execution-boundaries-during-review).

## Supported versions

Only the latest commit on the `main` branch is maintained. Fixes are not backported to older versions.

## Reporting a security problem in finecomb

Examples of what to report:

- an instruction in the skill that could lead an agent to take a harmful or unauthorized action, such as running commands, sending data out, or acting on production systems outside the documented boundaries;
- a way for content inside a reviewed target (comments, documents, tool output) to make an agent following the skill widen its scope or ignore its boundaries;
- content that could leak secrets or personal data from a reviewed project into a report or elsewhere;
- text that reads as exploitation instructions rather than review questions.

Email **finecomb@lianyue.com** with the file and line, what could happen, and how you found it. Please do not open a public issue for a problem that could be misused before it is fixed. Reports are handled as soon as possible; there is no guaranteed response time.

## Reporting misuse

If you believe finecomb is being used against systems without authorization, or that content in this repository crosses into attack tooling, email **finecomb@lianyue.com**. You can also report abuse to GitHub.

## Vulnerabilities in other projects

finecomb does not accept or triage reports about third-party software. Report those to the affected project's maintainers or security team. The vulnerabilities cited in this repository are already public.
