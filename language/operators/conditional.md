# Conditional

Aqua supports branching: you can return one value or another, recover from the error, or check a boolean expression.

### Contract

Second branch is executed iff the first branch failed.

Second branch has no access to the first branch's data.

Block is considered executed iff any branch was executed successfully.

Block is considered failed iff the second branch fails to execute.

### Conditional operations

#### try

Tries to perform operations or consumes the error if there's no catch or otherwise statement after the try block.

```text
try:
   -- If foo fails with an error, execution will continue
   -- You should write your logic in a non-blocking fashion:
   -- If your code below depends on `x`, it may halt as `x` is not resolved.
   -- See Conditional return below for workaround
   x <- foo()
```

#### catch

Catches the standard error from `try` block.

```text
try:
   foo()
catch e:
   logError(e)
```

Type of `e` is:

```text
data LastError:
  instruction: string -- What AIR instruction failed
  msg: string -- Human-readable error message
  peer_id: string -- On what peer the error happened
```

#### if

If corresponds to `match`, `mismatch` extension of π-calculus.

```text
x = true
if x:
  -- always executed
  foo()
  
if x == false:
  -- never executed
  bar()
  
if x != false:
  -- executed
  baz()  
```

Currently, you may only use one `==`, `!=` operator in the `if` expression or compare with true.

Both operators can be a variable. Variable types must intersect.

#### else

Just the second branch of `if`, in case the condition does not hold.

```text
if true:
  foo()
else:
  bar()  
```

If you want to set a variable based on condition, see Conditional return.

#### otherwise

You may add `otherwise` to provide recovery for any block or expression:

```text
x <- foo()
otherwise:
  -- if foo can't be executed, then do bar()
  y <- bar()
```

### Conditional return

In Aqua, functions may have only one return expression, which is on the last line of the function block,  and conditional expressions cannot define the same variable:

```text
try:
  x <- foo()
otherwise:
  x <- bar() -- Error: name x was already defined in scope, can't compile  
```

So to get the value based on condition, we need to use a [writeable collection](../types.md#collection-types).

```text
-- result may have 0 or more values of type string, and is writeable
resultBox: *string
try:
  resultBox <- foo()
otherwise:
  resultBox <- bar()
  
-- now result contains only one value, let's extract it!
result = resultBox!

-- Type of result is string
-- Please note that if there were no writes to resultBox, 
-- the first use of result will halt.
-- So you need to be careful about it and ensure that there's always a value.
```

