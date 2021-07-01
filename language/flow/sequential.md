# Sequential

By default, Aqua code is executed line by line, sequentially.

### Contract

* Data from the first arm is available in the second branch.
* The second arm is executed if and only if the first arm succeeded.
* If any arm failed, then the whole sequence is failed.
* If all arms executed successfully, then the whole sequence is executed successfully.

### Sequential operations

#### call arrow

Any runnable piece of code in Aqua is an arrow from its domain to the codomain.

```haskell
-- Call a function
foo()

-- Call a function that returns smth, assign results to a variable
x <- foo()

-- Call an ability function
y <- Peer.identify()

-- Pass an argument
z <- Op.identity(y)
```

When you write `<-`, this means not just "assign results of the function on the right to variable on the left". It means that all the effects are executed: [service](../abilities-and-services.md) may change state, the [topology](../topology.md) may be shifted. But you end up being \(semantically\) on the same peer where you have called the arrow.

#### on

`on` denotes the peer where the code must be executed. `on` is handled sequentially, and the code inside is executed line by line by default.

```haskell
func foo():
  -- Will be executed where `foo` was executed
  bar()
  
  -- Move to another peer
  on another_peer:
    -- To call bar, we need to leave the peer where we were and get to another_peer
    -- It's done automagically
    bar()
    
  on third_peer via relay:
    -- This is executed on third_peer
    -- But we denote that to get to third_peer and to leave third_peer
    -- an additional hop is needed: get to relay, then to peer
    bar()
 
  -- Will be executed in the `foo` call site again
  -- To get from the previous `bar`, compiler will add a hop to relay
  bar()
```

See more in the [Topology](../topology.md) section.

