---
description: Aqua implementation of DHT and PubSub
---

# @fluencelabs/aqua-dht

`aqua-dht` provides PubSub API that can be used for service discovery and event delivery. 

## API

For the API reference, take a look at [pubsub.aqua](https://github.com/fluencelabs/aqua-dht/blob/9a72961/aqua/pubsub.aqua) in the [fluencelabs/aqua-dht](https://github.com/fluencelabs/aqua-dht) repo.

## Terminology

* **@fluencelabs/aqua-dht-ts** - TypeScript/JS library on NPM. Provides a precompiled TS/JS PubSub API. Suits basic use-cases, easy to start with.
* **@fluencelabs/aqua-dht** - Aqua library on NPM. Provides high level and low-level APIs to develop custom Aqua scripts. Suits more advanced use-cases.

{% hint style="success" %}
Both **aqua-dht-ts** and **aqua-dht** can be used with TypeScript and JavaScript.
{% endhint %}

* **topic** - `string` with associated `peer_id` and a list of **subscribers**. Associated `peer_id` can be thought of as a **topic creator**.
* **topic creator** - `peer id` that created the topic. Other users can't create a topic with the same name.
* **subscriber** - a peer that has called `subscribe` on a **topic**, optionally with associated **relay\_id** and **service\_id**. Any peer can be a subscriber, including nodes and clients.
* subscriber's **relay\_id** - subscriber is available on the network through this relay. 

{% hint style="info" %}
When a **subscriber** doesn't have a publicly available address \(e.g. it's a browser\), it connects to the network through a relay node. That means that _other_ peers only can access that **subscriber** through a relay. 

In that case `subscribe` must be called with `relay_id`, so other peers can reach the **subscriber**. 
{% endhint %}

