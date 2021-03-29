# Aquamarine

Aquamarine is a programming language and executable choreography tool for distributed applications and backends. Aquamarine manages the communication and coordination between services, devices, and APIs without introducing any centralized gateway and can be used to express various distributed systems: from simple request-response to comprehensive network consensus algorithms.

At the core of Aquamarine is the design ideal and idea to pair concurrent systems, and especially decentralized networks, with a programing and execution tool chain to avoid centralized bottlenecks commonly introduced with [workflow engines](https://en.wikipedia.org/wiki/Workflow_engine) and [Business rule engines](https://en.wikipedia.org/wiki/Business_rules_engine). This not only makes Aquamarine the rosetta stone of the Fluence solution but also a very powerful generic coordination and composition medium.

## Background

When we build systems, we need to be able to model, specify, analyze and verify them and this is especially important to concurrent systems such as parallel and multi-threaded systems. [Formal specification](https://en.wikipedia.org/wiki/Formal_specification) are a family of formal approaches to design, model, and verify system. In the context of concurrent systems, there are two distinct formal specification techniques available. The state oriented approach is concerned with modeling verifying a systems state and state transitions and is often accomplished with [TLA+](https://en.wikipedia.org/wiki/TLA%2B). Modern blockchain design, modeling, and verification tend to rely on a state-based specification.

An alternative, complementary approach is based on [Process calculus](https://en.wikipedia.org/wiki/Process_calculus) to model and verify the sequence of communications operations of a system at any given time. [π-Calculs](https://en.wikipedia.org/wiki/%CE%A0-calculus) is a modern process calculus employed in a wide range of applications ranging from biology to games and business processes.

Aquamarine, Fluence's distributed composition language and runtime, is based on π-calculus and provides a solid theoretical basis toward the design, modeling, implementation, and verification of a wide class of distributed, peer-to-peer networks, applications and backends.

TODO: maybe add some non-fluence examples; definitely get to the verification bit.

## Language

[Aquamarine Intermediate Representation](https://github.com/boneyard93501/docs/tree/a512080f81137fb575a5b96d3f3e83fa3044fd1c/src/knowledge-base/knowledge_aquamarine__air.md) \(AIR\) is a low-level language modelled after the [WebAssembly text format](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format) and allows developers to manage network peers as well as services and backends. AIR, while intended as a compile target, is currently the only Aquamarine language implementation although a high level language \(HLL\) is currently under active development.

TODO: verify below and that we don't have parallel composition proper The Aquamarine language implementation includes

* sequential composition
* reduction semantics
* communication

## Runtime

The Aquamarine runtime is a virtual machine executed by the [Fluence Compute Engine](https://github.com/boneyard93501/docs/tree/a512080f81137fb575a5b96d3f3e83fa3044fd1c/src/knowledge-base/knowledge_fce.md) \(FCE\) running not only on every Fluence network peer but also on every frontend client. The distributed runtime availability per node not only aids in decentralized service discovery and execution at the same level of decentralization of the network, which is of significant importance. Moreover, with running execution scripts on both the client and the \(remote\) nodes, a high degree of auditability and verifiability can be attained.

