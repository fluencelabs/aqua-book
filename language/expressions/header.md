# Header

### Header expressions

`import`

Import expression brings everything defined within the imported file into the scope.

```text
import "path/to/file.aqua"
```

Imported file is first resolved relative to the source file path, then in `-imports` directories.

See [Imports & Exports](../statements-1.md) for details.

