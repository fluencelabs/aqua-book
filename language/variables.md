# Values

Aqua is all about combining data and computations. The runtime for the compiled Aqua code, [AquaVM](https://github.com/fluencelabs/aquavm), tracks what data comes from what origin, which constitutes the foundation for distributed systems security. That approach, driven by π-calculus and security considerations of open-by-default networks and distributed applications as custom application protocols, also puts constraints on the language that configures it.

Values in Aqua are backed by VDS (Verifiable Data Structures) in the runtime. All operations on values must keep the authenticity of data, prooved by signatures under the hood.

That's why values are immutable. Changing the value effectively makes a new one:

```haskell
x = "hello"
y = "world"

-- despite the sources of x and y, z's origin is "peer 1"
-- and we can trust value of z as much as we trust "peer 1"
on "peer 1":
  z <- concat(x, y)
```

More on that in the Security section. Now let's see how we can work with values inside the language.

## Arguments

Function arguments are available within the whole function body.

```haskell
func foo(arg: i32, log: string -> ()):
  -- Use data arguments
  bar(arg)

  -- Arguments can have arrow type and be used as strings
  log("Wrote arg to responses")
```

## Return values

You can assign the results of an arrow call to a name and use this returned value in the code below.

```haskell
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

Aqua functions may return more than one value.

```python
-- Define return types as a comma separated list
func myFunc() -> bool, string:
  -- Function must return values for all defined types
  <- true, "successful execution"
  
func otherFunc():
  -- Call a function, don't use returns
  myFunc()  
  -- Get any number of results out of the function
  flag <- myFunc()
  
  -- Push results to a stream
  results: *string
  is_ok, results <- myFunc()
  if is_ok:
    -- We know that it contains successful result
    foo(results!)
```

## Literals

Aqua supports just a few literals: numbers, quoted strings, booleans, and `nil`. You [cannot init a structure](https://github.com/fluencelabs/aqua/issues/167) in Aqua, only obtain it as a result of a function call.

```haskell
-- String literals cannot contain double quotes
-- No single-quoted strings allowed, no escape chars.
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

func takesMaybe(arg: ?string): ...

-- nil can be passed in every place
-- where a read-only collection fits
takesMaybe(nil)
```

## Arithmetic operators

Aqua offers a list of arithmetic and logic operators, introduced in Aqua 0.7.1.

```python
-- Addition
func foo(a: i32, b: i32) -> i32:
  c = a + b
  d = c - a
  e = d * c
  f = e / d
  -- power 3: unsigned number expected
  g = f ** 3
  -- remainder
  e = g % f
  
  -- Can use arithmetics anywhere
  -- Better use brackets to enforce ordering
  <- (a + b) - (c + d * (e - 6 ))
```

## Collections

With Aqua it is possible to create a [stream](crdt-streams.md), fill it with values, and use in place of any collection:

```haskell
-- foo returns an array
func foo() -> []string:
  -- Initiate a typed stream
  ret: *string
  
  -- Push values into the stream
  ret <<- "first"
  ret <<- "second"
  
  -- Return the stream in place of the array
  <- ret
```

Aqua provides syntax sugar for creating any of the collection types with `[ ... ]` for arrays, `?[ ... ]` for optional values, `*[ ... ]` for streams.

```haskell
func foo() -> []string, ?bool, *u32:
  <- ["string1", "string2"], ?[true, false], *[1, 3, 5]
```

The `?[]` expression takes any number of arguments, but returns an optional value that contains only `0` or `1` value. This is done by trying to yield these values one by one. The first value that yields without an error will be added to the resulting option.

```haskell
func getFlag(maybeFlagA: ?bool, maybeFlagB: ?bool, default: bool) -> bool:
  res = ?[maybeFlagA!, maybeFlagB!, default]
  <- res!
```

As of Aqua `0.6.3`, it is not possible to get an element by index directly from the collection creation expression.

## Getters

In Aqua, you can use a getter to peek into a field of a product or indexed element in an array.

```haskell
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
  bar(e.arr[2]) -- gets the 2nd element
  bar(e.arr[2].sub) -- string
  bar(e.arr[e.field]) -- can use any scalar as index with [] syntax
```

Note that the `!` operator may fail or halt:

* If it is called on an immutable collection, it will fail if the collection is shorter and has no given index; you can handle the error with [try](https://github.com/fluencelabs/aqua-book/tree/4177e00f9313f0e1eb0a60015e1c19a956c065bd/language/operators/conditional.md#try) or [otherwise](https://github.com/fluencelabs/aqua-book/tree/4177e00f9313f0e1eb0a60015e1c19a956c065bd/language/operators/conditional.md#otherwise).
* If it is called on an appendable stream, it will wait for some parallel append operation to fulfill, see [Join behavior](https://github.com/fluencelabs/aqua-book/tree/4177e00f9313f0e1eb0a60015e1c19a956c065bd/language/operators/parallel.md#join-behavior).

{% hint style="warning" %}
The `!` operator can currently only be used with literal indices.\
That is,`!2` is valid but`!x` is not valid.\
To access an index with non-literal, use the brackets index, like \[x].
{% endhint %}

## Assignments

Assignments, `=`, only give a name to a value with an applied getter or to a literal.

```haskell
func foo(arg: bool, e: Example):
  -- Rename the argument
  a = arg
  -- Assign the name b to value of e.child
  b = e.child
  -- Create a named literal
  c = "just string value"
```

## Constants

Constants are like assignments but in the root scope. They can be used in all function bodies, textually below the place of const definition. Constant values must resolve to a literal.

You can change the compilation results by overriding a constant but the override needs to be of the same type or subtype.

Constants are always `UPPER_CASE`.

```haskell
-- This FLAG is always true
const FLAG = true

-- This SETTING can be overwritten via CLI flag
const SETTING ?= "value"

func foo(arg: string): ...

func bar():
  -- Type of SETTING is string
  foo(SETTING)
```

## Visibility scopes

Visibility scopes follow the contracts of execution flow.

By default, everything defined textually above is available below. With some exceptions.

Functions have isolated scopes:

```haskell
func foo():
   a = 5

func bar():
   -- a is not defined in this function scope
   a = 7   
   foo() -- a inside fo is 5
```

[For loop](flow/iterative.md#export-data-from-for) does not export anything from it:

```haskell
func foo():
  x = 5
  for y <- ys:
    -- Can use what was defined above
    z <- bar(x)

  -- z is not defined in scope  
  z = 7
```

[Parallel](flow/parallel.md#join-behavior) branches have [no access](https://github.com/fluencelabs/aqua/issues/90) to each other's data:

```haskell
-- This will deadlock, as foo branch of execution will
-- never send x to a parallel bar branch
x <- foo()
par y <- bar(x)

-- After par is executed, all the can be used
baz(x, y)
```

Recovery branches in [conditional flow](https://github.com/fluencelabs/aqua-book/tree/4177e00f9313f0e1eb0a60015e1c19a956c065bd/language/operators/conditional.md) have no access to the main branch as the main branch exports values, whereas the recovery branch does not:

```haskell
try:
  x <- foo()
otherwise:
  -- this is not possible – will fail
  bar(x)  
  y <- baz()

-- y is not available below  
willFail(y)
```

## Streams as literals

Stream is a special data structure that allows many writes. It has [a dedicated article](crdt-streams.md).

To use a stream, you need to initiate it at first:

```haskell
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

One of the most frequently used patterns for streams is [Conditional return](flow/conditional.md#conditional-return).

You can create a stream with [Collection creation](variables.md#collections) operators.
