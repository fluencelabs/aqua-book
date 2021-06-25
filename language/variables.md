# Values

Aqua is all about combining data and computations. Underlying runtime \([AquaVM](https://github.com/fluencelabs/aquavm)\) tracks what data comes from what origin, which constitutes the foundation for distributed systems security. That approach, driven by π-calculus and security considerations of open-by-default networks and distributed applications as custom application protocols, also puts constraints on the language that configures it.

Namely, values form VDS \(Verifiable Data Structures\). All operations on values must retain security invariants. Hence values are immutable, except [writeable collections](types.md#collection-types) \(streams\).

### Arguments

Function arguments are available within the whole function body.

```text
func foo(arg: i32, log: string -> ()):
  -- Use data arguments
  bar(arg)
  
  -- Arguments can have arrow type and be used as strings
  log("Wrote arg to responses")
```

### Return values

That's the second way to get data with a name.

```text
-- Imagine a Stringify service that's always available
service Stringify("stringify"):
  i32ToStr(arg: i32) -> string

-- Define the result type of a function
func bar(arg: i32) -> string:
  -- Make a value, name it x
  x <- Stringify.i32ToStr(arg)
  -- Starting from there, you can use x
  -- Pass x out of the function scope as the return value
  <- x
  

func foo(arg: i32, log: *string):
  -- Use bar to convert arg to string, push that string
  -- to logs stream, return nothing
  log <- bar(arg)
```

### Literals

Aqua supports just a few literals: numbers, quoted strings, booleans. You [cannot make a structure](https://github.com/fluencelabs/aqua/issues/167) in Aqua.

```text
-- String literals cannot contain double quotes
foo("double quoted string literal")

-- Booleans are true and false
if x == false:
  foo("false is a literal")
  
-- Numbers are different
-- Any number:
bar(1)  

-- Signed number:
bar(-1)

-- Float:
bar(-0.2)
```

### Getters

In Aqua, you can use a getter to peak into a field of a product, or indexed element in an array. 

```text
data Sub:
  sub: string

data Example:
  field: u32
  arr: []Sub
  child: Sub
  
func foo(e: Example):
  bar(e.field) -- u32
  bar(e.child) -- Sub
  bar(e.child.sub) -- string
  bar(e.arr) -- []Sub
  bar(e.arr!) -- gets the 0 element
  bar(e.arr!.sub) -- string
  bar(e.arr!2) -- gets the 2nd element
  bar(e.arr!2.sub) -- string   
```

Note that `!` operator may fail or halt:

* If it is called on an immutable collection, it will fail if collection is shorter and has no given index; you can handle it with [try](operators/conditional.md#try) or [otherwise](operators/conditional.md#otherwise).
* If it is called on an appendable stream, it will wait for some parallel append operation to fulfill, see [Join behavior](operators/parallel.md#join-behavior).

`!` operator cannot be used with non-literal indices: you can use `!2`, but not `!x`. It's planned to be fixed.

### Assignments

Assignments do nothing new, just gives a name to a value with applied getter, or name a literal.

```text
func foo(arg: bool, e: Example):
  -- Rename the argument
  a = arg
  -- Assign the name b to value of e.child
  b = e.child
  -- Create a literal
  c = "just string value"
```

### Constants

Constants are like assignments, but in the root scope. Can be used in all function bodies, textually below the place of const definition.

You can change the compilation results with overriding a constant. Override should be of the same type \(or a subtype\).

Constant values must resolve to a literal.

```text
-- This flag is always true
const flag = true

-- This setting can be overwritten via CLI flag
const setting ?= "value"

func foo(arg: string): ...

func bar():
  -- Type of setting is string
  foo(setting)
```

### Visibility scopes

Visibility scopes follows the contracts of execution flow.

### Streams as literals

Stream is a special data that allows many writes. It worth a dedicated article.

Here we just explain how to initiate streams and use them as empty collection literals. Refer to conditional return.

