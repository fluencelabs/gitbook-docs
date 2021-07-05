# Setting Up Your Environment

In order to develop within the Fluence solution, [Rust](https://www.rust-lang.org/tools/install) and small number of tools are required.

### Rust

Install Rust:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

once Rust is installed, we need to expand the toolchain and include [nightly build](https://rust-lang.github.io/rustup/concepts/channels.html) and the [Wasm](https://doc.rust-lang.org/stable/nightly-rustc/rustc_target/spec/wasm32_wasi/index.html) compile target.

```bash
rustup install nightly
rustup target add wasm32-wasi
```

To keep Rust and the toolchains updated:

```bash
rustup self update
rustup update
```

There are a number of good Rust installation and IDE integration tutorials available. [DuckDuckGo](https://duckduckgo.com/) is your friend but if that's too much effort, have a look at [koderhq](https://www.koderhq.com/tutorial/rust/environment-setup/). Please note, however, that currently only VSCode is supported with Aqua syntax support.

### Aqua Tools

The Aqua compiler and standard library and be installed via npm:

```text
npm -g install @fluencelabs/aqua-cli
npm -g install @fluencelabs/aqua-lib
```



If you are a VSCode user, note that am Aqua syntax-highlighting extension is available. In VSCode, click on the Extensions button, search for `aqua`and install the extension.

![](https://gblobscdn.gitbook.com/assets%2F-MbmEhQUL-bljop_DzuP%2F-MdMDybZMQJ5kUjN4zhr%2F-MdME2UUjaxKs6pzcDLH%2FScreen%20Shot%202021-06-29%20at%201.06.39%20PM.png?alt=media&token=812fcb5c-cf28-4240-b072-a51093d0aaa4)

Moreover, the aqua-playground provides a ready to go Typescript template and Aqua example. In a directory of you choice:

```text
git clone git@github.com:fluencelabs/aqua-playground.git
```

### Marine Tools

Fluence provides several tools to support developers. `marine` is the command line compiler required to compile Rust modules to the necessary wasm32-wasi target. `mrepl`, on the other hand, is a command line  tool providing access to the Marine runtime to test and experiment with marine modules and services locally:

```bash
cargo install marine 
cargo +nightly install mrepl
```

### Fluence Tools

In addition, Fluence provides the `fldist` tool for the lifecycle management of services. From deploying services to the network to executing compiled Aqua scripts from the command line , `fldist` does it:

```bash
npm install -g @fluencelabs/fldist
```

### Fluence SDK

For frontend development, the Fluence [JS-SDK](https://github.com/fluencelabs/fluence-js) is currently the favored, and only, tool.

```bash
npm install @fluencelabs/fluence
```



