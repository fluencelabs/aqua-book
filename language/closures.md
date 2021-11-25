# Closures

In Aqua, you can create an arrow within the function, enclosing its context.

```python
service Hello:
  say_hello(to_name: string)

func bar(callback: string -> ()):
   callback("Fish")

func foo(peer: string):
  on peer:
    -- Capture service resolution
    Hello "world"
    -- Create a closure named "closure"
    closure = (name: string) -> string:
      -- Use a value that's available on the definition site
      -- Note this repeated `on`, it will be explained later
      on peer:
        -- Using an argument
        say_hello(name)
      -- Return a value from the closure; syntax is the same as in functions
      <- name
  -- Pass this closure to another function
  bar(closure)
```

Closures can be created anywhere in the function, starting with Aqua 0.4.1, and then used just like any other arrow (argument of arrow type, function, or service method): passed to another function as an argument, or called right there.

Closures are expected to enclose over three domains:

* Values in scope,
* Service resolutions,
* Topology: place where the closure is defined should be the place where it's executed.

{% hint style="danger" %}
As of Aqua 0.4.1, the topology is not captured by the closure, you have to state it explicitly.

If you don't do it, service resolutions may break, which will cause barely traceable bugs.
{% endhint %}

{% embed url="https://github.com/fluencelabs/aqua/issues/356" %}
Github issue for capturing the topology within closures
{% endembed %}

It is not yet possible to return an arrow from an arrow.
