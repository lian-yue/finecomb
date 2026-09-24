# 12 State machines and transitions

| Checkpoint | How to check | What counts as a problem |
| --- | --- | --- |
| State enumeration | List the states the object actually has and how each is represented | State combinations contradict each other, transition conditions are unclear; not using an enum type is not a defect by itself |
| **State × operation matrix** | Put the behavior of each public operation in each state into a table | Some cells are empty — not defined, not tested, not documented |
| Illegal transitions | Reopen after close, use before initialization, start twice, skip intermediate states | Illegal transitions silently accepted, or behavior undefined |
| Transition atomicity | Can others see an intermediate state during a transition | The intermediate state is observable and not defined |
| Terminal states | Can it recover after entering an error state; is the terminal state really terminal | Stuck in an intermediate state with no way out |
| Concurrent transitions | Two execution flows trigger a transition at the same time | See [18](18-concurrency-and-memory-model.md) |
| State and resources | Which resources are held in each state; who releases them on a transition | Release missed on some transition path |
| State persistence | Is state kept across restarts; which state does it land in after a restart | The state after restart does not match reality |
