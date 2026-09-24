# 附录 A：各语言运行时陷阱

本附录把[16](dimensions.md#16-语言与运行时陷阱)的类别落到具体语言，也收录各语言在其它维度上的高频陷阱。用法：

- 目标用到哪几种语言，就选哪几张表；混合语言的目标每种都选，跨语言边界另看[4.32](specialties.md#432-跨语言边界与本地扩展)。
- 表里只列高频、容易漏的项，**不代替 16 的类别检查**：16 的每一类仍要逐项找该语言的对应机制。
- 语义随版本变化。先确认目标声明并实际使用的语言、运行时和编译器版本，再判断某一条是否适用。
- 附录没有的语言，按[A.20 其它语言](#a20-其它语言)的方法自己建表。

## 语言索引

- [A.1 Go](lang-go.md)
- [A.2 Python](lang-python.md)
- [A.3 JavaScript 与 TypeScript](lang-javascript.md)
- [A.4 C 与 C++](lang-c-cpp.md)
- [A.5 Rust](lang-rust.md)
- [A.6 Java 与 Kotlin（JVM）](lang-jvm.md)
- [A.7 C# 与 .NET](lang-dotnet.md)
- [A.8 PHP](lang-php.md)
- [A.9 Ruby](lang-ruby.md)
- [A.10 Shell](lang-shell.md)
- [A.11 Swift 与 Objective-C](lang-swift-objc.md)
- [A.12 SQL 与查询方言](lang-sql.md)
- [A.13 Solidity 与 Vyper（EVM）](lang-solidity.md)
- [A.14 Solana 程序（Rust 与 Anchor）](lang-solana.md)
- [A.15 Move（Aptos、Sui）](lang-move.md)
- [A.16 零知识电路（Circom、halo2、Noir 等）](lang-zk.md)
- [A.17 CosmWasm 与 Cosmos SDK 模块](lang-cosmwasm.md)
- [A.18 Cairo（Starknet）](lang-cairo.md)
- [A.19 TON（FunC、Tact、Tolk）](lang-ton.md)

## A.20 其它语言

附录没有的语言，按下面的步骤自己建表，并在覆盖记录里写明来源：

1. 按[16](dimensions.md#16-语言与运行时陷阱)的每一类，找该语言的对应机制（空值、复制与别名、整数语义、异常、并发模型、模块加载……）。
2. 读该语言官方文档里的安全指南、内存模型与并发说明，以及主流静态检查工具的规则列表——规则列表通常就是该语言高频陷阱的清单。
3. 对照通用缺陷目录（如 CWE）里与该语言相关的条目。
4. 确认目标实际使用的版本，只保留在该版本下成立的条目。
5. 智能合约语言再对照该链官方的安全指南和公开的漏洞样例集（如 Trail of Bits 的 not-so-smart-contracts），重点看执行模型与 EVM 的差异：消息是同步还是异步、失败时回滚的范围、账户与存储怎么归属、谁付费。
