# CRUD All the Way

It's finally time to populate our shiny new storage as a service with some data. In order to suss out our system, let's run an AIR smoketest. Save the script below to ethqlite\_roundtrip.clj.

```text
(xor
    (seq
        (seq
            (seq
                (seq
                    (seq
                        (seq
                            (seq
                                (call relay ("op" "identity") [])
                                (call node_1 (service_1 "get_latest_block") [api_key] hex_block_result)
                            )
                            (seq
                                (call relay ("op" "identity") [])
                                (call %init_peer_id% (returnService "run") [hex_block_result])
                            )
                        )
                        (seq
                            (seq
                                (call relay ("op" "identity") [])
                                (call node_2 (service_2 "hex_to_int") [hex_block_result] int_block_result)
                            )
                            (seq
                                (call relay ("op" "identity") [])
                                (call %init_peer_id% (returnService "run") [int_block_result])
                            )
                        )
                    )
                    (seq
                        (seq
                            (call relay ("op" "identity") [])
                            (call node_1 (service_1 "get_block") [api_key int_block_result] block_result)
                        )
                        (seq
                            (call relay ("op" "identity") [])
                            (call %init_peer_id% (returnService "run") [block_result])
                        )
                    )
                )
                (seq    
                    (seq
                        (call relay ("op" "identity") [])
                        (call sqlite_node (sqlite_service "update_reward_blocks") [block_result] insert_result)
                    )
                    (seq
                        (call relay ("op" "identity") [])
                        (call %init_peer_id% (returnService "run") [insert_result])
                    )
                )
            )
            (seq
                (seq
                    (call relay ("op" "identity") [])
                    (call sqlite_node (sqlite_service "get_latest_reward_block") [] select_result)
                )
                (seq
                    (call relay ("op" "identity") [])
                    (call %init_peer_id% (returnService "run") [select_result])
                )
            )
        )
        (seq
            (seq
                (seq
                    (call relay ("op" "identity") [])
                    (call sqlite_node (sqlite_service "get_reward_block") [int_block_result] select_result_2)
                )
                (seq
                    (call relay ("op" "identity") [])
                    (call %init_peer_id% (returnService "run") [select_result_2])
                )
            )
            (seq
                (seq
                    (call relay ("op" "identity") [])
                    (call sqlite_node (sqlite_service "get_miner_rewards") [select_result_2.$.["block_miner"]!] select_result_3)
                )
                (seq
                    (call relay ("op" "identity") [])
                    (call %init_peer_id% (returnService "run") [select_result_3])
                )
            )
        )
    )
    (seq
        (call relay ("op" "identity") [])
        (call %init_peer_id% (returnService "run") ["XOR FAILED" %last_error%])
    )  
)
```

The top part of the script is identical what we used before:

* get the latest block number \(`get_latest_block`\), 
* convert the hex string to an integer \(`hex_to_int`\) and
* retrieve the reward block info \(`get_block`\)

The new service components called are:

* _update\_reward\_blocks_, which takes the `get_block` output and writes it to the db. Please note that this service requires authentication,
* _get\_latest\_reward\_block_, which is a read operation querying the most recent row in the reward block table,
* _get\_reward\_block_, which takes a miner address and in this cae the one produced by `get_block`, and finally
* _get\_miner\_rewards_, which returns a list of miner rewards for a particular miner address; in this case, the one provided by the `get_reward_block` result. Note the `$` operator to access the `block_miner` field in the return struct and the `!` operator to flatten the response


From the previous section we know that

* service\_1: 74d5c5da-4c83-4af9-9371-2ab5d31f8019 , node\_1: 12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H
* service\_2: 285e2a5e-e505-475f-a99d-15c16c7253f9 , node\_2: 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH
* service\_sqlite : 506528d3-3aaf-4ef5-a97d-18f1654fcf8d , node\_sqlite: 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH
* api\_key: you Etherscan API key
* client seed: Dq3rsUZUs25FGrZM3qpiUzyKJ3NFgtqocgGRqWq9YGsx

which allows us to construct the payload for our `fldist` command-line app:

