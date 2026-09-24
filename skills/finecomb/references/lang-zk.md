# A.16 Zero-knowledge circuits (Circom, halo2, Noir and others)

| Checkpoint | What counts as a problem |
| --- | --- |
| **Assigned but not constrained** | Circom assigns a signal with `<--` or `=` without a matching `===` constraint (use `<==` whenever possible); halo2 uses `assign_advice` to write a cell that should equal an existing value, without adding a copy constraint (`copy_advice` or `constrain_equal`). The forgery bugs in Tornado Cash in 2019 and in Zcash Orchard in 2026 both belong to this kind |
| Enabling selectors and gates | The selector of a halo2 custom gate is not enabled on every row that needs it; a lookup table constrains only some of the columns; whether equality constraints between regions connect back to the real source |
| Comparison and bit decomposition | "Less than" has no built-in meaning in a circuit: whether comparisons and bit decompositions constrain both that each bit can only be 0 or 1 and the upper bound on the number of bits |
| Division and inversion | When division uses a quotient and a remainder supplied by the witness, whether the remainder is constrained to be less than the divisor; how inverting zero is handled |
| Conditions and selection | A circuit has no real branches, and both paths are computed; when multiplication is used for selection, whether the selector bit is constrained to 0 or 1 |
| Indexes and lookups | How an index taken from a witness value is constrained; out-of-range indexes |
| Hashes and commitments | Whether the hash in the circuit matches the off-chain implementation; whether the conversion between field elements and bytes is unique |
| Public and private | A value that should be public and checked by the verifier is treated as a private witness; the order of public inputs differs from what the verifier expects |
| Unconstrained computation | Whether the results of computations that run only on the prover side, such as Noir `unconstrained` functions and Circom `<--`, are checked by later constraints |
| Toolchain versions | Versions and known bugs of the circuit compiler and the proving library; whether the trusted setup files and verification keys match the circuit version |
