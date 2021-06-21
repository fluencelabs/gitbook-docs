---
description: WIP
---

# Introduction

[Aqua](https://github.com/fluencelabs/aqua) is a new programming language aimed at programming distributed, and especially peer-to-peer, systems, backends, and applications.

At the core of the Aqua's peer-to-peer programming model is the notion of decoupling composable, distributed services' business logic from coordination workflow. Separating business logic from workflow concerns allows developers to take full advantage of the opportunities offered by peer-to-peer networks, such as a much more efficient request-response processing pattern, as well as much improved service reuse. A salient component of this distributed paradigm is what Fluence refers to as a _particle_: a data structure comprised of a conflict-free replicated data type \(CRDT\), execution sequence, and some metadata allowing particles to traverse a programmatically specified path of execution.

Todo: insert graphic 

Aqua provides the language and compiler to ergonomically coordinate distributed services, deployed across a permissionless network of peers, into a wide range of applications and backends. 

![Figure 1: Aqua Development Flow](../.gitbook/assets/image%20%286%29%20%281%29%20%281%29.png)

Aqua currently provides two compilation targets, a raw, low level language called Aquamarine Intermediate Representation \(AIR\) and Typescript, see Figure 1 above, with a runtime that is available for a variety of platforms ranging from browsers to servers to IoT devices.



