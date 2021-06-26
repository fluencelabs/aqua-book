# Values

Aqua is all about combining data and computations. The runtime for the compiled Aqua code, [AquaVM](https://github.com/fluencelabs/aquavm), tracks what data comes from what origin, which constitutes the foundation for distributed systems security. That approach, driven by π-calculus and security considerations of open-by-default networks and distributed applications as custom application protocols, also puts constraints on the language that configures it.

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

-- Booleans are true or false
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

Note that the `!` operator may fail or halt:

* If it is called on an immutable collection, it will fail if the collection is shorter and has no given index; you can handle it with [try](operators/conditional.md#try) or [otherwise](operators/conditional.md#otherwise).
* If it is called on an appendable stream, it will wait for some parallel append operation to fulfill, see [Join behavior](operators/parallel.md#join-behavior).

{% hint style="warning" %}
The `!` operator can currently only be used with literal indices.  
That is,`!2` is valid but`!x` is not valid.  
We expect to address this limitation soon.
{% endhint %}

### Assignments

Assignments, `=`, only give a name to a value with applied getter or to a literal.

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

Constants are like assignments but in the root scope. They can be used in all function bodies, textually below the place of const definition. Constant values must resolve to a literal.

You can change the compilation results with overriding a constant but the  override needs to be of the same type or subtype.

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

By default, everything defined textually above is available below. With some exceptions.

Functions have isolated scopes:

```text
func foo():
   a = 5
   
func bar():
   -- a is not defined in this function scope
   a = 7   
   foo() -- a inside fo is 5
```

[For loop](operators/iterative.md#export-data-from-for) does not export anything from it:

```text
func foo():
  x = 5
  for y <- ys:
    -- Can use what was defined above
    z <- bar(x)
    
  -- z is not defined in scope  
  z = 7  
```

[Parallel](operators/parallel.md#join-behavior) branches have [no access](https://github.com/fluencelabs/aqua/issues/90) to each other's data:

```text
-- This will deadlock, as foo branch of execution will
-- never send x to a parallel bar branch
x <- foo()
par y <- bar(x)

-- After par is executed, all the can be used
baz(x, y)
```

Recovery branches in [conditional flow](operators/conditional.md) have no access to the main branch as the main branch exports values, whereas the recovery branch does not:

```text
try:
  x <- foo()
otherwise:
  -- this is not possible – will fail
  bar(x)  
  y <- baz()
  
-- y is not available below  
willFail(y)  
  
```

### Streams as literals

Stream is a special data structure that allows many writes. It has [a dedicated article](crdt-streams.md).

To use a stream, you need to initiate it at first:

```text
-- Initiate an (empty) appendable collection of strings
resp: *string

-- Write strings to resp in parallel
resp <- foo()
par resp <- bar()

for x <- xs:
  -- Write to a stream that's defined above
  resp <- baz()
  
try:
  resp <- baz()
otherwise:
  on "other peer":
    resp <- baz()
    
-- Now resp can be used in place of arrays and optional values
-- assume fn: []string -> ()
fn(resp) 

-- Can call fn with empty stream: you can use it
-- to construct empty values of any collection types
nilString: *string
fn(nilString)                      
```

One of the most frequently used patterns for streams is [Conditional return](operators/conditional.md#conditional-return).

