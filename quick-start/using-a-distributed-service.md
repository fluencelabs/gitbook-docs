# Using A Distributed Service

Let's dive into peer-to-peer awesomeness by harnessing a distributed curl service, which pretty much keeps with its namesake: pass it a url and collect the response. Instead of developing our service from scratch, we reuse one already deployed to the [Fluence testnet](https://dash.fluence.dev/nodes).

The [Fluence Dashboard](https://dash.fluence.dev/) facilitates the discovery of deployed services, such as the [Curl Adapter](https://dash.fluence.dev/blueprint/b7d2454e-2a75-408c-a23a-fe35de3beeb9) service. For reference, please not that all services deployed on the Fluence peer-to-peer network are composed from Wasm modules. this brings enormous portability and development flexibility but also comes with a few constraints, such as no sockets available at the Wasm module level.  In order to get transfer capabilities across different network protocols, Wasm modules need to call, and be permissioned to call, the cUrl binary at the node level. Once these requirements are packed into a deployed service, we are ready to use that service's cUrl capabilities.

which allows us to harness http\(s\) requests as a service. Drilling down on the metadata provides a few useful parameters such as _service id_, _node id_ and _ip address_, which we need to execute our distributed curl service.

In order to execute the cUrl service and collect the result, i.e., response, we call upon our composition and coordination medium Aquamarine via an Aquamarine Intermediate Representation \(AIR\) script.

