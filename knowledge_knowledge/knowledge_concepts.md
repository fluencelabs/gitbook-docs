# Concepts

Distributed, peer-to-peer networks serve a wide variety of constituents and causes but generally with the common goal of providing decentralization benefits: no single point of failure or control. Alas, the programming of and developing for peer-to-peer networks is hard. Distributed service deployment and composition are quite different from the current REST, or GraphQL, microservices model, where discovery tends to be an inherently centralized process. In distributed, peer-to-peer networks, endpoint-based service requests are discovery-based searches across peers comprising the network. The Fluence solution makes peer-to-peer development accessible and productive and empowers developers to quickly and efficiently develop, deploy, manage, and monetize distributed applications and backends.

At the core of the Fluence solution are Aquamarine, a new, open-source language and runtime to choreograph distributed services into robust applications, and the Fluence Compute Engine \(FCE\), an open-source Wasm runtime. These Fluence innovations combine with [libp2p](https://libp2p.io/), [Kademlia DHT](https://en.wikipedia.org/wiki/Kademlia) at the peer-to-peer network level with the portability advantages of Wasm modules to provide developers with a modern, ergonomic p2p network and development model.

For the purpose of mastering peer-to-peer application development and the Fluence solution, developers need to be aware of the following concepts in addition to the Fluence tools and runtimes.

## Module

Modules are logical units of code that can be used individually or in combination to create a service. In Fluence, modules are written in Rust and compiled to [Wasm IT](https://wasi.dev/) for eventual execution by the Fluence Compute Engine \(FCE\). Modules fall into three conceptual categories, which developers need to be aware of as the module type has a direct impact on service configuration:

* Facade Modules
* Pure Modules
* Effector modules

Facade modules expose the API of the services comprised of one or more modules. Every service has exactly one facade module.

Pure modules perform computations without side-effects.

Effector modules contain at least one computation with a side-effect, e.g., write to file or database, access eternal binaries, etc.

## Service

Services are logical compute units derived from linking of one or more Wasm modules. The linking of modules is facilitated by the Fluence Compute Engine \(FCE\) using a specially prepared configuration file. The configuration file which governs the

* Order of instantiation
* Permission to resources
* Maximum memory allocation

FCE uses a [shared nothing](https://en.wikipedia.org/wiki/Shared-nothing_architecture) linking scheme meaning that modules only expose functions explicitly marked to be publicly available while not sharing memeory or any other resources. It should be further noted that services can **not** call on other services directly.

## Blueprint

The configuration map associating modules with service instantiations. This is a higher level construct than the low-level FCE linking described in the services section. Blueprints capture module names, blueprint name, and blueprint id. That allows the tracking and management of modules and services on a per-peer basis.

## Particle

A particle is a data structure combining data, service execution sequence, by means of an AIR script, and additional metadata. According to the AIR script specification, a particle travels through the network triggering execution at pre-defined service stops and peer nodes updating its data at every hop. Not surprisingly, the notion, implementation and processing of particles are a salient aspect of the Fluence solution.

## Application

An application in the Fluence solution is the "frontend" to one or more services and their execution sequence. Applications are developed by coordinating one or more services into a logical compute unit and tend to live outside the Fluence network. They can be executed in various runtime environments ranging from browsers to backend daemons with the help of Fluence command-line tools or the Fluence JS-SDK.

