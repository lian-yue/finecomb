# A.6 Java 与 Kotlin（JVM）

| 检查点 | 什么算问题 |
| --- | --- |
| equals 与 hashCode | 两者不一致；`compareTo` 与 `equals` 不一致导致 `TreeSet`、`TreeMap` 去重异常；可变对象作键 |
| 装箱 | `Integer` 用 `==` 比较只在缓存范围内（默认 -128 到 127）碰巧成立；自动拆箱遇到 `null` 抛 `NullPointerException` |
| 整数 | 溢出静默回绕；需要检查时用 `Math.addExact` 等 |
| **反序列化** | `ObjectInputStream`、`XMLDecoder`、开启多态类型的 JSON 库、不安全配置的 YAML 库会执行 gadget 链（见[4.22](specialties.md#422-子进程动态执行与解码器副作用)） |
| XML | 很多解析器默认处理外部实体，要显式关闭（见[4.3](specialties.md#43-解码不受信任的外部数据)） |
| 并发 | 多线程写 `HashMap`；双重检查锁定缺 `volatile`；`SimpleDateFormat` 非线程安全；线程池里的 `ThreadLocal` 串数据或泄漏；捕获 `InterruptedException` 后没有恢复中断标志 |
| 资源 | 没用 try-with-resources；依赖已弃用的 `finalize` |
| 区域设置 | `toUpperCase`、`toLowerCase`、`String.format` 默认用当前区域（如土耳其语的 i），机器可读输出要指定 `Locale.ROOT` |
| 日志查找 | 日志框架解释消息里的查找表达式（如 Log4j 2 的 JNDI 查找，见[4.13](specialties.md#413-日志指标与追踪)、[4.22](specialties.md#422-子进程动态执行与解码器副作用)） |
| 反射 | `setAccessible` 绕过封装；按外部输入加载类 |
| 容器限额 | 较老的 JVM 不感知容器限额；堆大小要按配额设置 |
| Kotlin 空安全 | 来自 Java 的平台类型绕过空安全检查；`!!`；未初始化的 `lateinit` |
| Kotlin 协程 | 取消是协作式的；`catch (e: Exception)` 吞掉 `CancellationException`；`GlobalScope` 启动的协程泄漏；在协程里调用 `runBlocking` |
