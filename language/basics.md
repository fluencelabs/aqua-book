# Basics

Aqua is an opinionated domain-specific language. It's structured with significant indentation.

```text
-- Comments begin with double-dash, ends on the end of line
func foo(): -- Comments can follow the code
  -- Body of the block expression is indented
  bar(5)
```

Values in Aqua have types. Types of values are designated by a colon `:`, like in the function arguments definition below. The type of return \(yielded when a function is executed\) is denoted by an arrow pointing to the right `->`.

Yielding is denoted by an arrow pointing to the left `<-`.

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



