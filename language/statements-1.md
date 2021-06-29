# Imports & exports

Aqua source file has head and body. The body contains function definitions, services, types, constants. Header manages what is imported from other files, and what is exported from this one.

### Import expression

The main way to import a file is via `import` expression:

```text
import "@fluencelabs/aqua-lib/builtin.aqua"

func foo():
  Op.noop()
```

Aqua compiler takes a source directory and a list of import directories \(usually with `node_modules` as a default\). You can use relative paths to `.aqua` files, relatively to the current file's path, and to import folders.

Everything defined in the file is imported into the current namespace.

### `Use` expression

Use expression makes it possible to import a subset of a file, or to alias the imports to avoid collisions.

{% embed url="https://github.com/fluencelabs/aqua/issues/30" %}



