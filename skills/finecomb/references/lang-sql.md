# A.12 SQL and query dialects

| Checkpoint | What counts as a problem |
| --- | --- |
| **NULL values** | Three-valued logic: `= NULL` is never true; `NOT IN` returns no rows at all when its subquery contains a `NULL` |
| Implicit conversion | Implicit conversion when comparing strings with numbers (for example MySQL treats `'1abc'` as 1), which stops indexes from being used or produces wrong matches |
| Collation | Whether case and accents matter; whether trailing spaces take part in comparison; the database's unique constraint disagrees with the application-level check |
| Non-strict mode | Overlong strings are truncated and invalid values are changed to defaults without an error |
| Default isolation level | Defaults differ between databases (for example MySQL InnoDB uses REPEATABLE READ, while PostgreSQL uses READ COMMITTED) |
| Result order | Without `ORDER BY` the order is not guaranteed, so `LIMIT` pagination results are unstable |
| Time types | Types with and without a time zone; how the session time zone affects reads and writes |
| Auto-increment and sequences | Auto-increment values have gaps and are not reclaimed on rollback; once exposed externally they can be enumerated |
