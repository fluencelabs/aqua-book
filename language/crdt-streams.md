# CRDT Streams

In Aqua, an ordinary value is a name that points to a single result:

```haskell
value <- foo()
```

A stream, on the other hand, is a name that points to zero or more results:

```haskell
values: *string

-- Write to a stream several times
values <- foo()
values <- foo()

-- A value can be pushed to a stream 
-- without an explicit function call with <<- operator:
values <<- "foo"
x <- foo()
values <<- x
```

Stream is a kind of [collection](types.md#collection-types) and can be used in place of other collections:

```haskell
func foo(peer: string, relay: ?string):
  on peer via relay:
    Op.noop() 

func bar(peer: string, relay: string):
  relayMaybe: *string
  if peer != %init_peer_id%:
    -- Wwrite into a stream
    relayMaybe <<- relay
  -- Pass a stream as an optional value  
  foo(peer, relayMaybe)
```

But the most powerful use of streams pertains to their use with parallel execution, which incurs non-determinism.

## Streams: Lifecycle And Guarantees

A stream's lifecycle can be separated into three stages:

* Source: (Parallel) Writes to a stream
* Map: Handles the stream values
* Sink: Converts the resulting stream into a scalar

Consider the following example:

```haskell
alias PeerId: string

func foo(peers: []PeerId) -> string:
  -- Store a list of peer IDs collected from somewhere
  -- This is a stream (denoted by *, which means "0 or more values")
  resp: *PeerId

  -- Will go to all peers in parallel
  for p <- peers par:
    -- Move execution flow to the peer p
    on p:
      -- Get a peer ID from a service call (called on p)
      resp <- Srv.call()

  -- You can think of resp2 as a locally consistent lazy list
  resp2: *PeerId    

  -- What is the value of resp at this point?
  -- Keep an eye on the `par` there: actually, it's FORKing execution
  -- to several branches on different peers.
  for r <- resp par:
    -- Move execution to peer r
    on r:
      -- Call Srv locally
      resp2 <- Srv.call()

  -- Wait for 6 responses on resp2: it's JOIN       
  Op.identity(resp2!5)
  -- Once we have 5 responses, merge them
  -- Function treats resp2 as an array of strings, and concatenates all
  -- of them into a single string.
  -- This function call "fixes" the content of resp2, making a single observation.
  -- This is a "stream canonicalization" event: values, order, and length
  -- is fixed at the moment of first function call, function will not be called
  -- again, with different data.
  r <- Srv.concat(resp2)
  -- You can keep writing to a stream after it's value is used
  
  <- r
```

In this case, for each peer `p` in `peers`, a new `PeerID` is going to be obtained from the `Srv.call`  and written into the `resp` stream.

Every peer `p` in peers does not know anything about how the other iterations proceed.

Once `PeerId` is written to the `resp` stream, the second `for` is triggered. This is the mapping stage.

And then the results are sent to the first peer, to call Op.identity there. This Op.identity waits until element number 5 is defined on `resp2` stream.

When the join is complete, the stream is consumed by the concatenation service to produce a scalar value, which is returned.

During execution, involved peers have different views on the state of execution: each of the `for` parallel branches has no view or access to the other branches' data and eventually, the execution flows to the initial peer. The initial peer then merges writes to the `resp` stream and to the `resp2` stream, respectively. These writes are done in a conflict-free fashion. Furthermore, the respective heads of the `resp`, `resp2` streams will not change from each peer's point of view as they are immutable and new values can only be appended. However, different peers may have a different order of the stream values depending on the order of receiving these values.

### Stream restrictions

Restriction is a part of π calculus that bounds (restricts) a name to a scope. For Aqua streams it means that the stream is not accessible outside of definition scope, and the stream is always fresh when execution enters the scope.

These behaviors are introduced in [Aqua 0.5](https://github.com/fluencelabs/aqua/releases/tag/0.5.0).

```python
-- Note: returns []string (immutable, readonly collection), not *string
func smth(xs: []string) -> []string:
  -- This stream is available within the function, and is empty
  outside: *string
  for x <- xs:
    -- This stream is empty at the beginning of each iteration
    inside: *string
    
    -- We can manipulate the inside stream within the scope
    if x == "ok"
      inside <<- "ok"
    else:
      inside <<- "not ok"
    
    -- Read the value    
    if inside! == "ok":
      outside <<- "result"  
  
  -- outside stream is not escaping this function scope:
  -- it is converted to an array (canonicalized) and cannot be appended after that        
  <- outside    
```

You still can keep streams as streams by using them as `*string` arguments, or by returning them as `*string`.&#x20;
