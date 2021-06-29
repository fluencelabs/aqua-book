# Abilities & Services

While Execution flow organizes the flow from peer to peer, Abilities & Services describe what exactly can be called on these peers, and how to call it.

Ability is a concept of "what is possible in this context": like a peer-specific trait or a typeclass. It will be better explained once abilities passing is implemented.

{% embed url="https://github.com/fluencelabs/aqua/issues/33" %}

### Services

A Service interfaces functions \(often WASM ones\) executable on a peer. Example of service definition:

```text
service MyService:
  foo(arg: string) -> string
  bar() -> bool
  baz(arg: i32)
```

Service functions in Aqua have no function body. Computations, of any complexity, are implemented with any programming language that fits, and then brought to the Aqua execution context. Aqua calls these functions but does not peak into what's going on inside.

#### Built-in Services

Some services may be singletons available on all peers. Such services are called built-ins, and are always available in any scope.

```text
-- Built-in service has a constant ID, so it's always resolved
service Op("op"):
  noop()
  
func foo():
  -- Call the noop function of "op" service locally
  Op.noop()  
```

#### Service Resolution

A peer may host many services of the same type. To distinguish services from each other, Aqua requires Service resolution to be done: that means, the developer must provide an ID of the service to be used on the peer.

```text
service MyService:
  noop()
  
func foo():
  -- Will fail
  MyService.noop()
  
  -- Resolve MyService: it has id "noop"
  MyService "noop"
  
  -- Can use it now 
  MyService.noop()
  
  on "other peer":
    -- Should fail: we haven't resolved MyService ID on other peer
    MyService.noop()
    
    -- Resolve MyService on peer "other peer"
    MyService "other noop"
    MyService.noop()
  
  -- Moved back to initial peer, here MyService is resolved to "noop"
  MyService.noop()
```

There's no way to call an external function in Aqua without defining all the data types and the service type. One of the most convenient ways to do it is to generate Aqua types from Wasm code in Marine. 

