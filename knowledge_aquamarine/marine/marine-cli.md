# Marine CLI

The [Marine command line tool](https://github.com/fluencelabs/marine) provides the project `marine build` functionality, analogous to `cargo build`,  that results in the Rust code to be compiled to _wasm32-wasi_ modules. In addition,  `marine` provides utilities to inspect Wasm modules, expose Wasm module attributes or manually set module properties.

```rust
mbp16~(:|✔) % marine --help
Fluence Marine command line tool 0.6.7
Fluence Labs

USAGE:
    marine [SUBCOMMAND]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    aqua     Shows data types of provided module in a format suitable for Aqua
    build    Builds provided Rust project to Wasm
    help     Prints this message or the help of the given subcommand(s)
    info     Shows manifest and sdk version of the provided Wasm file
    it       Shows IT of the provided Wasm file
    repl     Starts Fluence application service REPL
    set      Sets interface types and version to the provided Wasm file
mbp16~(:|✔) %
```



&#x20;



