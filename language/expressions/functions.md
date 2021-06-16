# Functions

Function in Aqua is a block of code expressing a workflow, a coordination scenario that works across many peers.

A function may take arguments of any type and may return a value.

A function can call other functions, take functions as arguments of [arrow type](../types.md#arrow-types), be provided as an arrow argument.

Essentially, a function is an arrow. The function call is an expression that connects named arguments to an arrow, and gives a name to the result.

Finally, all a function does is call its arguments or service functions.

```text
service MySrv:
  foo()

func do_something(): -- arrow of type: -> ()
  MySrv "srv id"
  MySrv.foo()  
```

* list all expressions
* for each, explain the contract and provide a use case

