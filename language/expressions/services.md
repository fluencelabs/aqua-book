# Services

### `service`

Service definition.

A service is a program running on a peer. Every service has an interface that consists of a list of functions. To call a service, the service must be identified: so, every service has an ID that must be resolved in the peer scope.

In the service definition, you enumerate all the functions, their names, argument, and return types, and optionally provide the default Service ID.

Services that are a part of the protocol, i.e. are available from the peer node, come along with IDs. Example of predefined service:

```haskell
service Peer("peer"):
  foo() -- no arguments, no return
  bar(i: bool) -> bool

func usePeer() -> bool:
  Peer.foo() -- results in a call of service "peer", function "foo", on current peer ID
  z <- Peer.bar(true)
  <- z
```

Example of a custom service:

```haskell
service MyService:
  foo()
  bar(i: bool, z: i32) -> string

func useMyService(k: i32) -> string:
  -- Need to tell the compiler what does "my service" mean in this scope
  MyService "my service id"
  MyService.foo()
  on "another peer id":
    -- Need to redefine MyService in scope of this peer as well
    MyService "another service id"
    z <- MyService.bar(false, k)
  <- z
```

Service definitions have types. Type of a service is a product type of arrows. See [Types](../types.md#type-of-a-service-and-a-file).

