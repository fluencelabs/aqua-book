# Imports And Exports

An Aqua source file has a head and a body. The body contains function definitions, services, types, constants. The header manages what is imported from other files and what is exported.

### Import Expression

The main way to import a file is via `import` expression:

```haskell
import "@fluencelabs/aqua-lib/builtin.aqua"

func foo():
  Op.noop()
```

Aqua compiler takes a source directory and a list of import directories \(usually with `node_modules` as a default\). You can use relative paths to `.aqua` files, relatively to the current file's path, and to import folders.

Everything defined in the file is imported into the current namespace.

### `use` Expression

The `use` expression makes it possible to import a subset of a file, or to alias imports to avoid namespace collisions.

{% embed url="https://github.com/fluencelabs/aqua/issues/30" %}



