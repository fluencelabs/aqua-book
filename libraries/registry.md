---
description: Aqua implementation of Fluence Registry and ResourcesAPI
---

# @fluencelabs/registry

Fluence Registry is an essential part of the Fluence network protocol. It provides a Resources API that can be used for service advertisement and discovery. For more details check out our [community call](https://youtu.be/Md0\_Ny\_5\_1o?t=770).

## Releases

You can find the latest `registry` release [on NPM](https://www.npmjs.com/package/@fluencelabs/registry) and the changelogs are in the [GitHub](https://github.com/fluencelabs/aqua-dht/releases) repo.

## API

For the API implementation, take a look at [resources-api.aqua](https://github.com/fluencelabs/registry/blob/main/aqua/resources-api.aqua) in the `registry` repo.

## Terminology

* **Registry** - a service that provides low-level API. `resources-api.aqua` is built on top of it.
* **@fluencelabs/registry** - an Aqua library on NPM that provides high-level and low-level APIs to develop custom registry scripts.
* **Resource/Provider** - a pattern for peer/service advertisement and discovery. **Providers** register for a **Resource** that can be discovered by its **resource\_id**.
* **Kademlia** - an algorithm for structuring a peer-to-peer network so that peers can find each other efficiently, i.e., in no more than O(logN) hops where N is the total number of peers in the network.
* **Resource** – a `string` label with associated `owner_peer_id` and a list of providers. A resource should be understood as a group of services or a group of peers united by some common feature. In low-level API **Resource** is a **Registry key.**
* **Registry Key** - a structure, signed by `owner_peer_id`, which holds information about Resource:

```
data Key:
  id: string
  label: string
  owner_peer_id: string
  timestamp_created: u64
  challenge: []u8
  challenge_type: string
  signature: []u8
```

* **resource\_id** - a stable identifier created from the hash of `label` and `owner_peer_id` used to identify any resource. `id` field of Registry `Key`
* **Resource owner** - the `owner_peer_id` that created the resource. Other users can create resources with the same label but the identifier will be different because of the `owner_peer_id.`
* **challenge/challenge\_type** – dummy fields which will be used for permission management in the next Registry iterations.
* **Provider** – a peer which is registered as a resource provider, optionally with an associated **relay\_id** and **service\_id**. Each provider is associated with a **Registry record.**
* **Registry record** - a structure, signed by `set_by` peer\_id, which holds information about Provider:

```
data Record:
  key_id: string
  value: string
  peer_id: string
  set_by: string
  relay_id: []string
  service_id: []string
  timestamp_created: u64
  solution: []u8
  signature: []u8
```

* provider's **value** – any string which can be defined and used in accordance with protocol requirements.
* provider's **peer\_id** – peer id of provider of this resource.
* provider's **relay\_id** - optional, set if provider is available on the network through this relay.&#x20;

{% hint style="info" %}
When a **provider** doesn't have a publicly accessible IP address, e.g. the client peer is a browser, it connects to the network through a relay node. That means that _other_ peers only can connect to this **provider** through a relay. In that case,`registerProvider implicitly set relay_id to`**`HOST_PEER_ID`**.&#x20;
{% endhint %}

* **provider**'s **service\_id** - optional, id of the service of that provider.
* **solution** – dummy field, will be used for permission checking in the next Registry iterations.
* **provider limit** - a **resource** can have at most 32 providers. Each new provider added after the provider limit has been reached results in removing an old provider following the FIFO principle. Soon provider's prioritization will be handled by TrustGraph.
* **host provider's record** - a **Registry record** with `peer_id`of a node. When a node is registered as a provider via `registerNodeProvider` or `createResourceAndRegisterNodeProvider`, the **record** is a **host record**. Host records live through garbage collection, unlike other Registry **records**. [See Register As A Provider ](registry.md#register-as-a-provider)for details.
* **resource** and **provider lifetime** - a **resource** and **provider record** are **** republished every  ****  [1 hour](https://github.com/fluencelabs/registry/blob/main/service/src/defaults.rs#L23) and evicted, i.e. removed, [24 hours](https://github.com/fluencelabs/registry/blob/main/service/src/defaults.rs#L24) after being unused.

{% hint style="info" %}
So there are two types of providers. First is a node provider which lifetime controlled by this node. Second is a JS peer provider and should be renewed periodically by this peer.
{% endhint %}

* **script caller** - a peer that executes a script by sending it to the network. In Aqua it's `INIT_PEER_ID`
* **node** - usually a Fluence node hosted by the community or Fluence Team. Nodes are long-lived, can host WebAssembly services and participate in the Kademlia network.

## How To Use Registry

{% hint style="info" %}
There are [several simple examples](https://github.com/fluencelabs/registry#how-to-use) in the `fluencelabs/registry` repo. Give them a look.
{% endhint %}

### Create A Resource

Before registering as a provider is possible, resource must be created. That's exactly what `createResource` does.&#x20;

Here's a simple Aqua example:

```haskell
import "@fluencelabs/registry/resources-api.aqua"
import "@fluencelabs/aqua-lib/builtin.aqua"

func my_function(label: string) -> ?string, *string:
    resource_id, errors <- createResource(resource)
    if resource_id != nil:
        -- resource created successfully
        Op.noop()
    else:
        -- resource creation failed
        Op.noop()
        
    <- resource_id, errors
```

### Register As A Provider

There are four functions that register providers. Let's review them.

These you would use for most of your needs:

* `registerProvider` - registers`INIT_PEER_ID` as a provider for existent resource.
* `createResourceAndRegisterProvider` - creates a resource first and then registers  `INIT_PEER_ID` as a provider for it.

And these are needed to register a node provider for a resource:

* `registerNodeProvider` - registers the given node as a provider for an existing resource.
* `createResourceAndRegisterNodeProvider` - creates a resource first and then registers the given node as a provider.

Now, let's review them in more detail.

#### `createResourceAndRegisterProvider` & `registerProvider`

These functions register the **caller** of a script as a provider:

* `createResourceAndRegisterProvider` creates a resource prior to registration
* `registerProvider` simply adds a registration as a provider for existing resource.

#### `createResourceAndRegisterNodeProvider` & `registerNodeProvider`

These two functions work almost the same as their non-`Node` counterparts, except that they register a **node** instead of a **caller**. This is useful when you want to register a **service** hosted on a **node**.

Records created by these two functions live through garbage collection unlike records created  by `registerProvider.`

Here's how you could use it in TypeScript:

{% hint style="info" %}
You first need to have `export.aqua` file and compile it to TypeScript, see [here](./#in-typescript-and-javascript)
{% endhint %}

```typescript
import {Fluence, KeyPair} from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";
import {registerNodeProvider, createResource, registerProvider, resolveProviders} from "./generated/export";
import assert from "assert";

async function main() {
    // create the first peer and connect it to the network
    await Fluence.start({ connectTo: krasnodar[1] });
    console.log(
        "📗 created a fluence peer %s with relay %s",
        Fluence.getStatus().peerId,
        Fluence.getStatus().relayPeerId
    );

    let label = "myLabel";
    console.log("Will create resource with label:", label);
    let [resource_id, create_error] = await createResource(label);
    assert(resource_id !== null, create_error.toString());
    console.log("resource %s created successfully", resource_id);
    
    let value = "myValue";
    let node_provider = krasnodar[4].peerId;
    let service_id = "identity";
    let [node_success, reg_node_error] = await registerNodeProvider(node_provider, resource_id, value, service_id);
    assert(node_success, reg_node_error.toString());
    console.log("node %s registered as provider successfully", node_provider);

    let [success, reg_error] = await registerProvider(resource_id, value, service_id);
    console.log("peer %s registered as provider successfully", Fluence.getStatus().peerId);
    assert(success, reg_error.toString());

    let [providers, error] = await resolveProviders(resource_id, 2);
    // as a result we will see two providers records
    console.log("resource providers:", providers);
}

main().then(() => process.exit(0))
    .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

#### Renew Record Periodically

After a non-host record is created, it must be used at least once an hour to keep it from being marked **stale** and deleted. Also, peers must renew themselves at least once per 24 hours to prevent record **expiration** and deletion.

While this collection schedule may seem aggressive, it keeps the Registry up-to-date and performant as short-lived client-peers, such as browsers, can go offline at any time or periodically change their relay nodes.

### Call A Function On Resource Providers

#### `executeOnProviders`

`registry` provides a function to callback on every **Record** associated with a resource:

```haskell
func executeOnProviders(resource_id: string, ack: i16, call: Record -> ())
```

It reduces boilerplate when writing an Aqua script that calls a (common) function on each provider. For example:

```haskell
import "@fluencelabs/registry/resources-api.aqua"

-- API that every subscriber must adhere to
-- You can think of it as an application protocol
service ProviderAPI:
    do_smth(value: string)

func call_provider(p: Record):
    -- topological move to provider via relay
    on p.peer_id via p.relay_id:
        -- resolve service on a provider
        ProviderAPI p.service_id
        -- call function
        ProviderApi.do_smth(p.value)

-- call ProviderApi.do_smth() on every provider
func call_everyone(resource_id: String, ack: i16):
    executeOnProviders(resource_id, ack, call_provider)
```

#### Passing Data To Providers

Due to the limitations in callbacks, `executeOnProviders` doesn't allow us to send dynamic data to providers. However, this limitation is easily overcome by using a `for` loop:

Consider this Aqua code:

```haskell
import "@fluencelabs/registry/resources-api.aqua"

-- Application event
data Event:
    value: string

-- API that every provider must adhere to
-- You can think of it as an application protocol
service ProviderAPI:
    receive_event(event: Event)

func call_provider(p: Record, event: Event):
    -- topological move to provider via relay
    on p.peer_id via p.relay_id:
        -- resolve service on a provider
        ProviderAPI p.service_id
        -- call function
        ProviderAPI.receive_event(event)

-- send event to every provider
func send_everyone(resource_id: String, ack: i16, event: Event):
    -- retrieve all providers of a resource
    providers <- resolveProviders(resource_id, ack)
    -- iterate through them
    for p <- providers par:
        call_provider(p, event)
```

### Handling Function Calls

[Fluence JS SDK](https://github.com/fluencelabs/fluence-js) allows JS/TS peers to define their API through services and functions.&#x20;

Let's take the `ProviderApi` from the previous example and extend it a little:

```haskell
data Event:
    value: string
    
service ProviderAPI:
    -- receive an event
    receive_event(event: Event)
    -- do something and return data
    do_something(value: string) -> u64

```

Let's save this file to `provider_api.aqua` and compile it

```
aqua -i . -o src/generated
```

```typescript
import { Fluence } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";
import { registerProviderAPI, ProviderAPIDef } from "./generated/provider_api"
    
async function main() {
    await Fluence.start({ connectTo: krasnodar[2] });
    
    let service_id = 'api';
    let counter = 0;
    
    await registerProviderAPI(service_id, {
        receive_event: (event: any) => {
            console.log("event received!", event);
        },
        do_something: (value: any) => {
            counter += 1;
            console.log("doing logging!", value, counter);
            return counter;
        }
    });
}

main().then(() => process.exit(0))
    .catch(error => {
        console.error(error);
        process.exit(1);
    });
```

### Overcoming The Record Limit

If your app requires more than 32 providers for a single resource, then it's time to think about a custom WebAssembly service that stores all these records. Basically a simple "records directory" service.

With such a service implemented and deployed, you can use `resources-api.aqua` to register that "records directory" service and host as provider. Depending on your app's architecture, you might want to have several instances of "records directory" service.&#x20;

The code to get all records from "directory" services might look something like this in Aqua:

```haskell
import "@fluencelabs/registry/resources-api.aqua"

service RecDir:
    get_records(resource_id: string) -> []Record

func dir_subscribers(resource_id: String, ack: i16) -> [][]Record:
    -- this stream will hold all records
    allRecs: *[]Record
    -- retrieve RecDir records from Registry
    providers <- resolveProviders(resource_id, ack)
    -- iterate through all RecDir services
    for dir <- providers:
        on dir.peer_id:
            RecDir dir.service_id
            -- get all records from RecDir and write to allRecs
            allRecs <- SubDir.get_records(resource_id)
    <- allRecs
```

## Concepts

### Kademlia Neighborhood

Fluence nodes participate in the Kademlia network. Kademlia organizes peers in such a way that given any key, you can find a set of peers that are "responsible" for that key. That set contains up to 20 nodes.

That set is called "neighborhood" or "K-closest nodes" (K=20). In Aqua, it is accessible in `aqua-lib` via the `Kademlia.neighbourhood` function.

The two most important properties of the Kademlia neighborhood are: \
1\) it exists for _any_ key \
2\) it is more or less stable

### Data Replication

#### On write

When a registration as a provider for a resource is done, it is written to the Kademlia neighborhood of that resource\_id. Here's a `registerProvider` implementation in Aqua:

```haskell
-- Register for a resource as provider
-- Note: resource must be already created
func registerProvider(resource_id: ResourceId, value: string, service_id: ?string) -> bool, *Error:
  success: *bool
  error: *Error
  relay_id: ?string
  relay_id <<- HOST_PEER_ID

  t <- Peer.timestamp_sec()

  on HOST_PEER_ID:
    record_sig_result <- getRecordSignature(resource_id, value, relay_id, service_id, t)

    if record_sig_result.success == false:
      error <<- record_sig_result.error!
      success <<- false
    else:
      record_signature = record_sig_result.signature!
      -- find resource
      key, error_get <- getResource(resource_id)
      if key == nil:
        appendErrors(error, error_get)
        success <<- false
      else:
        successful: *bool
        -- get neighbourhood for the resource_id
        nodes <- getNeighbours(resource_id)
        -- iterate through each node in the neighbourhood
        for n <- nodes par:
          error <<- n
          on n:
            try:
              -- republish key/resource
              republish_res <- republishKey(key!)
              if republish_res.success == false:
                error <<- republish_res.error
              else:
                -- register as a provider on each node in the neighbourhood
                put_res <- putRecord(resource_id, value, relay_id, service_id, t, record_signature)
                if put_res.success:
                  successful <<- true
                else:
                  error <<- put_res.error

        timeout: ?string
        -- at least one successful write should be performed
        join successful[0]
        par timeout <- Peer.timeout(5000, "provider hasn't registered: timeout exceeded")

        if timeout == nil:
          success <<- true
        else:
          success <<- false
          error <<- timeout!

  <- success!, error
```

This ensures that data is replicated across several peers.

#### At rest

Resource Keys and Provider records are also replicated "at rest". That is, once per hour all **stale** keys and records are removed and replicated to all nodes in the neighborhood, once per day all **expired** keys and records are removed. ****&#x20;

This ensures that even if a neighborhood for a **resource\_id** has changed due to some peers go offline and others join the network, data will be replicated to all nodes in the neighborhood.

...

{% hint style="info" %}
For advanced users accustomed to Aqua scripts:

There's an implementation of "at rest" replication for Registry [on GitHub](https://github.com/fluencelabs/registry/blob/main/aqua/registry-scheduled-scripts.aqua)
{% endhint %}
