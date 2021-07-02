# Thinking In Distributed

Permissionless peer-to-peer networks have a lot to offer to developers and solution architects such as decentralization, improved request-response data models and zero trust security at the application and service level. Of course, these capabilities and benefits don't just arise from putting libp2p to work. Instead, a peer-to-peer overlay is required. The Fluence protocol provides such an overlay enabling a powerful distributed data routing and management protocol allowing developers to implement modern and secure Web3 solutions.

**Improved Request-Response Model**

In some network models, such as client server, the request-response model generally entails a response returning to the request client. For example, a client application tasked to conduct a credit check of a customer and to inform them with a SMS typically would call a credit check API, consume the response, and then call a SMS API to send the necessary SMS.

Figure 2: Client Server Request Response Model

![](https://i.imgur.com/ZYLUzne.png)

The Fluence peer-to-peer protocol, on the other hand, allows for a much more effective Request-Response processing pattern where responses are forward-chained to the next consuming service\(s\) without having to make the return trip to the client. See Figure 3.

Figure 3: Fluence P2P Protocol Request Response Model

![](https://i.imgur.com/g3RGBRf.png)

In a Fluence p2p implementation, our client application would call a credit check API deployed or proxy-ed on some peer and then send the response directly to the SMS API service possibly deployed on another peer -- similar to the flow depicted in Figure 1.

Such a significantly flattened request-response model leads to much lower resource requirements for applications in terms of bandwidth and processing capacity thereby enabling a vast class of "thin" clients ranging from browsers to IoT and edge devices truly enabling decentralized machine-to-machine communication.

**Zero Trust Security**

The [zero trust security model](https://en.wikipedia.org/wiki/Zero_trust_security_model) assumes the worst reality, i.e., a breach, and proposes a "never trust, always verify" approach. This approach is inherent in the Fluence peer-to-peer protocol and Aqua programming model as every service request can be authenticated at the service API level.

Overall, the Fluence solution enables a modern Web3 runtime and development environment on top of a peer-to-peer stack that allows developers to build powerful and secure distributed applications on thin clients and powerful servers alike.

