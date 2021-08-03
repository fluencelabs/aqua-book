---
description: API of protocol-level service (a.k.a builtins)
---

# @fluencelabs/aqua-lib

## On NPM

You can find the latest releases of `aqua-lib` [on NPM](https://www.npmjs.com/package/@fluencelabs/aqua-lib)

## Services

* `Op` - different kinds of data transformations
* `Peer` - operations affecting peer's internal state
* `Kademlia` - api for libp2p Kademlia
* `Srv` - service manipulation
* `Dist` - module & blupeint disitrbution
* `Script` - scheduled scripts API

## API

The most up-to-date documentation of the API is in the code comments, please [check it out on GitHub](https://github.com/fluencelabs/aqua-lib/blob/main/builtin.aqua)

## How to use it

### In Aqua

Add `@fluencelabs/aqua-lib` to your dependencies as described in [Libraries doc](./), and then import it in your Aqua script:

```haskell
import "@fluencelabs/aqua-lib"

-- gather Peer.identify from all nodes in the neighbourhood
func getPeersInfo() -> []Info:
    infos: *Info
    nodes <- Kademlia.neighborhood(%init_peer_id%, nil, nil)
    for node in nodes:
        on node:
            infos <- Peer.identify()
    <- infos
```

### In TypeScript

`aqua-lib` is meant to be used to write Aqua scripts, and since`aqua-lib` doesn't export any top-level functions, it's not callable directly in the TypeScript. 

## Patterns

### Functions with variable number of arguments

Currently, Aqua doesn't allow to define functions with a variable number of arguments. And  that limits `aqua-lib` API. But there's a way around that.

Let's take `Op.concat_strings` as an example. You can use it to concatenate several strings. `aqua-lib` provides the following signature:

```haskell
concat_strings(a: string, b: string) -> string
```

Sometimes that is enough, but sometimes you need to concatenate more than 2 strings at a time. Happily, under the hood `concat_strings` accepts any number of arguments, so you can redeclare it with the number of arguments that you want:

```haskell
service MyOp("op"):
  concat_strings(a: string, b: string, c: string, d: string) -> string
```

#### List of operations with a variable number of arguments

Here's a full list of other Op-s that you can apply this pattern to

* `Op.concat` - can concatenate any number of arrays
* `Op.array` - wraps any number of arguments into an array
* `Op.concat_string` - concatenates any number of strings

