# 17 Time and clocks

| Checkpoint | What counts as a problem |
| --- | --- |
| Wall clock vs. monotonic clock | The wall clock is used to measure intervals → after a clock sync adjustment the interval becomes negative or huge |
| **Clock going backward** | What happens after the clock goes backward to every decision based on absolute timestamps (validity periods, grace windows, throttling, leases, fragment freshness windows): early expiry, never expiring, throttling stuck on forever |
| Precision and granularity | Under a shared coarse-grained clock, events in the same tick cannot be ordered — logic that depends on order breaks |
| Boundary comparisons | Less than or less than or equal; does the behavior on exact equality match the docs |
| Timeout propagation | Is the upstream's remaining deadline passed downstream; a downstream that uses its own fixed timeout makes the total duration uncontrollable |
| Clock skew | Comparing timestamps across machines; how much skew is tolerated |
| Time zones and daylight saving time | Is there a dependency on the local time zone; the repeated hour and the missing hour on the day daylight saving time switches |
| Units | Seconds / milliseconds / nanoseconds mixed up; the unit when converting between duration types and bare integers |
| Timers | Check collection, stop, reset and callback semantics against the actual runtime; after the logic has ended, scheduled tasks still hold resources or change state |
| Injectable | Can tests control time and the order of events; are assertions that rely on sleep or scheduling timing flaky — see [40](40-testability-and-fault-injection.md) |
| Calendar arithmetic | Leap years and February 29, adding one month at the end of a month, leap seconds, week numbers across the new year, "one day" not being 24 hours; is logic that bills, accrues interest or expires by date correct on these days |
| Limits of time representations | 32-bit seconds overflow in 2038; milliseconds or microseconds stored in 32-bit integers; the range of years a date library supports; the difference between treating a timestamp as signed or unsigned |
