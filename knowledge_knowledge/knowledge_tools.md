# Tools

## Fluence Marine REPL

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

## Fluence Proto Distributor: FLDIST

\`\`[`fldist`](https://github.com/fluencelabs/proto-distributor) is a command line interface \(CLI\) to Fluence peers allowing for the lifecycle management of services and offers the fastest and most effective way to service deployment.

```text
mbp16~(:|✔) % fldist --help
Usage: fldist <cmd> [options]

Commands:
  fldist completion      generate completion script
  fldist upload          Upload selected wasm
  fldist get_modules     Print all modules on a node
  fldist get_interfaces  Print all services on a node
  fldist get_interface   Print a service interface
  fldist add_blueprint   Add a blueprint
  fldist create_service  Create a service from existing blueprint
  fldist new_service     Create service from a list of modules
  fldist deploy_app      Deploy application
  fldist create_keypair  Generates a random keypair
  fldist run_air         Send an air script from a file. Send arguments to
                         "returnService" back to the client to print them in the
                         console. More examples in "scripts_examples" directory.
  fldist env             show nodes in currently selected environment

Options:
      --help             Show help                                     [boolean]
      --version          Show version number                           [boolean]
  -s, --seed             Client seed                                    [string]
      --env              Environment to use
            [required] [choices: "dev", "testnet", "local"] [default: "testnet"]
      --node-id, --node  PeerId of the node to use
      --node-addr        Multiaddr of the node to use
      --log              log level
       [required] [choices: "trace", "debug", "info", "warn", "error"] [default:
                                                                        "error"]
      --ttl              particle time to live in ms
                                            [number] [required] [default: 60000]
```

## Fluence JS SDK

The Fluence [JS SDK](https://github.com/fluencelabs/fluence-js) supports developers to build full-fledged applications for a variety of targets ranging from browsers to backend apps and greatly expands on the `fldist` capabilities.

