# Tools

### Fluence Proto Distributor: FLDIST

[`fldist`](https://github.com/fluencelabs/proto-distributor) is a command line interface \(CLI\) to Fluence peers allowing for the lifecycle management of services and offers the fastest and most effective way to service deployment.

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

### Fluence JS

The [Fluence JS](https://github.com/fluencelabs/fluence-js) supports developers to build full-fledged applications for a variety of targets ranging from browsers to backend apps and greatly expands on the `fldist` capabilities.

### Marine Tools

Marine offers multiple tools including the Marine CLI, REPL and SDK. Please see the [Marine section](knowledge_aquamarine/marine/) for more detail.

