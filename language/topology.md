---
description: Define where the code is to be executed and how to get there
---

# Topology


Aqua lets developers to describe the whole distributed workflow in a single script, link data, recover from errors, implement complex patterns like backpressure, and more. Hence,  topology is at the heart of Aqua. 

Topology in Aqua is declarative:  You just need to say where a piece of code must be executed, on what peer, and optionally how to get there. he Aqua compiler will add all the required network hops.

### On expression

`on` expression moves execution to the specified peer:

```haskell
on "my peer":
  foo()
```

Here, `foo` is instructed to be executed on a peer with id `my peer`. `on` supports variables of type `string` :

```haskell
-- foo, bar, baz are instructed to be executed on myPeer
on myPeer:
  foo()
  bar()
  baz()
```

### `%init_peer_id%`

There is one custom peer ID that is always in scope: `%init_peer_id%`. It points to the peer that initiated this request. 

{% hint style="warning" %}
Using `on %init_peer_id%` is an anti-pattern: There is no way to ensure that init peer is accessible from the currently used part of the network.
{% endhint %}

### More complex scenarios

Consider this example:

```go
func foo():
  on "peer foo":
    do_foo()
    
func bar(i: i32):
  do_bar()
  
func baz():
  bar(1)
  on "peer baz":
    foo()
    bar(2)
  bar(3)            
```

Take a minute to think about:

* Where `do_foo` is executed?
* Where `bar(1)` is executed?
* On what node `bar(2)` runs?
* What about `bar(3)`?

Declarative topology definition always works the same way.

* `do_foo` is executed on "peer foo", always.
* `bar(1)` is executed on the same node where `baz` was running. If `baz` is the first called function, then it's `init peer id`.
* `bar(2)` is executed on `"peer baz"`, despite the fact that foo does topologic transition. `bar(2)` is in the scope of `on "peer baz"`, so it will be executed there
* `bar(3)` is executed where `bar(1)` was: in the root scope of `baz`, wherever it was called from

### Accessing peers `via` other peers

In a distributed network it is quite common that a peer is not directly accessible. For example, a browser has no public network interface and you cannot open a socket to a browser at will. Such constraints warrant a `relay` pattern: there should be a well-connected peer that relays requests from a peer to the network and vice versa.

Relays are handled with `via`:

```haskell
-- When we go to some peer from some other peer,
-- the compiler will add an additional hop to some relay
on "some peer" via "some relay":
  foo()
  
-- More complex path: first go to relay2, then to relay1,
-- then to peer. When going out of peer, do it in reverse  
on "peer" via relay1 via relay2:
  foo()
  
-- You can pass any collection of strings to relay,
-- and it will go through it if it's defined,
-- or directly if not  
func doViaRelayMaybe(peer: string, relayMaybe: ?string):
  on peer via relayMaybe:
    foo()
```

`on`s nested or delegated in functions work just as you expect:

```haskell
-- From where we are, -> relay1 -> peer1
on "peer1" via "relay1":
  -- On peer1
  foo()
  -- now go -> relay1 -> relay2 -> peer2
  -- going to relay1 to exit peer1
  -- going to relay2 to enable access to peer2
  on "peer2" via "relay2":
    -- On peer2
    foo()
-- This is executed in the root scope, after we were on peer2
-- How to get there?
-- Compiler knows the path that just worked
-- So it goes -> relay2 -> relay1 -> (where we were)
foo()
```

With `on` and `on ... via`, significant indentation changes the place where the code will be executed, and paths that are taken when execution flow "bubbles up" \(see the last call of `foo`\). It's more efficient to keep the flow as flat as it could. Consider the following change of indentation in the previous script, and how it affects execution:

```haskell
-- From where we are, -> relay1 -> peer1
on "peer1" via "relay1":
  -- On peer1
  foo()
-- now go -> relay1 -> relay2 -> peer2
-- going to relay1 to exit peer1
-- going to relay2 to enable access to peer2
on "peer2" via "relay2":
  -- On peer2
  foo()
-- This is executed in the root scope, after we were on peer2
-- How to get there?
-- Compiler knows the path that just worked
-- So it goes -> relay2 -> (where we were)
foo()
```

When the `on` scope is ended, it does not affect any further topology moves. Until you stop indentation, `on` affects the topology and may add additional topology moves, which means more roundtrips and unnecessary latency.

### Callbacks

What if you want to return something to the initial peer? For example, implement a request-response pattern. Or send a bunch of requests to different peers, and render responses as they come, in any order.

This can be done with callback arguments in the entry function:

```go
func run(updateModel: Model -> (), logMessage: string -> ()):
  on "some peer":
    m <- fetchModel()
    updateModel(m)
  par on "other peer":
    x <- getMessage()
    logMessage(x)  
```

Callbacks have the [arrow type](types.md#arrow-types).

If you pass just ordinary functions as arrow-type arguments, they will work as if you hardcode them.

```haskell
func foo():
  on "peer 1":
    doFoo()
    
func bar(cb: -> ()):
  on "peer2":
    cb()
    
func baz():
  -- foo will go to peer 1
  -- bar will go to peer 2
  bar(foo)            
```

If you pass a service call as a callback, it will be executed locally on the node where you called it. That might change.

Functions that capture the topologic context of the definition site are planned, not yet there. **Proposed** syntax:

```text
func baz():
  foo = do (x: u32):
    -- Executed there, where foo is called
    Srv.call(x)
    <- x
  -- When foo is called, it will get back to this context
  bar(foo)
```

{% embed url="https://github.com/fluencelabs/aqua/issues/183" caption="Issue for adding \`do\` expression" %}

{% hint style="warning" %}
Passing service function calls as arguments is very fragile as it does not track that a service is resolved in the scope of the call. Abilities variance may fix that.
{% endhint %}

### Parallel execution and topology

When blocks are executed in parallel, it is not always necessary to resolve the topology to get to the next peer. The compiler will add topologic hops from the par branch only if data defined in that branch is used down the flow.

{% hint style="danger" %}
What if all branches do not return? Execution will halt. Be careful, use `co` if you don't care about the returned data.
{% endhint %}



