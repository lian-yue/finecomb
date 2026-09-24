# On-chain contract addresses

Applies to: targets given only as a contract address (or protocol name). Only do read-only on-chain queries and local fork simulation; do not send transactions, do not sign anything, and do not touch private keys.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Source matches bytecode** | Is there verified source code on the block explorer; do the compiler version, optimization settings and constructor arguments match the on-chain bytecode; without verified source code, you can only decompile, and the conclusion notes that it is based on bytecode |
| **Proxy and implementation** | Is it a proxy contract (read from standard slots such as EIP-1967); the current implementation contract address; whether the implementation contract itself has been initialized |
| Initialized version | Does the initialized version number recorded by an upgradeable contract match what the current implementation requires; are variables added in an upgrade still zero |
| **Privilege holders** | The actual addresses of the owner, administrators, upgraders, pausers and members of each role; whether they are externally owned accounts, multisigs or timelocks; multisig thresholds and signers; timelock delays; reconstruct all members from role grant and revoke events, focusing on deployer and developer addresses |
| Pending changes | Operations that are queued but not yet executed in timelock queues, multisig pending queues and governance proposals; decode the call data of each one |
| Upgrade and parameter history | Past upgrade, parameter change and privilege transfer events; whether there have been unusual recent changes |
| Current parameters | Are current values such as fees, caps, oracle addresses and allowlists reasonable, and do they match the documentation |
| Assets and approvals | Assets held by the contract; token approvals the contract has given to others; the size of the approvals users have given to the contract |
| Contracts it depends on | The addresses of the oracles, routers, tokens and other protocols it calls, and their own privileges and upgrade state |
| Multi-chain deployments | Are the addresses, code versions, parameters and privileges of the same protocol consistent across chains |
| **Cross-chain messaging configuration** | The actual configuration of a cross-chain application on the messaging layer: the number of verifiers and the threshold for each path, whether the default configuration is inherited, whether peer addresses are correct; bridge limits and pause privileges |
| Account types | Whether related addresses are externally owned accounts that have delegated code through EIP-7702; module and paymaster configuration of smart contract wallets |
| Transaction history | Unusual calls, failed transactions; preparatory moves that suggest an attack (small test transactions, interactions with newly deployed contracts) |

When source code is available, also review the code per [4.38](../specialties/4.38-smart-contracts-and-on-chain-interaction.md), [4.41](../specialties/4.41-defi-economics-and-oracles.md) to [4.43](../specialties/4.43-wallets-signing-and-off-chain-components.md) and the matching language tables; for contracts that verify zero-knowledge proofs, also see [4.44](../specialties/4.44-zero-knowledge-proofs-and-circuits.md); for smart contract wallets and delegation, see [4.50](../specialties/4.50-account-abstraction-and-smart-contract-wallets.md).
