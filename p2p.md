# Thinking In Aquamarine

Permissionless peer-to-peer networks have a lot to offer to developers and solution architects such as decentralization, control over data, improved request-response data models and zero trust security at the application and service level. Of course, these capabilities and benefits don't just arise from putting [libp2p](https://libp2p.io/) to work. Instead, a peer-to-peer overlay is required. The Fluence protocol provides such an overlay enabling a powerful distributed data routing and management protocol allowing developers to implement modern and secure Web3 solutions. See Figure 1 for a stylized representation decentralized applications development by programming the composition of services distributed across a peer-to-peer network.

Figure 1: Decentralized Applications Composed From Distributed Services On P2P Nodes

![](https://i.imgur.com/XxC7NN3.png)

## Aquamarine

As a complement to the protocol, Fluence provides the Aquamarine stack aimed at enabling developers to build high-quality, high-performance decentralized applications. Aquamarine is purpose-built to ease the programming demands commonly encountered in distributed, and especially peer-to-peer, development and is comprised of Aqua and Marine.

[Aqua](https://doc.fluence.dev/aqua-book/), is a new generation programming language allowing developers to program peer-to-peer networks and compose distributed services hosted on peer-to-peer nodes into decentralized applications and backends. Marine, on the other hand, provides the necessary Wasm runtime environment on peers to facilitate the execution of compiled Aqua code.

A major contribution of Aquamarine is that network and application layer, i.e., [Layer 3 and Layer 7](https://en.wikipedia.org/wiki/OSI_model), programming is accessible to developers as a seamless and ergonomic composition-from-services experience in Aqua thereby greatly reducing, if not eliminating, common barriers to distributed and decentralized application development.

## **Improved Request-Response Model**

In some network models, such as client server, the request-response model generally entails a response returning to the request client. For example, a client application tasked to conduct a credit check of a customer and to inform them with a SMS typically would call a credit check API, consume the response, and then call a SMS API to send the necessary SMS.

Figure 2: Client Server Request Response Model

![](.gitbook/assets/image%20%2811%29.png)

The Fluence peer-to-peer protocol, on the other hand, allows for a much more effective Request-Response processing pattern where responses are forward-chained to the next consuming service\(s\) without having to make the return trip to the client. See Figure 3.

Figure 3: Fluence P2P Protocol Request Response Model

![](.gitbook/assets/image%20%2810%29.png)

In a Fluence p2p implementation, our client application would call a credit check API deployed or proxy-ed on some peer and then send the response directly to the SMS API service possibly deployed on another peer -- similar to the flow depicted in Figure 1.

Such a significantly flattened request-response model leads to much lower resource requirements for applications in terms of bandwidth and processing capacity thereby enabling a vast class of "thin" clients ranging from browsers to IoT and edge devices truly enabling decentralized machine-to-machine communication.

## **Zero Trust Security**

The [zero trust security model](https://en.wikipedia.org/wiki/Zero_trust_security_model) assumes the worst, i.e., a breach, at all times and proposes a "never trust, always verify" approach. This approach is inherent in the Fluence peer-to-peer protocol and Aqua programming model as every service request can be authenticated at the service API level. That is, every service exposes functions which may require authentication and authorization. Aquamarine implements SecurityTetraplets as verifiable origins of the function arguments to enable fine-grained authorization.

## Service Granularity And Redundancy

Services are not capable to accept more than one request at any given time. Consider a service, FooBar, comprised of two functions, foo\(\) and bar\(\) where foo is a longer running function.

```text
-- Stylized FooBar service with two functions
-- foo() and bar()
-- foo is long-running
--- if foo is called before bar, bar is blocked
service FooBar("service-id"):
  bar() -> string
  foo() -> string --< long running function 

func foobar(node:string, service_id:string, func_name:string) -> string:
  res: *string
  on node:
    BlockedService service_id
    if func_name == "foo":
      res <- BlockedService.foo()
    else:
      res <- BlockedService.bar()
  <- res!
```

As long as  foo\(\) is running,  the entire FooBar service, including bar\(\), is blocked. This has implications with respect to both service granularity and redundancy, where service granularity captures to number of functions per service and redundancy refers to the  number of service instances deployed to different peers. 


## Summary

Programming distributed applications on the Fluence protocol with Aquamarine unlocks significant benefits from peer-to-peer networks while greatly easing the design and development processes. Nevertheless, a mental shift concerning peer-to-peer solution design and development process is required. Specifically, the successful mindset accommodates

* an application architecture based on the composition of distributed services across peer-to-peer networks by decoupling business logic from application workflow
* a services-first approach with respect to both the network and application layer allowing a unified network and application  programming model encapsulated by Aqua
* a multi-layer security approach enabling zero-trust models at the service level
* a flattened request-response model enabling data free from centralized control
* a services architecture with respect to granularity and redundancy influenced by service function runtime 

