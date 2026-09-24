# A.19 TON (FunC, Tact, Tolk)

TON contracts interact only through asynchronous messages, and each message runs in its own transaction. The EVM intuition that "a transaction either fully succeeds or fully reverts" does not hold here.

| Checkpoint | What counts as a problem |
| --- | --- |
| **Origin of transfer notifications** | When a Jetton transfer notification arrives, whether the sender is checked to be this contract's own Jetton wallet address; for contracts or off-chain services that must accept any Jetton (bridges, deposit endpoints), whether the sender wallet's code hash and the master contract it belongs to are checked; otherwise anyone can fake "tokens received" |
| **Non-atomicity across messages** | Once an operation is split into several messages, a failure in one of them partway through does not undo the state that earlier messages already committed; whether the state is still consistent after any step fails, and whether the assets can be recovered |
| Bounced messages | A message sent with the bounce flag comes back when it fails; whether the contract handles bounced messages and restores the balance or state it deducted earlier |
| Concurrent message flows | Several message flows arrive interleaved, and another flow changes the state between two messages; this is similar to reentrancy |
| **Accepting external messages** | When handling an external message, whether the contract checks the signature, sequence number and validity period before calling `accept_message()`; otherwise an attacker can send the same message again and again and drain the contract's balance |
| Gas and send modes | Whether the TON amount attached to a forwarded message and the send mode (such as the mode that carries the whole balance) are correct; a gas estimate that is too low makes a chain of messages fail partway |
| Storage fees | Once storage fees use up the contract's balance, the contract is frozen and may even be deleted |
| Code and data updates | Who can trigger `set_code` and `set_data`; whether the data format is still compatible after an upgrade |
| Integers and booleans | In FunC, true is `-1`; integers are 257-bit signed numbers; bitwise NOT and logical NOT are mixed up |
| Address formats | Addresses are written differently for different workchains and for bounceable versus non-bounceable forms; whether addresses are normalized to one form before they are compared |
| FunC `impure` | If a function is not marked `impure` and its return value is not used, the compiler removes the call, and the checks inside it disappear with it |
