# Iterative

π-calculus has a notion of the repetitive process: `!P = P | !P`. That means, you can always fork a new `P` process if you need it.

In Aqua, two operations correspond to it: you can call a service function \(it's just available when it's needed\), and you can use `for` loop to iterate on collections.

### `for` expression

In short, `for` looks like the following:

```haskell
xs: []string

for x <- xs:
  y <- foo(x)

-- x and y are not accessible there, you can even redefine them
x <- bar()
y <- baz()
```

## Contract

* Iterations of `for` loop are executed sequentially by default.
* Variables defined inside `for` loop are not available outside.
* `for` loop's code has access to all variables above.
* `for` can be executed on a variable of any [Collection type](../types.md#collection-types).

### Conditional `for`

For can be executed on a variable of any [Collection type](../types.md#collection-types). You can make several trials in a loop, and break once any trial succeeded.

```haskell
xs: []string

for x <- xs try:
  -- Will stop trying once foo succeeds
  foo(x)
```

The contract is changed as in [Parallel](parallel.md#contract) flow.

### Parallel `for`

Running many operations in parallel is the most commonly used pattern for `for`.

```text
xs: []string

for x <- xs par:
  on x:
    foo()

-- Once the fastest x succeeds, execution continues
-- If you want to make the subsequent execution independent from for,
-- mark it with par, e.g.:
par continueWithBaz()
```

The contract is changed as in [Conditional](conditional.md#contract) flow.

### Export data from `for`

The way to export data from `for` is the same as in [Conditional return](conditional.md#conditional-return) and [Race patterns](parallel.md#join-behavior).

```haskell
xs: []string
return: *string

-- can be par, try, or nothing
for x <- xs par:
  on x:
    return <- foo()

-- Wait for 6 fastest results -- see Join behavior    
baz(return!5, return)
```

### `for` on streams

`for` on streams is one of the most advanced and powerful parts of Aqua. See [CRDT streams](../crdt-streams.md) for details.

