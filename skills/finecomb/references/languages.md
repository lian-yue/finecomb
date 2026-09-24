# Appendix A: Language runtime pitfalls

This appendix maps the categories of [16](dimensions.md#16-language-and-runtime-pitfalls) onto specific languages, and also collects each language's frequent pitfalls in other dimensions. How to use it:

- Pick the tables for the languages the target uses; for a mixed-language target pick every one, and check cross-language boundaries separately in [4.32](specialties.md#432-cross-language-boundaries-and-native-extensions).
- The tables list only frequent, easy-to-miss items and **do not replace the category checks of 16**: for every category in 16 you still have to find that language's matching mechanism, item by item.
- Semantics change between versions. First confirm the language, runtime and compiler versions the target declares and actually uses, then decide whether an item applies.
- For a language not in this appendix, build your own table with the method in [A.13 Other languages](#a13-other-languages).

## Language index

- [A.1 Go](lang-go.md)
- [A.2 Python](lang-python.md)
- [A.3 JavaScript and TypeScript](lang-javascript.md)
- [A.4 C and C++](lang-c-cpp.md)
- [A.5 Rust](lang-rust.md)
- [A.6 Java and Kotlin (JVM)](lang-jvm.md)
- [A.7 C# and .NET](lang-dotnet.md)
- [A.8 PHP](lang-php.md)
- [A.9 Ruby](lang-ruby.md)
- [A.10 Shell](lang-shell.md)
- [A.11 Swift and Objective-C](lang-swift-objc.md)
- [A.12 SQL and query dialects](lang-sql.md)

## A.13 Other languages

For a language not in this appendix, build your own table with the steps below, and write down the sources in the coverage record:

1. For each category in [16](dimensions.md#16-language-and-runtime-pitfalls), find that language's matching mechanism (null values, copying and aliasing, integer semantics, exceptions, concurrency model, module loading...).
2. Read the security guides, memory model and concurrency notes in the language's official documentation, and the rule lists of mainstream static analysis tools. A rule list is usually a checklist of that language's frequent pitfalls.
3. Cross-check the entries related to that language in general weakness catalogs (such as CWE).
4. Confirm the version the target actually uses, and keep only the entries that hold for that version.
