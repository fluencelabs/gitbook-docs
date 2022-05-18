# Deploy A Local Fluence Node

A significant chunk of developing and testing of Fluence services can be accomplished on an isolated, local node. In this brief tutorial we set up a local, dockerized Fluence node and test its functionality. In subsequent tutorials, we cover the steps required to join an existing network or how to run your own network.

The fastest way to get a Fluence node up and running is to use [docker](https://docs.docker.com/get-docker/):

```bash
docker run -d --name fluence -e RUST_LOG="info" -p 7777:7777 -p 9999:9999 -p 5001:5001 -p 18080 fluencelabs/fluence
```

where the `-d` flag runs the container in detached mode, `-e` flag sets the environment variables, `-p` flag exposes the ports: 7777 is the tcp port, 9999 the websocket port, 5001 the ipfs port and, 18080 the Prometheus port. Note that Prometheus is called with  /`metrics` , e.g., `http://127.0.0.1:18080/metrics` .

Once the container is up and running, we can tail the log (output) with

```
docker logs -f fluence
```

Which gives os the logged output:

```bash
[2021-12-02T19:42:20.734559Z INFO  particle_node]
    +-------------------------------------------------+
    | Hello from the Fluence Team. If you encounter   |
    | any troubles with node operation, please update |
    | the node via                                    |
    |     docker pull fluencelabs/fluence:latest      |
    |                                                 |
    | or contact us at                                |
    | github.com/fluencelabs/fluence/discussions      |
    +-------------------------------------------------+

[2021-12-02T19:42:20.734599Z INFO  server_config::resolved_config] Loading config from "/.fluence/v1/Config.toml"
[2021-12-02T19:42:20.734842Z INFO  server_config::keys] Generating a new key pair to "/.fluence/v1/builtins_secret_key.ed25519"
[2021-12-02T19:42:20.735133Z INFO  server_config::keys] Generating a new key pair to "/.fluence/v1/secret_key.ed25519"
[2021-12-02T19:42:20.735409Z WARN  server_config::defaults] New management key generated. ed25519 private key in base64 = M2sMsy5qguJIEttNct1+OBmbMhVELRUzBX9836A+yNE=
[2021-12-02T19:42:20.736364Z INFO  particle_node] AIR interpreter: "/.fluence/v1/aquamarine_0.16.0-restriction-operator.9.wasm"
[2021-12-02T19:42:20.736403Z INFO  particle_node::config::certificates] storing new certificate for the key pair
[2021-12-02T19:42:20.736589Z INFO  particle_node] node public key = 3iMsSHKmtioSHoTudBAn5dTtUpKGnZeVGvRpEV1NvVLH
[2021-12-02T19:42:20.736616Z INFO  particle_node] node server peer id = 12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D
[2021-12-02T19:42:20.739248Z INFO  particle_node::node] Fluence listening on ["/ip4/0.0.0.0/tcp/7777", "/ip4/0.0.0.0/tcp/9999/ws"]
<snip>
```



For future interaction with the node, we need to retain the server peer id \`12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D\`, which may be different for you.&#x20;

And if you feel the need to snoop around the container:

```bash
docker exec -it fluence bash
```

will get you in.

Now that we have a local node, we can use `aqua cli` to interact with it. From the Quick Start, you may recall that we need the node-id and node-addr:

* node-id: `12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D`
* node-addr: `/ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D`

Let's inspect our node and check for any available modules and interfaces:

```
aqua remote list_modules \
     --addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D
```

Let's us check on available modules and gives us a list of the builtin modules:

```bash
Your peerId: 12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D
[
  {
    "config": {
      "max_heap_size": "100 KiB"
    },
    "hash": "558a483b1c141b66765947cf6a674abe5af2bb5b86244dfca41e5f5eb2a86e9e",
    "name": "sqlite3"
  },
  {
    "config": {
      "logger_enabled": true,
      "mounted_binaries": {
        "ipfs": "/usr/bin/ipfs"
      }
    },
    "hash": "f72aeaaef7075b8fbf09e101ba82e79cb08abefd0b6e602538bd440ff17c2329",
    "name": "ipfs_effector"
  },
  {
    "config": {},
    "hash": "82353f3ae7cab489b158f6b602acd82c603f0550ed56c7edaa77823a08596d12",
    "name": "trust-graph"
  },
  {
    "config": {},
    "hash": "1a5a7286dc29b76be4752f4cacac8c0122eea9f1d370d7777bcc51493bf3b6b7",
    "name": "sqlite3"
  },
  {
    "config": {},
    "hash": "e6454e0a0b5da5ab3c25f5190cea6d24189de34eaa2f9bce508db5a07ed1a465",
    "name": "aqua-dht"
  },
  {
    "config": {
      "logger_enabled": true
    },
    "hash": "96fc75aad39341626e7ebf610003e3be7c6c6b377281be4a93bb8205019223b2",
    "name": "ipfs_pure"
  }
]
```

And checking on available interfaces:

```
aqua remote list_interfaces \
      --addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D 
```

Results in:

```
Your peerId: 12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D
[]
```

Since we just initiated the node, we expect no modules and no interfaces and the `fldist` queries confirm our expectations. To further explore and validate the node, we can create a small [greeting](https://github.com/fluencelabs/fce/tree/master/examples/greeting) service.

```bash
mkdir fluence-greeter
cd fluence-greeeter
# download the greeting.wasm file into this directory:
# https://github.com/fluencelabs/marine/blob/master/examples/greeting/artifacts/greeting.wasm -- Download button to the right
echo '{ "name":"greeting"}' > greeting_cfg.json
```

We just grabbed the greeting Wasm file from the Fluence repo and created a service configuration file, `echo_greeter_deploy_cfg.json`, which allow us to create a new GreetingService:

```bash
aqua remote deploy_service \
     --addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D \
     --data-path configs/echo_greeter_deploy_cfg.json \
     --service echo-greeter
```

Which gives us the service id:

```
our peerId: 12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D
"Going to upload a module..."
2022.02.22 01:42:42 [INFO] created ipfs client to /ip4/127.0.0.1/tcp/5001
2022.02.22 01:42:42 [INFO] connected to ipfs
2022.02.22 01:42:44 [INFO] file uploaded
"Now time to make a blueprint..."
"Blueprint id:"
"de3e242cb4489f2ed04b4ad8ff0e7cee701b75d86422c51b691dfeee8ab4ed92"
"And your service id is:"
"0b42ec01-c79e-438b-bded-c0b967a532c6"
```

We now have a greeting service running on our node. As always, take note of the service id and we can check on the deployed interface with:&#x20;

```bash
aqua remote get_interface \
     --addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D \
     --id 0b42ec01-c79e-438b-bded-c0b967a532c6
```

Which now lists:

```
Your peerId: 12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D
{
  "function_signatures": [
    {
      "arguments": [
        [
          "name",
          "string"
        ],
        [
          "greeter",
          "bool"
        ]
      ],
      "name": "greeting",
      "output_types": [
        "string"
      ]
    }
  ],
  "record_types": []
}
```

Writing a small Aqua script allows us to use the service:

```python
service GreetingService("service-id"):
    greeting: string -> string

func greeting(name:string, node:string, greeting_service_id: string) -> string:
  on node:
    GreetingService greeting_service_id
    res <- GreetingService.greeting(name)
  <- res
```

We run the script with [`aqua`](https://doc.fluence.dev/aqua-book/getting-started/quick-start)

```
aqua run \
    -a /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D \
    -i aqua/ \
    -f 'greeting("Fluence", true, "12D3KooWCXj3BQuV5d4vhgyLFmv7rRYiy9MupFiyEWnqcUAGpS4D", "04ef4459-474a-40b5-ba8d-1e9a697206ab")'
```

```bash
Your peerId: 12D3KooWAMTVBjHfEnSF54MT4wkXB1CvfDK3XqoGXt7birVsLFj6
[
  "Hi, Fluence"
]
```

Yep, our node and the tools are working as expected. Going back to the logs, we can further verify the script execution:

```bash
docker logs -f fluence
```

And check from the bottom up:

```
<snip>
[2021-03-12T02:42:51.041267Z INFO  aquamarine::particle_executor] Executing particle 14db3aff-b1a9-439e-8890-d0cdc9a0bacd
[2021-03-12T02:42:51.041927Z INFO  particle_closures::host_closures] Executed host call "64551400-6296-4701-8e82-daf0b4e02751" "greeting" (96us 700ns)
[2021-03-12T02:42:51.046652Z INFO  particle_node::network_api] Sent particle 14db3aff-b1a9-439e-8890-d0cdc9a0bacd to 12D3KooWLFqJwuHNe2kWF8SMgX6cm24L83JUADFcbrj5fC1z3b21 @ [/ip4/172.17.0.1/tcp/61636/ws]
```

Looks like our node container and logging is up and running and ready for your development use. As the Fluence team is rapidly developing, make sure you stay up to date. Check the repo or [Docker hub](https://hub.docker.com/r/fluencelabs/fluence) and update with `docker pull fluencelabs/fluence:latest`.

Happy composing!