* subscriber's **service\_id** - id of the service provided by that subscriber. Sometimes subscriber wants **publishers** of a specific **topic** to call functions on this **service\_id**, so it's possible to distinguish service calls.
* **subscription** - a **dht record** associated with a **topic.** Holds information about a **subscriber**.
* **subscription lifetime** - **subscriptions** have two are evicted \(removed\) [24 hours](https://github.com/fluencelabs/aqua-dht/blob/9a72961/service/src/main.rs#L39) after their creation or after [1 hour](https://github.com/fluencelabs/aqua-dht/blob/9a72961/service/src/main.rs#L38) of being unused.
* **subscriptions limit** - each **topic** can have at most [20 subscribers](https://github.com/fluencelabs/aqua-dht/blob/9a72961/service/src/main.rs#L41). Each new subscriber will remove an old one, following a LIFO principle.
* **publisher** - any peer that has retrieved subscribers of a topic and calls functions on these subscribers
* **AquaDHT** - a service that provides low-level DHT API. `pubsub.aqua` is built on top of it.
* **dht key** - a low-level representation of a topic
* **dht record** - a low-level representation of a subscriber, see [Record](https://github.com/fluencelabs/aqua-dht/blob/9a729611c6da4930b5f145696bfdc975d1227e77/aqua/dht.aqua#L18) in Aqua
* **host value** - a **dht record** with `peer_id` being that of a node. When a node is subscribed to a topic via `subscribeNode` or `initTopicAndSubscribeNode`, the **subscription** is a **host value**. Host values live much longer \([10 days](https://github.com/fluencelabs/aqua-dht/blob/9a72961/service/src/main.rs#L40)\) than other **dht records**. [See Subscribe to a topic](aqua-dht.md#subscribe-to-a-topic) for details.
* **script caller** - peer that executes a script by sending it to the network. In Aqua it's `%init_peer_id%`
* **node** - usually a Fluence node hosted in the cloud. Nodes are long-lived, can host WebAssembly services and they participate in the Kademlia network.

## How to use

{% hint style="info" %}
There are [several simple examples](https://github.com/fluencelabs/aqua-dht/tree/9a72961/examples) in the fluencelabs/aqua-dht repo, give them a look.
{% endhint %}

### Create a topic

Before subscribing to a topic is possible, that topic must be created. That's exactly what `initTopic` does. 

Here's a rather synthetic example in Aqua:

```haskell
import "@fluencelabs/aqua-dht/pubsub.aqua"

type PeerId: string

func my_function(relay: PeerId, topic: string):
    initTopic(relay, topic)
```

In TypeScript, you would do:

```typescript
import { initTopic } from "@fluencelabs/aqua-dht-ts";
import { createClient } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";

// connect to the Fluence network
const client = await createClient(krasnodar[1]);
let topic = "myTopic";
await initTopic(client, client.relayPeerId!, "myTopic");
```

### Subscribe to a topic

There are 4 functions that achieve subscriptions. Let's review them.

#### `initTopicAndSubscribe` & `subscribe`

These two subscribe the **caller** of a script to a topic. `initTopicAndSubscribe` will create a topic before subscribing, so be careful **not** to call it on someone else's topic. `subscribe` simply adds a subscription on an existing topic.

Here's how you could use it in TypeScript:

```typescript
import { initTopicAndSubscribe, subscribe, findSubscribers } from "@fluencelabs/aqua-dht-ts";
import { createClient } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";

const relayA = krasnodar[1];
// connect to the Fluence network
const clientA = await createClient(relayA);
let topic = "myTopic";
let value = "put anything useful here";
let serviceId = "Foo";
// create topic and subscribe to it
await initTopicAndSubscribe(clientA, relayA, topic, value, relayA, serviceId);
// this will contain clientA's subscription
var subscribers = await findSubscribers(client, relayA, topic);

// now, let's create a second client
const relayB = krasnodar[2];
const clientB = await createClient(relayB);
// and subscribe it to the same topic
await subscribe(clientB, relayB, topic);
// this will contain both clientA and clientB subscriptions
subscribers = await findSubscribers(client, relayA, topic);
```

#### `initTopicAndSubscribeNode` & `subscribeNode`

These two functions work almost the same as their non-`Node` counterparts, except that they subscribe **node** instead of a **caller**. This is useful when you want to subscribe a **service** hosted on the **node** on a topic.

Such subscriptions live 10 times longer than these by `subscribe`, because services can't renew subscriptions on their own.

#### Renew subscription periodically

After a normal \(non-host\) subscription is created, it must be used at least once an hour to keep it from being marked **stale** and deleted. Also, peers must resubscribe themselves at least once per 24 hours to prevent subscription **expiration** and deletion.

That might sound harsh, but it makes sense for short-lived browser-based peers that can go offline at any time, and change their relay nodes from time to time. Steady clean-up of **stale** and **expired** subscriptions is required to keep PubSub up-to-date and performant.

### Call a function on subscribers

#### `executeOnSubscribers`

`aqua-dht` provides a function to call a callback on every **Record** associated with a topic:

```haskell
func executeOnSubscribers(node_id: PeerId, topic: string, call: Record -> ())
```

It saves typing when writing a custom Aqua script to call a function on each subscriber. Here's an example:

```haskell
import "@fluencelabs/aqua-dht/pubsub.aqua"

-- API that every subscriber must adhere to
-- You can think of it as an application protocol
service SubscriberAPI:
    something_happened(value: string)

func call_subscriber(sub: Record):
    -- topological move to subscriber via relay
    on sub.peer_id via sub.relay_id:
        -- resolve service on a subscriber
        SubscriberAPI sub.service_id
        -- call function
        SubscriberAPI.something_happened(sub.value)

-- call SubscriberAPI.something_happened() on every subscriber
func call_everyone(relay: PeerId, topic: string):
    executeOnSubscribers(relay, topic, call_subscriber)
```

#### Passing data to subscribers

However, because of how callbacks work, currently `executeOnSubscribers` doesn't allow us to send dynamic data to subscribers. Overcoming this is as easy as writing a for loop:

Consider this Aqua code:

```haskell
import "@fluencelabs/aqua-dht/pubsub.aqua"

-- Application event
data Event:
    value: string

-- API that every subscriber must adhere to
-- You can think of it as an application protocol
service SubscriberAPI:
    receive_event(event: Event)

func call_subscriber(sub: Record, event: Event):
    -- topological move to subscriber via relay
    on sub.peer_id via sub.relay_id:
        -- resolve service on a subscriber
        SubscriberAPI sub.service_id
        -- call function
        SubscriberAPI.receive_event(event)

-- send event to every subscriber
func send_everyone(relay: PeerId, topic: string, event: Event):
    -- retrieve all subscribers of a topic
    subscribers <- findSubscribers(relay, topic)
    -- iterate through them
    for sub <- subscribers par:
        call_subscriber(sub, event)
```

### Handle function calls

`subscribeToEvent` function from the [Fluence JS SDK](https://github.com/fluencelabs/fluence-js) allows JS/TS peers to define their API through services and functions. 

Let's take `SubscriberAPI` from the previous example, and extend it a little:

```haskell
data Event:
    value: string
    
data SubscriberAPI:
    -- receive an event
    receive_event(event: Event)
    -- do something and return data
    do_something(value: string) -> string

```

Here's how to define such API in TS/JS:

```typescript
import { createClient, registerServiceFunction } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";

const client = await createClient(krasnodar[1]);

let service_id = 'api';

registerServiceFunction(client, service_id, 'receive_event', args => {
    let [event] = args;
    console.log("event received! {}", event);
});

registerServiceFunction(client, service_id, 'do_something', args => {
    let [value] = args;
    console.log("doing logging! {}", value);
    return "OK";
});
```

### Overcome the limit of subscribers

If your app requires to have more than 20 subscribers on a single topic, then it's time to think about a custom WebAssembly service that would store all these subscriptions in memory or on disk. Basically a simple subscriptions "directory".

With such a service implemented and deployed, you can use `pubsub.aqua` to subscribe that "subscriptions directory" service to a topic. Depending on your app's architecture, you might want to have several instances of "subscriptions directory" service. 

The code to get all subscriptions from "directory" services might look something like this in Aqua:

```haskell
import "@fluencelabs/aqua-dht/pubsub.aqua"

service SubDir:
    get_subscribers(topic: string) -> []Record

func dir_subscribers(relay: PeerId, topic: string) -> [][]Record:
    -- this stream will hold all subscribers
    allSubs: *[]Record
    -- retrieve SubDir subscribers from AquaDHT
    subscribers <- getSubscribers(relay, topic)
    -- iterate through all SubDir services
    for dir <- subscribers:
        on dir.peer_id:
            SubDir dir.service_id
            -- get all subscribers from SubDir and write to allSubs
            allSubs <- SubDir.get_subscribers(topic)
    <- allSubs
```

## Concepts

### Kademlia neighbourhood

Fluence nodes participate in the Kademlia network. Kademlia organizes peers in such a way that given any key, you can find a set of peers that are "responsible" for that key. That set contains up to 20 nodes.

That set is called "neighbourhood" or "K-closest nodes" \(K=20\). In Aqua, it is accessible in `aqua-lib` via `Kad.neighbourhood` function.

The two most important properties of the Kademlia neighbourhood are:   
1\) it exists for _any_ key   
2\) it is more or less stable

A lot of DHTs rely on these properties to implement data replication. So does AquaDHT.

### Data replication

#### On write

When a subscription to a topic is created, it is written to the Kademlia neighbourhood of that topic. Here's a `subscribe` implementation in Aqua:

```haskell
func subscribe(node_id: PeerId, topic: string, value: string, relay_id: ?PeerId, service_id: ?string):
  -- get neighbourhood for a topic
  nodes <- getNeighbours(node_id, topic)
  -- iterate through each node in the neighbourhood
  for n <- nodes par:
    on n:
      try:
        t <- Peer.timestamp_sec()
        -- store subscription on each node in the neighbourhood
        AquaDHT.put_value(topic, value, t, relay_id, service_id, 0)
```

This ensures that data is replicated across several peers.

#### At rest

Subscriptions are also [replicated at rest](https://github.com/fluencelabs/fluence/tree/2320204/deploy/builtins/aqua-dht/scheduled). That is, once per hour all **stale** values are removed, and non-**stale** values are replicated to all nodes in the neighbourhood. 

This ensures that even if the neighbourhood for a topic has changed due to some peers go offline and others join the network, data is replicated to all nodes in the neighbourhood.

