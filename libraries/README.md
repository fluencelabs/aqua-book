# Libraries

`import` declaration allows to use functions, services and data types defined in another module on the file system. While it's a very simple mechanic, together with NPM flexibility it enables full-blown package management in Aqua.

## Available Aqua Libraries

* Builtin services API: [@fluencelabs/aqua-lib](aqua-lib.md) \(see on [NPM](https://www.npmjs.com/package/@fluencelabs/aqua-lib)\)
* PubSub & DHT: [@fluencelabs/aqua-dht](aqua-dht.md) \(see on [NPM](https://www.npmjs.com/package/@fluencelabs/aqua-dht)\)
* IPFS API: [@fluencelabs/aqua-ipfs](aqua-ipfs.md) \(see on [NPM](https://www.npmjs.com/package/@fluencelabs/aqua-ipfs)\)

## How To Use Aqua Libraries

To use a library, you need to download it by adding the library to `dependencies` in `package.json` and then running `npm install`.

{% hint style="info" %}
If you're not familiar with NPM, `package.json` is a project definition file. You can specify `dependencies` to be downloaded from [npmjs.org](https://npmjs.org) and define custom commands \(`scripts`\), which can be executed from the command line, e.g. `npm run compile-aqua.`

To create an NPM project, run `npm init`in a directory of your choice and follow the instructions.
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

After the library is downloaded, you can import it in your `.aqua` script as documented in [Imports And Exports](../language/header.md):

```javascript
import "@fluencelabs/aqua-lib/builtin.aqua"
```

Check out corresponding subpages for the API of available libraries.

### In TypeScript and JavaScript

To execute Aqua functions, you need to be connected to the Fluence network. The easiest way is to add JS SDK to `dependencies`:

```javascript
  "dependencies": {
    "@fluencelabs/aqua-dht": "0.2.4",
    "@fluencelabs/fluence": "0.13.0",
    "@fluencelabs/fluence-network-environment": "1.0.10"
  }
```

After executing `npm install`, the Aqua API is ready to use. Now you need to export `aqua-dht` functions to TypeScript, that's easy. Create a file `export.aqua`:

```python
import initTopicAndSubscribeBlocking, findSubscribers from "@fluencelabs/aqua-dht/pubsub.aqua"

export initTopicAndSubscribeBlocking, findSubscribers
```

Now, install Aqua compiler and compile your Aqua code to TypeScript: 

```bash
npm install --save-dev @fluencelabs/aqua
npx aqua -i . -o src/generated/
```

That's it. Now let's call some functions on the AquaDHT service:

```typescript
import { Fluence } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";
import { initTopicAndSubscribeBlocking, findSubscribers } from "./generated/export";

async function main() {
    // connect to the Fluence network
    await Fluence.start({ connectTo: krasnodar[1] });
    let topic = "myTopic";
    let value = "myValue";
    let relay = Fluence.getStatus().relayPeerId;
    // create topic (if not exists) and subscribe on it
    await initTopicAndSubscribeBlocking(
      topic, value, relay, null, 
      (s) => console.log(`node ${s} saved the record`)
    );
    // find other peers subscribed to that topic
    let subscribers = await findSubscribers(topic);
    console.log("found subscribers:", subscribers);
}
```

