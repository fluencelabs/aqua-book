# Quick Start

Every Fluence reference node comes with a set of builtin services that are accessible to Aqua programs. Let's use those readily available services to get the timestamp of a few of our peer-to-peer neighbourhood nodes with Aqua.

{% tabs %}
{% tab title="Timestamps With Aqua" %}
```haskell
-- timestamp_getter.aqua
-- bring the builtin services into scope
import "builtin.aqua"

-- create an identity service to join our results
service Op2("op"):
    identity(s: u64)
     array(a: string, b: u64) -> string

-- function to get ten timestamps from our Kademlia
-- neighborhood peers and return as an array of u64 timestamps
-- the function arguement node is our peer id
func ts_getter(node: string) -> []u64:
  -- create a streaming variable
  res: *u64
  -- execute on the pecified peer
  on node:
    -- get the base58 representation of the peer id
    k <- Op.string_to_b58(node)
    -- find all (default 20) neighborhood peers from k
    nodes <- Kademlia.neighborhood(k, nil, nil)
    -- for each peer in our neighborhood and in parallel
    for n <- nodes par:
      on n:
        -- try and get the peer's timestamp
        try:
          res <- Peer.timestamp_ms()
    -- flatten nine of our joined results
    Op2.identity(res!9)
  -- return an array of ten timestamps
  <- res
```
{% endtab %}
{% endtabs %}

The Aqua script essentially creates a workflow originating from our client peer to enumerate neighbor peers for our reference node, calls on the builtin timestamp service on each peer in parallel,  joins the results stream after we collect ten timestamps and return our u64 array of timestamps back to the client peer. 

 See the [ts-oracle example](https://github.com/fluencelabs/examples/tree/d52f06dfc3d30799fe6bd8e3e602c8ea1d1b8e8a/aqua-examples/ts-oracle) for the corresponding Aqua files in the `aqua-script` directory.  Now that we  have our script, let's compile it with the `aqua-cli` tool and find our AIR file in the `air-scripts` directory:

{% tabs %}
{% tab title="Compile" %}
```bash
aqua -i aqua-scripts -o air-scripts -a
```
{% endtab %}

{% tab title="Result" %}
```bash
# in the air-script dir you should have the following file
timestamp_getter.ts_getter.air
```
{% endtab %}
{% endtabs %}

Once we have our AIR file we can either use a Typescript or command line client. Let's use the command line client `flidst`see third tab for installation instructions, if needed:

{% tabs %}
{% tab title="Run Air scripts" %}
```bash
# execute the AIR script from our compile phase with a peer id
fldist run_air -p air-scripts/timestamp_getter.ts_getter.air  -d '{"node":"12D3KooWHLxVhUQyAuZe6AHMB29P7wkvTNMn7eDMcsqimJYLKREf"}'  --generated
```
{% endtab %}

{% tab title="Result" %}
```bash
# here we go: ten timestamps in micro seconds obtained in parallel
[
  [
    1624928596292,
    1624928596291,
    1624928596291,
    1624928596299,
    1624928596295,
    1624928596286,
    1624928596295,
    1624928596284,
    1624928596293,
    1624928596289
  ]
]
```
{% endtab %}

{% tab title="Installing fldist" %}
```bash
# if you don't have fldist on your machine: 
npm -g install @fluencelabs/fldist
```
{% endtab %}
{% endtabs %}

And that's it.  We now have ten timestamps right from our selected peer's neighbors.

