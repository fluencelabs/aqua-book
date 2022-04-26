# Control, Scope And Visibility

In Aqua, the default namespace of a module is the file name and all declarations, i.e., data, services and functions, are public.

For example, the default.aqua file:

```python
-- default_foo.aqua


func foo() -> string:
    <- "I am a visible foo func that compiles"
```

Which we compile with

```bash
aqua -i aqua-scripts -o compiled-aqua
```

to obtain Typescript wrapped AIR, `default_foo.ts` in the `compiled-aqua` directory:

```typescript
import { FluenceClient, PeerIdB58 } from '@fluencelabs/fluence';
import { RequestFlowBuilder } from '@fluencelabs/fluence/dist/api.unstable';
import { RequestFlow } from '@fluencelabs/fluence/dist/internal/RequestFlow';


// Services


// Functions

export async function foo(client: FluenceClient, config?: {ttl?: number}): Promise<string> {
    let request: RequestFlow;
    const promise = new Promise<string>((resolve, reject) => {
        const r = new RequestFlowBuilder()
            .disableInjections()
            .withRawScript(
                `
(xor
 (seq
  (call %init_peer_id% ("getDataSrv" "-relay-") [] -relay-)
  (xor
   (call %init_peer_id% ("callbackSrv" "response") ["I am a visible foo func that compiles"])
   (call %init_peer_id% ("errorHandlingSrv" "error") [%last_error% 1])
  )
 )
 (call %init_peer_id% ("errorHandlingSrv" "error") [%last_error% 2])
)

            `,
            )
            .configHandler((h) => {
                h.on('getDataSrv', '-relay-', () => {
                    return client.relayPeerId!;
                });

                h.onEvent('callbackSrv', 'response', (args) => {
    const [res] = args;
  resolve(res);
});

                h.onEvent('errorHandlingSrv', 'error', (args) => {
                    // assuming error is the single argument
                    const [err] = args;
                    reject(err);
                });
            })
            .handleScriptError(reject)
            .handleTimeout(() => {
                reject('Request timed out for foo');
            })
        if(config && config.ttl) {
            r.withTTL(config.ttl)
        }
        request = r.build();
    });
    await client.initiateFlow(request!);
    return promise;
}
```

Regardless of your output target, i.e. raw AIR or Typescript wrapped AIR, the default module namespace is `default_foo` and `foo` is the compiled function.

While this default approach is handy for single file, single module development, it makes for inefficient dependency management and unnecessary compilations for multi-module projects. The remainder of this section introduces the scoping and visibility concepts available in Aqua to effectively manage dependencies.

### Managing Visibility With `module` and `declare`

By default, all declarations in a module, i.e., _data_, _service_ and _func_, are public. With the `module` declaration, Aqua allows developers to create named modules and define membership visibility where the default visibility of `module` is private. That is, with the `module` declaration all module members are private and do not get compiled.

Let's create an `export.aqua` file like so:

```python
module Export

func foo() -> string:
    <- "I am Export foo"
```

When we compile `export.aqua`

```bash
aqua -i aqua-scripts -o compiled-aqua
```

nothing gets compiled as expected:

```bash
2021.09.02 11:31:41 [INFO] Aqua Compiler 0.2.1-219
2021.09.02 11:31:42 [INFO] Source /Users/bebo/localdev/aqua-245/documentation-examples/aqua-scripts/export.aqua: compilation OK (nothing to emit)
```

You can further check the output directory, `compiled-aqua`, in our case, for the lack of output files. By corollary, `foo` cannot be imported from another files. For example:

```python
-- import.aqua

import "export.aqua"


func wrapped_foo() -> string:
    res <- foo()
    <- res
```

Results in compile failure since `foo` is not visible to `import.aqua`:

```python
6 func wrapped_foo() -> string:
7     res <- foo()
             ^^^==
             Undefined arrow, available: HOST_PEER_ID, INIT_PEER_ID, nil, LAST_ERROR
8     <- res
```

We can use `declares` to create visibility for a `module` namespace for **consuming** modules. For example,

```python
-- export.aqua
module Export declares foo

func bar() -> string:
    <- " I am MyFooBar bar"

func foo() -> string:
    res <- bar() 
    <- res
```

in and by itself does not result in compiled Aqua:

```bash
aqua -i aqua-scripts -o compiled-aqua -a
Aqua JS: node /Users/bebo/.nvm/versions/node/v14.16.0/lib/node_modules/@fluencelabs/aqua/aqua.js -i aqua-scripts -o compiled-aqua -a
Aqua JS:
Aqua JS: 2021.09.08 13:36:17 [INFO] Aqua Compiler 0.3.0-222
2021.09.08 13:36:21 [INFO] Source /Users/bebo/localdev/aqua-245/documentation-examples/aqua-scripts/export.aqua: compilation OK (nothing to emit)
```

