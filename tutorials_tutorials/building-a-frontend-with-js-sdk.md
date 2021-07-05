# Building a Frontend with JS-SDK

Fluence provides means to connect to the network from a javascript environment. It is currently tested to work in nodejs and modern browsers.

To create an application you will need two main building blocks: the JS SDK itself and the aqua compiler. Both of them are provided in a form npm packages. JS SDK wraps the air interpreter and provides a connection to the network. There is low-level api for executing air scripts and registering for service calls handlers. Aqua compiler allows to write code in aqua language and compile it into typescript code which can be directly used with the SDK.

Even though all the logic could be programmed by hand with raw air it is strongly recommended to use aqua.

### Basic usage

As previously said you can use Fluence with any frontend or nodejs framework. JS SDK could be as any other npm library. For the purpose of the demo we will init a bare-bones nodejs package and show you the steps needed install JS SDK and aqua compiler. Feel free to use the tool most suitable for the framework used in application, the installation process should be roughly the same

#### 1. Start with npm package

For the purpose of the demo we will use a very minimal npm package with typescript support:

```text
src
 ┗ index.ts           (1)
package.json          (2)
tsconfig.json
```

index.ts \(1\):

```typescript
async function main() {
  console.log("Hello, world!");
}

main();
```

package.json \(2\):

```javascript
{
  "name": "demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "exec": "node -r ts-node/register src/index.ts"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "ts-node": "^9.1.1",
    "typescript": "^4.2.4"
  }
}
```

Let's test if it works:

```bash
$ npm install
$ npm run exec
```

The following should be printed

```bash
$ npm run exec

> demo@1.0.0 exec C:\work\demo
> node -r ts-node/register src/index.ts

Hello, world!
$ C:\work\demo>
```

#### 2. Setting JS SDK and connecting to Fluence network

Install the dependencies, you will need these two packages.

```bash
npm install @fluencelabs/fluence @fluencelabs/fluence-network-environment
```

The first one is the SDK itself and the second is a maintained list of Fluence networks and nodes to connect to.

All of the communication with the Fluence network is done by using `FluenceClient`. You can create one with `createClient` function. The client encapsulates air interpreter and connects to the network through the relay. Currently any node in the network can be used a relay.

```typescript
import { createClient } from "@fluencelabs/fluence";
import { testNet } from "@fluencelabs/fluence-network-environment";

async function main() {
  const client = await createClient(testNet[1]);
  console.log("Is client connected: ", client.isConnected);

  await client.disconnect();
}

main();
```

Let's try it out:

```bash
$ npm run exec

> demo@1.0.0 exec C:\work\demo
> node -r ts-node/register src/index.ts

Is client connected:  true
$
```

**Note**: typically you should have a single instance`FluenceClient` per application since it represents it's identity in the network. You are free to store the instance anywhere you like.

#### 3. Setting Up The  Aqua Compiler

Aqua is the proffered language for the Fluence network. It can be used with javascript-based environments via npm package.

{% hint style="warning" %}
**The package requires java to be installed as it calls "java -jar ... "** 
{% endhint %}

```bash
npm install --save-dev @fluencelabs/aqua-cli
```

We will also need the standard library for the language

```text
npm install --save-dev @fluencelabs/aqua-lib
```

Let's add our first aqua file:

```text
aqua                   (1)
 ┗ demo.aqua           (2)
node_modules
src
 ┣ compiled            (3)
 ┗ index.ts     
package-lock.json
package.json          
tsconfig.json
```

Add two directories, one for aqua sources \(1\) and another for the typescript output \(3\)

Create a new text file called `demo.aqua` \(2\):

```text
import "@fluencelabs/aqua-lib/builtin.aqua"

func demo(nodePeerId: PeerId) -> []string:
    on nodePeerId:
        res <- Peer.identify()
    <- res.external_addresses
```

This script will gather the list of external addresses from some node in the network. For more information about the aqua language refer to the aqua documentation.

The aqua code can now be compiled by using the compiler CLI. We suggest adding a script to the package.json file like so:

```javascript
...
  "scripts": {
    "exec": "node -r ts-node/register src/index.ts",
    "compile-aqua": "aqua-cli -i ./aqua/ -o ./src/compiled"
  },
...
```

Run the compiler:

```bash
npm run compile-aqua
```

A typescript file should be generated like so:

```text
aqua                   
 ┗ demo.aqua           
node_modules
src
 ┣ compiled
 ┃ ┗ demo.ts           <--
 ┗ index.ts     
package-lock.json
package.json          
tsconfig.json
```

#### 4. Consuming The Compiled Code

Using the code generated by the compiler is as easy as calling a function. The compiler generates all the boilerplate needed to send a particle into the network and wraps it into a single call. Note that all the type information and therefore type checking and code completion facilities are there!

Let's do it!

```typescript
import { createClient } from "@fluencelabs/fluence";
import { testNet } from "@fluencelabs/fluence-network-environment";

import { demo } from "./compiled/demo"; // import the generated file

async function main() {
  const client = await createClient(testNet[1]);
  console.log("Is client connected: ", client.isConnected);

  const otherNode = testNet[2].peerId;
  const addresses = await demo(client, otherNode); // call it like a normal function in typescript
  console.log(`Node ${otherNode} is connected to: ${addresses}`);

  await client.disconnect();
}

main();
```

if everything is fine you have similar result:

```text
$ npm run exec

> demo@1.0.0 exec C:\work\demo
> node -r ts-node/register src/index.ts

Is client connected:  true
Node 12D3KooWHk9BjDQBUqnavciRPhAYFvqKBe4ZiPPvde7vDaqgn5er is connected to: /ip4/138.197.189.50/tcp/7001,/ip4/138.197.189.50/tcp/9001/ws

$
```

### Advanced usage

Fluence JS SDK gives options to register own handlers for aqua vm service calls

**TBD**

### References

* For the list of compiler options see: [https://github.com/fluencelabs/aqua](https://github.com/fluencelabs/aqua)
* Repository with additional examples: [**https://github.com/fluencelabs/aqua-playground**](https://github.com/fluencelabs/aqua-playground)\*\*\*\*

