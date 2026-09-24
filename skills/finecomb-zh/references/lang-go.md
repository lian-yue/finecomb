# A.1 Go

| 检查点 | 什么算问题 |
| --- | --- |
| **接口 nil** | 空指针装进接口后，接口本身不为 nil，调用方永远判成有错；返回具体错误类型的空指针是典型来源 |
| **含锁/原子的值拷贝** | 首次使用后复制 `sync.Mutex`、`sync.WaitGroup`、`atomic.*` 或含有它们的结构体，保护与真实共享数据分离（`go vet` 的 copylocks 能查出一部分） |
| **64 位原子对齐** | 在 32 位平台上对裸 `int64`/`uint64` 字段做原子操作要求 8 字节对齐，否则崩溃；用 `atomic.Int64` 等带类型的包装规避 |
| 方法集 | 指针接收者的方法不在值类型的方法集里，接口断言失败 |
| 内嵌提升 | 意外把内嵌类型的方法提升成本类型的公开 API；内嵌具体的 `*net.TCPConn`、`*os.File` 等类型时，`io.Copy` 会探测到被提升的 `ReadFrom`/`WriteTo`，绕过包装层的 `Read`/`Write`；只内嵌接口时，又可能丢掉 `CloseWrite`、`SyscallConn` 等可选能力 |
| 闭包捕获 | `go.mod` 的 `go` 版本低于 1.22 时，`for` 循环变量在各次迭代间共享，闭包和 goroutine 读到的是后来的值；1.22 起每次迭代新建 |
| 循环里的 defer | 循环体内注册的 `defer` 要到函数结束才执行，资源迟迟不释放 |
| context 存字段 | 结构体里存 `context.Context`，生命周期错配，取消传播不可控；`context.WithCancel` 等返回的取消函数没调用（`go vet` 的 lostcancel） |
| 切片别名 | `append` 可能复用底层数组，导致远处被改；截断复用时旧引用仍指向同一数组 |
| 遍历复制 | `range` 的值变量是副本，改它无效；大结构体值遍历有拷贝开销 |
| 字符串与字节 | `string` 与 `[]byte` 互转的不必要拷贝；`unsafe.String`、`unsafe.Slice` 零拷贝转换后原数据被改；`len` 是字节数，`range` 按 rune 遍历，非法 UTF-8 变成 U+FFFD |
| 非安全操作 | 每处 `unsafe`：依赖的内存布局假设有没有写下来；GC 可见性；存成 `uintptr` 的地址不会让对象保持存活 |
| 反射 | 类型断言失败路径没处理；用在热路径上；可缓存的类型信息没缓存 |
| 泛型 | 约束太宽导致运行时才失败；零值语义；类型参数为指针与值时行为不同 |
| 包级初始化与全局 | `init` 的副作用、失败与执行顺序未定义；可变全局的所有权和并发契约不清 |
| panic 与 recover | 没有 recover 的 goroutine panic 会终止整个进程；`recover` 只在被 defer 直接调用的函数里生效；并发读写 map 是不可恢复的致命错误，不是 panic |
| nil 容器与通道 | 向 nil map 写入会 panic；nil 通道的收发永久阻塞；关闭 nil 或已关闭的通道、向已关闭的通道发送都会 panic |
| 不可比较类型 | 接口里装着切片、map、函数时，作为 map 键或用 `==` 比较会在运行时 panic |
| goroutine 泄漏 | 调用方超时返回后，子 goroutine 仍阻塞在无缓冲通道的发送上，永远不退出 |
| `io` 契约 | 实现 `io.Reader` 时可以同时返回 `n > 0` 和 `err != nil`，调用方要先处理这 n 字节；`Read` 返回 `0, nil` 不代表结束；`Write` 返回 `n < len(p)` 时必须带错误 |
| 错误判定 | 用 `==` 比较被包装过的错误；自定义错误类型没实现需要的 `Unwrap`、`Is`、`As`；超时错误不满足 `net.Error` 的 `Timeout()` 或 `os.ErrDeadlineExceeded`，调用方无法识别 |
| HTTP | `resp.Body` 没关闭；需要复用连接时没读完 Body；`http.DefaultClient` 没有超时；修改 `http.DefaultTransport` 等默认对象是进程级副作用 |
| 计时器 | `go.mod` 的 `go` 版本低于 1.23 时，没停止的 `time.Timer`、`time.Tick` 在触发前不会被回收；1.23 起计时器通道的语义也变了，按版本核对 |
| `os.File` 与描述符 | 调用 `Fd()` 后在 Unix 上 `SetDeadline` 失效；`*os.File` 被回收时终结器会关闭描述符，描述符号复用后可能误关别的文件；`os.NewFile` 接管描述符后不能再另行关闭 |
| 线程绑定 | 切换网络命名空间等线程级系统状态时要 `runtime.LockOSThread`，否则 goroutine 会被调度到别的线程 |
| cgo | 向 C 传 Go 指针要遵守 cgo 指针规则；C 分配的内存不受 GC 管理；C 回调进入 Go 时的线程与栈 |
| JSON | `encoding/json` 匹配字段名不区分大小写；重复键取最后一个；未知字段默认忽略；解到 `any` 的数字默认是 `float64`，超过 2^53 失真（需要时用 `UseNumber`） |
| 整数 | 有符号整数溢出静默回绕；`int` 的宽度随平台变化 |
| 容器限额 | Go 1.25 之前 `GOMAXPROCS` 不感知 cgroup 的 CPU 限额；内存上限要配合 `GOMEMLIMIT` |
| 工作区与本地覆盖 | `go.work` 和 `replace` 让仓库内构建通过，模块被单独使用时却缺依赖；用 `GOWORK=off` 核对 |
| `//go:linkname` | 引用标准库内部符号，新版本工具链可能拒绝链接（1.23 起收紧） |
| `iota` 重排 | 插入常量让 `iota` 重新编号，持久化或跨进程传递的值随之改变 |
