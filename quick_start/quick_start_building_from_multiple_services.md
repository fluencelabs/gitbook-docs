# Building An Application From Multiple Services

In this section, we compose multiple services into an application to catalog block miner addresses and block rewards for the latest block created on the \[Ethereum\[\([https://ethereum.org/en/](https://ethereum.org/en/)\) mainnet. This block reward data is useful to track miner and pool dominance as well ETH supply and related indexes. For convenience purposes, we use the \[Etherscan API\]\([https://etherscan.io/apis](https://etherscan.io/apis)\) for this portion of the tutorial and in order to proceed, you should have an Etherscan API Key or get one from [Etherscan](https://etherscan.io/apis).

Since we are composing our application from first principles, we use the following services to compose our app:

* _get\_latest\_block_: takes an api key and returns the latest block number as a hex string
* _hex2int_: takes a hex string and returns up to a u64
* _get\_block_: takes an api key and block number as int and returns the reward block info
* _extract\_miner\_address_: takes a json string and return the miner address

Let's find these services on the [Fluence Dashboard](https://dash.fluence.dev/) and make a note of the corresponding associated service and network ids:

* [Ethereum Block Getter](https://dash.fluence.dev/blueprint/801037186238469ce354d2eb6d884091aaf9622ba7b1a83816cc45d39ab2000d) provides methods to retrieve the latest, most recent Ethereum block number as a hex string and a reward block getter method, which gets the block information for a given integer block number. And yes, this service exposes two methods
  * service id: `74d5c5da-4c83-4af9-9371-2ab5d31f8019`
  * node id: `12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H`
* [Hex Converter](https://dash.fluence.dev/blueprint/63ff63360ef64651f712a2ecf08868d1a71f9dff0af04e234e4d543a66872806), which exposes the hex\_to\_int method to convert a hex string \(starting with 0x\) to an integer value
  * service id: `285e2a5e-e505-475f-a99d-15c16c7253f9`
  * node id: `12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH`
* [Extract Miner Address](https://dash.fluence.dev/blueprint/16a22a4033b6e98c45ac603fb520db77f4dcf42bf143f0d935262cb43136647e) extracts the miner address from a reward block json string
  * service id: `d13da294-004a-4c71-8631-a351c5f3489b`
  * node id: `12D3KooWCKCeqLPSgMnDjyFsJuWqREDtKNHx1JEBiwaMXhCLNTRb`

Let's test the hex conversion test in isolation with the following AIR script:

```text
(xor
    (seq
        (seq
            (call relay ("op" "identity") [])
            (call hex_node (hex_service "hex_to_int") [hex_string] hex_result)
        )
        (seq
            (call relay ("op" "identity") [])
            (call %init_peer_id% (returnService "run") [hex_result])
        )
    )
    (seq
        (call relay ("op" "identity") [])
        (call %init_peer_id% (returnService "run") ["XOR FAILED" %last_error%])
    )   
)
```

{% hint style="info" %}
Aquamarine Intermediate Representation \(AIR\) is a low level implementation of Aquamarine to coordinate services into applications. For a detailed introduction to AIR, please refer to the [Knowledgebase](../knowledge_knowledge/knowledge_aquamarine/knowledge_aquamarine_air.md). For the remainder of this section, we will see three AIR _instructions_: _seq, xor,_ and _call._

_seq_ is the _sequential_ instruction that wraps arguments and executes them, you guessed it, sequentially. And yes, there is a _parallel_ instruction in the language.

_xor_ is the _branching_ instruction that takes two **instructions,** e.g., two _seq_ instructions as arguments and evaluates the first argument only to proceed to the second instruction if the first one failed.

_call_ is the _execution_ instruction to launch distributed service methods and takes the following data:  
            **\(**_call_  **node-id \(service-id  service-method\) \[input parameters\] result\)**
{% endhint %}

As with the previous AIR script, the _xor_ takes care of capturing errors in case things don't pan out the way we've planned. Other than that, we are calling the `hex_to_int` method and we need to supply the service and node ids as well the the hex value. Save the above script to a local file called _hex2int.clj_ and use `fldist` to deploy the script:

```bash
fldist run_air -p hex2int.clj -d '{"hex_service":"285e2a5e-e505-475f-a99d-15c16c7253f9", "hex_node": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH", "hex_string":"0xF"}'
```

which rewards our efforts with:

```bash
client seed: 4G1GSe9sd38wSG4Uh9JVGYXzHY4nacYXJmAdYGGoM5Xz
client peerId: 12D3KooWC3ntvNvUP6K4XADrkkZJcJs4ZK4kkgoMWZ3ou8C4AQ12
node peerId: 12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb
Particle id: c0f44da7-3bfb-445b-896a-537c10143392. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  15
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
```

We input the hex string 0xF and, as expected, got 15 radius 10 back. Whoever implemented the hex conversion service seemingly got it right. So let's keep using it as we coordinate an application from multiple services.

Beware but do not fear the nesting and parenthesis!! As we're building a more complex application, our script of course grows a bit. Next, we use the get\__latest\_block_ function and feed the result, a hex string, into the _hex\_to\_int c\_onversion function and feed its output, an integer, to the \_get\_block_ function to arrive at the reward block data. Of course, we wrap it all into the trusty XOR just in case something goes wrong.

```text
(xor
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
        (call relay ("op" "identity") [])
        (call %init_peer_id% (returnService "run") ["XOR FAILED" %last_error%])
    )   
)
```

Before we deploy the script, notice that we made explicit provisions for service and node id associated with each method and we used the output, i.e., result, as input parameters for subsequent method calls. This further illustrates how Aquamarine allows developers to efficiently write applications from distributed network services.

Save the script locally to a file named _block\_geter_.clj and deploy it with `fldist`:

```bash
fldist run_air -p block_getter.clj -d '{"service_1":"74d5c5da-4c83-4af9-9371-2ab5d31f8019", "service_2":"285e2a5e-e505-475f-a99d-15c16c7253f9", "node_1": "12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H","node_2": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH", "api_key":<your api key}'
```

Let's see what we got:

```bash
client seed: DZEJcJGCKgoZ6CPCnrZrrjdqkHu8LdnVVuPkKyfsXSjE
client peerId: 12D3KooWNMF2yNXCjtCXCk7MnrjtXqj7Ej2jLa2kPoEBiseavvso
node peerId: 12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb
Particle id: 50f54bad-03f3-41ba-9950-9f18b47fbdee. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  "0xb70466"
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
  11994214
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
  "{\"status\":\"1\",\"message\":\"OK\",\"result\":{\"blockNumber\":\"11994214\",\"timeStamp\":\"1615158157\",\"blockMiner\":\"0xea674fdde714fd979de3edf0f56aa9716b898ec8\",\"blockReward\":\"4345475914435035590\",\"uncles\":[],\"uncleInclusionReward\":\"0\"}}"
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
```

Very cool. Our coordinated service flow generates the expected latest block hex string, which serves as an input to the hex conversion and the resulting integer value is used as an input in the get\_block method, which returns the associated reward block information. Just as planned. Beautiful.

Of course, that leaves us wanting as our goal was to get the reward miner address. Not to worry, we incorporate the missing _extract\_miner\_address_ service call into our AIR script:

```text
(xor
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
                (call node_3 (service_3 "extract_miner_address") [block_result] reward_result)
            )
            (seq
                (call relay ("op" "identity") [])
                (call %init_peer_id% (returnService "run") [reward_result])
            )
        )
    )
    (seq
        (call relay ("op" "identity") [])
        (call %init_peer_id% (returnService "run") ["XOR FAILED" %last_error%])
    )   
)
```

Update your _block\_getter.clj_ file with the updated AIR script and fire up our trusty `fldist`:

```bash
fldist run_air -p block_getter.clj -d '{"service_1":"74d5c5da-4c83-4af9-9371-2ab5d31f8019", "service_2":"285e2a5e-e505-475f-a99d-15c16c7253f9","service_3":"d13da294-004a-4c71-8631-a351c5f3489b" ,"node_1": "12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H","node_2": "12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH", "node_3":"12D3KooWCKCeqLPSgMnDjyFsJuWqREDtKNHx1JEBiwaMXhCLNTRb", "api_key":<your api key>}'
```

and let's admire our handiwork:

```bash
client seed: Az2SSwrLxEgbRZFdTR7ezcFgVu8D2YzND3ceLcBbReZ1
client peerId: 12D3KooWK3uZKW9kk1p6zt4NHNx9HB16T17kBjxkt8JB7vAFmvUC
node peerId: 12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb
Particle id: cdf7d2f0-3e99-4a9e-bb17-28b3e0326c97. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  "0xb70519"
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
  11994393
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
  "{\"status\":\"1\",\"message\":\"OK\",\"result\":{\"blockNumber\":\"11994393\",\"timeStamp\":\"1615160413\",\"blockMiner\":\"0x2f731c3e8cd264371ffdb635d07c14a6303df52a\",\"blockReward\":\"3527180289300104721\",\"uncles\":[],\"uncleInclusionReward\":\"0\"}}"
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
  "\"0x2f731c3e8cd264371ffdb635d07c14a6303df52a\""
]
[
  [
    {
      peer_pk: '12D3KooWCKCeqLPSgMnDjyFsJuWqREDtKNHx1JEBiwaMXhCLNTRb',
      service_id: 'd13da294-004a-4c71-8631-a351c5f3489b',
      function_name: 'extract_miner_address',
      json_path: ''
    }
  ]
]
===================
```

As expected, we now also have the miner address for the latest block mined.\[^1\]

In summary, we set out to obtain the miner address of the most recent, aka latest, block created on Ethereum and identified a number of suitable, reusable services already available on different peers of the Fluence testnet to achieve our goal.

We again utilized Aquamarine and wrote an increasingly comprehensive AIR script to coordinate and compose these services into our application and successfully obtained the miner addresses for the latest Ethereum block.

AIR, once again, proved to be a powerful and efficient tool and illustrates the power of the p2p reusable services model and the ease to change and expand an application's or backend's feature and capabilities set by means of coordination.

In the next section, we add state to persist our results for future use by composing Sqlite into our application workflow.

\[^1\]: If you get an error instead of the well-formed result, it's most likely due to a null return from _get\_block_, which can happen if the block hasn't finalized, has been dropped, etc. at the time of the call.

