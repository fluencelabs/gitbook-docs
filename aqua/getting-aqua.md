---
description: WIP
---

# Setting Up Aqua

In order to run the Aqua compiler, you need java installed. Please see [AdoptOpenJDK](https://adoptopenjdk.net/) for installation options for your environment if you choose to install the binary or compile from source.

### With Docker

Fluence provides a docker container that contains the complete development tool chain for developing peer-to-peer applications including Aqua. Moreover, the container is ready for VSCode IDE integration.

#### For VSCode users

* Install the [Remote-Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension
* Run `Remote-Containers: Clone Repository in Container Volume`

  * Enter `fluencelabs/devcontainer`
  * Select `main` for branch
  * Confirm `unique` for volume, if asked

The download may take some time but once it's complete, you should see the following VSCode terminal tab.

```text
Unpacking objects: 100% (97/97), done.

=======
Source code to learn from:
    fluence/examples         –  AIR + Rust examples                      
    fluence/aqua-playground  –  Aqua + Typescript examples              
    fluence/fluentpad        -  Frontend + backend project written in Aqua


=========
Welcome to the Fluence devcontainer!
To download example projects, run /download_examples.sh
Available tools:
    $ marine           – build wasm from Rust
    $ marine repl      – run wasm services locally
    $ aqua-cli         – compile Aqua to AIR + Typescript
    $ fldist           – deploy & query services
    
Have fun!


Done. Press any key to close the terminal.
```

After the installation, simply connect to your docker container by

* Run Remote-Containers: Attach To Running Container
* Select your instance

In a \(new\) terminal tab, you see the root directory of the container and 

```text
todo: run  aqua version once we can do it
```

#### 

#### 

#### Docker Without IDE Integration

To pull the container from Docker Hub:

```text
docker pull fluencelabs/devcontainer:latest
```

Once the image is downloaded, connect to the container shell with:

```text
docker run -it fluencelabs/devcontainer:latest  /bash
```

which gets you to the bash shell root directory:

```text
=========
Welcome to the Fluence devcontainer!
To download example projects, run /download_examples.sh
Available tools:
    $ marine           – build wasm from Rust
    $ marine repl      – run wasm services locally
    $ aqua-cli         – compile Aqua to AIR + Typescript
    $ fldist           – deploy & query services

Have fun!
root@95aa2b3404aa:/#
```



**From Binary**

You can download 



#### From Source

```text
git clone git@github.com:fluencelabs/aqua.git
```

