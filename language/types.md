# Types

### Scalars

Scalar types follow the Wasm IT notation.

* Unsigned numbers: `u8`, `u16`, `u32`, `u64`
* Signed numbers: `i8`, `i16`, `i32`, `i64`
* Floats: `f32`, `f64`
* Boolean: `bool`
* String: `string`
* Records \(product type\): see below
* Arrays: see Collection Types below

### Literals

You can pass booleans \(true, false\), numbers, double-quoted strings as literals.

### Products

```python
data ProductName:
  field_name: string
  
data OtherProduct:
  prod: ProductName
  flag: bool  
```

Fields are accessible with the `.` operator, e.g. `product.field`.

### Collection Types

Aqua has three different types with variable length, denoted by quantifiers `[]`, `*`, and `?`.

Immutable collection with 0..N values: `[]`

Immutable collection with 0 or 1 value: `?`

Appendable collection with 0..N values: `*`

Any data type can be prepended with a quantifier: `*u32`, `[][]string`, `?ProductType` – these types are absolutely correct.

You can access a distinct value of a collection with `!` operator, optionally followed by an index.

Examples:

```text
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

### Arrow Types

Every function has an arrow type that maps a list of input types to an optional output type.

It can be denoted as: `Type1, Type2 -> Result`

In the type definition, absense of result is denoted with `()`: `string -> ()`

The absence of arguments is denoted by nothing: `-> ()` this arrow takes nothing as an argument and has no return type.

Note that there's no `Unit` type in Aqua: you cannot assign the non-existing result to a value.

```python
-- Assume that arrow has type: -> ()

-- This is possible:
arrow()

-- This will lead to error:
x <- arrow()
```

### Type Alias

For convinience, you can alias a type:

```python
alias MyAlias = ?string
```

### Type Variance

Aqua is made for composing data on the open network. That means that you want to compose things if they do compose, even if you don't control its source code.

Therefore Aqua follows the structural typing paradigm: if a type contains all the expected data, than it fits. For example, you can pass `u8` in place of `u16` or `i16`. Or `?bool` in place of `[]bool`. Or `*string` instead of `?string` or `[]string`. The same holds for products.

For arrow types, Aqua checks variance on arguments, contravariance on the return type.

### Type of a Service and a file

A service type is a product of arrows. File is a product of all defined constants and functions \(treated as arrows\). Type definitions in the file does not go to the file type.

{% embed url="https://github.com/fluencelabs/aqua/blob/main/types/src/main/scala/aqua/types/Type.scala" caption="See the types system implementation" %}



