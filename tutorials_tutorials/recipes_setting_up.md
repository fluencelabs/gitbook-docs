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

There are a number of good Rust installation and IDE integration tutorials available. [DuckDuckGo](https://duckduckgo.com/) is your friend but if that's too much effort, have a look at [koderhq](https://www.koderhq.com/tutorial/rust/environment-setup/).



### Aqua Tools

```text
# aqua compiler
```

### Marine Tools

Fluence provides several tools to support developers. Fluence cli, `flcli`, facilitates the compilation of modules to the necessary wasm32-wasi target. Fluence REPL, `fce-repl`, on the other hand, is a cli tool to test and experiment with FCE modules and services locally.

```bash
cargo install marine 
cargo +nightly install mrepl
```

In addition, Fluence provides the [proto-distributor](https://github.com/fluencelabs/proto-distributor) tool, aka `fldist`, for service lifecyle management. From deploying services to the network to executing AIR scripts, `fldist` does it all.

### Fluence Tools

```bash
npm install -g @fluencelabs/fldist
```

### Fluence SDK

For frontend development, the Fluence [JS-SDK](https://github.com/fluencelabs/fluence-js) is currently the favored, and only, tool.

```bash
npm install @fluencelabs/fluence
```



