---
description: WIP -- need to circle back
---

# Setup

The Fluence and Aquamarine stacks depend on Rust, Typescript, and several tools. Luckily, we provide a dockerized development environment that bundles all the pre-requisites, VSCode integration and examples.

**VSCode Installation**

In the VSCode, install the [Remote-Containers ](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)extension and in the comand palette:

* Run _**Remote\_containers: Clone a repository from  GitHub in container volume**_
* Enter _**fluencelabs/devcontainer**_
* Select _**main**_, if prompted for branch
* Select _**unique**_, when asked for volume

Once installed, select _**Remote-Containers: Attach to Running Container**_ 

#### Manual Installation

Get the development environment:

```text
% docker pull fluencelabs/devcontainer:latest
```

This is going to take a minute, or ten, as the download is about 2.4GB. Once you have the container, you can start it with:

```text
% docker run -d  -name fluence-devenv fluencelabs/devcontainer:latest bash
```

which puts us in the root of the container `root@9d747e268a87:/#` now clone the Fluence examples repo:

```text
root@9d747e268a87:/# bash -i -c 'git clone https://github.com/fluencelabs/examples'
```



 

