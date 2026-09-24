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
| 静态分析 | Go `go vet './<pkg>'` 或 `staticcheck './<pkg>'`；Python `ruff check`、`pylint`；JavaScript/TypeScript `eslint`；C/C++ 编译器 `-Wall -Wextra`、`clang-tidy`、`cppcheck`、`scan-build`；Rust `cargo clippy`；JVM SpotBugs、Error Prone、PMD，Kotlin `detekt`；.NET 构建分析器；PHP `phpstan`、`psalm`；Ruby `rubocop`；Shell `shellcheck`；跨语言 `semgrep`、CodeQL | 按需要选择工具，确认平台、构建条件、生成代码与规则范围；未报告不等于无缺陷 |
| 类型检查 | Python `mypy`、`pyright`；TypeScript `tsc --noEmit`；PHP `phpstan`、`psalm` | 只覆盖有类型信息的部分；外部输入的运行时校验仍要人工核对 |
| 安全规则扫描 | 跨语言 `semgrep`；Python `bandit`；Ruby `brakeman`；Go `gosec` | 规则命中是线索，按[27](dimensions.md#27-安全与信任边界)和专项研判；联网拉取规则集先按授权处理 |
| 死代码线索 | Go `deadcode`、`staticcheck` 的未使用检查；Python `vulture`；JavaScript/TypeScript `knip`；Rust 编译器的未使用警告；依赖层面 Rust `cargo udeps` | 先确认该版本能报告什么，判据见[1](dimensions.md#1-死代码与可达性) |
| 重复与复杂度 | 多语言 `jscpd`（重复）、`lizard`（复杂度）；Go `gocyclo` | 度量只是线索，判据见[3](dimensions.md#3-重复)、[7](dimensions.md#7-复杂度与可维护性) |
| 依赖图与环 | Go `go list -deps`、`go mod why`、`go mod graph`；JavaScript/TypeScript `madge --circular`；Python `pipdeptree`；Rust `cargo tree`；JVM `mvn dependency:tree`、`gradle dependencies` | 用于[5](dimensions.md#5-关联与影响面)的依赖方向和[35](dimensions.md#35-依赖)的必要性判断 |
| 依赖漏洞 | Go `govulncheck './<pkg>'`；Python `pip-audit`；JavaScript `npm audit`、`pnpm audit`；Rust `cargo audit`、`cargo deny`；JVM OWASP Dependency-Check；.NET `dotnet list package --vulnerable`；PHP `composer audit`；Ruby `bundler-audit`；跨生态 `osv-scanner`、`trivy`、`grype` | 使用项目已选且已安装的工具；工具联网查询或刷新漏洞库数据属于默认允许的只读联网（见[目标项目的硬边界](scope.md#目标项目的硬边界)），升级或安装工具本身先按授权处理；结果按[36](dimensions.md#36-供应链与制品完整性)研判 |
| 依赖版本与安全公告 | Go `go list -m -u all`；JavaScript `npm outdated`、`pnpm outdated`；Python `pip list --outdated`；PHP `composer outdated`；Ruby `bundle outdated`；.NET `dotnet list package --outdated`；没有审计工具时，按包名和版本查 [OSV](https://osv.dev) 或 [GitHub Advisory Database](https://github.com/advisories) | 只读联网查询，默认允许；「有更新版本」不等于「有安全问题」，要对照公告与修复版本，按[36](dimensions.md#36-供应链与制品完整性)研判 |
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
| 智能合约 | Solidity：`slither`、`aderyn`、编译器警告；符号执行 `mythril`、`halmos`；模糊与不变量测试 `forge test`（Foundry）、`echidna`、`medusa`。Solana：`anchor test`。Move：`sui move test`、`aptos move test`、Move Prover | 规则命中是线索，按[4.38](specialties.md#438-智能合约与链上交互)、[4.41](specialties.md#441-defi-经济机制与预言机) 到 [4.45](specialties.md#445-区块链节点共识与协议实现)、[4.50](specialties.md#450-账户抽象与合约钱包)和对应语言表研判；分叉主网只做本地模拟，不发送真实交易 |
| 非 EVM 合约 | Solana 模糊测试 Trident；Starknet 静态分析 `caracal`、测试 `snforge test`；TON 静态分析 Misti；CosmWasm `cargo test` 配合 `cw-multi-test` | 规则命中是线索；判据见对应语言表 |
| 合约升级核对 | 新旧两版各跑 `forge inspect '<合约>' storageLayout` 后对比；`slither-check-upgradeability`；OpenZeppelin Upgrades 的 `validate` | 只核对存储布局与初始化写法；升级时实际执行了哪些初始化，按[4.38](specialties.md#438-智能合约与链上交互)（升级流程）人工核对 |
| 部署版本核对 | `forge verify-bytecode '<地址>' '<路径>:<合约>'`；Solana `solana-verify get-program-hash` | 只读查询；只证明链上代码与指定提交一致，不证明该提交安全 |
| 交易与签名核对 | `cast run '<交易哈希>'`（本地重放并打印调用追踪）；`cast decode-calldata`；Safe 多签用 `safe-tx-hashes-util` 在签名前独立计算交易哈希 | 重放只在本地进行，不发送交易；判据见[4.43](specialties.md#443-钱包签名与链下组件) |
| 零知识电路 | 欠约束检测：Circom 用 `circomspect`、Picus；halo2 的 `MockProver` 只检查诚实见证能否满足约束，要另写改动见证后必须失败的测试 | 工具没报不代表约束完整；判据见[4.44](specialties.md#444-零知识证明与电路) |
| 链上读取 | Foundry `cast code`、`cast storage`、`cast call`、`cast implementation`；区块浏览器上的已验证源码和合约读取页 | 只读查询，记录链标识与块高；判据见[非源码目标：链上合约地址](targets.md#链上合约地址) |
| 编译产物与二进制 | `file`、`checksec`、`objdump`、`strings`；Linux `readelf -d`（RPATH、RUNPATH、BIND_NOW）、`readelf -lW`（GNU_STACK）、`readelf -n`（CET 属性）；Windows `winchecksec '<文件>'`、`binskim analyze '<文件>'`；macOS `otool -L`、`otool -l '<文件>'`（`LC_RPATH`）、`codesign -dv`、`codesign -d --entitlements - '<文件>'`；内嵌依赖 `go version -m '<文件>'`、`govulncheck -mode=binary '<文件>'`、`cargo audit bin '<文件>'`；托管代码 `ilspycmd`（.NET）、`jadx`（JVM 与 Android）；反编译 Ghidra、radare2；组件识别 `syft` | 只做静态查看；需要运行时只在隔离环境；判据见[非源码目标：编译产物与二进制](targets.md#编译产物与二进制) |
| 桌面安装包 | Windows 签名 `osslsigncode verify '<文件>'` 或 PowerShell `Get-AuthenticodeSignature '<文件>'`；MSI 用 msitools 的 `msiinfo export '<文件>.msi' CustomAction`、`msidump`、`msiextract`；macOS `pkgutil --check-signature '<文件>.pkg'`、`pkgutil --expand '<文件>.pkg' '<tmp>/x'`、`spctl --assess -vv --type install '<文件>.pkg'`、`xcrun stapler validate '<文件>'`；deb `dpkg-deb -e '<文件>.deb' '<tmp>/ctl'`；rpm `rpm -qp --scripts '<文件>.rpm'`；Snap `unsquashfs -d '<tmp>/x' '<文件>.snap'` 后读 `meta/snap.yaml`；Flatpak 读清单的 `finish-args`，已安装的用 `flatpak info --show-permissions '<应用 ID>'`；Electron 用 `@electron/fuses` 的 `read --app '<应用>'`、`@electron/asar` 的 `asar extract '<app.asar>' '<tmp>/x'` | 只解包，不安装；安装、修复、运行放到隔离虚拟机；AppImage 的 `--appimage-extract` 会运行包里自带的启动程序，只在隔离环境用；判据见[非源码目标：桌面安装包与桌面应用](targets.md#桌面安装包与桌面应用) |
| 移动应用包 | `apktool`、`jadx`、`aapt dump badging`、`apksigner verify --print-certs -v '<文件>.apk'`、MobSF；iOS 包 `codesign -d --entitlements - '<应用>'`、`plutil`，`otool -l '<主程序>'` 查 `LC_ENCRYPTION_INFO` 的 `cryptid`；动态分析 Frida、objection | 动态分析只在隔离的测试设备上做；判据见[非源码目标：移动应用包](targets.md#移动应用包)和[4.49](specialties.md#449-移动应用组件与进程间通信) |
| 浏览器扩展 | 解包 `unzip -d '<tmp>/x' '<文件>'`（CRX 前面有签名头，`unzip` 会提示多余字节，但能解开）；Firefox 扩展 `web-ext lint --source-dir '<目录>'` | lint 只按 Firefox 的规则查；权限、通信与远程代码仍要人工读 `manifest.json` 和脚本；判据见[非源码目标：浏览器扩展](targets.md#浏览器扩展) |
| 固件 | `binwalk`、`unblob`、EMBA；仿真 FirmAE；组件版本 `cve-bin-tool '<解包目录>'` | 在隔离目录解包；`cve-bin-tool` 首次运行会下载漏洞库；判据见[非源码目标：固件与设备镜像](targets.md#固件与设备镜像) |
| 物联网协议 | MQTT `nmap -p 1883,8883 --script mqtt-subscribe '<地址>'`、`mosquitto_sub -h '<代理>' -t '#' -v -C '<条数>'`；BLE 用 `bluetoothctl`、nRF Sniffer 配合 Wireshark，旧式配对的抓包用 `crackle` 分析；Zigbee 用 KillerBee | 订阅、配对、重放都是主动操作，只对获准的设备和代理做；订阅 `#` 会读到别人的数据；无线发射受法规约束；判据见[非源码目标：设备通信与物联网协议](targets.md#设备通信与物联网协议) |
| 容器镜像 | `docker history --no-trunc`、`dive`、`trivy image`、`grype`、`syft`；不拉镜像层读清单与配置 `crane manifest`、`crane config`、`skopeo inspect 'docker://<镜像>'`；验签 `cosign verify '<镜像>' --certificate-identity '<身份>' --certificate-oidc-issuer '<签发方>'` | 拉取镜像会联网下载，按授权处理；不带身份参数的验签证明不了是谁签的；判据见[非源码目标：容器镜像](targets.md#容器镜像) |
| 已发布的包 | npm `npm pack '<包>@<版本>'`、`npm diff --diff='<包>@<旧版本>' --diff='<包>@<新版本>'`，在隔离副本里 `npm ci --ignore-scripts` 后运行 `npm audit signatures`；PyPI 取 wheel 用 `pip download --no-deps --only-binary ':all:' -d '<tmp>' '<包>==<版本>'`，取 sdist 按 `https://pypi.org/pypi/<包>/<版本>/json` 给出的地址直接下载，数字证明用 `pypi-attestations verify pypi --repository '<仓库地址>' 'pypi:<文件名>'`；Ruby `gem fetch '<包>' -v '<版本>'` 后 `gem unpack`；Rust 从 `https://static.crates.io/crates/<包>/<包>-<版本>.crate` 下载后读 `.cargo_vcs_info.json`；.NET `dotnet nuget verify '<包>.nupkg'`；JVM `jarsigner -verify -verbose '<文件>.jar'`；与源码比对用 `diff -r` 或 `diffoscope` | 下载制品属于联网取数据，按授权处理；`npm pack` 从注册表取包时不跑生命周期脚本；不要用 `pip download` 取 sdist，它会执行构建后端，等于运行包里的代码；判据见[非源码目标：已发布的包](targets.md#已发布的包) |
| 签名与来源证明 | 镜像 `cosign verify` 带 `--certificate-identity`、`--certificate-oidc-issuer`，证明用 `cosign verify-attestation --type '<类型>'` 并带同样的身份参数；GitHub `gh attestation verify '<文件>' --repo '<所有者>/<仓库>'`；SLSA `slsa-verifier verify-artifact '<文件>' --provenance-path '<证明>' --source-uri '<仓库>' --source-tag '<标签>'`；in-toto `in-toto-verify` | 必须写明预期的签名身份和源码来源，不带身份参数的验证证明不了什么；查询透明日志属于只读联网；判据见[非源码目标：签名与来源证明](targets.md#签名与来源证明) |
| SBOM 与 VEX | 格式校验 `cyclonedx validate --input-file '<文件>' --fail-on-errors`（cyclonedx-cli）；SPDX 最小要素 `sbomcheck '<文件>'`（ntia-conformance-checker）；质量评分 `sbomqs score '<文件>'`；按清单查漏洞 `grype 'sbom:<文件>'`、`trivy sbom '<文件>'`、`osv-scanner scan -L '<文件>'`；带 VEX 过滤 `grype 'sbom:<文件>' --vex '<VEX 文件>'` | 格式校验通过不代表内容完整；被 VEX 过滤掉的漏洞单独列出，逐条核对理由；判据见[非源码目标：依赖清单、锁文件与 SBOM](targets.md#依赖清单锁文件与-sbom) |
| 线上服务 | 被动观察：`curl -sI`、`openssl s_client`、`dig`、证书透明日志查询。主动测试：`nmap`、`testssl.sh`、`nuclei`、ZAP 主动扫描 | 被动观察默认低频进行；主动测试须书面授权；判据见[非源码目标：线上服务](targets.md#线上服务) |
| 前端包与源映射 | 还原源码 `sourcemapper -url '<映射地址或文件>' -output '<tmp>/src'`；反混淆 `webcrack '<文件>' -o '<tmp>/out'`；识别库版本 `retire --path '<目录>'`；秘密 `gitleaks detect --no-git --redact --source '<目录>'` | 从线上取映射文件属于被动访问公开资源，保持低频；按特征识别的库版本注明是推断；判据见[非源码目标：前端包与源映射](targets.md#前端包与源映射) |
| 接口规格文件 | `spectral lint '<规格文件>'`（配 `@stoplight/spectral-owasp-ruleset` 规则集）、`vacuum lint '<规格文件>'`、`redocly lint '<规格文件>'` | 规则只查规格的写法；认证与授权是否落实，要对照实现或做授权测试；判据见[非源码目标：接口规格文件](targets.md#接口规格文件) |
| 域名、DNS 与邮件 | `dig +short TXT '<域名>'`、`dig +short TXT '_dmarc.<域名>'`、`dig +short TXT '_mta-sts.<域名>'`、`dig +short CAA '<域名>'`；DNSSEC `delv '<域名>'`，`dnsviz probe -o '<tmp>/p.json' '<域名>'` 后 `dnsviz print -r '<tmp>/p.json'`；邮件记录汇总 `checkdmarc '<域名>'`；MTA-STS 策略 `curl -s 'https://mta-sts.<域名>/.well-known/mta-sts.txt'`；区域传送 `dig AXFR '<域名>' '@<名称服务器>'` | 公开 DNS 查询和读取公开策略文件属于被动观察；区域传送与开放解析测试要授权；判据见[非源码目标：域名、DNS 与邮件配置](targets.md#域名dns-与邮件配置) |
| 主机与操作系统 | Linux `lynis audit system --logfile '<tmp>/lynis.log' --report-file '<tmp>/lynis-report.dat'`、OpenSCAP `oscap xccdf eval --profile '<配置档>' --results '<tmp>/results.xml' '<数据流文件>'`、包完整性 `rpm -Va`、`dpkg --verify`；Windows 用 HardeningKitty 的 `Invoke-HardeningKitty -Mode Audit -FileFindingList '<清单>'`；macOS 用 mSCP 生成的检查脚本；Docker 宿主用 `docker-bench-security` | 完整检查通常要管理员权限；Lynis 默认写 `/var/log`；docker-bench 以特权容器运行；基线条目按主机用途取舍，不把「不满足基线」直接当成漏洞；判据见[非源码目标：主机与操作系统](targets.md#主机与操作系统) |
| 云账号 | Prowler（除各家云外也支持 `kubernetes`、`m365`、`github`）、ScoutSuite，或云厂商命令行的只读查询 | 使用只读身份；ScoutSuite 最后一次发布是 v5.14.0（2024-05-10），注意规则的更新日期；判据见[非源码目标：云账号与基础设施状态](targets.md#云账号与基础设施状态) |
| Kubernetes 集群 | 权限 `kubectl auth can-i --list`（查别的主体加 `--as`，需要模拟权限）；基线 `kubescape scan framework '<框架>'`、`kube-bench`；资源与镜像 `trivy k8s`；秘密只列名字 `kubectl get secrets -A` | kube-bench 和 kubescape 的主机扫描会在集群里创建特权工作负载，属于写操作，先取得授权；不用 `-o yaml` 读 Secret 的取值；托管集群的控制面条目记为不适用；判据见[非源码目标：Kubernetes 集群](targets.md#kubernetes-集群) |
| Helm 与状态文件 | `helm template '<chart>' -f '<取值文件>'` 渲染后交给 `kube-linter`、`checkov`；`helm pull --prov` 后 `helm verify '<chart 包>'`；Terraform 状态用 `jq '.resources[].instances[].attributes' '<状态文件>'` 直接读，秘密用 `gitleaks detect --no-git --redact --source '<目录>'` 扫 | `terraform show -json` 要先 `terraform init`，会下载提供方插件；`terraform plan`、`refresh` 会用凭据访问真实环境，默认不跑；判据见[非源码目标：配置、部署清单与基础设施代码](targets.md#配置部署清单与基础设施代码) |
| 网络设备配置 | Batfish 离线分析导出的设备配置（可达性、ACL 与防火墙规则的遮蔽和冗余） | 只分析导出的配置，不连接设备；导出之后的变更看不到；判据见[非源码目标：网络设备与防火墙配置](targets.md#网络设备与防火墙配置) |
| SaaS 租户与代码托管 | Microsoft 365 `Invoke-SCuBA`（ScubaGear）；Google Workspace `scubagoggles gws`（ScubaGoggles）；Prowler `prowler m365`、`prowler github`；GitHub 与 GitLab 组织 `legitify analyze`；单个仓库 `scorecard --repo='<仓库>'` | 多数工具要在租户里注册应用或授予读取权限，这是配置变更，先取得授权；使用只读管理角色；判据见[非源码目标：SaaS 租户与第三方集成](targets.md#saas-租户与第三方集成) |
| 智能体配置 | 隐藏字符 `rg -nP '[\x{200B}-\x{200F}\x{202A}-\x{202E}\x{2060}-\x{2064}\x{2066}-\x{2069}\x{E0000}-\x{E007F}]' -- '<目录>'`；秘密 `gitleaks detect --no-git --redact --source '<目录>'`；专用扫描 `snyk-agent-scan inspect`、`snyk-agent-scan scan` | 专用扫描器会执行配置里的服务器命令、连接远程服务器，把工具描述发到厂商接口，还需要厂商令牌，只在隔离环境、取得授权后使用；默认只静态读配置；判据见[非源码目标：智能体配置与 MCP 服务器](targets.md#智能体配置与-mcp-服务器) |
| WebAssembly 模块 | WABT 的 `wasm-objdump -x '<文件>'`、`wasm2wat '<文件>'`、`wasm-decompile '<文件>'`；`wasm-tools validate '<文件>'`、`wasm-tools print '<文件>'` | 只做静态查看；需要运行时放在隔离宿主里；判据见[非源码目标：WebAssembly 模块](targets.md#webassembly-模块) |
| 办公文档与 PDF | oletools 的 `oleid`、`olevba`、`mraptor`、`msodde`、`oleobj`、`rtfobj`；PDF 用 `pdfid.py`、`pdf-parser.py`、`qpdf --check`，`qpdf --qdf --object-streams=disable '<文件>' '<tmp>/x.pdf'` 展开后查看；元数据 `exiftool '<文件>'` | 不用办公软件或阅读器打开，不启用宏；不上传到在线扫描服务；判据见[非源码目标：办公文档与 PDF](targets.md#办公文档与-pdf) |
| 日志与抓包 | `capinfos -H '<文件>'`（摘要与时间范围）、`tshark -r '<文件>' -q -z conv,tcp`、在 `<tmp>` 目录里运行 `zeek -r '<文件>'`（日志写到当前目录）、`suricata -r '<文件>' -l '<tmp>'` | 在副本上分析；抓包和 HAR 里有第三方的个人数据和会话凭据，按敏感数据处置；判据见[非源码目标：日志、流量与运行数据](targets.md#日志流量与运行数据) |

命令执行后保留原始退出状态和必要输出，不能只统计匹配到的失败行。零匹配、工具不兼容、数据缺失或环境失败按[五](report.md#五证据等级与报告格式)记录。秘密扫描只输出脱敏位置，不能把凭据原值带进报告或上传外部服务。

工具结果与人工分析相互补充；工具未发现问题，只能说明该工具在记录的规则、路径和输入下未报告。数据流、业务授权、不变量和攻击链仍按[二](questions.md)逐对象核对。
