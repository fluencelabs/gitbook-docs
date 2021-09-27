# Running app in nodejs

## Intro

It is easy to use Fluence JS in NodeJS applications. You can take full advantage of the javascript ecosystem and at the save time expose service to the Fluence Network. That makes is an excellent choice for quick prototyping of applications for Fluence Stack.

## Calc app example

Lets implement a very simple app which simulates a desk calculator. The calculator has internal memory and implements the following set of operations:

* Add a number
* Subtract a number
* Multiply by a number
* Divide by a number
* Get the current memory state
* Reset the memory state to 0.0

First, let's write the service definition in aqua:

```text
-- service definition
service Calc("calc"):
    add(n: f32)
    subtract(n: f32)
    multiply(n: f32)
    divide(n: f32)
    reset()
    getResult() -> f32
```

Now write the implementation for this service in typescript:

```typescript
import { Fluence } from "@fluencelabs/fluence";
import { krasnodar } from "@fluencelabs/fluence-network-environment";
import { registerCalc, CalcDef } from "./_aqua/calc";

class Calc implements CalcDef {
  private _state: number = 0;

  add(n: number) {
    this._state += n;
  }

  subtract(n: number) {
    this._state -= n;
  }

  multiply(n: number) {
    this._state *= n;
  }

  divide(n: number) {
    this._state /= n;
  }

  reset() {
    this._state = 0;
  }

  getResult() {
    return this._state;
  }
}

const keypress = async () => {
  process.stdin.setRawMode(true);
  return new Promise<void>((resolve) =>
    process.stdin.once("data", () => {
      process.stdin.setRawMode(false);
      resolve();
    })
  );
};

async function main() {
  await Fluence.start({
    connectTo: krasnodar[0],
  });

  registerCalc(new Calc());

  console.log("application started");
  console.log("peer id is: ", Fluence.getStatus().peerId);
  console.log("relay is: ", Fluence.getStatus().relayPeerId);
  console.log("press any key to continue");
  await keypress();

  await Fluence.stop();
}

main();
```

As you can see all the service logic has been implemented in typescript. You have full power of npm at your disposal.

Now try running the application:

```bash
> node -r ts-node/register src/index.ts

application started
peer id is:  12D3KooWLBkw4Tz8bRoSriy5WEpHyWfU11jEK3b5yCa7FBRDRWH3
relay is:  12D3KooWSD5PToNiLQwKDXsu8JSysCwUt8BVUJEqCHcDe7P5h45e
press any key to continue
```

And the service can be called from aqua. For example:

```bash
const peer ?= "12D3KooWLBkw4Tz8bRoSriy5WEpHyWfU11jEK3b5yCa7FBRDRWH3"
const relay ?= "12D3KooWSD5PToNiLQwKDXsu8JSysCwUt8BVUJEqCHcDe7P5h45e"

func demoCalculation() -> f32:
    on peer via relay
        Calc.add(10)
        Calc.multiply(5)
        Calc.subtract(8)
        Calc.divide(6)
        res <- Calc.getResult()
    <- res
```

