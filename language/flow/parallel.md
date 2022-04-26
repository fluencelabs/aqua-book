# Parallel

Parallel execution is where Aqua fully shines.

## Contract

* Parallel arms have no access to each other's data. Sync points must be explicit (see [Join behavior](parallel.md#join-behavior)).
* If any arm is executed successfully, the flow execution continues.
* All the data defined in parallel arms is available in the subsequent code.

## Implementation limitation

Parallel execution has some implementation limitations:

* Parallel means independent execution on different peers
* No parallelism when executing a script on a single peer
* No concurrency in services: every service instance does only one job simultaneously.
* Keep services small in terms of computation and memory (WebAssembly limitation)

These limitations might be overcome in future Aqua updates. But for now, plan your application design having this in mind.

## Parallel operations

#### par

`par` syntax is derived from π-calculus notation of parallelism: `A | B`

```haskell
-- foo and bar will be executed in parallel, if possible
foo()
par bar()

-- It's useful to combine `par` with `on` block,
-- to delegate further execution to different peers.

-- In this case execution will continue on two peers, independently
on "peer 1":
  x <- foo()
par on "peer 2":
  y <- bar()

-- Once any of the previous functions return x or y,
-- execution continues. We don't know the order, so 
-- if y is returned first, hello(x) will not execute  
hello(x)
hello(y)  

-- You can fix it with par
-- What's comes faster, will advance the execution flow
hello(x)
par hello(y)
```

`par` works in an infix manner between the previously stated function and the next one.

### co

`co` , short for `coroutine`, prefixes an operation to send it to the background. From π-calculus perspective, it's the same as `A | null`, where `null`-process is the one that does nothing and completes instantly.

```haskell
-- Let's send foo to background and continue
co foo()

-- Do something on another peer, not blocking the flow on this one
co on "some peer":
  baz()

-- This foo does not wait for baz()  
foo()  

-- Assume that foo is executed on another machine
co try:
  x <- foo()
-- bar will not wait for foo to be executed or even launched
bar()
-- bax will wait for foo to complete
-- if foo failed, x will never resolve
-- and bax will never execute
bax(x)
```

## Join behavior

Join means that data was created by different parallel execution flows and then used on a single peer to perform computations. It works the same way for any parallel blocks, be it `par`, `co` or something else (`for par`).

In Aqua, you can refer to previously defined variables. In case of sequential computations, they are available, if execution not failed:

```haskell
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

```haskell
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

Actually, the script will be executed twice: the first time it will be sent from `peer1`, and the second time – from `peer2`. Or another way round: `peer2` then `peer1`, we don't know who is faster.

When execution will get to `baz` for the first time, Aqua VM will realize that it lacks some data that is expected to be computed above in the parallel branch. And halt.

After the second branch executes, VM will be woken up again, reach the same piece of code and realize that now it has enough data to proceed.

This way you can express race (see [Collection types](../types.md#collection-types) and [Conditional return](conditional.md#conditional-return) for other uses of this pattern):

```haskell
-- Initiate a stream to write into it several times
results: *string

on peer1:
  results <- foo()

par on peer2:
  results <- bar()

-- When any result is returned, take the first (the fastest) to proceed
baz(results!)
```

## Explicit join expression

Consider the case when you want to wait for a certain amount of results computed in parallel, and then return.

```python
func collectRespondingPeers(peers: []string, n: i16) -> *string:
  responded: *string
  for p <- peers par:
     responded <<- p
  -- ...
```

How to return no less than `n+1` responsible peers?

{% hint style="warning" %}
Keep in mind that indices start from `0`.

If the expected length of a stream equals `n`, and you wait for element `stream[n]`, your code will hang forever, as it exceeds the length!
{% endhint %}

One way is to use a useless stream:

```python
  useless: *string
  useless <<- responded[n]
  <- responded
```

Actually `useless` stream is useless, we create it just to push the nth element into it. However, it forces waiting for `responded[n`] to be available. When `responded` is returned, it will be at least of length `n+1` or longer.

To eliminate the need for such workarounds, Aqua has the `join` expression that does nothing except consuming its arguments, hence waiting for them:

```python
  join responded[n]
  <- responded
```

You can use any number of arguments to `join`, separating them with a comma. `join` is executed on a particular node – so `join` respects the `on` scopes it's in.

```python
func getTwo(peerA: string, peerB: string) -> string, string:
  co on peerA:
    a <- foo()
  co on peerB:
    b <- foo()
    
  -- Without this join, a and b will still be returned,
  -- But in the join case they are returned by-value,
  -- While without join they are returned by-name
  -- and it could happen that they're not actually available in time.
  join a, b
  <- a, b 
```

## Timeout and race patterns

To limit the execution time of some part of an Aqua script, you can use a pattern that's often called "race". Execute a function in parallel with `Peer.timeout`, and take results from the first one to complete.&#x20;

This way, you're racing your function against `timeout`. If `timeout` is the first one to complete, consider your function "timed out".

`Peer.timeout` is defined in [`aqua-lib`](https://github.com/fluencelabs/aqua-lib/blob/1193236/builtin.aqua#L135).

For this pattern to work, it is important to keep an eye on where exactly the timeout is scheduled and executed. One caveat is that you cannot timeout the unreachable peer by calling a timeout on that peer.

Here's an example of how to put a timeout on peer traversal:

```haskell
-- Peer.timeout comes from the standard library
import "@fluencelabs/aqua-lib/builtin"

func traverse_peers(peers: []string) -> []string:
    -- go through the array of peers and collect acknowledgments
    acks: *string
    for peer <- peers par:
        on peer:
            acks <- Service.long_task()
    
    -- if 10 acks collected or 1 second passed, return acks
    join acks[9]
    par Peer.timeout(1000, "timeout")
    <- acks
```

And here's how to approach error handling when using `Peer.timeout`

```haskell
-- Peer.timeout comes from the standard library
import "@fluencelabs/aqua-lib/builtin"

func getOrNot() -> string:
  status: *string
  res: *string
  -- Move execution to another peer
  on "other peer":
    res <- Srv.someFunction()
    status <<- "ok"
  -- In parallel with the previous on, run timeout on this peer  
  par status <- Peer.timeout(1000, "timeout") 
  
  -- status! waits for the first write to happen
  if status! == "timeout":
    -- Now we know that "other peer" was not able to respond within a second
    -- Do some failover
    res <<- "providing a local failover value"
    
  <- res!  
```
