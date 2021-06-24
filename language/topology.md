---
description: Define where the code is to be executed & how to get there
---

# Topology

In Aqua, topology is the major thing. Aqua lets developers to describe the whole distributed workflow in a single script, link data, recover from errors, implement complex patterns like backpressure, and more.

Topology in Aqua is made declarative way. You need just to say where a piece of code must be executed, on what peer, and optionally – how to get there. Compiler will add all the required network hops.

### On expression

`on` expression moves execution to the peer:

```text
on "my peer":
  foo()
```

Foo is instructed to be executed on a peer with id `my peer`. `on` supports variables of type `string` :

```text
-- Foo, bar, baz are instructed to be executed on myPeer
on myPeer:
  foo()
  bar()
  baz()
```

### `%init_peer_id%`

There's one custom peer ID that's always available in scope: `%init_peer_id%`. It points to the peer who initiated this request. But using `on %init_peer_id%` is an anti-pattern: there's no way to ensure that init peer is accessible from the part of the network you're currently in.

More complex scenarios

Consider this example:

```text
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

Declarative topology definition always work the same way.

* `dofoo` is executed on "peer foo", always.
* `bar(1)` is executed on the same node where `baz` was running. If `baz` is the first called function, then it's `init peer id`.
* `bar(2)` is executed on `"peer baz"`, despite the fact that foo does topologic transition. `bar(2)` is in the scope of `"peer baz"`, so it will be executed there
* `bar(3)` is executed where `bar(1)` was: in the root scope of `baz`, wherever it was called

### Accessing peers `via` other peers

In the distributed network it's a very common situation when a peer is only accessible indirectly. For example, a browser: it has no public network interface, you cannot open a socket to a browser at will. This leads to a `relay` pattern: there should be a well connected peer that relays requests from a peer to the network and vice versa.

Relays are handled with `via`:

```text
-- When we go to some peer and from some peer,
-- compiler will add an additional hop to some relay
on "some peer" via "some relay":
  foo()
  
-- More complex path: first go to relay2, then to relay1,
-- then to peer. When going out of peer, do it reversed way  
on "peer" via relay1 via relay2:
  foo()
  
-- You can pass any collection of strings to relay,
-- and it will go through it if it's defined,
-- or directly if not  
func doViaRelayMaybe(peer: string, relayMaybe: ?string):
  on peer via relayMaybe:
    foo()
```

Nested `on`s, or delegated in functions, should work just as you expect:

```text
-- From where we are, -> relay1 -> peer1
on "peer1" via "relay1":
  -- On peer1
  foo()
  -- now go -> relay1 -> relay2 -> peer2
  on "peer2" via "relay2":
    -- On peer2
    foo()
-- This is executed in the root scope, after we were on peer2
-- How to get there?
-- Compiler knows the path that just worked
-- So it goes -> relay2 -> relay1 -> (where we were)
foo()
```

### Callbacks

What if you want to return something to the initial peer? For example, send the request to a bunch of services, and then render the responses as they come.

This can be done with callback arguments in the entry function:

```text
func run(updateModel: Model -> (), logMessage: string -> ()):
  on "some peer":
    m <- fetchModel()
    updateModel(m)
  par on "other peer":
    x <- getMessage()
    logMessage(x)  
```

Callbacks has the [arrow type](types.md#arrow-types).

If you pass just ordinary functions as arrow-type arguments, they will work as is you hardcode them.

```text
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

If you pass service call as a callback, it will be executed locally to the node where you called it. That might change.

Lambda functions that capture topologic context of definition site are planned, not yet there.

{% hint style="warning" %}
Passing service function calls as arguments are very fragile, as it does not track that a service is resolved in the scope of calling. Abilities variance may fix that.
{% endhint %}

### Parallel execution and topology

When blocks are executed in parallel, it's not always necessary to resolve topology to get to the next peer. Compiler will add topologic hops from the par branch only if data defined in that branch is used down the flow.

{% hint style="danger" %}
What if all branches do not return? Execution will halt. Be careful, use `co` if you don't care about the returned data.
{% endhint %}