```bash
fldist --node-id 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH run_air  -p air-scripts/ethqlite_roundtrip.clj -d '{"service_1":"74d5c5da-4c83-4af9-9371-2ab5d31f8019", "node_1":"12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H","service_2":"285e2a5e-e505-475f-a99d-15c16c7253f9", "node_2": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH", "sqlite_service":"506528d3-3aaf-4ef5-a97d-18f1654fcf8d", "sqlite_node":"12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH", "api_key": "your api key"}' -s Dq3rsUZUs25FGrZM3qpiUzyKJ3NFgtqocgGRqWq9YGsx
```

and upon execution gives us the expected results flow:

```bash
===================
[
  "0xb736bc"
]
[
  [
    {
      peer_pk: '12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H',
      service_id: '74d5c5da-4c83-4af9-9371-2ab5d31f8019',
      function_name: 'get_latest_block',
      json_path: ''
    }
  ]
]
===================
===================
[
  12007100
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '285e2a5e-e505-475f-a99d-15c16c7253f9',
      function_name: 'hex_to_int',
      json_path: ''
    }
  ]
]
===================
===================
[
  "{\"status\":\"1\",\"message\":\"OK\",\"result\":{\"blockNumber\":\"12007100\",\"timeStamp\":\"1615330221\",\"blockMiner\":\"0x04668ec2f57cc15c381b461b9fedab5d451c8f7f\",\"blockReward\":\"3799386136990487274\",\"uncles\":[],\"uncleInclusionReward\":\"0\"}}"
]
[
  [
    {
      peer_pk: '12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H',
      service_id: '74d5c5da-4c83-4af9-9371-2ab5d31f8019',
      function_name: 'get_block',
      json_path: ''
    }
  ]
]
===================
===================
[
  {
    "err_str": "",
    "success": 1
  }
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'update_reward_blocks',
      json_path: ''
    }
  ]
]
===================
===================
[
  {
    "block_miner": "\"0x04668ec2f57cc15c381b461b9fedab5d451c8f7f\"",
    "block_number": 12007100,
    "block_reward": "3799386136990487274",
    "timestamp": 1615330221
  }
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'get_latest_reward_block',
      json_path: ''
    }
  ]
]
===================
===================
[
  {
    "block_miner": "\"0x04668ec2f57cc15c381b461b9fedab5d451c8f7f\"",
    "block_number": 12007100,
    "block_reward": "3799386136990487274",
    "timestamp": 1615330221
  }
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'get_reward_block',
      json_path: ''
    }
  ]
]
===================
===================
[
  {
    "miner_address": "\"0x04668ec2f57cc15c381b461b9fedab5d451c8f7f\"",
    "rewards": [
      "3799386136990487274"
    ]
  }
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'get_miner_rewards',
      json_path: ''
    }
  ]
]
===================
```

Feel free to re-run the workflow without the `-s` flag. Before we conclude, let's cleanup our digital footprint and reset the Sqlite service for other tutorial users.

```text
(xor
    (seq
        (seq
            (call relay ("op" "identity") [])
            (call sqlite_node (sqlite_service "owner_nuclear_reset") [] result)
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

save the script to _ethqlite\_reset.clj_ and run with:

```bash
fldist --node-id 12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH run_air  -p air-scripts/ethqlite_reset.clj -d '{"service": "506528d3-3aaf-4ef5-a97d-18f1654fcf8d", "node_1": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH"}' -s Dq3rsUZUs25FGrZM3qpiUzyKJ3NFgtqocgGRqWq9YGsx
```

to get

```bash
[
  1
]
[
  [
    {
      peer_pk: '12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH',
      service_id: '506528d3-3aaf-4ef5-a97d-18f1654fcf8d',
      function_name: 'owner_nuclear_reset',
      json_path: ''
    }
  ]
]
```

Of course, if you tried that without the `-s` flag, the return result would be 0. Try re-running the _ethqlite\_roundtrip.clj_ script after reset and without initialization.

In summary, you have extended the multi-service application with a Sqlite database service by simply adding the service methods to your existing workflow. We further extended the workflow by adding read queries which of course could be run separately or even as a separate application that may include a service-side paywall for data retrieval.

