# Node

The Fluence protocol is implemented as the Fluence [reference node](https://github.com/fluencelabs/fluence) which includes the

* Peer-to-peer communication layer
* Marine interpreter
* Aqua VM
* Builtin services

and more.

Builtin services are available on every Fluence peer and can be programmatically accessed and composed using Aqua just like any other service. For a complete list of builtin services see the `builtin.aqua` file in the [Aqua Lib](https://github.com/fluencelabs/aqua-lib) repo.&#x20;

To find out how to create your own builtin service, see the [Add Your Own Builtins](tutorials\_tutorials/add-your-own-builtin.md) tutorial.

## Node distribution

All infrastructure-related information is kept in [fluencelabs/node-distro](https://github.com/fluencelabs/node-distro) repository on GitHub.

Node is distributed as a docker container [fluencelabs/fluence](https://hub.docker.com/r/fluencelabs/fluence). Version information can be found on the [releases page](https://github.com/fluencelabs/node-distro/releases) in GitHub.

It comes with IPFS, AquaDHT and TrustGraph bundled.

### How to run a node

Just a simple docker run:

```
docker run --rm -e RUST_LOG="info" -p 7777:7777 -p 9999:9999 fluencelabs/fluence
```

Or take a look at the [docker-compose.yml](https://github.com/fluencelabs/node-distro/blob/main/docker-compose.yml) in the node-distro repository. It starts node with a web dashboard to explore deployed services.

```
docker compose up -d
```
