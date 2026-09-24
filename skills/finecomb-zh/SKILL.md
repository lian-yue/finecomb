---
name: finecomb-zh
description: 穷尽式代码审查与安全审计清单（中文版）。不限语言，也适用于多种语言混合的仓库。用户要求审查、审计、代码评审、全面检查、安全审计、挑刺、找问题时使用；目标可以是一个或多个目录、包、模块、仓库、文件、改动或合并请求，可以带排除项（例如「排除 vendor 和生成的代码」）。覆盖 45 个通用维度、38 类按目标类型追加的专项、13 张语言陷阱表、五张逐对象追问清单、威胁模型、证据等级与报告格式。Keywords: code review, security audit, exhaustive checklist, any language.
license: Apache-2.0
metadata:
  author: lian-yue
  version: "1.0.0"
---

# finecomb：穷尽式代码审查清单

这份技能用于检查代码的正确性、安全性和可维护性。调用示例：「用 finecomb 审 `<目标>`」「用 finecomb 审 `<目录1>`、`<目录2>` 和这次改动」「用 finecomb 审计 `<目录>`，排除 `<a>`、`<b>` 和生成的代码」。目标可以是一个，也可以是多个：包、目录、模块、仓库、文件、改动或合并请求，可以混着给；代码可以是任意语言，也可以是多种语言混合。调用怎么解析成范围见[调用与范围解析](references/scope.md#调用与范围解析)。

**它不绑定具体项目，也不绑定具体语言。** 审查范围由调用时指定，清单用于追踪检查对象和证据，不能保证发现所有缺陷。检查项是追问方向；是否构成问题，要结合真实契约、可达路径和影响判断，不能因没有采用某项技术就直接报缺陷。

通用约定：

- [通用维度](references/dimensions.md)、[专项](references/specialties.md)、[报告与修复](references/report.md)里的检查项与语言无关。语言相关的部分集中在两处：[16 语言与运行时陷阱](references/dimensions.md#16-语言与运行时陷阱)列跨语言的陷阱类别，[附录 A](references/languages.md)把这些类别落到具体语言。目标用到几种语言，就按附录 A 选几张表；附录没有的语言按 16 的类别自行找对应机制。
- [七、机器可验证检查命令](references/tools.md)按「要检查什么」列出各生态的常见工具。项目已选定的工具优先；没有列出的生态选同目的的工具，**要检查的东西不变**。
- [四、按目标类型追加的专项](references/specialties.md)按目标**是什么类型的东西**来选，不看它叫什么名字、用什么语言、放在哪个目录。
- 凡是提到「项目规范」「说明文档」「流程文档」「测试矩阵」的地方，一律读作**目标项目里承担该职责的那份文件**，不假定它叫什么名字。常见位置和项目没有规定时的默认做法见[目标项目的硬边界](references/scope.md#目标项目的硬边界)。
- 适用性按实际行为判断：没有持久化行为可以排除落盘恢复；内部工具仍可能处理个人信息。报告状态统一见[五](references/report.md#五证据等级与报告格式)。

## 工作流程

按顺序做。每一步只读当步需要的参考文件，用不到的不读。

1. **解析调用，确定边界。** 读 [references/scope.md](references/scope.md)：把调用解析成目标、排除项和规模；找到目标项目的规范和硬边界，项目没有规定时用其中的保守默认；确认执行边界。目标太大时按其中的分片规则做。
2. **建立事实基线。** 读 [references/baseline.md](references/baseline.md)：语言与构建清单、对象清单、上下游、威胁模型；用「历史机制对号」表选出要追加的专项。
3. **选语言表。** 读 [references/languages.md](references/languages.md)，再读目标用到的每种语言的 `lang-*.md`。
4. **逐对象追问。** 读 [references/questions.md](references/questions.md)，对每个公开入口、共享状态、不变量、外部副作用、后台执行流过对应的追问清单。
5. **逐项判定通用维度。** 读 [references/dimensions.md](references/dimensions.md)，按规模判定 45 个维度的适用性并检查。
6. **追加专项。** 读 [references/specialties.md](references/specialties.md) 里第 2 步选中的专项。
7. **按需执行验证。** 需要复现或机器检查时，从 [references/tools.md](references/tools.md) 选工具，遵守执行边界。
8. **写报告。** 按 [references/report.md](references/report.md) 的证据等级、严重性、问题字段和整体结构写。
9. **修复（仅在已获授权时）。** 按 [references/report.md](references/report.md#六修复纪律) 的修复纪律。
10. **收尾自检。** 按本文件末尾的[八、收尾自检](#八收尾自检)。

## 索引

- [零、开工前](references/scope.md)（含[调用与范围解析](references/scope.md#调用与范围解析)、[大目标分片与多轮审查](references/scope.md#大目标分片与多轮审查)）
- [一、建立事实基线](references/baseline.md)（含[威胁模型与攻击链](references/baseline.md#威胁模型与攻击链)、[历史机制对号](references/baseline.md#历史机制对号)）
- [二、五张追问清单（找真问题的主力）](references/questions.md)
- [三、通用维度检查表](references/dimensions.md)
  - 代码形态：[1 死代码与可达性](references/dimensions.md#1-死代码与可达性)｜[2 遗留标记与被抑制项](references/dimensions.md#2-遗留标记与被抑制项)｜[3 重复](references/dimensions.md#3-重复)｜[4 抽象与分层](references/dimensions.md#4-抽象与分层)｜[5 关联与影响面](references/dimensions.md#5-关联与影响面)｜[6 命名、注释与可读性](references/dimensions.md#6-命名注释与可读性)｜[7 复杂度与可维护性](references/dimensions.md#7-复杂度与可维护性)
  - 功能与契约：[8 功能正确性与需求符合度](references/dimensions.md#8-功能正确性与需求符合度)｜[9 业务逻辑与流程完整性](references/dimensions.md#9-业务逻辑与流程完整性)｜[10 API 与契约](references/dimensions.md#10-api-与契约)｜[11 易用性与误用防护](references/dimensions.md#11-易用性与误用防护)｜[12 状态机与状态转移](references/dimensions.md#12-状态机与状态转移)
  - 行为细节：[13 错误处理与失败语义](references/dimensions.md#13-错误处理与失败语义)｜[14 边界、数值与文本](references/dimensions.md#14-边界数值与文本)｜[15 算法与数据结构正确性](references/dimensions.md#15-算法与数据结构正确性)｜[16 语言与运行时陷阱](references/dimensions.md#16-语言与运行时陷阱)｜[17 时间与时钟](references/dimensions.md#17-时间与时钟)
  - 运行时正确性：[18 并发与内存模型](references/dimensions.md#18-并发与内存模型)｜[19 生命周期与资源](references/dimensions.md#19-生命周期与资源)｜[20 资源上界与背压](references/dimensions.md#20-资源上界与背压)｜[21 可伸缩性与容量](references/dimensions.md#21-可伸缩性与容量)｜[22 故障隔离与降级](references/dimensions.md#22-故障隔离与降级)
  - 数据与持久化：[23 事务、原子性与一致性](references/dimensions.md#23-事务原子性与一致性)｜[24 编解码与持久化格式](references/dimensions.md#24-编解码与持久化格式)｜[25 崩溃与恢复](references/dimensions.md#25-崩溃与恢复)｜[26 发布、升级、迁移与回滚](references/dimensions.md#26-发布升级迁移与回滚)
  - 安全：[27 安全与信任边界](references/dimensions.md#27-安全与信任边界)｜[28 授权与访问控制](references/dimensions.md#28-授权与访问控制)｜[29 失效安全与危险操作](references/dimensions.md#29-失效安全与危险操作)｜[30 隐私、数据治理与合规](references/dimensions.md#30-隐私数据治理与合规)
  - 性能：[31 性能、内存与延迟](references/dimensions.md#31-性能内存与延迟)
  - 可运维：[32 配置与默认值](references/dimensions.md#32-配置与默认值)｜[33 可观测性与可诊断性](references/dimensions.md#33-可观测性与可诊断性)｜[34 运行环境与部署契约](references/dimensions.md#34-运行环境与部署契约)
  - 工程：[35 依赖](references/dimensions.md#35-依赖)｜[36 供应链与制品完整性](references/dimensions.md#36-供应链与制品完整性)｜[37 互操作性与共存](references/dimensions.md#37-互操作性与共存)｜[38 可移植性与构建上下文](references/dimensions.md#38-可移植性与构建上下文)｜[39 生成产物与工具链](references/dimensions.md#39-生成产物与工具链)
  - 验证：[40 可测试性与故障注入](references/dimensions.md#40-可测试性与故障注入)｜[41 测试质量](references/dimensions.md#41-测试质量)｜[42 覆盖率](references/dimensions.md#42-覆盖率)
  - 文档与规则：[43 文档一致性](references/dimensions.md#43-文档一致性)｜[44 项目规则合规](references/dimensions.md#44-项目规则合规)
  - 长期运行：[45 常驻与长时运行](references/dimensions.md#45-常驻与长时运行)
- [四、按目标类型追加的专项](references/specialties.md)
  - 通信：[4.1 网络与连接](references/specialties.md#41-网络与连接)｜[4.2 协议与帧解析](references/specialties.md#42-协议与帧解析)｜[4.3 解码不受信任的外部数据](references/specialties.md#43-解码不受信任的外部数据)｜[4.21 出站请求与服务端请求伪造](references/specialties.md#421-出站请求与服务端请求伪造)｜[4.28 隧道、代理与网络数据面](references/specialties.md#428-隧道代理与网络数据面)
  - 身份：[4.4 加密与凭据](references/specialties.md#44-加密与凭据)｜[4.5 认证、会话与令牌](references/specialties.md#45-认证会话与令牌)
  - 数据：[4.6 缓存与存储](references/specialties.md#46-缓存与存储)｜[4.7 数据库与查询](references/specialties.md#47-数据库与查询)｜[4.8 文件系统与路径](references/specialties.md#48-文件系统与路径)｜[4.37 数据处理、批处理与机器学习](references/specialties.md#437-数据处理批处理与机器学习)
  - 异步：[4.9 事件、发布订阅与观察者](references/specialties.md#49-事件发布订阅与观察者)｜[4.10 消息队列与异步任务](references/specialties.md#410-消息队列与异步任务)｜[4.11 定时与调度](references/specialties.md#411-定时与调度)｜[4.24 Webhook 与外部事件](references/specialties.md#424-webhook-与外部事件)
  - 进程：[4.12 依赖装配与服务生命周期](references/specialties.md#412-依赖装配与服务生命周期)｜[4.13 日志、指标与追踪](references/specialties.md#413-日志指标与追踪)｜[4.14 服务端请求处理与中间件](references/specialties.md#414-服务端请求处理与中间件)｜[4.15 命令行与进程入口](references/specialties.md#415-命令行与进程入口)｜[4.22 子进程、动态执行与解码器副作用](references/specialties.md#422-子进程动态执行与解码器副作用)
  - 表达：[4.16 数据模型与生成契约](references/specialties.md#416-数据模型与生成契约)｜[4.17 模板与文本输出](references/specialties.md#417-模板与文本输出)｜[4.18 用户界面与可访问性](references/specialties.md#418-用户界面与可访问性)｜[4.19 国际化与本地化](references/specialties.md#419-国际化与本地化)
  - 基础件：[4.20 并发原语与通用容器](references/specialties.md#420-并发原语与通用容器)｜[4.25 分布式协调与租约](references/specialties.md#425-分布式协调与租约)
  - 安全记录：[4.23 安全审计日志](references/specialties.md#423-安全审计日志)
  - 模型应用：[4.26 大模型与工具调用](references/specialties.md#426-大模型与工具调用)
  - 语言与规则：[4.27 解释器、编译器与虚拟机](references/specialties.md#427-解释器编译器与虚拟机)｜[4.29 规则与策略匹配](references/specialties.md#429-规则与策略匹配)｜[4.31 代码生成器与编译期工具](references/specialties.md#431-代码生成器与编译期工具)
  - 系统与平台：[4.30 操作系统接口、系统调用与描述符](references/specialties.md#430-操作系统接口系统调用与描述符)｜[4.32 跨语言边界与本地扩展](references/specialties.md#432-跨语言边界与本地扩展)｜[4.35 客户端应用、扩展与自动更新](references/specialties.md#435-客户端应用扩展与自动更新)｜[4.36 嵌入式、固件与实时约束](references/specialties.md#436-嵌入式固件与实时约束)
  - 交付：[4.33 构建脚本、持续集成与基础设施即代码](references/specialties.md#433-构建脚本持续集成与基础设施即代码)｜[4.34 可发布的库、SDK 与包](references/specialties.md#434-可发布的库sdk-与包)
  - 链上：[4.38 智能合约与链上交互](references/specialties.md#438-智能合约与链上交互)
- [五、证据等级与报告格式](references/report.md#五证据等级与报告格式)
- [六、修复纪律](references/report.md#六修复纪律)
- [七、机器可验证检查命令](references/tools.md)
- [八、收尾自检](#八收尾自检)
- [附录 A：各语言运行时陷阱](references/languages.md)：[Go](references/lang-go.md)｜[Python](references/lang-python.md)｜[JavaScript 与 TypeScript](references/lang-javascript.md)｜[C 与 C++](references/lang-c-cpp.md)｜[Rust](references/lang-rust.md)｜[Java 与 Kotlin](references/lang-jvm.md)｜[C# 与 .NET](references/lang-dotnet.md)｜[PHP](references/lang-php.md)｜[Ruby](references/lang-ruby.md)｜[Shell](references/lang-shell.md)｜[Swift 与 Objective-C](references/lang-swift-objc.md)｜[SQL](references/lang-sql.md)｜[其它语言](references/languages.md#a13-其它语言)

## 八、收尾自检

收尾只核对记录是否完整，不触发新一轮检查、全套测试或额外专项。

- [ ] [事实基线](references/baseline.md)与选定范围完整，安全相关目标已有威胁模型和明确假设。
- [ ] 目标、排除项、类别识别结果和疑似项已按[调用与范围解析](references/scope.md#调用与范围解析)列出；被排除代码只用于追踪，没有漏掉经过它的调用链。
- [ ] 目标用到的每种语言都已选用[附录 A](references/languages.md) 的对应表（或写明按 16 自行映射的依据）；跨语言边界已按[4.32](references/specialties.md#432-跨语言边界与本地扩展)检查。
- [ ] 分片审查已做接缝核对；沿用上一轮结论的单元已核对指纹未变。
- [ ] [逐对象追问](references/questions.md)、通用维度与所选专项已按[报告状态](references/report.md#五证据等级与报告格式)记录，未完成部分没有冒充已查。
- [ ] 问题已去重，触发条件、阻断点、影响与建议有证据；推断、待验证项和一般建议与已确认问题分开。
- [ ] 已授权修复按[六](references/report.md#六修复纪律)完成或明确留下边界，验证结果与问题状态一致。
- [ ] 命令、缓存复用、失败与未运行状态已记录，未把工具通过写成完整安全证明。
- [ ] 全程遵守[目标项目的硬边界](references/scope.md#目标项目的硬边界)，报告已脱敏，范围外问题和未采取动作的原因已说明。
