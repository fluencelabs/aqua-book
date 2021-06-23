# Parallel

Parallel execution is where everything becomes shiny.

### Contract

Parallel branches have no access to each other's data. Sync points must be explicit \(see Join behavior\).

If any branch is executed successfully, the flow execution continues.

All the data defined in parallel branches is available in the subsequent code.

### Parallel operations

As of time of writing, there's only one parallel expression: `par`

Its syntax is derived from π-calculus notation of parallelism: `A | B`

```text
-- foo and bar will be executed in parallel, if possible
foo()
par bar()
```

Parallel execution has some implementation limitations:

* Parallel means independent execution on different peers
* No parallelism when executing a script on single peer \(fix planned\)
* No concurrency in services: one service instance does only one job in a time. Keep services small \(wasm limitation\)

### Join behavior

Join means that data was created by different parallel execution flows and then used on a single peer to perform computations.

In Aqua, you can refer to previously defined variables. In case of sequential computations, they are available, if execution not failed:

```text
-- Start execution somewhere
on peer1:
  -- Go to peer1, execute foo, remember x
  x <- foo()
  
-- x is available at this point
  
on peer2:
  -- Go to peer2, execute bar, remember y
  y <- bar()

-- Both x and y are available at this point
-- Use them in a function
baz(x, y)        
```

Let's make this script parallel: execute `foo` and `bar` on different peers in parallel, then use both to compute `baz`.

```text
-- Start execution somewhere
on peer1:
  -- Go to peer1, execute foo, remember x
  x <- foo()
  
-- Notice par on the next line: it means, go to peer2 in parallel with peer1
  
par on peer2:
  -- Go to peer2, execute bar, remember y
  y <- bar()

-- Remember the contract: either x or y is available at this point
-- As it's enough to execute just one branch to advance further
baz(x, y)
```

What will happen when execution comes to `baz`?

Actually, the script will be executed twice: first time it will be sent from `peer1`, and second time – from `peer2`. Or another way round: `peer2` then `peer1`, we don't know who is faster.

When execution will get to `baz` for the first time, [Aqua VM](../../aqua-vm.md) will realize that it lacks some data that is expected to be computed above in the parallel branch. And halt.

After the second branch executes, VM will be woken up again, reach the same piece of code and realize that now it has enough data to proceed.

This way you can express race \(see [Collection types](../types.md#collection-types) and [Conditional return](conditional.md#conditional-return) for other uses of this pattern\):

```text
-- Initiate a stream to write into it several times
results: *string

on peer1:
  results <- foo()

par on peer2:
  results <- bar()

-- When any result is returned, take the first (the fastest) to proceed
baz(results!0)
```

