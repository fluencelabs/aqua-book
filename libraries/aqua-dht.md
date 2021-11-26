---
description: Aqua implementation of DHT and PubSub
---

# @fluencelabs/aqua-dht

`aqua-dht` provides PubSub API that can be used for service discovery and event delivery.&#x20;

## Releases

You can find the latest releases of `aqua-dht` [on NPM](https://www.npmjs.com/package/@fluencelabs/aqua-dht) and changelogs are [on GitHub](https://github.com/fluencelabs/aqua-dht/releases)

## API

For the API reference, take a look at [pubsub.aqua](https://github.com/fluencelabs/aqua-dht/blob/9a72961/aqua/pubsub.aqua) in the AquaDHT repo.

## Terminology

* **@fluencelabs/aqua-dht** - Aqua library on NPM. Provides high level and low-level APIs to develop custom Aqua scripts. Suits more advanced use-cases.
* **PubSub **- short for Publish-Subscribe. A pattern for messaging and discovery. **Subscribers** create **subscriptions** on a **topic **so that **publishers** send messages to subscribers.
* **Kademlia** - algorithm for organizing a peer-to-peer network in such a way that peers can find each other in no more than O(logN) hops, where N is the size of the network.
* **topic** - `string` with associated `peer_id` and a list of **subscribers**. Associated `peer_id` can be thought of as a **topic creator**.
* **topic creator **- `peer id` that created the topic. Other users can't create a topic with the same name.
* **subscriber **- a peer that has called `subscribe` on a **topic**, optionally with an associated **relay\_id **and **service\_id**. Any peer can be a subscriber, including nodes and clients.
* subscriber's **relay\_id** - subscriber is available on the network through this relay.&#x20;

{% hint style="info" %}
When a **subscriber** doesn't have a publicly available IP address, e.g. the client peer is a browser, it connects to the network through a relay node. That means that _other_ peers only can connect to that **subscriber** through a relay.&#x20;

In that case `subscribe` must be called with the`relay_id`, so other peers can reach the **subscriber**.&#x20;
{% endhint %}

* subscriber's **service\_id** - id of the service provided by that subscriber. Sometimes a subscriber may want **publishers **of a specific **topic** to call functions on this **service\_id** in order to be able to distinguish among service calls.
* **subscription **- a **DHT record** associated with a **topic. **Holds information about a **subscriber**.
* **subscription lifetime **- **subscriptions **have two are evicted (removed) [24 hours](https://github.com/fluencelabs/aqua-dht/blob/9a72961/service/src/main.rs#L39) after their creation or after [1 hour](https://github.com/fluencelabs/aqua-dht/blob/9a72961/service/src/main.rs#L38) of being unused.
* **subscriptions limit **- each **topic** can have at most [20 subscribers](https://github.com/fluencelabs/aqua-dht/blob/9a72961/service/src/main.rs#L41). Each new subscriber will remove an old one, following a LIFO principle.
* **publisher **- any peer that has retrieved subscribers of a topic and calls functions on these subscribers
* **AquaDHT** - a service that provides low-level DHT API. `pubsub.aqua` is built on top of it.
* **DHT key **- a low-level representation of a topic
* **DHT record **- a low-level representation of a subscriber, see [Record](https://github.com/fluencelabs/aqua-dht/blob/9a729611c6da4930b5f145696bfdc975d1227e77/aqua/dht.aqua#L18) in Aqua
* **host value** - a **DHT record** with `peer_id` being that of a node. When a node is subscribed to a topic via `subscribeNode` or `initTopicAndSubscribeNode`, the **subscription** is a **host value**. Host values live much longer ([10 days](https://github.com/fluencelabs/aqua-dht/blob/9a72961/service/src/main.rs#L40)) than other **DHT records**. [See Subscribe to a topic](aqua-dht.md#subscribe-to-a-topic) for details.
* **script caller** - peer that executes a script by sending it to the network. In Aqua it's `%init_peer_id%`
* **node **- usually a Fluence node hosted in the cloud. Nodes are long-lived, can host WebAssembly services and they participate in the Kademlia network.

## How To Use Aqua-DHT

{% hint style="info" %}
There are [several simple examples](https://github.com/fluencelabs/aqua-dht/tree/3598c9044b90845a69c8f1c3306409a41d49e7e3#how-to-use) in the fluencelabs/aqua-dht repo, give them a look.
{% endhint %}

### Create A Topic

Before subscribing to a topic is possible, that topic must be created. That's exactly what `initTopic` does.&#x20;

Here's a rather simplistic example in Aqua:

```haskell
import "@fluencelabs/aqua-dht/pubsub.aqua"

type PeerId: string

func my_function(relay: PeerId, topic: string):
    initTopic(relay, topic)
```

### Subscribe To A Topic

There are 4 functions that create subscriptions. Let's review them.

There are four functions that achieve subscriptions. Let's review them.

These you would use for most of your needs:

* `subscribe` - subscribes `%init_peer_id%` to an existing topic.
* `initTopicAndSubscribe` - creates a topic first and then subscribes `%init_peer_id%` to it.

And these are needed to subscribe a service to a topic:

* `subscribeNode` - subscribes a node to an existing topic.
* `initTopicAndSubscribeNode` - creates a topic first and then subscribes a node to it.

Now, let's review them in more detail.

#### `initTopicAndSubscribe` & `subscribe`

These two subscribe the **caller** of a script to a topic. `initTopicAndSubscribe` will create a topic before subscribing, so be careful **not** to call it on someone else's topic. `subscribe` simply adds a subscription on an existing topic.

Here's how you could use it in TypeScript:

{% hint style="info" %}
You first need to have `export.aqua` file and compile it to TypeScript, see [here](./#in-typescript-and-javascript)
{% endhint %}

```typescript
import { FluencePeer } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";
import { initTopicAndSubscribeBlocking, findSubscribers, subscribe } from "./generated/export";

const relayA = krasnodar[1];
// create the first peer and connect it to the network
const peerA = new FluencePeer();
await peerA.start({ connectTo: relayA });

let topic = "myTopic";
let value = "put anything useful here";
let serviceId = "Foo";
// create topic and subscribe to it
await initTopicAndSubscribeBlocking(
    peerA,
    topic, value, relayA, serviceId, 
    (s) => console.log(`node ${s} saved the record`)
);
// this will contain peerA's subscription
var subscribers = await findSubscribers(peerA, topic);

const relayB = krasnodar[2];
// now, let's create the second peer and connect to the network
const peerB = new FluencePeer();
await peerB.start({ connectTo: relayB });

// and subscribe it to the same topic
await subscribe(peerB, topic);
// this will contain both peerA and peerB subscriptions
subscribers = await findSubscribers(peerB, topic);
```

{% hint style="info" %}
There is`initTopicAndSubscribeBlocking and initTopicAndSubscribe. `

`Blocking` version waits until at least a single write is done. The latter version is "fire and forget": it `awaits`instantly, but doesn't guarantee that write has happened.
{% endhint %}



#### `initTopicAndSubscribeNode` & `subscribeNode`

These two functions work almost the same as their non-`Node` counterparts, except that they subscribe **node** instead of a **caller**. This is useful when you want to subscribe a **service** hosted on the **node **on a topic.

Such subscriptions live 10 times longer than these by `subscribe`, because services can't renew subscriptions on their own.

#### Renew Subscription Periodically

After a normal (non-host) subscription is created, it must be used at least once an hour to keep it from being marked **stale** and deleted. Also, peers must resubscribe themselves at least once per 24 hours to prevent subscription **expiration **and deletion.

While this collection schedule may seem aggressive, it keeps keep PubSub up-to-date and performant as short-lived client-peers, such as browsers, can go offline at any time or periodically change their relay nodes.

### Call A Function On Subscribers

#### `executeOnSubscribers`

`aqua-dht` provides a function to call a callback on every **Record** associated with a topic:

```haskell
func executeOnSubscribers(node_id: PeerId, topic: string, call: Record -> ())
```

It reduces boilerplate when writing a custom Aqua script to call a function on each subscriber. Here's an example:

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

### Handle Function Calls

`subscribeToEvent` function from the [Fluence JS SDK](https://github.com/fluencelabs/fluence-js) allows JS/TS peers to define their API through services and functions.&#x20;

Let's take the  `SubscriberAPI` from the previous example and extend it a little:

```haskell
data Event:
    value: string
    
service SubscriberAPI:
    -- receive an event
    receive_event(event: Event)
    -- do something and return data
    do_something(value: string) -> u64

```

Let's save this file to `subscriber_api.aqua` and compile it

```
npx aqua -i . -o src/generated
```

```typescript
import { Fluence } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";
import { registerSubscriberAPI, SubscriberAPIDef } from "./generated/subscriber_api"

await Fluence.start({ connectTo: krasnodar[2] });

let service_id = 'api';
let counter = 0;

await registerSubscriberAPI(service_id, {
    receive_event: (event: any) => {
        console.log("event received!", event);
    },
    do_something: (value: any) => {
        counter += 1;
        console.log("doing logging!", value, counter);
        return counter;
    }
});

```

### Overcome The Subscriber Limit

If your app requires to have more than 20 subscribers on a single topic, then it's time to think about a custom WebAssembly service that would store all these subscriptions in memory or on disk. Basically a simple subscriptions "directory".

With such a service implemented and deployed, you can use `pubsub.aqua` to subscribe that "subscriptions directory" service to a topic. Depending on your app's architecture, you might want to have several instances of "subscriptions directory" service.&#x20;

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

### Kademlia Neighborhood

Fluence nodes participate in the Kademlia network. Kademlia organizes peers in such a way that given any key, you can find a set of peers that are "responsible" for that key. That set contains up to 20 nodes.

That set is called "neighborhood" or "K-closest nodes" (K=20). In Aqua, it is accessible in `aqua-lib` via the `Kad.neighbourhood` function.

The two most important properties of the Kademlia neighborhood are: \
1\) it exists for _any_ key \
2\) it is more or less stable

A lot of DHTs rely on these properties to implement data replication. So does AquaDHT.

### Data replication

#### On write

When a subscription to a topic is created, it is written to the Kademlia neighborhood of that topic. Here's a `subscribe` implementation in Aqua:

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

Subscriptions are also replicated "at rest". That is, once per hour all **stale** values are removed, and non-**stale** values are replicated to all nodes in the neighborhood.&#x20;

This ensures that even if a neighborhood for a topic has changed due to some peers go offline and others join the network, data will be replicated to all nodes in the neighborhood.

{% hint style="info" %}
For advanced users accustomed to AIR scripts:

There's an implementation of "at rest" replication for AquaDHT [on GitHub](https://github.com/fluencelabs/fluence/tree/2320204/deploy/builtins/aqua-dht/scheduled)
{% endhint %}
