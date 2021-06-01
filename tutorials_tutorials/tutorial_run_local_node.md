# Deploy A Local Fluence Node

A significant chunk of developing and testing of Fluence services can be accomplished on an isolated, local node. In this brief tutorial we set up a local, dockerized Fluence node and test its functionality. In subsequent tutorials, we cover the steps required to join an existing network or how to run your own network.

The fastest way to get a Fluence node up and running is to use [docker](https://docs.docker.com/get-docker/):

```bash
docker run -d --name fluence -e RUST_LOG="info" -p 7777:7777 -p 9999:9999 -p 18080 fluencelabs/fluence
```

where the `-d` flag runs the container in detached mode, `-e` flag sets the environment variables, `-p` flag exposes the ports: 7777 is the tcp port, 9999 the websocket port, and, optionally, 18080 the Prometheus port.

Once the container is up and running, we can tail the log \(output\) with

```bash
docker logs -f fluence

[2021-03-11T01:31:17.574274Z INFO  particle_node]
    +-------------------------------------------------+
    | Hello from the Fluence Team. If you encounter   |
    | any troubles with node operation, please update |
    | the node via                                    |
    |     docker pull fluencelabs/fluence:latest      |
    |                                                 |
    | or contact us at                                |
    | github.com/fluencelabs/fluence/discussions      |
    +-------------------------------------------------+

[2021-03-11T01:31:17.575062Z INFO  server_config::fluence_config] Loading config from "/.fluence/Config.toml"
[2021-03-11T01:31:17.575461Z INFO  server_config::keys] generating a new key pair
[2021-03-11T01:31:17.575768Z WARN  server_config::defaults] New management key generated. private in base64 = VE0jt68kqa2B/SMOd3VuuPd14O2WTmj6Dl//r6VM+Wc=; peer_id = 12D3KooWNGuGgQVUA6aJMGMGqkBCFmLZqMwmp6pzmv1WLYdi7gxN
[2021-03-11T01:31:17.575797Z INFO  particle_node] AIR interpreter: "./aquamarine_0.7.3.wasm"
[2021-03-11T01:31:17.575864Z INFO  particle_node::config::certificates] storing new certificate for the key pair
[2021-03-11T01:31:17.577028Z INFO  particle_node] public key = BRqbUhVD2XQ6YcWqXW1D21n7gPg15STWTG8C7pMLfqg2
[2021-03-11T01:31:17.577848Z INFO  particle_node::node] server peer id = 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
<snip>
```

For future interaction with the node, we need to retain the `server peer id` 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx. And if you feel the need to snoop around the container:

```bash
docker exec -it fluence bash
```

will get you in.

Now that we have a local node, we can use the `fldist` tool to interact with it. From the Quick Start, you may recall that we need the node-id and node-addr:

* node-id: 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
* node-addr: /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx

Let's inspect our node and check for any available modules and interfaces:

```bash
fldist get_modules --node-id 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
client seed: 43PmCycRqLt9h3t5Dbmkc3vpNjF9qrNDEVLvQhjCQYSj
client peerId: 12D3KooWQXTe2aFzUsYFf9mBHe4poey45nmAoa8PQwCc2iy9BLMW
node peerId: 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
[[]]

fldist get_interfaces --node-id 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
client seed: DGf3E48yr73tJbxXpfxyNiRNFsoeRgxKUCpUDYafkXaN
client peerId: 12D3KooWEY37spzSbrg1GTFEo67p9X8cFqmYDHuzaBWWJ9aRT1G2
node peerId: 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
60000
[ [] ]
to expand interfaces, use get_interfaces --expand
```

Since we just initiated the node, we expect no modules and no interfaces and the `fldist` queries confirm our expectations. To further explore and validate the node, we can create a small [greeting](https://github.com/fluencelabs/examples/tree/master/greeting) service.

```bash
mkdir fluence-greeter
cd fluence-greeeter
# download the greeting.wasm file into this directory
# https://github.com/fluencelabs/fce/blob/master/examples/greeting/artifacts/greeting.wasm -- Download button to the right
echo '{ "name":"greeting"}' > greeting_cfg.json
```

We just grabbed the greeting wasm file from the Fluence repo and created a service configuration file, greeting\_cfg.json, which allow us to create a new GreetingService:

```bash
fldist --node-id 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx new_service --ms  /Users/bebo/localdev/fce/examples/greeting/artifacts/greeting.wasm:greeting_cfg.json -n GreetingService
client seed: 7VtMT7dbdfuU2ewWHEo42Ysg5B9KTB5gAgM8oDEs4kJk
client peerId: 12D3KooWRSmoTL64JVXna34myzAuKWaGkjE6EBAb9gaR4hyyyQDM
node peerId: 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
uploading blueprint GreetingService to node 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx via client 12D3KooWRSmoTL64JVXna34myzAuKWaGkjE6EBAb9gaR4hyyyQDM
NON-CONSTANT BLUEPRINT ID: Expected blueprint id to be predefined as 88b9b328-7c2b-44fe-8f2c-01b52db12fd9, but it was generated by node as 94d02dfe696549a98e23c5de8713e7c6d6f91694e823790a2f6dcfcc93843be3
service id: 64551400-6296-4701-8e82-daf0b4e02751
service created successfully
```

We now have a greeting service running on our node. As always, make a note of the `service id`, 64551400-6296-4701-8e82-daf0b4e02751.

```bash
fldist get_modules --node-id 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
client seed: HXoV5UfoBAtT8vM2zibm6oiTt7ecFBbP3xSF2dec4RTF
client peerId: 12D3KooWGJ8crCtYy4es835v5dVhTbD7snyLxCQupuiq2sLSXMyA
node peerId: 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
[[{"config":{"logger_enabled":true,"logging_mask":null,"mem_pages_count":100,"mounted_binaries":null,"wasi":{"envs":null,"mapped_dirs":null,"preopened_files":[]}},"hash":"80a992ec969576289c61c4a911ba149083272166ffec2949d9d4a066532eec1d","name":"greeting"}]]
```

Yep, checking once again for modules, the output confirms that the greeting service is available. Writing a small AIR script allows us to use the service:

```text
(xor
    (seq
        (call relay (service  "greeting") [name] result)
        (call %init_peer_id% (returnService "run") [result])
    )
    (call %init_peer_id% (returnService "run") [%last_error%])
)
```

Copy and save the script to greeting.clj and we can use our trusted `fldist` tool:

```bash
fldist --node-id 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx run_air -p greeting.clj -d '{"service": "64551400-6296-4701-8e82-daf0b4e02751", "name":"Fluence"}'
client seed: 8eXzEhypvkYST82sakeS4NeGFSyxqyCSpv2GQj3tQK5E
client peerId: 12D3KooWLFqJwuHNe2kWF8SMgX6cm24L83JUADFcbrj5fC1z3b21
node peerId: 12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx
Particle id: 14db3aff-b1a9-439e-8890-d0cdc9a0bacd. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  "Hi, Fluence"
]
[
  [
    {
      peer_pk: '12D3KooWLFCmDq4vDRfaxW2GA6kYnorxAiie78XzQrVDVoWEZnPx',
      service_id: '64551400-6296-4701-8e82-daf0b4e02751',
      function_name: 'greeting',
      json_path: ''
    }
  ]
]
===================
```

Yep, our node and the tools are working as expected. Going back to the logs, we can further verify the script execution:

```bash
docker logs -f fluence
<snip>
[2021-03-12T02:42:51.041267Z INFO  aquamarine::particle_executor] Executing particle 14db3aff-b1a9-439e-8890-d0cdc9a0bacd
[2021-03-12T02:42:51.041927Z INFO  particle_closures::host_closures] Executed host call "64551400-6296-4701-8e82-daf0b4e02751" "greeting" (96us 700ns)
[2021-03-12T02:42:51.046652Z INFO  particle_node::network_api] Sent particle 14db3aff-b1a9-439e-8890-d0cdc9a0bacd to 12D3KooWLFqJwuHNe2kWF8SMgX6cm24L83JUADFcbrj5fC1z3b21 @ [/ip4/172.17.0.1/tcp/61636/ws]
```

Looks like our node container and logging is up and running and ready for your development use. As the Fluence team is rapidly developig, make sure you stay up to date. Check the repo or [Docke hub](https://hub.docker.com/r/fluencelabs/fluence) and update with `docker pull fluencelabs/fluence:latest`.

Happy composing!

