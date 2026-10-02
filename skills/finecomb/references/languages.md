# Appendix A: Language runtime pitfalls

This appendix maps the categories of [16](dimensions/16-language-and-runtime-pitfalls.md) onto specific languages, and also collects each language's frequent pitfalls in other dimensions. How to use it:

- Pick the tables for the languages the target uses; for a mixed-language target pick every one, and check cross-language boundaries separately in [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md).
- The tables list only frequent, easy-to-miss items and **do not replace the category checks of 16**: for every category in 16 you still have to find that language's matching mechanism, item by item.
- Semantics change between versions. First confirm the language, runtime and compiler versions the target declares and actually uses, then decide whether an item applies.
- Select the actual hosts and deployment forms separately in [Appendix B](environments.md). The same language may use several backends with different contracts.
- For a language not in this appendix, build your own table with the method in [A.34 Other languages](#a34-other-languages).

## General-purpose languages

- [A.1 Go](lang-go.md)
- [A.2 Python](lang-python.md)
- [A.3 JavaScript and TypeScript](lang-javascript.md)
- [A.4 C and C++](lang-c-cpp.md)
- [A.5 Rust](lang-rust.md)
- [A.6 Java and the JVM](lang-jvm.md)
- [A.7 C#, F# and .NET](lang-dotnet.md)
- [A.8 PHP](lang-php.md)
- [A.9 Ruby](lang-ruby.md)
- [A.10 Shell](lang-shell.md)
- [A.11 Swift and Objective-C](lang-swift-objc.md)
- [A.12 SQL and query dialects](lang-sql.md)

## Contract languages and chain runtimes

- [A.13 Solidity and Vyper (EVM)](lang-solidity.md)
- [A.14 Solana programs (Rust and Anchor)](lang-solana.md)
- [A.15 Move (Aptos, Sui)](lang-move.md)
- [A.16 Zero-knowledge circuits (Circom, halo2, Noir and others)](lang-zk.md)
- [A.17 CosmWasm and Cosmos SDK modules](lang-cosmwasm.md)
- [A.18 Cairo (Starknet)](lang-cairo.md)
- [A.19 TON (FunC, Tact, Tolk)](lang-ton.md)

## Additional language families

- [A.20 PowerShell](lang-powershell.md)
- [A.21 Lua and LuaJIT](lang-lua.md)
- [A.22 Dart and Flutter](lang-dart.md)
- [A.23 Erlang and Elixir (BEAM)](lang-beam.md)
- [A.24 Scala and Clojure](lang-scala-clojure.md)
- [A.25 Haskell and OCaml](lang-haskell-ocaml.md)
- [A.26 R and Julia](lang-r-julia.md)
- [A.27 Perl](lang-perl.md)
- [A.28 Zig](lang-zig.md)
- [A.29 Fortran and COBOL](lang-fortran-cobol.md)
- [A.30 Kotlin](lang-kotlin.md)

## Embedded languages and device programs

- [A.31 Configuration and serialization languages](lang-configuration.md)
- [A.32 Regular expressions](lang-regex.md)
- [A.33 GPU kernels and shading languages](lang-gpu.md)

## A.34 Other languages

For a language not in this appendix, build your own table with the steps below, and write down the sources in the coverage record:

1. For each category in [16](dimensions/16-language-and-runtime-pitfalls.md), find that language's matching mechanism (null values, copying and aliasing, integer semantics, exceptions, concurrency model, module loading...).
2. Use available official security guides, memory-model and concurrency notes, and the rule lists of mainstream static analysis tools. Consult only what the selected mechanisms need, within [the review boundaries](scope.md#general-review-discipline); missing documentation is recorded, not treated as permission to fetch it.
3. Cross-check the entries related to that language in general weakness catalogs (such as CWE).
4. Confirm the version the target actually uses, and keep only the entries that hold for that version.
5. For smart contract languages, also cross-check the chain's official security guide and public collections of vulnerable examples (such as Trail of Bits' not-so-smart-contracts). Focus on how the execution model differs from the EVM: whether messages are synchronous or asynchronous, how much is rolled back on failure, how accounts and storage are owned, and who pays.
6. Include embedded and declarative languages: regular expressions, templates, build expressions, policy languages, configuration DSLs and hardware description languages. Map their evaluation phase, name resolution, coercions and effects; a data-looking file may execute code. For RTL use [4.66](specialties/4.66-hardware-designs-and-hardware-security.md).
