# Execution flow

Aqua's main goal is to express how the execution flows: moves from peer to peer, forks to parallel flows and then joins back, uses data from one step in another.

As the foundation of Aqua is based on π-calculus, finally flow is decomposed into [sequential](sequential.md) (`seq`, `.`), [conditional](conditional.md) (`xor`, `+`), [parallel](parallel.md) (`par`, `|`) computations and [iterations](iterative.md) based on data (`!P`). For each basic way to organize the flow, Aqua follows a set of rules to execute the operations:

* What data is available for use?
* What data is exported and can be used below?
* How errors and failures are handled?

These rules form a contract, as in [design-by-contract](https://en.wikipedia.org/wiki/Design\_by\_contract) programming.
