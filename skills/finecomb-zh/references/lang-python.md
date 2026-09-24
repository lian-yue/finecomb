# A.2 Python

| 检查点 | 什么算问题 |
| --- | --- |
| 可变默认参数 | `def f(x=[])` 的默认值只创建一次，跨调用共享 |
| 闭包晚绑定 | 循环里创建的 `lambda` 或内部函数在调用时才读变量，全部读到最后的值 |
| 别名与浅拷贝 | `[[0] * 3] * 3` 的三行是同一个列表；`copy.copy` 只拷一层 |
| 身份与相等 | 用 `is` 比较数字或字符串，依赖小整数缓存和字符串驻留这类实现细节 |
| 迭代中修改 | 遍历字典时增删键抛 `RuntimeError`；遍历列表时删元素会跳过元素 |
| **异常捕获过宽** | 裸 `except:` 或 `except BaseException` 吞掉 `KeyboardInterrupt`、`SystemExit` 和 `asyncio.CancelledError`；`except Exception: pass` 静默吞错；`finally` 里 `return` 吞掉异常；上下文管理器的 `__exit__` 返回真值会吞掉异常 |
| 全局解释器锁 | GIL 不让 `x += 1`、检查后写入这类复合操作变成原子；自由线程构建（3.13 起可选）下，依赖 GIL 隐含串行化的代码和 C 扩展可能失效 |
| asyncio | 协程里调用阻塞函数卡住整个事件循环；`asyncio.create_task` 的返回值不保存，任务可能在执行中被回收；`gather` 里一个失败时其它任务仍在运行；捕获 `CancelledError` 后不再抛出会破坏取消 |
| **反序列化即执行** | `pickle`、`marshal`、`shelve`、`yaml.load` 配不安全的 Loader、`torch.load`（2.6 之前默认不限制，旧版本里 `weights_only=True` 也能被绕过，按实际锁定的版本核对）都可能执行代码（见[4.22](specialties.md#422-子进程动态执行与解码器副作用)）；YAML 用 `yaml.safe_load` |
| 动态执行与命令 | `eval`、`exec`、`subprocess` 配 `shell=True`、`os.system`、`os.popen` |
| 导入与路径 | 导入时的副作用；循环导入拿到半初始化的模块；脚本所在目录排在 `sys.path` 前面，同名文件可以劫持标准库或依赖 |
| 任意精度整数 | 不会溢出，但外部输入可以构造巨大整数耗尽 CPU；较新版本默认限制整数与十进制字符串互转的位数（4300 位），超出抛 `ValueError` |
| 金额与精度 | 金额用 `float`；`Decimal` 的上下文精度与舍入方式没有显式设置 |
| 文本与编码 | `str` 与 `bytes` 混用；`open()` 不指定 `encoding` 时依赖区域设置 |
| 类型注解 | 注解在运行时不校验；外部输入要靠显式校验 |
| 迭代顺序 | `dict` 保持插入顺序（3.7 起语言保证）；`set` 不保证，且字符串哈希按进程随机，同一段代码每次运行的 `set` 遍历顺序可能不同 |
| 相等与哈希 | 类定义了 `__eq__` 却没定义 `__hash__`，实例变得不可哈希；可变对象作键 |
| 递归 | 默认递归上限约 1000，深嵌套的外部输入触发 `RecursionError` |
| 路径 | `os.path.join` 遇到绝对路径的组件会丢掉前面的部分，形成路径穿越；`tarfile`、`zipfile` 解包的条目路径（`tarfile` 的 `filter` 参数与默认值按版本核对）；`os.path.realpath` 默认不严格，路径不存在、成环或超长时不报错，只返回部分解析的结果，做包含检查要用严格模式，并在使用点重查 |
| 临时文件 | `tempfile.mktemp` 有竞争，用 `mkstemp` 或 `NamedTemporaryFile` |
| 网络默认值 | `requests` 等客户端不传 `timeout` 时可能永久等待 |
| 随机数 | `random` 不是密码学随机源，用 `secrets` |
| 时间 | 不带时区的 `datetime` 与带时区的混用；`datetime.utcnow()` 返回不带时区的值（3.12 起弃用） |
| 多进程 | 多线程进程里用 `fork` 启动方式可能死锁；默认启动方式随平台和版本变化（见[18](dimensions.md#18-并发与内存模型)「fork 与线程」） |
| 资源释放 | 依赖 `__del__` 或引用计数及时释放文件和连接；换到其它实现（如 PyPy）后不再及时 |
| 断言 | `python -O` 会移除 `assert`，用它做校验在优化模式下失效 |
| 依赖安装 | 源码包安装时执行构建脚本；`--extra-index-url` 让内部包名可能从公网源解析（依赖混淆） |
