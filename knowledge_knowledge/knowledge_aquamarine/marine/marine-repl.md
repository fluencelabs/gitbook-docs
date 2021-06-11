# Marine Repl

[`mrepl`](https://crates.io/crates/mrepl) is a command line tool \(CLI\) to locally run a Marine instance to inspect, run, and test module and service configurations.

```text
mbp16~(:|✔) % mrepl
Welcome to the Marine REPL (version 0.7.2)
Minimal supported versions
  sdk: 0.6.0
  interface-types: 0.20.0

New version is available! 0.7.2 -> 0.7.4
To update run: cargo +nightly install mrepl --force

app service was created with service id = d81a4de5-55c3-4cb7-935c-3d5c6851320d
elapsed time 486.234µs

1> help
Commands:

n/new [config_path]                       create a new service (current will be removed)
l/load <module_name> <module_path>        load a new Wasm module
u/unload <module_name>                    unload a Wasm module
c/call <module_name> <func_name> [args]   call function with given name from given module
i/interface                               print public interface of all loaded modules
e/envs <module_name>                      print environment variables of a module
f/fs <module_name>                        print filesystem state of a module
h/help                                    print this message
q/quit/Ctrl-C                             exit

2>
```



