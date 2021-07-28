# Libraries

`import` declaration allows to use functions, services and data types defined in another module on the file system. While it's a very simple mechanic, together with NPM flexibility it enables full-blown package management in Aqua.

As Aqua is itself compiled to TypeScript, there are usually two kinds of libraries:

* With TypeScript API. Such libraries contain TS precompiled from Aqua, use them when you just want to call functions from TypeScript.
* With Aqua API. You want to use them when writing your own Aqua scripts.

## Available Aqua libraries

* Builtin services API: [@fluencelabs/aqua-lib](changelog/aqua-lib.md) \(see on [NPM](https://www.npmjs.com/package/@fluencelabs/aqua-lib)\)
* PubSub & DHT: [@fluencelabs/aqua-dht](changelog/aqua-dht.md) \(see on [NPM](https://www.npmjs.com/package/@fluencelabs/aqua-dht)\)
* IPFS API: [@fluencelabs/aqua-ipfs](changelog/aqua-ipfs.md) \(see on [NPM](https://www.npmjs.com/package/@fluencelabs/aqua-ipfs)\)

To use library precompiled to TypeScript/JS, add `-ts` suffix, for example:  
`@fluencelabs/aqua-lib` =&gt; `@fluencelabs/aqua-lib-ts` 

## How to use Aqua libraries

To use a library, you first need to download it. It's best done by adding the library to `dependencies` in `package.json` and then doing `npm install`.

{% hint style="info" %}
If you're not familiar with NPM, `package.json` is a project definition. You can specify `dependencies` to be downloaded from [npmjs.org](https://npmjs.org), define custom commands \(`scripts`\) and run them, e.g. `npm run compile-aqua.`

To create an NPM project, run `npm init`and follow instructions.
{% endhint %}

Here's an example of adding `aqua-lib` to `package.json` dependencies:

```javascript
{
  "name": "my-awesome-project",
  "version": "0.0.1",
  "dependencies": {
    "@fluencelabs/aqua-lib": "0.1.10"
  }
}
```

After running `npm i`, you will have `@fluencelabs/aqua-lib` in `node_modules` 

### In Aqua

After the library is downloaded, you can import it in your `.aqua` script as documented in [Imports And Exports](https://fluence.gitbook.io/aqua-book/imports-exports):

```javascript
import "@fluencelabs/aqua-lib/builtin.aqua"
```

Check out corresponding subpages for the API of available libraries.

#### In TypeScript and JavaScript

{% hint style="info" %}
For this example, we'll use  `@fluencelabs/aqua-dht-ts`

\(note the `-ts` suffix, we're using the library with TS/JS API\)
{% endhint %}

To execute Aqua functions, you need to be connected to the Fluence network. The easiest way is to add JS SDK to `dependencies`:

```javascript
  "dependencies": {
    "@fluencelabs/aqua-dht-ts": "0.1.0",
    "@fluencelabs/fluence": "0.9.53",
    "@fluencelabs/fluence-network-environment": "1.0.10"
  }
```

After `npm install` libraries are downloaded, you can use it in TS/JS code right away:

```typescript
import { initTopicAndSubscribe, findSubscribers } from "@fluencelabs/aqua-dht-ts";
import { createClient } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";

async function main() {
    // connect to the Fluence network
    const client = await createClient(krasnodar[1]);
    let topic = "myTopic";
    let value = "myValue";
    // create topic (if not exists) and subscribe on it
    await initTopicAndSubscribe(client, client.relayPeerId!, topic, value, client.relayPeerId!, null);
    // find other peers subscribed to that topic
    let subscribers = await findSubscribers(client, client.relayPeerId!, topic);
    console.log("found subscribers:", subscribers);
}
```

