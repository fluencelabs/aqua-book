# Variables

Variables are immutable, except [writeable collections](types.md#collection-types) \(streams\).

### Arguments

Function arguments are available within the whole function body.

### Return values

That's the second way to get data with a name.

### Literals

Aqua supports just a few literals: numbers, quoted strings, booleans. You cannot make a structure in Aqua.

### Lenses

Can get fields from objects, and elements by id from collections.

### Assignments

Assignments do nothing new, just gives a name to a variable with applied lens.

### Constants

Constants are like assignments, but in the root scope. Can be used in all function bodies, textually below the place of const definition.

You can change the compilation results with overriding a constant. Override should be of the same type \(or a subtype\).

Only literals can be values of constants.

### Visibility scopes

Visibility scopes follows the contracts of execution flow.

### Streams as literals

Stream is a special data that allows many writes. It worth a dedicated article.

Here we just explain how to initiate streams and use them as empty collection literals. Refer to conditional return.

