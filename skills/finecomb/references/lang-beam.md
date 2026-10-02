# A.23 Erlang and Elixir (BEAM)

Applies to: BEAM applications, OTP supervision trees and their native extensions.

| Checkpoint | What counts as a problem |
| --- | --- |
| Atom lifetime | Unbounded external names become atoms and exhaust a VM-wide resource; accepting an existing atom is mistaken for authorizing its use |
| Mailboxes | Selective receives leave unmatched messages accumulating; timeout or cancellation leaves stale replies that affect a later request |
| Correlation and ordering | A message is accepted without binding its reference and sender to the outstanding operation; ordering across different senders is assumed |
| Supervision | Restart repeats external effects or loses required state; linked process failure or a restart storm takes unrelated services down |
| Term decoding | External terms are trusted after decoding; the `safe` option is treated as schema validation or a resource limit |
| Native functions | A NIF blocks schedulers or corrupts the VM; process isolation is assumed to contain native memory errors |
| Shared storage | ETS ownership, access mode or multi-step updates violate the intended isolation or atomicity |
| Distributed nodes | Node connectivity grants more authority than intended; distribution cookies or network location substitute for application authorization |

Semantic references: [Erlang common caveats](https://www.erlang.org/doc/system/commoncaveats.html), [robustness](https://www.erlang.org/doc/system/robustness.html), [term decoding API](https://www.erlang.org/doc/apps/erts/erlang.html). For retries and native boundaries also use [4.10](specialties/4.10-message-queues-and-async-jobs.md) and [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md).
