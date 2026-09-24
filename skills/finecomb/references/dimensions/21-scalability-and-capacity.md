# 21 Scalability and capacity

> The previous dimension asks "will it be overwhelmed". This one asks "**what happens at ten times the size, and does adding machines help**".

| Checkpoint | What counts as a problem |
| --- | --- |
| Growth dimensions | Data volume / concurrency / connection count / tenant count / request rate — do you know which one hits the bottleneck first |
| **Tenfold growth** | What breaks first at ten times the current scale; does it degrade linearly or fall off a cliff |
| Linear scaling | Does adding instances / shards raise capacity linearly; is there a global serialization point that cancels it out |
| State and scaling | Is there local state that blocks horizontal scaling; can it be shared-nothing |
| Sharding and rebalancing | The choice of shard key and hot spots; the cost of data migration and rebalancing when scaling out |
| Uneven hot and cold load | Hot keys, hot partitions, long tails; is there a way to spread them out |
| Single points | Are there single points (global locks, single-instance components, a sole sequence number generator, a single writer) |
| Capacity headroom and alerts | How much headroom is left now; is there an alert when nearing the limit |
| Backpressure passed upstream | When capacity is reached, can the pressure be passed back upstream correctly instead of piling up locally |
| Startup amplification | Can many instances starting / reconnecting at the same time take down downstream services (thundering herd, all caches empty) |
| Running cost | How its own running cost grows with scale: pay-per-use outbound traffic, log and metric storage, per-call prices of third-party APIs, expensive queries; is there a cost cap and an alert |
