# Incident reports and postmortems

Applies to: targets where you only have incident reports, postmortem documents, tickets or timelines, without the original evidence. All conclusions are "Unverified" by default unless original evidence is attached.

| Checkpoint | What counts as a problem |
| --- | --- |
| Complete timeline | Are the times of the first intrusion, detection, containment and recovery all present, with sources and time zones noted; how was the dwell time determined |
| **Root cause separated from trigger** | Does it state the root cause, or only the trigger; why the protections did not stop it and why monitoring did not catch it (detection gaps); classify it per the [root-cause facets](../facets.md) |
| Basis for the impact scope | What logs back "no data exfiltration found", and does log retention cover the whole dwell time; when logs are missing, the conclusion should be "cannot be determined" |
| **Complete remediation** | Are all leaked or possibly leaked credentials, tokens and keys rotated; are persistence mechanisms (backdoor accounts, scheduled tasks, OAuth authorizations, SSH public keys) removed; are hosts from the same batch and similar systems all fixed |
| Checking for the same issue elsewhere | Does the same mechanism also exist in other systems, repositories and environments (compare with [historical vulnerability patterns](../history/index.md)) |
| Improvement items | Does each item have an owner, a deadline and a way to verify it; items that only say "raise awareness" do not count |
| Notification obligations | Are the notification deadlines required by applicable regulations and contracts met; leave legal judgment to the legal team (see [30](../dimensions/30-privacy-data-governance-and-compliance.md) ("Source of requirements")) |
| Evidence preservation | Are the original logs, images and memory preserved with their digests recorded, and can they still be re-examined now |
