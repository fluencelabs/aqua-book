# CRDT Streams

In Aqua, ordinary value is a name that points to a single result:

```text
value <- foo()
```

Stream is a name that points to a number of results \(zero or more\):

```text
value: *string
value <- foo()
value <- foo()
```

Stream is a kind of [collection](types.md#collection-types), and can be used where other collections are:

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

But the most powerful uses of streams come along with parallelism, which incurs non-determinism.

### Streams lifecycle and guarantees

Streams lifecycle can be divided into three stages:

* Source: \(Parallel\) Writes to a stream
* Map: Handling the stream values
* Sink: Converting the resulting stream into scalar

Consider the following example:

```text
func foo(peers: []string) -> string:
  resp: *string
  
  -- Will go to all peers in parallel
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

In this case, for each peer in peers, something is going to be written into resp stream.

Every peer p in peers does not know anything about how the other iterations proceed.

Once something is written to resp stream, the second for is triggered. It's the mapping stage.

And then the results are sent to the first peer, to call Op.identity there. This Op.identity waits until element number 5 is defined on resp2 stream.

When it is, stream as a whole is consumed to produce a scalar value, which is returned.

During execution, involved peers have different views on the state of execution: parallel branches of for have no access to each other's data. Finally, execution flows to the initial peer. Initial peer merges writes to the resp stream, and merges writes to the resp2 stream. It's done in conflict-free fashion. More than that, head of resp, resp2 streams will not change from each peer's point of view: it's immutable, and new values are only appended. However, different peers may have different order of the stream values, depending on the order of receiving these values. 

