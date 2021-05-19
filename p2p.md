# Thinking In Distributed

Building and operating distributed networks, backends and applications have long been thought of as non-trivial undertakings. Aquamarine's primary intent is to empower and enable developers to much more easily and rapidly build reliable and verifiable peer-to-peer applications. 

Peer-to-peer networks offer a set of advantages including architectural advantages over centralized solutions, such as client-server architectures. A prominent improvement made possible with peer-to-peer networks is a fundamental shift in Request-Response processing.

 Consider a workflow tasked with calling multiple REST endpoints, in sequence, where the response of the previous call is the input to the current call. As illustrated in Figure 1, the application is the focal point and data relay and a typical client-server solution tends to require the client to process approximately n-to-n request-response calls.

![Figure 1: Stylized Data Flow For Application With Multiple Endpoint Calls](.gitbook/assets/image%20%283%29.png)

Peer-to-peer networks allow for a different data management pattern. Instead of a client-centric Request-Response processing pattern, peer-to-peer networks allow the forward-chaining of responses directly into the intended consumers: no "client relay" necessary. Hence, a peer-to-peer client can be notably reduced to a n-to-one pattern. See Figure 2. 

![Figure 2: Stylized Data Flow For Application With Fluence Distributed Services ](.gitbook/assets/image%20%284%29.png)

Of course, such capabilities don't just come with the availability of peer-to-peer networking but are the result of additional runtimes and programming tools to enable such behavior on each node. Fluence provides the required open source runtime and tooling components to allow developers to take full advantage of inherent peer-to-peer capabilities. Specifically, the Fluence framework allows the separation of services' business logic from the workflows coordinating services which in turn allows a much more favorable request-response processing model. 

Programming peer-to-peer applications in the Fluence framework entails scripting a workflow in  the Aqua language to coordinate deployed distributed network services into the desired application logic. In a probably somewhat novel design to most developers, the workflow is part of a data structure that also includes the data, in form of a conflict-free replication data type, which follows the programmatically specific execution route for services and nodes. In Fluence parlance, we call the data structure a _particle_.

Hence, an application client is not your typical workflow intermediary and relay but merely the initiator of deploying particles on the their programmatically specified journey and, as a programming model, may provide a significant departure for most developers.

This is a rather cursory overview of probably the most salient conceptual difference developers need to take into consideration in order to succeed programming distributed networks and applications. Luckily, Aquamarine and the Fluence stack eliminate most, if not all, of the heavy lifting necessary to develop high-value per-to-peer networks, backends, and applications.

