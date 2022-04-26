# Aqua CLI

Aqua CLI allows you to manage all aspects of [Aqua](https://doc.fluence.dev/aqua-book/) development and includes:

* Compiler
* Client Peer
* Utilities

To install the Aqua CLI package:

```bash
npm install -g @fluencelabs/aqua
```

### Compiler

The compiler turns high-level Aqua code into either JS/TS handlers wrapping AIR, the default, or pure AIR.

The quickest way to compile Aqua code is to take all `.aqua` files from the specified _input_ directory, e.g., `src/aqua`, and place the generated JavaScript code in some _output_ directory of your choosing, e.g. `src/generated`. Please note that if the specified output directory does not exist, the CLI creates it for you:

```bash
aqua --input src/aqua --output src/generated
```

Of course, we can be more specific and name a filename:

```bash
aqua --input src/aqua/some_file.aqua --output src/generated
```

As mentioned in the intro, the Aqua compiler generates `.js` with `.d.ts` TypeScript files by default. Output files will contain functions exported from `.aqua` files and methods for registering defined services. You can read more about calling functions and service registration in the [FluenceJS documentation](https://doc.fluence.dev/docs/fluence-js/3\_in\_depth).

Additional compiler options are:

* `--js` flag, which generates only `.js` files
* `--air` or `-a` flag, which generates pure AIR code
* `--scheduled`, which generates AIR code suitable for script storage

Use `aqua --help` for a complete listing of available flags, subcommands and explanations.

### Subcommands

The CLI provides additional features and utilities via subcommands.

#### Aqua Run

The `aqua run` command creates a one-shot client peer over the compiled Aqua code specified allowing you to quickly and effectively test Aqua code against deployed services on the network.

```bash
aqua run --addr <relay multidaddress> --input <your aqua filepath> --func '<function name>(<args>)'
```

For the following Aqua script:

```python
-- some-dir/hello.aqua
service Hello("service_id"):
    hello(name:string) -> string
    
func hello(name: string, node:string, sid: string) -> string:
    on node:
        Hello sid
        res <- Hello.hello(name)
    <- res    
```

We instantiate our aqua client peer:

```bash
aqua run --addr /dns4/.../wss/p2p/12D3 ...aoHI --input some-dir/hello.aqua --func 'hello("reader", "peer id", ["service id1", "service id2"])'
```

The `aqua run` command provides additional features such as:

* `--sk` or `-s`  allows you to provide your secret key (sk) in base64
* `--addr` or `-a`  allows you to specify a relay in [_multiaddr_](https://github.com/multiformats/multiaddr) format, e.g., `/dns4/kras-04.fluence.dev/tcp/19001/wss/p2p/12D3KooWFEwNWcHqi9rtsmDhsYcDbRUCDXH84RC4FW6UfsFWaoHi` . The `aqua config default_peers <krasnodar, testnet, stage>` command enumerates you the respective network multiaddresses of available Fluence nodes.&#x20;

`--import` or `-m` allows you to [import functionality](https://doc.fluence.dev/aqua-book/language/header) from one or more source folders by using the flag repeatedly

*   `--data` or `-d` allows you to specify data arguments as a json map:

    ```
    aqua run --addr /dns4/.../wss/p2p/12D3 ... oHI --input my_code.aqua --func 'my_aqua_func(a, b)' --data '{"a": "some_string", "b": 123}'
    ```
* `--data-path` or `p` allows you to specify data arguments, see `--data`, as a file. _Note that `--data` and `--data-path` are mutually exclusive._

Use `aqua run --help` for a complete listing of available flags and explanations.

#### Aqua Create Keypair

The `aqua key create` utility allows you to create an ed25519-based keypair in _base64_:

```bash
aqua key create
```

And produces the following json document:

```json
{
    "peerId": "12D3KooWMC6picJQNruwFMFWqP62FWLtbM94TGYEzCsthsKa46CQ",
    "secretKey": "QG3Ot2i1kD4Mpw0RpsKtUjbA/0XjZ0WP7dajDBwLQi0=",
    "publicKey": "CAESIKkB+6eYhFDsEZhn0u+xwIKVhE+1xvgJoV5/csc+CS6R"
}
```

#### Aqua Module Distribution

A critical step is to get our WASm modules and service configuration files to our target hosts. The Aqua cli provides the capability to upload and configure our assets into hosted services under the `aqua remote` namespace:

```
aqua remote --help
Usage:
    ...
    aqua remote deploy_service
    aqua remote remove_service
    ...

Subcommands:
    ...
    deploy_service
        Deploy service from WASM modules
    remove_service
        Remove service
    ...
```



See the [service management](service-management.md) section for details and examples.

#### Aqua Environments Listing

The `aqua config default_peers` utility shows a list of peers in [multiaddr](https://github.com/multiformats/multiaddr) format for a specific Fluence network. Currently, there are three environments: `krasnodar`, the default network,`stage` and `testnet`.

```
aqua config default_peers testnet
```

shows a list of `testnet` peers:

```
dns4/net01.fluence.dev/tcp/19001/wss/p2p/12D3KooWEXNUbCXooUwHrHBbrmjsrpHXoEphPwbjQXEGyzbqKnE9
/dns4/net01.fluence.dev/tcp/19990/wss/p2p/12D3KooWMhVpgfQxBLkQkJed8VFNvgN4iE6MD7xCybb1ZYWW2Gtz
/dns4/net02.fluence.dev/tcp/19001/wss/p2p/12D3KooWHk9BjDQBUqnavciRPhAYFvqKBe4ZiPPvde7vDaqgn5er
/dns4/net03.fluence.dev/tcp/19001/wss/p2p/12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb
/dns4/net04.fluence.dev/tcp/19001/wss/p2p/12D3KooWJbJFaZ3k5sNd8DjQgg3aERoKtBAnirEvPV8yp76kEXHB
/dns4/net05.fluence.dev/tcp/19001/wss/p2p/12D3KooWCKCeqLPSgMnDjyFsJuWqREDtKNHx1JEBiwaMXhCLNTRb
/dns4/net06.fluence.dev/tcp/19001/wss/p2p/12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH
/dns4/net07.fluence.dev/tcp/19001/wss/p2p/12D3KooWBSdm6TkqnEFrgBuSkpVE3dR1kr6952DsWQRNwJZjFZBv
/dns4/net08.fluence.dev/tcp/19001/wss/p2p/12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H
/dns4/net09.fluence.dev/tcp/19001/wss/p2p/12D3KooWF7gjXhQ4LaKj6j7ntxsPpGk34psdQicN2KNfBi9bFKXg
/dns4/net10.fluence.dev/tcp/19001/wss/p2p/12D3KooWB9P1xmV3c7ZPpBemovbwCiRRTKd3Kq2jsVPQN4ZukDf
```

