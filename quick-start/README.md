# Quick Start

Welcome to our quick-start tutorials which guide you through the necessary steps to

1. Create a browser-to-browser messaging web application
2. Create and deploy a hosted service
3. Enhance a browser-to-browser application with a network-hosted service

For your development convenience, Fluence provides a [docker-based development environment](https://github.com/fluencelabs/devcontainer) that comes with the necessary dependencies, tooling and quick-start code pre-installed.

### Fluence Devcontainer

Fluence's devcontainer is a ready to use dockerized development environment with VSCode integration containing the following tools:

* [`aqua`](https://www.npmjs.com/package/@fluencelabs/aqua-cli) to compile [Aqua](https://doc.fluence.dev/aqua-book/) to AIR or wrapped in Typescript
* [`fldist`](https://www.npmjs.com/package/@fluencelabs/fldist) to manage services and optionally execute compiled Aqua from the command line
* [`marine`](https://crates.io/crates/marine) to compile services developed in Rust to the wasm32-wasi target
* [`mrepl`](https://crates.io/crates/mrepl) to run, test and debug WebAssembly \(Wasm\) services locally

### How to install

Docker and optionally VSCode need to be available on your system. For Docker installation, follow the [Get Docker](https://docs.docker.com/get-docker/) instructions for your OS. For VSCode, see [VSCode](https://code.visualstudio.com/) for instructions.

With Docker and VSCode in place:

* Install Remote-Containers extension in VSCode

![Install Remote - Containers in VSCode](../.gitbook/assets/image%20%2813%29.png)

* `Run Remote-Containers: Clone Repository in Container Volume...` via the command palette \(F1 or Cmd-Shift-P\)

![Select Remote Container Clone Repository](../.gitbook/assets/image%20%2814%29.png)

* Enter `fluencelabs/devcontainer`

![Select \`fluencelabs/devcontainer\`](../.gitbook/assets/image%20%2815%29.png)

* When asked for branch, press enter \(main\)
* When asked for volume, press enter \(unique\)
* Open Terminal in VSCode \(ctrl-\`\)

![Installed And Ready Devcontainer in VSCode](../.gitbook/assets/image%20%2818%29%20%281%29.png)

Congratulations, you now have a fully functional Fluence development environment. For a variety of container management options, click on the `Dev Container: Fluence` button in the lower left of your tool bar:

![Container Management Option Menu](../.gitbook/assets/image%20%2816%29.png)

If you encounter any problems or have suggestions, please open an issue or submit a PR. You can also reach out in [Discord](https://fluence.chat) or [Telegram](https://t.me/fluence_project). For more detailed reference resources, see the [Fluence documentation](https://doc.fluence.dev/docs/) and [Aqua book](https://doc.fluence.dev/aqua-book/).

All right, now we are ready to proceed to the first application on Fluence.

