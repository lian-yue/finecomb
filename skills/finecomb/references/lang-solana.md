# A.14 Solana programs (Rust and Anchor)

| Checkpoint | What counts as a problem |
| --- | --- |
| **Account owner checks** | The owner program of an incoming account is not checked, so an attacker can pass in a data account they forged |
| **Signer checks** | An operation that needs authorization does not check whether the matching account signed |
| **Account identity and substitution** | The real addresses of system accounts, sysvars and program accounts are not checked, so forged accounts can take their place (in 2022 Wormhole did not check the instructions sysvar account, an already verified signature was forged, and about 320 million US dollars was lost) |
| **Account relationship chains** | Only the adjacent one-level links between accounts are checked (A records B, B records C), and the start of the chain does not land on a trusted root; an attacker can forge the whole chain (in 2022 Cashio was exploited this way, and forged collateral accounts were used to mint tokens) |
| Remaining accounts | `remaining_accounts` do not go through Anchor's constraint checks; whether the program itself checks the owner, type and identity of each of these accounts |
| Derived address seeds | The seeds of a program derived address (PDA) do not include enough distinguishing fields, so different users or purposes collide on the same address; the canonical bump is not used |
| Cross-program invocation | The ID of the called program is not checked; signing authority is passed through a cross-program invocation to an untrusted program |
| Data after cross-program invocation | After a cross-program invocation changes an account, the old copy the program deserialized earlier is not refreshed automatically (Anchor requires calling `reload()`), so later checks use stale data |
| Duplicate accounts | The same account is passed in as two parameters at once (such as the same account as sender and receiver), which breaks the balance calculation |
| Account closing and revival | Data is not zeroed after an account is closed; after the rent is refunded, the account is funded again in the same transaction and "revives" |
| Repeated initialization | `init_if_needed` or hand-written initialization does not stop accounts that are already initialized, so fields such as the owner can be written again |
| Token-2022 extensions | Extensions such as transfer fees, transfer hooks, permanent delegates, default frozen state and confidential transfers change transfer semantics; whether the program accepts only the classic token program, or handles the amount actually received according to the extensions; whether the token program ID is checked |
| Instruction introspection | When the instructions sysvar is used to check other instructions in the same transaction, whether instructions are located by absolute index and their program ID is checked; whether an attacker can insert extra instructions to shift the positions |
| Program upgrade authority | Whether the program is upgradeable, and who holds the upgrade authority; assets held by an upgradeable program are in effect handed to whoever holds the upgrade authority |
| Durable nonce transactions | A transaction that uses a durable nonce does not expire when its blockhash becomes too old, so once signed it can be submitted much later; when multisig or admin operations are pre-signed as this kind of transaction, the signers are signing an authorization that stays valid for a long time (see also [4.43](specialties.md#443-wallets-signing-and-off-chain-components) ("Pre-signed and long-lived signatures")) |
| Type confusion | Account data of different types has no discriminator field, so one kind of account can be parsed as another |
| Integers | Release builds do not check overflow by default; checked arithmetic should be used |
| Compute budget and space | The compute unit limit keeps an operation from completing; not enough account space or rent |
| Anchor constraints | Constraints missing from `#[account(...)]` (`has_one`, `seeds`, `owner`, `signer`); whether the reason for using `UncheckedAccount` is written down |
