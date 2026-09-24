# A.15 Move (Aptos, Sui)

| Checkpoint | What counts as a problem |
| --- | --- |
| Abilities | Whether the `copy`, `drop`, `store` and `key` abilities of resource types are granted too broadly; assets that should not be copyable or droppable are given these abilities |
| Objects and sharing | Access control for Sui shared objects and owned objects; when anyone can pass in a shared object, whether the function checks the caller's permission |
| **Shared object identity** | When a function receives a shared object or a pool object, whether it checks that the object is the expected instance (by object ID or a registered field), not just that the type matches; an attacker can create an object of the same type and pass it in |
| Singleton objects | Whether config, registry and pool objects that should be globally unique can be created again by anyone; whether the one-time witness is used correctly |
| Object ownership | Aptos objects allow free transfer by default; Sui objects with `store` can be sent to any address with `public_transfer`; after an object is wrapped into another object, whether the original access checks still hold |
| Aptos object references | An `Object<T>` is only an address plus a type, and anyone can construct one; the real permissions live in references such as `ConstructorRef`, `TransferRef`, `ExtendRef` and `DeleteRef`; whether these are stored, returned or handed to someone who should not get them |
| Capabilities | Whether capability objects can be copied, transferred, or end up with someone who should not have them; whether a public function returns a capability by accident |
| Mutable references handed to other modules | After a `&mut` reference or a mutably borrowed object is passed to a function in another module, which fields that module can change |
| Flash loan receipts | The "hot potato" receipt type used for flash loans must not have `drop` or `store`; whether the repay function checks the amount, the coin type and the pool the receipt belongs to |
| Function values and reentrancy | Aptos Move 2.2 and later support function values: when this module calls a function passed in by the caller, that function may call back into this module; how far the runtime restricts reentrancy depends on the chain's implementation; whether this module's state is already consistent before the call |
| **Arithmetic and shifts** | Overflow in addition, subtraction, multiplication and division aborts, but a left shift does not check for overflow and simply drops the high bits; whether hand-written overflow checks are correct (in 2025 Cetus lost about 220 million US dollars because the threshold of an overflow check before a shift was wrong) |
| Visibility | The choice among `public`, `public(friend)` or `public(package)`, and `entry`; functions meant for internal use are made public |
| Upgrade policy | The package's upgrade policy and compatibility; who holds the upgrade authority |
| Old versions after an upgrade | After a Sui package upgrade, the old version can still be called; whether shared objects record a version number and check it at every entry point; whether new functions allowed by the upgrade compatibility rules give old objects permissions they did not have before |
| Generics and type parameters | Type parameters have no constraints, so callers can pass in unexpected coin or object types |
| Time and randomness | The source of on-chain time; correct use of the randomness module |
| Composing randomness and gas | Whether a function that uses on-chain randomness can be called by other contracts as part of a larger call: the caller can abort when the result is unfavorable (Sui requires `Random` to be used only in non-public `entry` functions, and Aptos requires entry functions marked `#[randomness]`); after the random result is known, when the favorable and unfavorable paths use different amounts of gas, an attacker can set the gas limit so that the unfavorable result fails and reverts |
