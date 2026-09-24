# A.7 C# 与 .NET

| 检查点 | 什么算问题 |
| --- | --- |
| async void | 事件处理以外的 `async void`：异常可能直接终止进程，调用方也无法等待 |
| 同步等待异步 | `.Result`、`.Wait()` 在有同步上下文的环境里死锁；库代码缺 `ConfigureAwait(false)`；不等待的任务的异常无人观察 |
| 资源 | 没用 `using` 释放 `IDisposable`；每次请求新建 `HttpClient` 导致端口耗尽 |
| **反序列化** | `BinaryFormatter`（新版本已禁用或移除）、`TypeNameHandling` 不为 `None` 的 Json.NET 会执行 gadget 链 |
| 字符串比较 | `StartsWith(string)`、`string.Compare` 等默认按当前文化比较；标识符和安全判断要用 `StringComparison.Ordinal` |
| 整数 | 默认不检查溢出（`checked` 上下文除外） |
| 值类型 | `struct` 按值复制；改集合里可变 `struct` 的副本无效 |
| LINQ 延迟执行 | 多次枚举会重复执行查询；闭包捕获的变量在执行前被改 |
| 时间 | `DateTime.Kind` 混用；`DateTime.Now` 与 `DateTime.UtcNow` 混用 |
| 可空引用类型 | 只是编译期警告，运行时不校验 |
| 正则 | 默认没有超时，回溯可被利用；设置超时，或用 .NET 7 起的 `NonBacktracking` |
| 锁对象 | `lock(this)`、锁类型对象或字符串，外部代码可能锁同一个对象 |
| 随机数 | `System.Random` 不是密码学随机源，用 `RandomNumberGenerator` |
