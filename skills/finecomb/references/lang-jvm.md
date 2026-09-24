# A.6 Java and Kotlin (JVM)

| Checkpoint | What counts as a problem |
| --- | --- |
| equals and hashCode | The two are inconsistent; `compareTo` inconsistent with `equals` breaks deduplication in `TreeSet` and `TreeMap`; mutable objects used as keys |
| Boxing | Comparing `Integer` values with `==` only happens to work within the cache range (-128 to 127 by default); auto-unboxing a `null` throws `NullPointerException` |
| Integers | Overflow wraps around silently; use `Math.addExact` and similar when overflow must be checked |
| **Deserialization** | `ObjectInputStream`, `XMLDecoder`, JSON libraries with polymorphic typing turned on, and YAML libraries with unsafe configuration execute gadget chains (see [4.22](specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md)) |
| XML | Many parsers process external entities by default, and this must be turned off explicitly (see [4.3](specialties/4.3-decoding-untrusted-external-data.md)) |
| Concurrency | Multiple threads writing to a `HashMap`; double-checked locking without `volatile`; `SimpleDateFormat` is not thread-safe; `ThreadLocal` in thread pools carries data over between tasks or leaks; the interrupt flag is not restored after catching `InterruptedException` |
| Resources | Not using try-with-resources; relying on the deprecated `finalize` |
| Locale | `toUpperCase`, `toLowerCase` and `String.format` use the current locale by default (such as the Turkish i); machine-readable output must specify `Locale.ROOT` |
| Log lookups | The logging framework interprets lookup expressions in messages (such as the JNDI lookup of Log4j 2, see [4.13](specialties/4.13-logs-metrics-and-tracing.md), [4.22](specialties/4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md)) |
| Reflection | `setAccessible` bypasses encapsulation; loading classes by names from external input |
| Container limits | Older JVMs are not aware of container limits; the heap size must be set according to the quota |
| Random numbers | `java.util.Random`, `Math.random` and `ThreadLocalRandom` are not cryptographic random sources; use `SecureRandom` |
| Kotlin null safety | Platform types coming from Java bypass null-safety checks; `!!`; `lateinit` that is never initialized |
| Kotlin coroutines | Cancellation is cooperative; `catch (e: Exception)` swallows `CancellationException`; coroutines launched in `GlobalScope` leak; calling `runBlocking` inside a coroutine |
