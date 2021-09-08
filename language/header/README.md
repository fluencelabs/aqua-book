# Imports And Exports

An Aqua source file has a head and a body. The body contains function definitions, services, types, constants. The header manages what is imported from other files and what is exported.

## Module

By default, `.aqua` file exports and declares everything it contains. With `module` header you can describe the `.aqua` file's interface.

```python
-- Module expression may be only on the very first line of the file
module ModuleName declares *
```

`Module.Name` may contain dots.

`ModuleName` can be used as the module's name when this file is `use`d. In this case, only what is enumerated in `declares` section will be available. `declares *` allows you to declare everything in the file as the module interface.

```text
module ModuleName declares CONSTNAME, ServiceName, MyType, fn

const CONSTNAME = "smth"

service ServiceName:
  do_smth()
  
data MyType:
  result: i32  

function fn() -> string:
  <- CONSTNAME
```

## Import Expression

The main way to import a file is via `import` expression:

```haskell
import "@fluencelabs/aqua-lib/builtin.aqua"

func foo():
  Op.noop()
```

Aqua compiler takes a source directory and a list of import directories \(usually with `node_modules` as a default\). You can use relative paths to `.aqua` files, relatively to the current file's path, and to import folders.

Everything defined in the file is imported into the current namespace.

You can cherry-pick and rename imports using `import ... from` expression:

```python
import Op as Noop from "@fluencelabs/aqua-lib/builtin.aqua"

func foo():
  Noop.noop()
```

## Use Expression

The `use` expression makes it possible to import a file into a named scope.

```python
use Op from "@fluencelabs/aqua-lib/builtin.aqua" as BuiltIn

func foo():
  BuiltIn.Op.noop()
```

If the imported file has a `module` header, `from` and `as` sections of `use` may be omitted.

```python
use "@fluencelabs/aqua-lib/builtin.aqua"
-- Assume that builtin.aqua's header is `module BuiltIn declares *`

func foo():
  BuiltIn.Op.noop()
```

## Export

While it's useful to split the code into several functions into different files, it's not always a good idea to compile everything into the host language.

Another problem is libraries distribution. If a developer wants to deliver an `.aqua` library, he or she often needs to provide it in compiled form as well.

`export` lets a developer decide what exactly is going to be exported, including imported functions.

```python
import bar from "lib.aqua"

-- Exported functions and services will be compiled for the host language
-- You can use several `export` expressions
export foo as my_foo
export bar, MySrv

service MySrv:
  call_smth()
  
func foo() -> bool:
  <- true  
```

