# A.5 Rust

| 检查点 | 什么算问题 |
| --- | --- |
| **unsafe** | `unsafe` 块违反别名、初始化或生命周期规则会导致未定义行为，即使调用它的是安全代码；每个 `unsafe` 块的前提有没有写明；手写的 `Send`、`Sync` 实现是否成立 |
| 整数 | 调试构建溢出会 panic，发布构建默认回绕（除非开启 `overflow-checks`）；`as` 转换会截断，浮点转整数会饱和 |
| panic 来源 | 公开输入可触发的 `unwrap`、`expect`、下标越界、除零、`RefCell` 重复借用；`panic = "abort"` 时 `Drop` 不运行；panic 穿过 FFI 边界（按版本和 ABI，结果是终止或未定义行为） |
| 锁 | `Mutex` 中毒后的处理；`Mutex` 不可重入；持有 `std::sync::Mutex` 的守卫跨越 `.await` |
| async | async 函数里调用阻塞操作卡住执行器；取消安全：future 在 `.await` 处被丢弃会留下半完成的状态（如 `select!` 的分支）；丢弃 tokio 的 `JoinHandle` 不会取消任务 |
| 资源 | `mem::forget` 和 `Rc` 循环引用会泄漏，且都算安全代码；`Drop` 的执行顺序 |
| 构建期执行 | `build.rs` 与过程宏在构建时执行任意代码；特性合并让依赖意外启用了某些功能 |
| 反序列化 | `serde` 默认忽略未知字段（需要时用 `deny_unknown_fields`）；`untagged` 枚举的歧义匹配 |
| 哈希 | 标准 `HashMap` 默认用抗碰撞哈希；换成更快的非随机哈希后，对不可信键失去保护 |
