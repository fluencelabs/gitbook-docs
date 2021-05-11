# Aqua

### Overview

[Aqua](https://github.com/fluencelabs/aqua) is a new scripting language to program distributed and especially peer-to-peer systems, backends, and applications.

At the core of the Fluence's peer-to-peer programming model is the notion of decoupling composable, distributed services' business logic from coordination workflow to allow developers to take advantage of the opportunities offered by peer-to-peer networks, such as a much more efficient Request-Response processing pattern, as well as encourage service reuse. A salient component of this system is what Fluence refers to as a _particle_: a data structure comprised of a conflict-free replicated data type \(CRDT\), execution sequence, and some metadata traversing a programmatically specified path of execution.

Aqua provides the programmatic tool and execution model allowing developers to coordinate distributed services deployed across a permissionless network of peers into a wide range of applications and backends.

Aqua currently provides two compilation targets, Aquamarine Intermediate Representation \(AIR\) and Typescript, with a runtime that is available for a variety of platforms ranging from browsers to servers to IoT devices.

For a detailed review, see the [Fluence Protocol](https://github.com/fluencelabs/rfcs/blob/main/0-overview.md).

