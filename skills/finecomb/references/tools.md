# Part VII: Machine-checkable commands

This part is a table of tool options, not a list of things you must run. For when to run anything, see [execution boundaries during review](scope.md#execution-boundaries-during-review). Choosing the smallest entry point, reusing caches and handling failures follow the target project's test strategy.

Rows are grouped by "what to check", and each row gives common tools for each ecosystem. Before using them, do three things:

- **Replace and check the placeholders.** `<target-dir>` and `<file>` are paths within this review's scope; `<pkg>`, `<module>` and `<crate>` are specific packages or modules; `<tmp>` is a temporary directory used only by this run. Test names, budgets, platforms and arguments must all come from the actual target. Run commands in the owning module or at the project root, and take module and workspace options from the nearest testing notes.
- **Confirm the tool is already on the machine; do not install it automatically.** Commands such as `npx`, `pipx run` and `go run <module>@<version>` download over the network when the tool is missing, so handle them per authorization first.
- **Prefer tools the project has already chosen.** The ones listed are only examples; for ecosystems not listed, pick a tool with the same purpose.

## Read-only discovery

| Purpose | Example | Limits of the result |
| --- | --- | --- |
| File inventory and leftovers | `rg --files --hidden -g '!.git/**' -g '!**/.git/**' '<target-dir>'` | Lists only the given directory; ignore rules may exclude files, so empty output does not prove the directory has no content |
| Languages and size | `tokei '<target-dir>'` or `scc '<target-dir>'`; without such tools, count extensions from the previous row's output | Judges only by extension and file header, and cannot recognize languages embedded in strings; use the result for sharding and for picking [Appendix A](languages.md) |
| Identifying generated code | `rg -l -F -e 'DO NOT EDIT' -e '@generated' -e 'auto-generated' -e 'automatically generated' -- '<target-dir>'`; then read `linguist-generated` and `linguist-vendored` in `.gitattributes` | A hit is only one piece of evidence; list the hits for checking per [invocation and scope resolution](scope.md#invocation-and-scope-resolution) |
| Symbol and document references | `rg -n -F -- '<symbol>' '<target-dir>'` | Search hits are leads for locating code; dynamic calls and external callers still need manual checking |
| Leftover markers | See the code block below | Judge per [2](dimensions.md#2-leftover-markers-and-suppressions); the number of hits is not the number of issues |
| Format checks | Go `gofmt -l '<file>'`; Python `ruff format --check '<file>'` or `black --check '<file>'`; JavaScript/TypeScript `prettier --check '<file>'`; Rust `cargo fmt --check`; C/C++ `clang-format --dry-run -Werror '<file>'`; Shell `shfmt -d '<file>'` | List differences only when the chosen format check applies; do not format automatically |
| Mapping documents to entry points | The project's existing tool, with explicit directory, package and document arguments | Check manually when there is no tool; a passing mapping does not prove that branches and contract descriptions are correct |
| Document links | `lychee --offline '<target-dir>'` or the project's existing link check | Offline mode checks only local files and anchors; checking external addresses needs the network, so handle that per authorization first |

Example search for leftover markers and suppression markers (fixed strings; multiple `-e` flags are combined as a union; add or remove patterns for the target languages):

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
  -- '<target-dir>'
```

## On-demand verification

| Purpose | Example | Choosing and interpreting |
| --- | --- | --- |
| Ordinary correctness | Go `go test './<pkg>' -run '^TestName$'`; Python `pytest '<file>::<test>'`; JavaScript/TypeScript `npx vitest run '<file>' -t '<test-name>'` or `npx jest '<file>' -t '<test-name>'`; Rust `cargo test -p '<crate>' '<test>' -- --exact`; JVM `mvn -Dtest='<class>#<method>' test` or `gradle test --tests '<class>.<method>'`; .NET `dotnet test --filter 'FullyQualifiedName=<full-name>'`; C/C++ `ctest -R '^<test>$'` | Use test names confirmed in the source; for several tests, use a limited, exact selection; keep the tool's valid cache by default |
| Static analysis | Go `go vet './<pkg>'` or `staticcheck './<pkg>'`; Python `ruff check`, `pylint`; JavaScript/TypeScript `eslint`; C/C++ compiler `-Wall -Wextra`, `clang-tidy`, `cppcheck`, `scan-build`; Rust `cargo clippy`; JVM SpotBugs, Error Prone, PMD, Kotlin `detekt`; .NET build analyzers; PHP `phpstan`, `psalm`; Ruby `rubocop`; Shell `shellcheck`; cross-language `semgrep` | Pick tools as needed, and confirm the platform, build conditions, generated code and rule scope; no report does not mean no defect |
| Type checking | Python `mypy`, `pyright`; TypeScript `tsc --noEmit`; PHP `phpstan`, `psalm` | Covers only the parts with type information; runtime validation of external input still needs manual checking |
| Security rule scanning | Cross-language `semgrep`; Python `bandit`; Ruby `brakeman`; Go `gosec` | A rule hit is a lead; assess it per [27](dimensions.md#27-security-and-trust-boundaries) and the specialties; pulling rule sets over the network needs authorization first |
| Dead code leads | Go `deadcode`, the unused checks in `staticcheck`; Python `vulture`; JavaScript/TypeScript `knip`; Rust compiler unused warnings; at the dependency level, Rust `cargo udeps` | First confirm what that version can report; criteria in [1](dimensions.md#1-dead-code-and-reachability) |
| Duplication and complexity | Multi-language `jscpd` (duplication), `lizard` (complexity); Go `gocyclo` | Metrics are only leads; criteria in [3](dimensions.md#3-duplication) and [7](dimensions.md#7-complexity-and-maintainability) |
| Dependency graph and cycles | Go `go list -deps`, `go mod why`, `go mod graph`; JavaScript/TypeScript `madge --circular`; Python `pipdeptree`; Rust `cargo tree`; JVM `mvn dependency:tree`, `gradle dependencies` | Used for dependency direction in [5](dimensions.md#5-coupling-and-blast-radius) and for judging necessity in [35](dimensions.md#35-dependencies) |
| Dependency vulnerabilities | Go `govulncheck './<pkg>'`; Python `pip-audit`; JavaScript `npm audit`, `pnpm audit`; Rust `cargo audit`, `cargo deny`; JVM OWASP Dependency-Check; .NET `dotnet list package --vulnerable`; PHP `composer audit`; Ruby `bundler-audit`; cross-ecosystem `osv-scanner`, `trivy`, `grype` | Use tools the project has chosen and already installed; updating a tool or a vulnerability database may use the network, so get authorization first; assess results per [36](dimensions.md#36-supply-chain-and-artifact-integrity) |
| Dependency integrity | Go `go mod verify`; Rust `cargo build --locked`; Python `--require-hashes` at install time; npm `npm ci --ignore-scripts` in an isolated copy | Only shows that local dependencies match the lock file; commands that download or write the dependency directory run only in an isolated copy |
| Standalone build | Go `GOWORK=off go build -o '<tmp>/' './...'`; other ecosystems build in an isolated copy with workspace links and local overrides removed | Criteria in [38](dimensions.md#38-portability-and-build-context); do not change the source tree |
| Platform builds | Go `GOOS='<os>' GOARCH='<arch>' go build -o '<tmp>/artifact' './<pkg>'`; Rust `cargo build --target '<target-triple>'`; C/C++ cross toolchains. 32-bit targets can be used to check word size and atomic alignment | Only verifies the build for the given platform; it does not mean it runs correctly on that platform |
| API and ABI differences | Go `apidiff`, `gorelease`; Rust `cargo semver-checks`; JVM `japicmp`; C/C++ `abidiff` from libabigail; TypeScript `api-extractor` | Compares only two confirmed versions; interpret per [4.34](specialties.md#434-publishable-libraries-sdks-and-packages) |
| Coverage | Go `go test './<pkg>' -run '^TestName$' -coverprofile='<tmp>/cov.out'`, read with `go tool cover -func='<tmp>/cov.out'`; Python `coverage run -m pytest '<file>::<test>'`; JavaScript/TypeScript `c8`, `vitest --coverage`, `jest --coverage`; Rust `cargo llvm-cov`; JVM JaCoCo; .NET coverlet; C/C++ `gcov`, `llvm-cov` | Collect only when that measurement is authorized; interpret per [42](dimensions.md#42-coverage) |
| Concurrency detection | Go `go test './<pkg>' -race -run '^TestName$'`; C/C++/Rust ThreadSanitizer (`-fsanitize=thread`); Rust `loom` (needs matching test code); JVM `jcstress` | Select only the concurrent behavior that needs verification; limits of the conclusion in [18](dimensions.md#18-concurrency-and-memory-model) |
| Memory error detection | C/C++ `-fsanitize=address,undefined`, `valgrind`; Rust `cargo miri test`; Go with C code `go test -asan` | Select only the paths that need verification; no report does not mean no memory errors |
| Fuzzing | Go `go test './<pkg>' -run '^$' -fuzz '^FuzzName$' -fuzztime '<budget>'`; Python `atheris`; JavaScript `@jazzer.js/core`; C/C++ libFuzzer, AFL++; Rust `cargo fuzz`; JVM Jazzer; .NET SharpFuzz | Run only in an approved isolated copy; failing inputs may be written into the test data directory, so confirm ownership and the resource budget; not crashing does not prove safety |
| Property testing | Python `hypothesis`; JavaScript/TypeScript `fast-check`; Rust `proptest`; JVM `jqwik`; Go `testing/quick` | This is writing tests; whether to add them follows the project's test strategy |
| Diagnosing flaky issues | Go `go test './<pkg>' -run '^TestName$' -count '<count>'`; other frameworks use their own repeat options or plugins | Needs a failure analysis and a reason to repeat first; the count follows the authorized budget; keep the first failure and do not let a passing rerun overwrite it |
| Benchmarks and allocations | Go `go test './<pkg>' -run '^$' -bench '^BenchmarkName$' -benchmem`; Rust `cargo bench`; JVM JMH; Python `pyperf`, `pytest-benchmark` | Choose only when the user asks for performance measurement, and check the cost, input size and comparability conditions |
| Escape and allocation analysis | Go `go build -gcflags='-m' './<pkg>'`; other languages use an allocation profiler | Compiler reports only explain optimization decisions; costs on hot paths need evidence per the discipline in [31](dimensions.md#31-performance-memory-and-latency) |
| Secret scanning | `gitleaks detect --no-git --redact --source '<target-dir>'` | Use only tools and arguments that redact output and do not verify secrets over the network |
| Component inventory | `syft '<target-dir>'` | For comparison against the actual artifacts, see [36](dimensions.md#36-supply-chain-and-artifact-integrity) |
| Build, CI and deployment configuration | Shell `shellcheck`; Dockerfile `hadolint`; GitHub Actions `actionlint`, `zizmor`; infrastructure `checkov`, `trivy config`; Kubernetes `kube-linter` | Assess per [4.33](specialties.md#433-build-scripts-ci-and-infrastructure-as-code) |

After running a command, keep the original exit status and the necessary output; do not just count the matched failure lines. Record zero matches, tool incompatibility, missing data or environment failures per [Part V](report.md#part-v-evidence-levels-and-report-format). Secret scans output only redacted locations; never carry raw credential values into the report or upload them to external services.

Tool results and manual analysis complement each other. A tool finding no issues only shows that the tool reported nothing under the recorded rules, paths and inputs. Data flows, business authorization, invariants and attack chains are still checked object by object per [Part II](questions.md).
