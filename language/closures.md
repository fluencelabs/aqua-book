# Closures

In Aqua, you can create an arrow within the function, enclosing its context.

```python
service Hello:
  say_hello(to_name: string, peer: string)

func bar(callback: string -> ()):
   callback("Fish")

func foo(peer: string):
  on peer:
    -- Capture service resolution
    Hello "world"
    -- Create a closure named "closure"
    closure = (name: string) -> string:
      -- Use a value that's available on the definition site
      -- To call a service that's resolved on the definition site
      Hello.say_hello(name, peer)
      -- Return a value from the closure; syntax is the same as in functions
      <- name
  -- Pass this closure to another function
  bar(closure)
```

Closures can be created anywhere in the function, starting with Aqua 0.4.1, and then used just like any other arrow (argument of arrow type, function, or service method): passed to another function as an argument, or called right there.

Closures enclose over three domains:

* Values in scope,
* Service resolutions,
* Topology: place where the closure is defined should be the place where it's executed.

Comparing with functions, closures have one important difference: functions are detached from topology by default. `func` keyword can be used to bring this behavior to closures, if needed.

```python
service Hello:
  say_hello()

func foo():
  on HOST_PEER_ID:
    Hello "hello"
    
    -- This closure will execute on HOST_PEER_ID
    closure = () -> ():
      Hello.say_hello() 
      
    fn = func () -> ():
      Hello.say_hello()
      
  -- Will go to HOST_PEER_ID, where Hello service is resolved, and call say_hello     
  closure()        
  
  -- Will be called on current peer, probably INIT_PEER_ID, and may fail
  -- in case Hello service is not defined or has another ID
  fn()
```

It is not yet possible to return an arrow from an arrow.
