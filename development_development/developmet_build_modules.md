# From Module To Service

In Fluence, a service is based on one or more [Wasm](https://webassembly.org/) modules suitable to be deployed to the Fluence Compute Engine \(FCE\). In order to develop our  modules, we use Rust and the [Fluence Rust SDK](https://github.com/fluencelabs/rust-sdk).

## Preliminaries

The general process to create a Fluence \(module\) project is to:

```bash
cargo +nightly create your_module_name --release
```

and add the [binary target](https://doc.rust-lang.org/cargo/reference/cargo-targets.html#binaries) and [Fluence Rust SDK](https://crates.io/crates/fce) to the Cargo.toml:

```text
<snip>

[[bin]]  # <- binary target
name = "<your_module_name>"
path = "src/main.rs"

[dependencies]
fluence = { version = "=0.5.0", features = ["logger"] }
log = "0.4.14"
```

## Developing A Simple Wasm Module

Let's build a simple greeting module to verify our setup and quickly go through the steps we need to complete to build a simple service.

```bash
cargo +nightly new greeting --release
cd greeting
```

and update _main.rs_:

```rust
use fluence::fce;                               // 1
use fluence::module_manifest;                   // 2

module_manifest!();                             // 3

pub fn main() {}                                // 4

#[fce]                                          // 5
pub fn greeting(name: String) -> String {
    format!("Hi, {}", name)
}
```

Let's go line by line: 

1. Import the [fce](https://github.com/fluencelabs/fce/tree/5effdcba7215cd378f138ab77f27016024720c0e) module from the [Fluence crate](https://crates.io/crates/fluence), which allows us to compile our code to the [wasm32-wasi](https://docs.rs/crate/wasi/0.6.0) target 

2. Import the [module\_manifest](https://github.com/fluencelabs/rust-sdk/blob/master/crates/main/src/module_manifest.rs), which allows us to embed the SDK version in our module 

3. Initiate the module\_manifest macro 

4. Initiate the main function which generally stays empty or is used to instantiate a logger 

5. Markup the public function we want to expose with the FCE macro which, among other things, checks that only Wasm IT types are used

Once we compile our code, we generate the wasm32-wasi file, which can be found in the `target/wasm32-wasi` path of your directory. The `greeting.wasm` file is what we need for testing and eventual upload to the peer-to-peer network.

To make things a little easier on us, let's create a build script, _build.sh_:

```bash
#!/bin/sh
# This script builds all sub-projects and puts our Wasm module(s) in a high-level dir

fce build --release                                            // 1

mkdir -p artifacts                                             // 2
rm artifacts/*                                                     
cp target/wasm32-wasi/release/greeting.wasm artifacts/         // 3
```

Our script executes the following steps in one handy executable:

1. Compile the FCE annotated Rust code to the wasm32-wasi target generating the wasm module we so very much desire
2. Make a higher-level artifacts directory to hold wasm file\(s\) in a more convenient location
3. Copy the wasm build to the artifacts directory

Before we can run the script we need to `chmod +x build.sh` to make the file executable. Now we can run it:

```bash
./build.sh
```

which starts the build and compilation of the project and eventually, you should see the `greeting.wasm` file in the artifacts directory.

```bash
ll artifacts
-rwxr-xr-x  1 bebo  staff    81K Mar 15 19:41 greeting.wasm
```

Before we can actually create a service from our module, one more file needs to be added to our project called the service configuration file. Service config files control the order in which the modules are instantiated, their permissions, maximum memory limits and some other parameters. In general, a service configuration file contains:

* modules\_dir  -- the path to the directory with all the Wasm modules
* \[\[module\]\]  -- a list of modules comprising the service
* name -- the  \(file\) name of the corresponding Wasm file in the modules\_dir

For our greeting service, we add the following _Config.toml_ file:

```text
# Config.toml
modules_dir = "artifacts/"

[[module]]
    name = "greeting"
```

The source code for the module can be found in the [examples repo](https://github.com/fluencelabs/examples/tree/main/greeting).

## Taking The Greeting Module For A Spin

Now that we have a Wasm module and service configuration, we can explore and test our achievements locally with the Fluence REPL tool `fce-repl`. Load the service for inspection and testing:

```bash
fce-repl Config.toml

Welcome to the FCE REPL (version 0.5.2)
app service was created with service id = 10afa1aa-22e6-4c8a-b668-6be95d2d3530
elapsed time 54.290336ms

1> interface
Loaded modules interface:

greeting:
  fn greeting(name: String) -> String

2>
```

Using our service config file, we loaded the module and associated config info into the `fce-repl` tool and with the `interface` command, we obtain a listing of module name\(s\) and associated interfaces which when can execute in the tool:

```bash
2> call greeting greeting ["Fluence"]
result: String("Hi, Fluence")
 elapsed time: 98.02µs

3>
```

The _interface_ command lists the available interfaces by module, i.e., the functions we designated as public and marked up with the _FCE_ macro in our source code. For more command info, use the _help_ command:

```text
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
```

The command we'll be using the most is the _call_ command to execute module functions locally as an effective way to test services, such as calling a function with an incorrect type:

```bash
3> call greeting greeting [5]
call failed with: JsonArgumentsDeserializationError: error Error("invalid type: integer `5`, expected a string", line: 0, column: 0) occurred while deserialize output result to a json value

4> call greeting greeting ["5"]
result: String("Hi, 5")
 elapsed time: 61.505µs

5>
```

The interface `fn greeting(name: String) -> String` specified a string input and an integer input will cause the method to fail. Looks like all is working as designed and expected.

## Deploying The Greeting Module To A Local Node

Now that we are reasonably satisfied that our greeting service works, it is time to deploy it to the local network and test it with an AIR script. Before we do that, however, we need configuration files for each of the modules comprising our service. In our greeting service case, we only have one module and our configuration reads as follows:

```javascript
// greeting_cfg.json
{
    "name" : "greeting"
}
```

The configuration files are the per-module equivalents of the service configuration file we've seen earlier. They allow nodes to establish the meta-data and permission requirements, per module, before modules are linked to a service. The resulting configuration \(json\) object for a service over the underlying modules. This is called a _blueprint :_

```text
{
  "id": "uuid-1234-...",
  "name": "some name",
  "dependencies": [ "module_a", "module_b", "facade_module" ]
}
```

Back to our use case at hand: our config file only needs a name specifier and we are ready to deploy our service to the network or local development node. For detailed information with respect to running a local node, see the [tutorial](https://github.com/boneyard93501/docs/tree/575ff7b260d1014bdaf4d26e791f0ce8f2841d0d/src/tutorials/tutorial_run_local_node.md).

To run the local node:

```bash
# start the local dev node
docker run --rm --name fluence -e RUST_LOG="info" -p 7777:7777 -p 9999:9999 -p 18080 fluencelabs/fluence
```

and pull the node id, _12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17_ in this case, from the log:

```bash
[docker run --rm --name my_fluence -e RUST_LOG="info" -p 7777:7777 -p 9999:9999 -p 18080 fluencelabs/fluence:latest
[2021-03-16T21:01:01.347081Z INFO  particle_node]
    +-------------------------------------------------+
    | Hello from the Fluence Team. If you encounter   |
    | any troubles with node operation, please update |
    | the node via                                    |
    |     docker pull fluencelabs/fluence:latest      |
    |                                                 |
    | or contact us at                                |
    | github.com/fluencelabs/fluence/discussions      |
    +-------------------------------------------------+

[2021-03-16T21:01:01.347925Z INFO  server_config::fluence_config] Loading config from "/.fluence/Config.toml"
[2021-03-16T21:01:01.348061Z INFO  server_config::keys] generating a new key pair
[2021-03-16T21:01:01.348410Z WARN  server_config::defaults] New management key generated. private in base64 = SDB6bW/9Vwwy8KvLONkqPwPzaRnb51MzoNkm18fJ790=; peer_id = 12D3KooWCArczSKMzpnyfxKradjE25NEzcfQghkKrtDNuPbsvSU9
[2021-03-16T21:01:01.348455Z INFO  particle_node] AIR interpreter: "./aquamarine_0.7.5.wasm"
[2021-03-16T21:01:01.348608Z INFO  particle_node::config::certificates] storing new certificate for the key pair
[2021-03-16T21:01:01.348862Z INFO  particle_node] public key = FbBMwyYsRvutVSaPNhLYzUyghzHZFewXvmE7SdowNPHB
[2021-03-16T21:01:01.350296Z INFO  particle_node::node] server peer id = 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
[2021-03-16T21:01:01.353939Z INFO  particle_node::node] Fluence listening on ["/ip4/0.0.0.0/tcp/7777", "/ip4/0.0.0.0/tcp/9999/ws"]
[2021-03-16T21:01:01.356075Z INFO  particle_node] Fluence has been successfully started.
[2021-03-16T21:01:01.356098Z INFO  particle_node] Waiting for Ctrl-C to exit...
[2021-03-16T21:01:01.358364Z INFO  tide::server] Server listening on http://0.0.0.0:18080
[2021-03-16T21:01:02.067989Z INFO  particle_node::network_api] Connected bootstrap 12D3KooWB9P1xmV3c7ZPpBemovbwCiRRTKd3Kq2jsVPQN4ZukDfy @ [/dns4/net10.fluence.dev/tcp/7001]
[2021-03-16T21:01:02.068067Z INFO  particle_node::network_api] Connected bootstrap 12D3KooWEXNUbCXooUwHrHBbrmjsrpHXoEphPwbjQXEGyzbqKnE9 @ [/dns4/net01.fluence.dev/tcp/7001]
<snip>
```

Now we are in a position to deploy our service using the `fldist` tool to the local node. In your project directory, run:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 new_service --ms artifacts/greeting.wasm:greeting_cfg.json -n MyGreeting
```

And if all went well, you should see output similar to:

```text
client seed: 3XUwhqLs7yLHqwE4xnh2C7LitvmT3dFq6Tj1shSRWw1A
client peerId: 12D3KooWH2tx7ywW8nvZuGztJMFHhFh16g9fR63BkEQS6QYbG95o
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
uploading blueprint MyGreeting to node 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 via client 12D3KooWH2tx7ywW8nvZuGztJMFHhFh16g9fR63BkEQS6QYbG95o
service id: 9712f9ca-7dfd-4ff5-817d-aef9e1e92e03
service created successfully
```

Which not only confirms the success of our operation but also gives us the _service id, 9712f9ca-7dfd-4ff5-817d-aef9e1e92e03_ in this case. We can further check the success of our operation by checking the installed modules on our local node with `fldist get_modules`:

```text
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 get_modules
client seed: AgZjbuMvZmCWbqZBABXXtv3cjGTqYFfiVj7aqg8dm2fA
client peerId: 12D3KooWFhUMisVC2VtXAertXt5oQQ7Xj1qppFZRM4mvEQ1iUaBP
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
[{"config":{"logger_enabled":true,"logging_mask":null,"mem_pages_count":100,"mounted_binaries":null,"wasi":{"envs":null,"mapped_dirs":null,"preopened_files":[]}},"hash":"c8aec6cbbc0a9632bf532b9553092ae6f66d2e3a5f71e11d1fe65e423c2204e2","name":"greeting"},{"config":{"logger_enabled":true,"logging_mask":null,"mem_pages_count":100,"mounted_binaries":null,"wasi":{"envs":null,"mapped_dirs":null,"preopened_files":[]}},"hash":"915d7487d4ae99f6136a7fe053c4ebd52cde1650c47492a315287117cedd0d3a","name":"greeting"}]

```

 Which confirms our recent upload!!

Now that we have a service on our local node, we need to construct our AIR script to build our frontend. 

```text
(xor
    (seq
        (call relay (service  "greeting") [name] result)
        (call %init_peer_id% (returnService "run") [result])
    )
    (call %init_peer_id% (returnService "run") [%last_error%])
)
```

As we've seen in the Quick Start section, we call the service _"greeting"_ with service id _service_  and the method parameter _name_.  As usual, we use the `fldist` tool to execute the AIR script:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air -p greeting.clj -d '{"service":"9712f9ca-7dfd-4ff5-817d-aef9e1e92e03", "name": "Fluence"}'
```

Giving us the expected response:

```bash
client seed: EV3bFK7mnqk58HrssTfCdXeYSzrVeiTzxWmh2B7k2g6R
client peerId: 12D3KooWLYtUhCj392W8XMhToCVrrsjowVdLirBzNHkEqCDmpe17
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: 3dbbdfa6-7401-438d-89b9-53413b0022e4. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  "Hi, Fluence"
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: '9712f9ca-7dfd-4ff5-817d-aef9e1e92e03',
      function_name: 'greeting',
      json_path: ''
    }
  ]
]
===================
```

And that's a wrap.

## Summary

In this section we worked through the various requisites and requirements to develop modules and services. To recap:

1. Create a Rust bin project and update the Cargo.toml to reflect our binary target
2. Mark public module functions with the _FCE_ macro
3. Build and compile the project with the `fce` tool
4. Create a service config toml file to specify wasm file location, included modules, module permissions and more
5. Use `fce-repl` to inspect and test modules and services
6. Create a deployment json config file for each module for service deployment
7. Deploy a service with `fldist` tool
8. Execute the service with an AIR script from the `fldst` command-line tool

