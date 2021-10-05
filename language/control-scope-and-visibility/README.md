# Control, Scope And Visibility

## 

In Aqua, the default namespace of a module is the file name and all declarations, i.e., data, services and functions, are public.

For example, a `default.aqua` file:

```text
-- default_foo.aqua


func foo() -> string:
    <- "I am a visible foo func that compiles"
```

Provides visibility to all members, i.e. `foo`.

Let's compile the file:

```text
aqua -i aqua-scripts -o compiled-aqua
```

To obtain Typescript wrapped AIR, `default_foo.ts` in the `compiled-aqua` directory:

```text
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

Regardless of your output target, i.e. AIR or Typescript, the default module namespace is `default_foo` and `foo` is the compiled function. If you are following along, check your `compiled-aqua` directory for the output file.

While the default approach is handy for single file, single module development, it makes for inefficient dependency management. The remainder of this section introduces the scoping and visibility concepts available in Aqua to effectively manage dependencies.

### Visibility In Aqua Exports

By default, all declarations in a module, i.e., _data_, _service_ and _func_, are public. With the `module` declaration, Aqua allows developers to create named modules and define membership visibility where the default visibility of `module` is private and module members do not get compiled.

Let's create an `export.aqua` file to check things out:

```text
-- export.aqua
module Export

func foo() -> string:
    <- "I am Export foo"
```

When we compile `export.aqua`

```text
aqua -i aqua-scripts -o compiled-aqua
```

nothing gets compiled as expected:

```text
2021.09.02 11:31:41 [INFO] Aqua Compiler 0.2.1-219
2021.09.02 11:31:42 [INFO] Source /Users/bebo/localdev/aqua-245/documentation-examples/aqua-scripts/export.aqua: compilation OK (nothing to emit)
```

Again, you can check the output directory, `compiled-aqua` , for the lack of output files. By corollary, `foo` cannot be imported from another files. For example:

```text
-- import.aqua

import "module_foo.aqua"


func wrapped_foo() -> string:
    res <- foo()
    <- res
```

Results in a compile failure since `foo` is not visible to `import.aqua`:

```text
this is not working as expected. Issue filed.
```



In order to make module members available for use by other modules, we can use `declares`. For example,

```text
-- export.aqua
module ExportModule declares foo

func bar() -> string:
    <- " I am MyFooBar bar"

func foo() -> string:
    res <- bar() 
    <- res
```

Keeps `bar` private but exposes `foo` as public for use by another module only. That is, compiling just `expert.aqua` does not create output if compiled directly:

```text
2021.09.02 16:33:53 [INFO] Aqua Compiler 0.2.1-219
2021.09.02 16:33:54 [INFO] Source /Users/bebo/localdev/aqua-245/documentation-examples/aqua-scripts/export.aqua: compilation OK (nothing to emit)
```

Let's add a module consuming `foo`:

```text
-- import.aqua
import "export.aqua"

func foo_wrapper() -> string:
    res <- foo()
    <- res
```

results in the compilation of `foo` and `foo_wrapper`:

```text
2021.09.02 16:37:18 [INFO] Aqua Compiler 0.2.1-219
2021.09.02 16:37:18 [INFO] Source /Users/bebo/localdev/aqua-245/documentation-examples/aqua-scripts/export.aqua: compilation OK (nothing to emit)
2021.09.02 16:37:18 [INFO] Result /Users/bebo/localdev/aqua-245/documentation-examples/compiled-aqua/import.ts: compilation OK (1 functions)
```

Check out the output file, `compiled-aqua/import.ts`:  


```text
export async function foo_wrapper(client: FluenceClient, config?: {ttl?: number}): Promise<string> {
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
                reject('Request timed out for foo_wrapper');
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



Of course, if we change `import.aqua` to:

```text
import bar from "export.aqua"

func bar_wrapper() -> string:
    res <- bar()
    <- res
```

we get the expected error:

```text
import bar from "export.aqua"
         ^^^===================
         Imported file declares [foo], no bar declared. Try adding `declares *` to that file.
```

since `bar` was not declared to be visible outside the module namespace.

As indicated in the error message, `declares *` makes all members of the namespace public.

Note: `exports` isn't working yet

### Scoping Aqua Imports

We already encountered the `import` statement earlier. Using `import` with the file name, e.g., `import "export.aqua"`, imports all namespace members visible for export. However, we can scope import with the `from` modifier, e.g., `import foo from "file.aqua"`, to limit our imports and subsequent compilation outputs.

Moreover, we can associate a namespace with an `import`, e.g., `import foo from "export.aqua" as ExportModule`, which then allows us to use `foo` as:

```text
import foo from "export.aqua" as ExportModule

func demo_foo() -> string:
    res <- ExportModule.foo()
    <- res
```

and we can rename our imports with the `as` modifier with or without the `module` modifier:

```text
import foo as ex_foo from "export.aqua" as ExportModule

func demo_foo() -> string:
    res <- ExportModule.ex_foo()
    <- res
```

In addition to `import`, we also have the `use` keyword available to link and scope. The difference between`use` and `import` is that `use` brings in module namespaces declared in the referenced file. For example:

```text
-- export.aqua
module ExportModule declares foo

func foo() -> string:
    <- "I am a foo fighter"
```

declares the `ExportModule` namespace and makes `foo` visible. We can now bring `foo` into scope by means of its module namespace `ExportModule` in our import file without having to \(re-\) declare anything:

```text
-- import.aqua
use "export.aqua"

func foo_wrapper() -> string:
    res <- ExportModule.foo()
    <- res
```



If, for validation purposes, we use:

```text
-- import.aqua
use "export.aqua"

func foo_wrapper() -> string:
    res <- foo()
    <- res
```

We get the expected error:

```text
3 func foo_wrapper() -> string:
4     res <- foo()
5     <- res
         ^^^
         Undefined name, available: ExportModule.foo, host_peer_id, ExportModule.host_peer_id, ExportModule.%last_error%, ExportModule.nil, nil, %last_error%
```

If we don't declare a module name in our reference file and use `use` in our consumer file, the compiler will throw an error:

```text
-- export.aqua
func foo() -> string:
    <- "I am a foo fighter"
```

```text
use "export.aqua"
      ^^^^^^^^^^^^^
      Used module has no `module` header. Please add `module` header or use ... as ModuleName, or switch to import
```

Following the compiler instructions as can declare a module name in the importing file:

```text
use "export.aqua" as ModuleName

func foo_wrapper() -> string:
    res <- ModuleName.foo()
    <- res
```

Note: also doesn't compile just yet

