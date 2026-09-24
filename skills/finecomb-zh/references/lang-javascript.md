# A.3 JavaScript 与 TypeScript

| 检查点 | 什么算问题 |
| --- | --- |
| 宽松相等与转换 | `==` 的类型转换；`typeof null === 'object'`；`NaN !== NaN` |
| **数值精度** | 所有 `number` 都是双精度，超过 2^53 的整数失真（JSON 里的 64 位 ID 尤其常见）；`BigInt` 不能与 `number` 混算，`JSON` 也不直接支持 |
| **原型污染** | 深合并、按路径赋值等处理外部对象时写入 `__proto__`、`constructor.prototype`；外部键用 `Object.create(null)` 或 `Map` 存 |
| 对象键顺序 | 整数样的键按数值升序排在前面，其余按插入顺序 |
| `this` 与作用域 | 回调里丢失 `this`；`var` 是函数作用域，循环里的闭包共享同一个变量（`let` 每次迭代新建） |
| **浮动 Promise** | 没有 `await` 或 `.catch` 的 Promise，错误丢失或变成未处理的拒绝（Node 15 起默认终止进程）；`forEach` 里的 `async` 回调不会被等待 |
| await 交错 | `await` 前检查、`await` 后据此写入，中间状态已被别的任务改掉（见[18](dimensions.md#18-并发与内存模型)） |
| `Promise.all` | 一个失败就立即拒绝，其余仍在运行且结果被丢弃；需要全部结果时用 `allSettled` |
| 事件循环阻塞 | 同步的 CPU 密集计算、同步文件或加密调用、大 `JSON.parse` 阻塞所有请求 |
| 流与背压 | 忽略 `write()` 返回的 `false`；`pipe` 不传播错误，用 `pipeline` |
| 事件监听泄漏 | 反复添加监听器而不移除 |
| 正则 | 回溯型引擎的灾难性回溯；带 `g` 或 `y` 标志的正则被复用时 `lastIndex` 有残留 |
| 字符串 | `length` 和下标按 UTF-16 码元计，截断可能切开代理对 |
| 数组排序 | `sort()` 默认按字符串比较（`[10, 9, 1]` 排成 `[1, 10, 9]`） |
| 日期 | 月份从 0 开始；只有日期的 ISO 字符串按 UTC 解析，有时间没时区的按本地时间解析 |
| TypeScript 类型 | 类型只在编译期存在；外部输入用 `as` 断言不做任何校验；`any` 扩散；非空断言 `!`；关闭了 `strict` |
| 模块 | CommonJS 与 ES 模块两份同时加载时状态分裂；循环依赖拿到未初始化的导出；模块顶层的副作用 |
| Node 进程与文件 | `child_process.exec` 经过 shell，`execFile`、`spawn` 默认不经过；`path.join` 挡不住路径穿越，要规范化后再检查前缀；`Buffer.allocUnsafe` 返回未初始化内存 |
| 浏览器 | `innerHTML`、`eval`、`new Function`、字符串形式的 `setTimeout`；`postMessage` 没校验来源（见[4.18](specialties.md#418-用户界面与可访问性)） |
| 依赖安装 | 安装脚本（`preinstall`、`postinstall`）执行任意代码；锁文件没提交；私有作用域没有绑定到私有源（依赖混淆） |
| 定时器 | `setInterval` 回调里有异步工作时会重叠执行 |
| 随机数 | `Math.random` 不是密码学随机源，用 `crypto.getRandomValues` 或 Node 的 `crypto.randomBytes`、`crypto.randomUUID` |
