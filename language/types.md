# Types

## Scalars

Scalar types follow the Wasm IT notation.

* Unsigned numbers: `u8`, `u16`, `u32`, `u64`
* Signed numbers: `i8`, `i16`, `i32`, `i64`
* Floats: `f32`, `f64`
* Boolean: `bool`
* String: `string`
* Records \(product type\): see below
* Arrays: see Collection Types below

## Literals

You can pass booleans \(true, false\), numbers, double-quoted strings as literals.

## Products

```haskell
data ProductName:
  field_name: string

data OtherProduct:
  product: ProductName
  flag: bool
```

Fields are accessible with the dot operator `.` , e.g. `product.field`.

## Collection Types

Aqua has three different types with variable length, denoted by quantifiers `[]`, `*`, and `?`.

Immutable collection with 0..N values: `[]`

Immutable collection with 0 or 1 value: `?`

Appendable collection with 0..N values: `*`

Any data type can be prepended with a quantifier, e.g. `*u32`, `[][]string`, `?ProductType` are all correct type specifications.

You can access a distinct value of a collection with `!` operator, optionally followed by an index.

Examples:

```haskell
strict_array: []u32
array_of_arrays: [][]u32
element_5 = strict_array!5
element_0 = strict_array!0
element_0_anotherway = strict_array!

-- It could be an argument or any other collection
maybe_value: ?string
-- This ! operator will FAIL if maybe_value is backed by a read-only data structure
-- And will WAIT if maybe_value is backed with a stream (*string)
value = maybe_value!
```

## Arrow Types

Every function has an arrow type that maps a list of input types to an optional output type.

It can be denoted as: `Type1, Type2 -> Result`

In the type definition, the absence of a result is denoted with `()`, e.g., `string -> ()`

The absence of arguments is denoted `-> ()`.That is, this mapping takes no argument and has no return type.

Note that there's no `Unit` type in Aqua: you cannot assign a non-existing result to a value.

```haskell
-- Assume that arrow has type: -> ()

-- This is possible:
arrow()

-- This will lead to error:
x <- arrow()
```

## Type Alias

For convenience, you can alias a type:

```haskell
alias MyAlias = ?string
```

## Type Variance

Aqua is made for composing data on the open network. That means that you want to compose things if they do compose, even if you don't control its source code.

Therefore Aqua follows the structural typing paradigm: if a type contains all the expected data, then it fits. For example, you can pass `u8` in place of `u16` or `i16`. Or `?bool` in place of `[]bool`. Or `*string` instead of `?string` or `[]string`. The same holds for products.

For arrow types, Aqua checks the variance on arguments and contravariance on the return type.

```haskell
-- We expect u32
xs: *u32

-- u16 is less then u32
foo1: -> u16
-- works
xs <- foo1()

-- i32 has sign, so cannot fit into u32
foo2: -> i32
-- will fail
xs <- foo2()

-- Function takes an arrow as an argument
func bar(callback: u32 -> u32): ...


foo3: u16 -> u16

-- Will not work
bar(foo3)  

foo4: u16 -> u64

-- Works
bar(foo4)

-- Works for Product Types
data Smaller:
    field: i16
    
data Bigger:
    field: i32
    
service Service("s"):
    take_smaller(s: Smaller) -> Smaller
    take_bigger(b: Bigger) -> Bigger
    
func take_smaller(small: Smaller):
    Service.take_bigger(small)
    
func return_bigger(small: Smaller) -> Bigger:
    <- small
    
-- Will not work    
func take_bigger(big: Bigger):
    Service.take_smaller(big)

func return_smaller(big: Bigger) -> Smaller:
    <- big    
```

Arrow type `A: D -> C` is a subtype of `A1: D1 -> C1`, if `D1` is a subtype of `D` and `C` is a subtype of `C1`.

## Type Of A Service And A File

A service type is a product of arrows.

```haskell
service MyService:
  foo(arg: string) -> bool

-- type of this service is:
data MyServiceType:
  foo: string -> bool
```

The file is a product of all defined constants and functions \(treated as arrows\). Type definitions in the file do not go to the file type.

```haskell
-- MyFile.aqua

func foo(arg: string) -> bool:
  ...

const flag ?= true  

-- type of MyFile.aqua
data MyServiceType:
  foo: string -> bool
  flag: bool
```

{% embed url="https://github.com/fluencelabs/aqua/blob/main/types/src/main/scala/aqua/types/Type.scala" caption="See the types system implementation" %}

