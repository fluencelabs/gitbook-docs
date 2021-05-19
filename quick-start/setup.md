---
description: WIP -- need to circle back
---

# Setup

Aquamarine depends on Rust, Typescript, and several tools, which are provided as a dockerized development environment that bundles all the pre-requisites, VSCode integration and examples.

**VSCode Installation**

In the VSCode, install the [Remote-Containers ](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)extension and in the command palette:

* Run _**Remote-Containers: Clone a repository from  GitHub in container volume**_
* Enter/Select _**fluencelabs/devcontainer**_
* Select _**main**_, if prompted for branch
* Select _**unique**_, if prompted for volume

Once installed, select _**Remote-Containers: Attach to Running Container**_ and select the _**fluencelabs/devcontainer**_ instance.

#### Manual Installation

Get the development environment:

```text
docker pull fluencelabs/devcontainer:latest
```

This is going to take a minute, or ten, as the download is about 2.4GB. Once you have the container, you can start it with:

```text
docker run -d  -name fluence-devenv fluencelabs/devcontainer:latest bash
```

which puts us in the root of the container `root@9d747e268a87:/#` now clone the Fluence examples repo:

```text
root@9d747e268a87:/# bash -i -c 'git clone https://github.com/fluencelabs/examples'
```



 

