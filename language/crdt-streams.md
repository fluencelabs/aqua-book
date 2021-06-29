# CRDT Streams

In Aqua, an ordinary value is a name that points to a single result:

```text
value <- foo()
```

A stream , on the other hand, is a name that points to zero or more results:

```text
value: *string
value <- foo()
value <- foo()
```

Stream is a kind of [collection](types.md#collection-types) and can be used in place of other collections:

```text
func foo(peer: string, relay: ?string):
  on peer via relay:
    Op.noop()
    
-- Dirty hack for lack of type variance, and lack of cofunctors    
service OpStr("op"):
  identity: string -> string    
    
func bar(peer: string, relay: string):
  relayMaybe: *string
  if peer != %init_peer_id%:
    -- To write into a stream, function call is required
    relayMaybe <- OpStr.identity(relay)
  -- Pass a stream as an optional value  
  foo(peer, relayMaybe)  
```

But the most powerful use of streams pertains to their use with parallel execution, which incurs non-determinism.

### Streams: Lifecycle And Guarantees

A stream's lifecycle can be separated into three stages:

* Source: \(Parallel\) Writes to a stream
* Map: Handling the stream values
* Sink: Converting the resulting stream into a scalar

Consider the following example:

```text
func foo(peers: []string) -> string:
  resp: *string
  
  -- Go to all peers in parallel
  for p <- peers par:
    on p:
      -- Do something
      resp <- Srv.call()
      
  resp2: *string    
      
  -- What is resp at this point?
  for r <- resp par:
    on r:
      resp2 <- Srv.call()
          
  -- Wait for 6 responses        
  Op.identity(resp2!5)
  -- Once we have 5 responses, merge them
  r <- Srv.concat(resp2)
  <- r
    
```

In this case, for each peer in peers, something is going to be written into `resp` stream.

Every peer `p` in peers does not know anything about how the other iterations proceed.

Once something is written to `resp` stream, the second for is triggered. This is the mapping stage.

And then the results are sent to the first peer, to call Op.identity there. This Op.identity waits until element number 5 is defined on `resp2` stream.

When the join is complete, the stream is consumed by the concatenation service to produce a scalar value, which is returned.

During execution, involved peers have different views on the state of execution: each of the `for` parallel branches have no view or access to the other branches' data and eventually, the execution flows to the initial peer. The initial peer then merges writes to the `resp` stream and to the `resp2` stream, respectively. These writes are done in conflict-free fashion. Furthermore,  the respective heads of the `resp`, `resp2` streams will not change from each peer's point of view as they are immutable and new values can only be appended. However, different peers may have a different order of the stream values depending on the order of receiving these values.

