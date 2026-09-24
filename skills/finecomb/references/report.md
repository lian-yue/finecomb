# Reporting and fixes

## Part V: Evidence levels and report format

Each issue record states its evidence level and its severity separately. The evidence standards below apply equally to security vulnerabilities, correctness defects and documentation issues:

| Level | Meaning |
| --- | --- |
| **Reproduced** | There is a reproducible input and an observable result under the recorded source, configuration and environment. Simulations, test doubles or isolated reproductions must state how they differ from the real environment |
| **Statically confirmed** | The required input, preconditions and reachable path have all been verified, and the code and contracts are enough to determine the result. It does not mean a production exploit was carried out or the scale of impact was measured |
| **Unverified** | Evidence of a key precondition, reachability, environment or result is missing; state what is still missing. List these separately; they do not count toward the number of confirmed issues |

A judgement of "not exploitable" or "limited impact" also needs evidence, and it must survive the same search for counter-evidence as a confirmed issue: state which guard blocks exploitation, where it is, and that it holds on every path. Judging something not exploitable only from the shape of the input (a restricted character set, a short length, digits only) is not evidence. When you cannot show this, write "Unverified", not "not exploitable".

Assess severity by actual impact, exploitability conditions, scope of impact and recovery cost. Do not assign it automatically from a vulnerability name or a tool label. A missing protection is not necessarily high risk by itself. If the project has a confirmed severity scale, use it; otherwise use:

| Severity | Criteria |
| --- | --- |
| **Critical** | Can lead to severe outcomes such as broad control of the system or a major loss of assets or secrets, the required conditions hold in the reviewed scenario, and immediate action is needed |
| **High** | Can lead to real unauthorized access, sensitive data leaks, corruption of important data or a sustained service outage, with evidence supporting the impact and the reachability conditions |
| **Medium** | The impact or the trigger conditions are limited, but it still breaks correctness, a security goal or availability |
| **Low** | A confirmed defect with small impact. Pure style choices, general hardening suggestions and unproven concerns are listed separately and are not passed off as vulnerabilities |

When a CVSS, CWE or other external standard mapping is needed, give the version, vector or item identifier used and the basis for it. Classification labels cannot replace evidence of the root cause and of reachability.

Write each issue in this shape:

| Field | Content |
| --- | --- |
| ID and location | A stable issue ID, `file:line`, a function or a configuration entry |
| Symptom and root cause | Expected behavior, actual behavior, and which contract or security goal is broken |
| Trigger conditions | The identity, input, configuration, execution order and environment needed; for security issues, add what the attacker can control |
| Evidence | The path from source to result, why the relevant guards did not block it, the minimal reproduction input or the static derivation; for Reproduced issues, attach the exact command, the exit status and the necessary output |
| Impact | What is affected, the scope, the cost and the recovery conditions; keep potential wider impact separate from proven results |
| Suggestion | The smallest fix location and the affected callers, and what acceptance should observe; describe out-of-scope changes separately |
| Severity / evidence / status | For example High / Statically confirmed / Not fixed; if it is fixed but not verified, say so explicitly |

The overall report contains:

- **Conclusion and scope**: the review target, the baseline, the verified configuration, the exclusions, and the confirmed issues sorted by severity. For a remote repository target, state the repository URL, the branch or tag, and the commit hash actually reviewed. For targets that are not source code, also state the artifact digest and version, the address and access time of a live target, the chain ID, address and block height at read time of an on-chain target, or the identifier of a cloud account, cluster or SaaS tenant together with the identity used to read it and that identity's permissions. If no issues were found, it can only be stated as "no confirmed issues found within the reviewed scope and conditions". At the top, list the checks that were deliberately skipped: which check, why it was skipped, and what would be needed to do it.
- **Coverage record**: for the chosen scale, list the general dimensions, objects and specialties, each with a status of "Checked, issues found", "Checked, none found", "Partially checked", "Not checked" or "Not applicable". For unfinished items, give the reason and the missing evidence; for not-applicable items, give the actual basis. Items with the same status can be listed together; hits in the mechanism table only need to be filed under the matching item.
- **Verification record**: the exact entry points run or reused, the input identity and the results. Clearly distinguish passed this time, reused from cache, failed, zero matches, skipped, timed out, environment error and not run. For scans, attach the tool version, the date of the rules or vulnerability database, the configuration and the exclusions; for network lookups, attach the sources queried and the time. For targets that are not source code, attach an evidence list: the digest, time obtained and storage location of each piece of raw evidence (artifacts, exports, responses, screenshots, packet captures, command output), and what was done with the obtained data at the end of the review.
- **Changes and remaining items**: which issues were fixed, which passed verification, and which were not changed and why. Distinguish "pre-existing", "introduced this time" and "origin unconfirmed". Without a comparable baseline, use the last one; do not temporarily restore the workspace to guess.

**Deduplication and disclosure**: merge issues with the same root cause and the same fix point into one entry, and list all affected entry points and consequences. Record different root causes separately, and link combined attacks through references. Known issues or false positives need current evidence; they cannot be closed based only on an old label. Redact accounts, credentials, user data and reachable addresses in the report as needed. Sending the report outside follows the confirmed authorization and the project's disclosure process.

**Machine-readable output**: when the report must feed CI or a code scanning platform, also produce it in a format such as SARIF, with locations, rules, severities and evidence levels that match the written report.

## Part VI: Fix discipline

Enter this part only when fixes are authorized. Change scope, compatibility cleanup, writing tests, the number of verification runs and documentation sync all follow [the target project's hard boundaries](scope.md#the-target-projects-hard-boundaries). This part only adds how review results are put into practice.

1. First state the root cause, the impact and the smallest fix point. Fix at the authoritative layer that performs the sensitive operation, not just by blocking one sample input or one UI entry. Check the equivalent reachable paths in scope at the same time; only report migration items that are out of scope.
2. Acceptance targets the success, rejection and real failure behavior of the original issue. In particular, check that no unwanted side effect happens after a rejection. Judge whether tests can catch the issue per [41](dimensions.md#41-test-quality). Mutation testing or repeatedly switching guards off is not required by default.
3. After verifying with the chosen smallest entry point, update the original issue's status, and distinguish fix done, verification done and remaining boundaries. When a mitigation blocks only some paths, state the residual risk; do not write it up as the root cause being removed.
4. A fix is itself a change: per the "review this change" row of [scaling to the request](scope.md#scaling-to-the-request), go through the five question lists and confirm no new issues were introduced. For the convergence criteria of multi-round fixes, see [sharding large targets and multi-round review](scope.md#sharding-large-targets-and-multi-round-review).
