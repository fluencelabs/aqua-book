# Basics

Aqua is an opinionated domain-specific language. It's structured with significant indentation.

```text
-- Comments begin with double-dash and end with the line (inline)
func foo(): -- Comments are allowed almost everywhere
  -- Body of the block expression is indented
  bar(5)
```

Values in Aqua have types, which are designated by a colon, `:`,  as seen in function signature below. The type of a return, which is yielded when a function is executed, is denoted by an arrow pointing to the right `->` , whereas yielding is denoted by an arrow pointing to the left `<-`.

```text
-- Define a function that yields a string
func bar(arg: i16) -> string:
  -- Call a function
  smth(arg)
  
  -- Yield a value from a function
  x <- smth(arg)
  
  -- Return a yielded results from a function
  <- "return literal"
```

Subsequent sections explain the main parts of Aqua.

Data:

* [Types](types.md)
* [Values of that types](variables.md)

Execution:

* [Topology](topology.md) – how to express where the code should be executed
* [Execution flow](flow/) – control structures

Computations:

* [Abilities & Services](abilities-and-services.md)

Advanced parallelism:

* [CRDT Streams](crdt-streams.md)

Code management:

* [Imports & exports](statements-1.md)

Reference:

* [Expressions](expressions/)