But once we link from another module, e.g.:

```python
import foo from "export.aqua"

func foo_wrapper() -> string:
    res <- foo()
    <- res
```

We get the appropriate result:

```bash
2021.09.08 13:40:17 [INFO] Source /Users/bebo/localdev/aqua-245/documentation-examples/aqua-scripts/export.aqua: compilation OK (nothing to emit)
2021.09.08 13:40:17 [INFO] Result /Users/bebo/localdev/aqua-245/documentation-examples/compiled-aqua/import.ts: compilation OK (1 functions)
```

in form of  `import.ts`:

```typescript
// compiled-aqua/import.ts
import { FluencePeer } from '@fluencelabs/fluence';
import {
    ResultCodes,
    RequestFlow,
    RequestFlowBuilder,
    CallParams,
} from '@fluencelabs/fluence/dist/internal/compilerSupport/v1';


// Services


// Functions

 export function foo_wrapper(config?: {ttl?: number}) : Promise<string>;
 export function foo_wrapper(peer: FluencePeer, config?: {ttl?: number}) : Promise<string>;
 export function foo_wrapper(...args) {
     let peer: FluencePeer;

     let config;
     if (args[0] instanceof FluencePeer) {
         peer = args[0];
         config = args[1];
     } else {
         peer = FluencePeer.default;
         config = args[0];
     }

     let request: RequestFlow;
     const promise = new Promise<string>((resolve, reject) => {
         const r = new RequestFlowBuilder()
                 .disableInjections()
                 .withRawScript(
                     `
     (xor
 (seq
  (call %init_peer_id% ("getDataSrv" "-relay-") [] -relay-)
  (xor
   (call %init_peer_id% ("callbackSrv" "response") [" I am MyFooBar bar"])
   (call %init_peer_id% ("errorHandlingSrv" "error") [%last_error% 1])
  )
 )
 (call %init_peer_id% ("errorHandlingSrv" "error") [%last_error% 2])
)

                 `,
                 )
                 .configHandler((h) => {
                     h.on('getDataSrv', '-relay-', () => {
                    return peer.connectionInfo.connectedRelay ;
                });

                h.onEvent('callbackSrv', 'response', (args) => {
    const [res] = args;
  resolve(res);
});

                h.onEvent('errorHandlingSrv', 'error', (args) => {
                    const [err] = args;
                    reject(err);
                });
            })
            .handleScriptError(reject)
            .handleTimeout(() => {
                reject('Request timed out for foo_wrapper');
            })
        if(config && config.ttl) {
            r.withTTL(config.ttl)
        }
        request = r.build();
    });
    peer.internals.initiateFlow(request!);
    return promise;
}
```

Of course, if we change `import.aqua` to include the private `bar`:

```python
import bar from "export.aqua"

func bar_wrapper() -> string:
    res <- bar()
    <- res
```

We get the expected error:

```python
import bar from "export.aqua"
         ^^^===================
         Imported file declares [foo], no bar declared. Try adding `declares *` to that file.
```

As indicated in the error message, `declares *` makes all members of the namespace public, although we can be quite fine-grained and use a comma separated list of members we want to be visible, such as `declares foo, bar`.

### Scoping Inclusion With `use` and `import`

We already encountered the `import` statement earlier. Using `import` with the file name, e.g., `import "export.aqua"`, imports all visible, i.e., public, members from the dependency. We can manage import granularity with the `from` modifier, e.g., `import foo from "file.aqua"`, to limit our imports and subsequent compilation outputs. Moreover, we can alias imported declarations with the `as` modifier, e.g.,`import foo as HighFoo, bar as LowBar from "export_file.aqua"`.

In addition to `import`, we also have the `use` keyword available to link and scope. The difference between`use` and `import` is that `use` brings in module namespaces declared in the referenced source file. For example:

```python
-- export.aqua
module ExportModule declares foo

func foo() -> string:
    <- "I am a foo fighter"
```

declares the `ExportModule` namespace and makes `foo` visible. We can now bring `foo` into scope by means of its module namespace `ExportModule` in our import file without having to (re-) declare anything:

```python
-- import.aqua
use "export.aqua"

func foo -> string:
    res <- ExportModule.foo()
    <- res
```

This example already illustrates the power of `use` as we now can declare a local `foo` function rather than the `foo_wrapper` we used earlier. `use` provides very clear namespace separation that is fully enforced at the compiler level allowing developers to build, update and extend complex code bases with clear namespace separation by default.

The default behavior for `use` is to use the dependent filename if no module declaration was provided. Moreover, we can use the `as` modifier to change the module namespace. Continuing with the above example:

```python
-- import.aqua
use "export.aqua" as RenamedExport

func foo() -> string:
    -- res <- ExportModule.foo()  --< this fails
    res <- RenamedExport.foo()
    <- res
```
