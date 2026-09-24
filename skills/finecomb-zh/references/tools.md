# 七、机器可验证检查命令

本节是工具选项表，不是必跑清单。执行条件统一见[审查执行边界](scope.md#审查执行边界)；选择最小入口、复用缓存和处理失败按目标项目的测试策略。

按「要检查什么」分行，每行给出各生态的常见工具。使用前先做三件事：

- **替换并核对占位符。** `<目标目录>`、`<文件>` 是本次范围内的路径；`<pkg>`、`<模块>`、`<crate>` 是明确的包或模块；`<tmp>` 是本次独占的临时目录；用例名、预算、平台和参数都要来自实际目标。命令在所属模块或项目根运行，模块与工作区选项沿用最近的测试说明。
- **确认工具已在本机，不自动安装。** `npx`、`pipx run`、`go run <模块>@<版本>` 这类命令在工具缺失时会联网下载，先按授权处理。
- **项目已选定的工具优先。** 列出的只是例子；没有列出的生态，选同目的的工具。

## 只读定位

| 目的 | 示例 | 结果边界 |
| --- | --- | --- |
| 文件清单与残留 | `rg --files --hidden -g '!.git/**' -g '!**/.git/**' '<目标目录>'` | 只清点已指定目录；忽略规则可能排除文件，不能据空输出断言目录没有内容 |
| 语言与规模 | `tokei '<目标目录>'` 或 `scc '<目标目录>'`；没有这类工具时按上一行结果的扩展名统计 | 只按扩展名和文件头判断，认不出嵌在字符串里的语言；结果用于分片和选[附录 A](languages.md) |
| 生成代码识别 | `rg -l -F -e 'DO NOT EDIT' -e '@generated' -e 'auto-generated' -e 'automatically generated' -- '<目标目录>'`；再读 `.gitattributes` 的 `linguist-generated`、`linguist-vendored` | 命中只是证据之一，按[调用与范围解析](scope.md#调用与范围解析)列出供核对 |
| 符号与文档引用 | `rg -n -F -- '<符号>' '<目标目录>'` | 搜索命中是定位线索，动态调用和外部调用方仍需人工核对 |
| 遗留标记 | 见下方代码块 | 按[2](dimensions.md#2-遗留标记与被抑制项)判定，不把命中数当问题数 |
| 格式检查 | Go `gofmt -l '<文件>'`；Python `ruff format --check '<文件>'` 或 `black --check '<文件>'`；JavaScript/TypeScript `prettier --check '<文件>'`；Rust `cargo fmt --check`；C/C++ `clang-format --dry-run -Werror '<文件>'`；Shell `shfmt -d '<文件>'` | 仅在已选格式检查适用时列出差异，不自动格式化 |
| 文档与入口映射 | 项目已有工具的明确目录、包和文档参数 | 没有工具时人工核对；映射通过不证明分支与契约描述正确 |
| 文档链接 | `lychee --offline '<目标目录>'` 或项目已有的链接检查 | 离线模式只查本地文件和锚点；检查外部地址要联网，先按授权处理 |

遗留标记与抑制标记的搜索示例（固定字符串，多个 `-e` 取并集；按目标语言增删）：

```sh
rg -n -F \
  -e TODO -e FIXME -e XXX -e HACK \
  -e 'nolint' -e 'lint:ignore' -e 'nosec' \
  -e 'noqa' -e 'type: ignore' -e 'pylint: disable' -e 'pragma: no cover' \
  -e 'eslint-disable' -e '@ts-ignore' -e '@ts-expect-error' -e '@ts-nocheck' -e 'istanbul ignore' -e 'c8 ignore' \
  -e 'NOLINT' -e 'pragma GCC diagnostic' -e 'cppcheck-suppress' \
  -e '#[allow(' -e '#[expect(' \
  -e '@SuppressWarnings' -e '@Suppress(' -e 'pragma warning disable' \
  -e '@phpstan-ignore' -e '@psalm-suppress' -e 'phpcs:ignore' \
  -e 'rubocop:disable' -e ':nocov:' \
  -e 'shellcheck disable' -e 'swiftlint:disable' \
  -e 'NOSONAR' -e 'nosemgrep' -e 'gitleaks:allow' \
  -- '<目标目录>'
```

## 按需执行的验证

| 目的 | 示例 | 选择与解释 |
| --- | --- | --- |
| 普通正确性 | Go `go test './<pkg>' -run '^TestName$'`；Python `pytest '<文件>::<用例>'`；JavaScript/TypeScript `npx vitest run '<文件>' -t '<用例名>'` 或 `npx jest '<文件>' -t '<用例名>'`；Rust `cargo test -p '<crate>' '<用例>' -- --exact`；JVM `mvn -Dtest='<类>#<方法>' test` 或 `gradle test --tests '<类>.<方法>'`；.NET `dotnet test --filter 'FullyQualifiedName=<全名>'`；C/C++ `ctest -R '^<用例>$'` | 使用源码中确认的用例名；多用例用有限、精确的选择，默认保留工具有效缓存 |
| 静态分析 | Go `go vet './<pkg>'` 或 `staticcheck './<pkg>'`；Python `ruff check`、`pylint`；JavaScript/TypeScript `eslint`；C/C++ 编译器 `-Wall -Wextra`、`clang-tidy`、`cppcheck`、`scan-build`；Rust `cargo clippy`；JVM SpotBugs、Error Prone、PMD，Kotlin `detekt`；.NET 构建分析器；PHP `phpstan`、`psalm`；Ruby `rubocop`；Shell `shellcheck`；跨语言 `semgrep` | 按需要选择工具，确认平台、构建条件、生成代码与规则范围；未报告不等于无缺陷 |
| 类型检查 | Python `mypy`、`pyright`；TypeScript `tsc --noEmit`；PHP `phpstan`、`psalm` | 只覆盖有类型信息的部分；外部输入的运行时校验仍要人工核对 |
| 安全规则扫描 | 跨语言 `semgrep`；Python `bandit`；Ruby `brakeman`；Go `gosec` | 规则命中是线索，按[27](dimensions.md#27-安全与信任边界)和专项研判；联网拉取规则集先按授权处理 |
| 死代码线索 | Go `deadcode`、`staticcheck` 的未使用检查；Python `vulture`；JavaScript/TypeScript `knip`；Rust 编译器的未使用警告；依赖层面 Rust `cargo udeps` | 先确认该版本能报告什么，判据见[1](dimensions.md#1-死代码与可达性) |
| 重复与复杂度 | 多语言 `jscpd`（重复）、`lizard`（复杂度）；Go `gocyclo` | 度量只是线索，判据见[3](dimensions.md#3-重复)、[7](dimensions.md#7-复杂度与可维护性) |
| 依赖图与环 | Go `go list -deps`、`go mod why`、`go mod graph`；JavaScript/TypeScript `madge --circular`；Python `pipdeptree`；Rust `cargo tree`；JVM `mvn dependency:tree`、`gradle dependencies` | 用于[5](dimensions.md#5-关联与影响面)的依赖方向和[35](dimensions.md#35-依赖)的必要性判断 |
| 依赖漏洞 | Go `govulncheck './<pkg>'`；Python `pip-audit`；JavaScript `npm audit`、`pnpm audit`；Rust `cargo audit`、`cargo deny`；JVM OWASP Dependency-Check；.NET `dotnet list package --vulnerable`；PHP `composer audit`；Ruby `bundler-audit`；跨生态 `osv-scanner`、`trivy`、`grype` | 使用项目已选且已安装的工具；更新工具或漏洞库可能联网，先按授权处理；结果按[36](dimensions.md#36-供应链与制品完整性)研判 |
| 依赖完整性 | Go `go mod verify`；Rust `cargo build --locked`；Python 安装时 `--require-hashes`；npm 在隔离副本中 `npm ci --ignore-scripts` | 只说明本地依赖与锁文件一致；会下载或写依赖目录的命令只在隔离副本里执行 |
| 独立构建 | Go `GOWORK=off go build -o '<tmp>/' './...'`；其它生态在隔离副本中去掉工作区链接和本地覆盖后构建 | 判据见[38](dimensions.md#38-可移植性与构建上下文)；不改源码树 |
| 平台构建 | Go `GOOS='<系统>' GOARCH='<架构>' go build -o '<tmp>/artifact' './<pkg>'`；Rust `cargo build --target '<目标三元组>'`；C/C++ 交叉工具链。32 位目标可用来检查字长与原子对齐 | 只验证指定平台的构建，不代表在该平台运行通过 |
| API 与 ABI 差异 | Go `apidiff`、`gorelease`；Rust `cargo semver-checks`；JVM `japicmp`；C/C++ libabigail 的 `abidiff`；TypeScript `api-extractor` | 只比较两个已确认的版本；按[4.34](specialties.md#434-可发布的库sdk-与包)解读 |
| 覆盖率 | Go `go test './<pkg>' -run '^TestName$' -coverprofile='<tmp>/cov.out'`，用 `go tool cover -func='<tmp>/cov.out'` 读取；Python `coverage run -m pytest '<文件>::<用例>'`；JavaScript/TypeScript `c8`、`vitest --coverage`、`jest --coverage`；Rust `cargo llvm-cov`；JVM JaCoCo；.NET coverlet；C/C++ `gcov`、`llvm-cov` | 仅在该测量已获授权时采集，按[42](dimensions.md#42-覆盖率)解释 |
| 并发检测 | Go `go test './<pkg>' -race -run '^TestName$'`；C/C++/Rust ThreadSanitizer（`-fsanitize=thread`）；Rust `loom`（需要测试代码配合）；JVM `jcstress` | 仅选择需要验证的并发行为，结论边界见[18](dimensions.md#18-并发与内存模型) |
| 内存错误检测 | C/C++ `-fsanitize=address,undefined`、`valgrind`；Rust `cargo miri test`；Go 带 C 代码时 `go test -asan` | 仅选择需要验证的路径；未报告不代表没有内存错误 |
| 模糊测试 | Go `go test './<pkg>' -run '^$' -fuzz '^FuzzName$' -fuzztime '<预算>'`；Python `atheris`；JavaScript `@jazzer.js/core`；C/C++ libFuzzer、AFL++；Rust `cargo fuzz`；JVM Jazzer；.NET SharpFuzz | 仅在已获准的隔离副本中执行；失败样本可能写入测试数据目录，须确认归属和资源预算，不能因未崩溃就证明安全 |
| 性质测试 | Python `hypothesis`；JavaScript/TypeScript `fast-check`；Rust `proptest`；JVM `jqwik`；Go `testing/quick` | 属于写测试，是否补按项目测试策略 |
| 不稳定问题诊断 | Go `go test './<pkg>' -run '^TestName$' -count '<次数>'`；其它框架用各自的重复运行选项或插件 | 先有失败分析和重复执行依据；次数按授权预算确定，保留首次失败，不用重跑通过覆盖它 |
| 基准与分配 | Go `go test './<pkg>' -run '^$' -bench '^BenchmarkName$' -benchmem`；Rust `cargo bench`；JVM JMH；Python `pyperf`、`pytest-benchmark` | 仅在用户要求性能测量时选择，并核对成本、输入规模和可比条件 |
| 逃逸与分配分析 | Go `go build -gcflags='-m' './<pkg>'`；其它语言用分配剖析器 | 编译器报告只说明优化决策；热路径成本按[31](dimensions.md#31-性能内存与延迟)的纪律给依据 |
| 秘密扫描 | `gitleaks detect --no-git --redact --source '<目标目录>'` | 只用能脱敏输出、且不联网验证秘密的工具和参数 |
| 组件清单 | `syft '<目标目录>'` | 与实际制品比对见[36](dimensions.md#36-供应链与制品完整性) |
| 构建、持续集成与部署配置 | Shell `shellcheck`；Dockerfile `hadolint`；GitHub Actions `actionlint`、`zizmor`；基础设施 `checkov`、`trivy config`；Kubernetes `kube-linter` | 按[4.33](specialties.md#433-构建脚本持续集成与基础设施即代码)研判 |

命令执行后保留原始退出状态和必要输出，不能只统计匹配到的失败行。零匹配、工具不兼容、数据缺失或环境失败按[五](report.md#五证据等级与报告格式)记录。秘密扫描只输出脱敏位置，不能把凭据原值带进报告或上传外部服务。

工具结果与人工分析相互补充；工具未发现问题，只能说明该工具在记录的规则、路径和输入下未报告。数据流、业务授权、不变量和攻击链仍按[二](questions.md)逐对象核对。
