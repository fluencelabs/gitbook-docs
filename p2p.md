# Thinking In Distributed

Building and operating distributed networks, backends and applications are non-trivial undertakings not only posing technical challenges but also requiring a significant shift in how to view and think in distributed.

Consider a workflow tasked with calling multiple REST endpoints, in sequence, where the response of the previous call is the input to the current call. As illustrated in Figure 1, the application is the focal point and data relay.

![Figure 1: Stylized  Data Flow For Application With Multiple Endpoint Calls](.gitbook/assets/image%20%283%29.png)

Programming a frontend application in the Fluence peer-to-peer solution, an application is not a workflow intermediary but merely the initiator of a workflow as workflow logic and data traverses the network from service to service.  See Figure 2 for an illustration and please note that services may be deployed to different nodes as well as to more than one node.

![Figure 2: Stylized Data Flow For Application With Fluence Distributed Services ](.gitbook/assets/image%20%284%29.png)

In Fluence parlance, we call the workflow + data construct a _particle_ where the workflow is expressed in an AIR script.

This is a rather cursory overview of probably the most salient conceptual difference developers need to take into consideration in order to succeed in a distributed ecosystem. Luckily, Aquamarine and the Fluence stack eliminate most, if not all, of the heavy lifting necessary to develop high-value per-to-peer networks, backends, and applications.

