# Header

## Header expressions

### `import`

The `import` expression brings everything defined within the imported file into the scope.

```haskell
import "path/to/file.aqua"
```

The to be imported file path is first resolved relative to the source file path followed by checking for an `-imports` directories.

See [Imports & Exports](../header.md) for details.

