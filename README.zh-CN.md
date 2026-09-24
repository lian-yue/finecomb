# finecomb

<a href="https://skills.sh/lian-yue/finecomb"><img alt="skills.sh" src="https://skills.sh/b/lian-yue/finecomb?style=for-the-badge" height="28"></a>

[English](README.md)

finecomb 是一份穷尽式的代码审查与安全审计清单，按开放的 [Agent Skills](https://agentskills.io) 格式编写，不绑定某一种智能体或某一种安装方式：

- 用 [skills.sh](https://skills.sh) 的 skills 命令行一条命令安装；
- 在 Claude Code 里当插件市场添加；
- 对智能体说一句「帮我安装 https://github.com/lian-yue/finecomb」，让它自己装；
- 或者手工复制技能目录。

Claude Code、Codex、Cursor、Gemini CLI、GitHub Copilot、OpenCode 等支持技能的智能体都能用，安装方法见[安装](#安装)。它适用于任意语言的代码，也适用于多种语言混合的仓库。你把一个或多个目标（目录、包、仓库、文件、改动）交给智能体，技能告诉它：查什么、怎么查、什么算问题、怎么报告。

名字来自英语 "go over with a fine-tooth comb"（用细齿梳梳一遍）：一根一根地过，不漏掉任何一处。

## 覆盖什么

- **根因面**：从真实漏洞根因归纳出的跨领域追问，每个对象都要逐面过一遍，例如验证的完备性、声明的控制是否真的生效、校验之后的转换、先算大小与实际写入的两趟是否一致、关联字段是否一起更新、同一对象会不会被结束两次、旧的合法版本会不会被重新接受。这些面用 nginx、Linux 内核、OpenSSL、OpenSSH、glibc、主流框架与企业设备、链上与零知识证明的真实漏洞做过命中测试，没命中的根因都已补成面。
- **45 个通用维度**：死代码、重复、API 契约、错误处理、并发、资源、崩溃恢复、安全、隐私、性能、配置、供应链、测试、文档、长时运行等。
- **50 类按目标类型追加的专项**：网络、协议解析、加密、认证、单点登录与联合身份、数据库、文件系统、消息队列、调度、服务端中间件、服务端请求伪造、子进程、解释器与虚拟机、代理与隧道、规则引擎、系统调用、本机特权组件、内核驱动与虚拟化、移动应用组件、代码生成器、跨语言边界、持续集成与基础设施即代码、可发布的包、客户端应用、固件、数据管道、大模型与智能体、支付与账务、通知与外发消息、智能合约、DeFi 与预言机、跨链桥、钱包与签名、账户抽象、零知识证明、区块链节点与共识等。
- **语言陷阱表**：Go、Python、JavaScript/TypeScript、C/C++、Rust、Java/Kotlin、C#/.NET、PHP、Ruby、Shell、Swift/Objective-C、SQL，合约语言 Solidity/Vyper、Solana、Move、CosmWasm、Cairo、TON 与零知识电路，以及为其它语言自建陷阱表的方法。
- **历史漏洞模式**：从 CWE Top 25、CISA 高频被利用漏洞和重大安全事件（Log4Shell、Heartbleed、xz 后门、MOVEit、Citrix Bleed、跨链桥与 DeFi 被盗事件等）归纳出的可泛化机制，每条带审查时要找什么。
- **非源码目标**：审计对象是二进制、安装包、浏览器扩展、固件、容器镜像、已发布的包、线上地址、域名与邮件配置、主机、Kubernetes 集群、云账号、SaaS 租户、数据库、日志与抓包、链上合约地址、设计文档、智能体配置或依赖清单时，能拿到什么证据、先查什么、哪些看不到、哪些需要授权。
- **五张逐对象追问清单**（公开入口、共享状态、不变量、外部副作用、后台执行流），以及威胁模型、证据等级、严重性、报告格式和各生态的工具选项。

审查默认只读。技能会先找目标项目自己的规范（例如 `AGENTS.md`、`CLAUDE.md`、`CONTRIBUTING.md`、`SECURITY.md`）并遵守；项目没有规定时，使用保守默认：不写源码树、不安装东西；联网只做只读查询，用来核对依赖版本有没有已知安全问题（只外发依赖名称和版本，不外发源码和秘密）。目标是线上服务、账号、租户或链上合约时，默认只做被动、只读的访问；主动扫描、登录尝试和漏洞验证要有资产所有者的书面授权，结束时清理测试产物、处置取得的数据；链上不发交易、不签名。某一项检查因授权、环境、工具或审查者自身的规则做不了时，只跳过这一项并在报告开头写明原因和还缺什么，其余照常做完，不整体退出，也不悄悄降级。

## 如何验证覆盖面

根因面和检查点是用真实漏洞做「命中测试」补出来的，不是凭经验列的：

1. 从补丁或官方公告查到漏洞的真实根因。
2. 只看技能里「要问的问题」和「什么算问题」两列，不看案例列，判断审查者照着追问能否问到这个根因。
3. 问不到的，归纳成与领域、语言无关的追问补进技能，再用没参与编写的新样本复测。

两轮留出集（共 111 个没参与编写技能的漏洞，每个样本只写补丁或公告层面的根因）：

| 样本 | 命中 | 部分 | 未命中 |
| --- | --- | --- | --- |
| Linux 内核，2024–2026 年（15 个） | 14 | 1 | 0 |
| nginx，2009–2021 年（8 个） | 8 | 0 | 0 |
| Apache httpd、HAProxy、Envoy、OpenSSL，2024–2025 年（8 个） | 8 | 0 | 0 |
| Web 框架与业务应用（10 个） | 9 | 1 | 0 |
| 内存安全语言的库：Go、Rust、Java、npm、Python、.NET（10 个） | 8 | 2 | 0 |
| 浏览器、脚本引擎与桌面客户端（10 个） | 8 | 2 | 0 |
| 开源云原生组件与构建工具（10 个） | 6 | 4 | 0 |
| 密码学库、认证协议与身份系统（10 个） | 6 | 4 | 0 |
| 智能合约、DeFi、跨链与零知识证明（10 个） | 7 | 3 | 0 |
| AI 智能体与大模型应用（10 个） | 9 | 1 | 0 |
| 开源数据库、存储与消息系统（10 个） | 6 | 4 | 0 |

没有一个样本需要新增根因面；所有「部分」缺的都是已有面下面的具体检查点，或者某个通用问题只写在了某个领域的专项里。这些都已补进技能。命中测试检验的是「追问能否把人引到根因」，不等于照着清单一定能在代码里找到它。

## 仓库里的技能

| 技能 | 语言 | 路径 |
| --- | --- | --- |
| `finecomb` | 英文 | [skills/finecomb](skills/finecomb/SKILL.md) |
| `finecomb-zh` | 中文 | [skills/finecomb-zh](skills/finecomb-zh/SKILL.md) |

两个技能内容相同，只装一个：它们响应的是同一类请求。

## 安装

### 让智能体安装

对你的编码智能体（Claude Code、Codex、Cursor、Gemini CLI、GitHub Copilot 等）说一句就行：

```text
帮我安装 https://github.com/lian-yue/finecomb
```

智能体读到这里时，按下面的步骤安装：

1. **选版本。** 用户用中文交流就装 `finecomb-zh`，否则装 `finecomb`；用户点名的以用户为准。只装一个，两个版本响应的是同一类请求。
2. **选范围。** 用户没说时装到当前用户，所有项目都能用；用户说只装到当前项目时，装到项目里。
3. **本机有 Node.js 时用 skills 命令行。** `-a` 填你自己对应的智能体标识（如 `claude-code`、`codex`、`cursor`、`gemini-cli`、`github-copilot`、`opencode`）；装到项目里时去掉 `-g`：

   ```sh
   npx skills add https://github.com/lian-yue/finecomb --skill finecomb-zh -a claude-code -g -y
   ```

4. **没有 Node.js 时手工安装。** 把仓库浅克隆到临时目录，把 `skills/finecomb-zh`（或 `skills/finecomb`）整个目录复制到你的技能目录，例如 Claude Code 的 `~/.claude/skills/finecomb-zh`，其它智能体的目录见下方表格；复制完删掉临时目录。
5. **核对并告知。** 确认目标目录里有 `SKILL.md`，告诉用户装在哪里、要不要重启智能体，以及怎么用，例如「用 finecomb 审 `./src`」「帮我审计 `https://github.com/<所有者>/<仓库>`」。

技能只有 Markdown 文件，不带脚本，安装时不会执行其中任何代码。

### skills 命令行

用 [skills 命令行](https://github.com/vercel-labs/skills)（[skills.sh](https://skills.sh)）安装中文版：

```sh
npx skills add lian-yue/finecomb --skill finecomb-zh
```

英文版把 `--skill` 换成 `finecomb`。常用选项：

| 选项 | 作用 |
| --- | --- |
| `-l`、`--list` | 只列出仓库里的技能，不安装 |
| `-g`、`--global` | 装到当前用户，而不是当前项目 |
| `-a`、`--agent <智能体>` | 指定智能体，例如 `claude-code`、`codex`、`cursor`、`gemini-cli`、`github-copilot`、`opencode` |
| `--copy` | 复制文件，而不是软链接到智能体目录 |
| `-y`、`--yes` | 跳过确认 |

安装源也可以写成完整地址 `https://github.com/lian-yue/finecomb`，只装一个技能时可以直接指向它的目录 `https://github.com/lian-yue/finecomb/tree/main/skills/finecomb-zh`，从本地副本安装时写副本路径。

装好之后：

```sh
npx skills list
```

```sh
npx skills update finecomb-zh
```

```sh
npx skills remove finecomb-zh
```

不安装、只用一次（把技能生成为提示词交给智能体）：

```sh
npx skills use lian-yue/finecomb --skill finecomb-zh --agent claude-code
```

命令行按智能体把技能放到它的技能目录，例如：

| 智能体 | 项目内 | 当前用户 |
| --- | --- | --- |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `.agents/skills/` | `~/.codex/skills/` |
| Cursor | `.agents/skills/` | `~/.cursor/skills/` |
| Gemini CLI | `.agents/skills/` | `~/.gemini/skills/` |
| GitHub Copilot | `.agents/skills/` | `~/.copilot/skills/` |

完整的智能体列表见 skills 命令行的说明。

### Claude Code 插件市场

仓库带有 `.claude-plugin/marketplace.json`，可以在 Claude Code 里当插件市场添加：

```text
/plugin marketplace add lian-yue/finecomb
/plugin install finecomb-zh@finecomb
```

英文版装 `finecomb@finecomb`。

### 手工安装

把 `skills/finecomb-zh` 整个目录复制到智能体的技能目录（例如 Claude Code 的 `.claude/skills/`）。

## 用法

直接用自然语言交代，例如：

- 「用 finecomb 审 `./server`。」
- 「用 finecomb 审 `./server`、`./client` 和这次改动。」
- 「用 finecomb 审计 `src/`，排除 `vendor/`、`third_party/` 和生成的代码。」
- 「用 finecomb 审这次改动。」
- 「用 finecomb 看看 `pkg/cache` 的并发。」
- 「用 finecomb 审计以太坊主网上的合约 `0x…`。」
- 「帮我审计 `https://github.com/<所有者>/<仓库>`。」（GitHub、GitLab 等代码仓库的网址都行，可以指向分支、标签、子目录或合并请求；智能体只读地克隆到临时目录再审，报告里写明审查的提交哈希。）

智能体会解析目标和排除项，建立事实基线（语言、入口、共享状态、威胁模型），执行适用的追问清单、根因面、维度、专项和语言表，然后写报告。每条问题都有位置、触发条件、证据、影响、建议和证据等级。被排除的代码只免于报告问题；调用链经过它时照样会读。

## 目录结构

```text
finecomb/
├── .claude-plugin/
│   └── marketplace.json     Claude Code 插件市场清单
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
            ├── facets.md        根因面
            ├── dimensions.md    45 个通用维度
            ├── specialties.md   50 类专项
            ├── history.md       历史漏洞模式
            ├── targets.md       非源码目标
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
- 技能目录按 skills 命令行的发现规则放在 `skills/<名字>/SKILL.md`，`name` 与目录名一致；新增或改名技能时同步改 `.claude-plugin/marketplace.json`。

## 许可证

[Apache-2.0](LICENSE)

## 致谢

感谢 Anthropic 的 [Claude for Open Source](https://claude.com/contact-sales/claude-for-oss) 计划为开源维护者免费提供 Claude Max。finecomb 的编写、命中测试和中英文翻译都是用它完成的。
