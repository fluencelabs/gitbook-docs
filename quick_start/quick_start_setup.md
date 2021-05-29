# Setup

Fluence provides developers with nodes, runtimes and tools to ease and accelerate the development of distributed networks, backends, and applications. In order to be able to utilize the Fluence support system, we need to install a few things on our machines.

If you don't have [Rust](https://www.rust-lang.org/) installed, this is as good a time as any to do so:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

and follow the [instructions](https://www.rust-lang.org/tools/install). Once Rust is in place, install Rust _nightly_ and the Wasm tool chain:

```bash
rustup toolchain install nightly
rustup target add wasm32-wasi
```

In addition, install the Marine REPL,`mrepl`, and CLI, `marine`, tools:

```bash
$ cargo install marine --force
$ cargo +nightly install mrepl --force
```

Finally, you need [node](https://nodejs.org/en/) installed, and if you don't have it already, you may be best served by installing [NVM](https://github.com/nvm-sh/nvm):

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
nvm install --lts
```

which allows us to install the \[Fluence Service Distribution and Management too\]\([https://github.com/fluencelabs/proto-distributor](https://github.com/fluencelabs/proto-distributor)\), `fldist`.

```bash
npm install -g @fluencelabs/fldist
```

Throughout the tutorial you will be asked to copy AIR scripts and run them with the `fldist` tool. In order to use this tool, you need to use the terminal. The repeat action is to copy an AIR script from this book to a file on your system and then run the file with `fldist` in the terminal.

That's all we need to work through the examples. Let's get to it.

