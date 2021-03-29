# Setting Up

For our database solution we have two goals in mind:

1. Persist reward block information and 
2. Allow for the querying of the database

while maintaining not inly data integrity but also access control, which implies that we need some sort of authentication and authorization scheme that allows us to nominate and process

Again using the [Fluence Dashboard](https://dash.fluence.dev) we can find for a Sqlite database service solution, such as this [service](https://dash.fluence.dev/blueprint/b67afc58ed7d15757303b60e694ed5083faedb466b7cc36242fa0979d4f8b1b7), which gives us:

* service id: 506528d3-3aaf-4ef5-a97d-18f1654fcf8d and
* node id: 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH

As with most stateful solutions, we need to take care of a little pre-work to set up the database, e.g., create tables, insert default values, etc., which requires us to fire a one time initialization call. But before we get to the service initialization, let's write a small script to check whether we have the necessary owner privileges and we do that with an AIR script:

```text
(xor
    (seq
        (seq
            (call relay ("op" "identity") [])
            (call node (service "am_i_owner") [] result)
            ; (call node_1 (service "get_tplet") [] result)
        )
        (seq
            (call relay ("op" "identity") [])
            (call %init_peer_id% (returnService "run") [result])
        )
    )
    (seq
        (call relay ("op" "identity") [])
        (call %init_peer_id% (returnService "run") ["XOR FAILED" %last_error%])
    )   
)
```

This should look rather familiar as we're calling a am\_i\_owner method of the remote service to see if `owner == us` using the familiar `fldist` tool:

```bash
fldist --node-id 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH run_air  -p air-scripts/ethqlite_owner.clj -d '{"service": "506528d3-3aaf-4ef5-a97d-18f1654fcf8d", "node_1": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH"}'
```

which yields

```bash
[
  0
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'am_i_owner',
      json_path: ''
    }
  ]
]
```

Looking at that pesky 0, i.e. false, return value, it seems that we have no owner privileges. Let's see if the service really enforces ownership requirements and execute the init script discussed earlier. Our AIR script is pretty short and executes only the `init_service` method to handle table creation.

```text
(xor
    (seq
        (seq
            (call relay ("op" "identity") [])
            (call node_1 (service "init_service") [] result)
        )
        (seq
            (call relay ("op" "identity") [])
            (call %init_peer_id% (returnService "run") [result])
        )
    )
    (seq
        (call relay ("op" "identity") [])
        (call %init_peer_id% (returnService "run") ["XOR FAILED" %last_error%])
    )   
)
```

Run with:

```bash
fldist --node-id 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH run_air  -p air-scripts/ethqlite_init.clj -d '{"service": "506528d3-3aaf-4ef5-a97d-18f1654fcf8d", "node_1": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH"}'fldist --node-id 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH run_air  -p air-scripts/ethqlite_init.clj -d '{"service": "506528d3-3aaf-4ef5-a97d-18f1654fcf8d", "node_1": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH"}'
```

which yields

```bash
[
  {
    "err_msg": "Not authorized to use this service",
    "success": 0
  }
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'init_service',
      json_path: ''
    }
  ]
]
```

Looks like the service wasn't joking and we need to have ownership privileges to even initiate the service. In order to prove ownership for the service, we need what's called the client seed, or just seed, in Fluence parlance, which is derived from a the private key of a user's keypair and used in the module upload and service deployment process. For our purposes, it suffices to say that the seed is `Dq3rsUZUs25FGrZM3qpiUzyKJ3NFgtqocgGRqWq9YGsx` and re-running the scripts with the seed information largely improves our lot.

```bash
fldist --node-id 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH run_air  -p air-scripts/ethqlite_owner.clj -d '{"service": "506528d3-3aaf-4ef5-a97d-18f1654fcf8d", "node_1": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH"}' -s Dq3rsUZUs25FGrZM3qpiUzyKJ3NFgtqocgGRqWq9YGsx
```

now gives us a juicy thumbs up regarding ownership:

```bash
[
  1
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'am_i_owner',
      json_path: ''
    }
  ]
]
```

and the db init

```bash
fldist --node-id 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH run_air  -p air-scripts/ethqlite_init.clj -d '{"service": "506528d3-3aaf-4ef5-a97d-18f1654fcf8d", "node_1": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH"}' -s Dq3rsUZUs25FGrZM3qpiUzyKJ3NFgtqocgGRqWq9YGsx
```

also executes:

```bash
[
  {
    "err_msg": "",
    "success": 1
  }
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'init_service',
      json_path: ''
    }
  ]
]
```

We now have the tools and confidence that we can authenticate as the service owner and that the service actually implements some authorization schema. Specifically, we expect all state changing operations to utilize the authorization guard and the read operations to be available without authorization.

