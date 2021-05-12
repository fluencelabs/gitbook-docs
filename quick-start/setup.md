---
description: WIP -- need to circle back
---

# Setup

The Fluence and Aquamarine stacks depend on Rust, Typescript, and several tools. Luckily, we provide a dockerized development solution that bundles all the pre-requisites and even provides VSCode integration, if that happens to be your IDE of choice.

#### Manual Installation

Get the development environment:

```text
% docker pull fluencelabs/devcontainer:latest
```

This is going to take a minute or ten as the download is about 2.4GB. Once you have the container, you can start it with:

```text
% docker run -d  -name fluence-devenv fluencelabs/devcontainer:latest bash
```

which puts you in the root of the container `root@9d747e268a87:/#` now clone the fluence examples repo:

```text
root@9d747e268a87:/# bash -i -c 'git clone https://github.com/fluencelabs/examples'
```



VSCode Installation

In VSCode, 

