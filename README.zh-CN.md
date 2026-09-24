# finecomb

[English](README.md)

finecomb 是一份穷尽式的代码审查与安全审计清单，打包成 [Agent Skill](https://agentskills.io)。它适用于任意语言的代码，也适用于多种语言混合的仓库。你把一个或多个目标（目录、包、仓库、文件、改动）交给智能体，技能告诉它：查什么、怎么查、什么算问题、怎么报告。

名字来自英语 "go over with a fine-tooth comb"（用细齿梳梳一遍）：一根一根地过，不漏掉任何一处。

## 覆盖什么

- **45 个通用维度**：死代码、重复、API 契约、错误处理、并发、资源、崩溃恢复、安全、隐私、性能、配置、供应链、测试、文档、长时运行等。
- **38 类按目标类型追加的专项**：网络、协议解析、加密、认证、数据库、文件系统、消息队列、调度、服务端中间件、服务端请求伪造、子进程、解释器与虚拟机、代理与隧道、规则引擎、系统调用、代码生成器、跨语言边界、持续集成与基础设施即代码、可发布的包、客户端应用、固件、数据管道、智能合约等。
- **语言陷阱表**：Go、Python、JavaScript/TypeScript、C/C++、Rust、Java/Kotlin、C#/.NET、PHP、Ruby、Shell、Swift/Objective-C、SQL，以及为其它语言自建陷阱表的方法。
- **五张逐对象追问清单**（公开入口、共享状态、不变量、外部副作用、后台执行流），以及威胁模型、证据等级、严重性、报告格式和各生态的工具选项。

审查默认只读。技能会先找目标项目自己的规范（例如 `AGENTS.md`、`CLAUDE.md`、`CONTRIBUTING.md`、`SECURITY.md`）并遵守；项目没有规定时，使用保守默认：不写源码树、不安装东西；联网只做只读查询，用来核对依赖版本有没有已知安全问题（只外发依赖名称和版本，不外发源码和秘密）。

## 仓库里的技能

| 技能 | 语言 | 路径 |
| --- | --- | --- |
| `finecomb` | 英文 | [skills/finecomb](skills/finecomb/SKILL.md) |
| `finecomb-zh` | 中文 | [skills/finecomb-zh](skills/finecomb-zh/SKILL.md) |

两个技能内容相同，只装一个：它们响应的是同一类请求。

## 安装

用 [skills 命令行](https://skills.sh)：

```sh
npx skills add lian-yue/finecomb --skill finecomb-zh
```

英文版把 `--skill` 换成 `finecomb`。加 `-g` 装到当前用户而不是当前项目，加 `-a <agent>` 指定智能体。从本地副本安装时，把 `lian-yue/finecomb` 换成副本路径。

也可以手工把 `skills/finecomb-zh` 复制到智能体的技能目录（例如 Claude Code 的 `.claude/skills/`）。

## 用法

直接用自然语言交代，例如：

- 「用 finecomb 审 `./server`。」
- 「用 finecomb 审 `./server`、`./client` 和这次改动。」
- 「用 finecomb 审计 `src/`，排除 `vendor/`、`third_party/` 和生成的代码。」
- 「用 finecomb 审这次改动。」
- 「用 finecomb 看看 `pkg/cache` 的并发。」

智能体会解析目标和排除项，建立事实基线（语言、入口、共享状态、威胁模型），执行适用的追问清单、维度、专项和语言表，然后写报告。每条问题都有位置、触发条件、证据、影响、建议和证据等级。被排除的代码只免于报告问题；调用链经过它时照样会读。

## 目录结构

```text
finecomb/
├── LICENSE
├── README.md
├── README.zh-CN.md
└── skills/
    ├── finecomb/            英文技能
    └── finecomb-zh/         中文技能
        ├── SKILL.md         工作流程、索引、收尾自检
        └── references/
            ├── scope.md         调用解析、范围、硬边界、分片
            ├── baseline.md      事实基线、威胁模型、历史机制对号
            ├── questions.md     五张逐对象追问清单
            ├── dimensions.md    45 个通用维度
            ├── specialties.md   38 类专项
            ├── report.md        证据等级、报告格式、修复纪律
            ├── tools.md         各生态的工具
            ├── languages.md     语言表的用法
            └── lang-*.md        每种语言一张表
```

`SKILL.md` 保持简短；智能体只在当前步骤需要时才读对应的参考文件。

## 维护

- 中文技能（`skills/finecomb-zh`）是源，英文技能是它的翻译。两边在同一次提交里一起改，标题、表格和行数保持一致。
- `SKILL.md` 控制在 500 行以内，细节放进 `references/`。
- 每个技能都可能被单独安装，所以相对链接和锚点必须在各自的技能目录内就能解析。

## 许可证

[Apache-2.0](LICENSE)
