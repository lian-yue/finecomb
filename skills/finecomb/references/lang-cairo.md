# A.18 Cairo (Starknet)

For messages between layer 1 and layer 2, see also [4.42](specialties.md#442-cross-chain-bridges-and-messaging).

| Checkpoint | What counts as a problem |
| --- | --- |
| **`felt252` arithmetic** | `felt252` wraps around modulo a large prime and does not check overflow, so a subtraction can also produce a very large number; comparison semantics differ from ordinary integers; whether amounts and counters use checked integer types such as `u256` and `u128` instead |
| **Layer-1 message origin** | Whether an `#[l1_handler]` function checks that `from_address` is the expected layer-1 contract; otherwise any layer-1 contract can send it messages |
| Layer-1 to layer-2 addresses | Ethereum addresses and Starknet addresses have different value ranges; whether address conversion checks the range and the zero address |
| Failed layer-1 to layer-2 messages | A message from layer 1 to layer 2 is not retried automatically after it fails on layer 2; whether the layer-1 contract provides a flow to cancel the message and refund the assets |
| Signature replay | Whether account contracts and off-chain signatures include the chain ID, a nonce and the contract address |
| Storage variable name collisions | A storage address comes from the hash of the variable name; variables with the same name in different components, or before and after an upgrade, land in the same slot |
| Upgrades | Who has the permission to call `replace_class_syscall` to replace the code; whether the storage of the old and new classes is compatible |
