# A.17 CosmWasm and Cosmos SDK modules

CosmWasm contracts are usually written in Rust, so check them with [A.5 Rust](lang-rust.md) first and then with this table. Cosmos SDK chain modules are written in Go, so check them with [A.1 Go](lang-go.md) first; general checks for nodes and consensus are in [4.45](specialties/4.45-blockchain-nodes-consensus-and-protocol-implementations.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| **Address validation** | After an address is received as a `String`, it is stored or compared without being validated and normalized through `addr_validate`; the same address in different letter case is treated as two addresses; storage should hold validated `Addr` values |
| **Funds and denominations** | Whether `info.funds` is checked for denomination and amount; with several denominations, only the first one is looked at; when an entry point that needs no payment receives funds, the money stays in the contract |
| Access control and migration | The permission check of each execute message; the contract admin can migrate the contract to any code, so who is this admin; whether the `migrate` entry point checks the old and new versions |
| Submessages and replies | The `reply_on` setting of a `SubMsg`; after a failed submessage is handled by `reply`, which state is rolled back and which is not; whether the temporary state that `reply` relies on is reliable |
| Unbounded iteration | Unbounded `range` over storage, or query interfaces without pagination; once the data grows past the gas limit, the contract is stuck for good |
| Integers and panics | Plain arithmetic on types such as `Uint128` panics on overflow and aborts the whole execution; in some `cosmwasm-std` versions, a few operations wrap around on overflow instead of panicking (CWA-2024-002) |
| Unsaved state | A struct read from storage is changed but never written back |
| **IBC packet handling** | An error returned from `ibc_packet_receive` aborts the whole transaction, and the relayer cannot submit it; a failed acknowledgement should be returned instead; whether assets are refunded on timeouts and failed acknowledgements; whether the channel and the counterparty port are checked |
| Block hooks | In contract entry points that the chain calls automatically every block (`sudo` or chain-specific hooks), unbounded work or a panic can slow down or even halt the whole chain |
| Cross-module accounting | The contract's internal ledger disagrees with the balances in the chain's bank module; the admin of a token factory token can mint more or burn; callbacks from chain modules that the contract depends on |
| Signers | The signer of an SDK message is decided by the signer field in the message definition (`cosmos.msg.v1.signer`); whether it is the same address that the handler actually uses |
| Nondeterminism | A chain module on the consensus path iterates over Go maps, uses floating point, reads local time, starts goroutines or calls external services; different results on different nodes make the chain fork or halt |
