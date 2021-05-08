# Additional Concepts

In the previous sections we obtained block reward data by discovering the latest Ethereum block created. Of course, Ethereum produces a new block about every 13 seconds or so and it would be nice to automate the data acquisition process. One way, of course, would be to, say, cron or otherwise daemonize our frontend application. But where's the fun in that and we'd rather hand that task to the p2p network.

As we have seen in our AIR workflows, particles travel the path, trigger execution, and update their data. So far, we have only seen services consume previous outputs as \(complete\) inputs, which means that service at workflow sequence s needs to be fairly tightly coupled to service at sequence s-1, which is less than ideal. Luckily, Fluence provides a solution to access certain types of results as j_son paths_.


## Peer-Based Script Storage And Execution

As discussed previously, a peer-based ability to "poll" is a valuable feature to some applications. Fluence nodes come with a set of built-in services including the ability to store scripts on a peer with the intent of periodic execution.

This service, just as all distributed services, is managed by Aquamarine. The AIR script looks like:

```text
; add a script to 
(call node ("script" "add") [script interval] id)
```

where:

* _node_ -- takes the peer id parameter
* _"script"_ -- is the \(hard-coded\) service id
* _script_  -- takes the AIR script as a **string**
* _interval_ -- the execution interval in seconds, optional, default is three \(3\) seconds; provide as **string**, e.g. five seconds are expressed as "5" 
* _id_ -- is the return value

In addition to the service "add" method, there are also service "list" and service "remove" methods available:

```text
; list 
(call node ("script" "list") [] list)

; remove
(call node ("script" "remove") [script_id] result)
```

where remove takes the id \(returned by "add"\) and returns a boolean.

Let's check on any stored services on our local node \(make sure you use your node id\) and as expected, no services have been uploaded for storage and execution.

```text
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air -p air-scripts/list_stored_services.clj -d '{"node":"12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17"}'
client seed: 5ydZWdJAzMHAGQ2hCVJCa5ByYq7obp2yc9gRD43ajXrZ
client peerId: 12D3KooWBgzuiNn5mz1DwqDbqapBf3NSF8mRjSJV1KC3VphjAyWL
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: 17986fb7-36e7-4f10-b311-d2512f5fe2e5. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  []
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: 'script',
      function_name: 'list',
      json_path: ''
    }
  ]
]
===================
```

In order to upload the periodic "block to db poll", we can use parts of the _ethqlite\_roundtrip.clj_ script and hard-code the parameters since currently there is no option to separately upload the script and data. Make sure you replace the `node_*`, `service_*` and `api_key` placeholders with your actual values in the file!

```text
; air-scripts/ethqlite_block_committer.clj
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
                (call sqlite_node (sqlite_service "update_reward_blocks") [block_result] insert_result)
            )
            (seq
                (call relay ("op" "identity") [])
                (call %init_peer_id% (returnService "run") [insert_result])
            )
        )
    )
    (seq
        (call relay ("op" "identity") [])
        (call %init_peer_id% (returnService "run") ["XOR FAILED" %last_error%])
    )  
)
)
```


```bash
# script file to string variable
AIR=`cat air-scripts/ethqlite_block_committer.clj`
# interval variable in seconds to string variable
INT="10"
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air  -p air-scripts/add_stored_service.clj -d '{"node":"12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17", "interval":"'"$INT"'", "script":"'"$AIR"'"}'
client seed: Cwhf8VuyqPCUPi8keyZAcRVBkaGNLWviHMRwDL2hG8D4
client peerId: 12D3KooWJgFCCeHpcEoVyxT5Fmg47ok43MPU7hfT9cNv5R3KeDEw
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: dd3ad854-b10d-4664-846d-42c59c59335f. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  "a1791c0f-084e-4b4d-a85c-a3eb65a18d57"        # <= Take note of the storage id !
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: 'script',
      function_name: 'add',
      json_path: ''
    }
  ]
]
===================
```

Checking once more for listed services hits pay dirt:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air -p air-scripts/list_stored_services.clj -d '{"node":"12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17"}'
client seed: HpHQc1as9zGdiHaMQzyPDaPWrdMVEvAA8DwdJiAvczWS
client peerId: 12D3KooWFiiS7FMo18EbrtWZi38Nwe1SiYCRqcsJNEPtYh28zHNm
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: 5fb0af87-310f-4b12-8c73-e044cfd8ef6e. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  [
    {
      "failures": 0,
      "id": "a1791c0f-084e-4b4d-a85c-a3eb65a18d57",
      "interval": "10s",
      "owner": "12D3KooWJgFCCeHpcEoVyxT5Fmg47ok43MPU7hfT9cNv5R3KeDEw",
      "src": "$AIR"
    }
  ]
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: 'script',
      function_name: 'list',
      json_path: ''
    }
  ]
]
===================
```

And we are golden. Give it some time and start checking Ethqlite for latest block and reward info!!

{% hint style="info" %}
Unfortunately, our daemonized service won't work just yet as the current implementation cannot take the \(client\) seed we need in order to get our SQLite write working. It's on the to-do list but if you need it, please contact us and we'll see about juggling priorities. 
{% endhint %}


For completeness sake, let's remove the stored service with the following AIR script:

```bash
; remove a script to 
(call node ("script" "remove") [script_id] result)
```

## Advanced Service Output Access

As Aquamarine advances a particle's journey through the network, output from a service at workflow sequence s-1 method tends to be the input for a service at sequence s method. For example, the _hex\_to\_int_ method, as used earlier, takes the output from the _get\_latest\_block_ method. With single parameter outputs, this is a pretty straight forward and inherently decoupled dependency relation. However, when result parameters become more complex, such as structs, we still would like to keep services as decoupled as possible.


Fluence provides this capability by facilitating the conversion of \(Rust\) struct returns into [json values](https://github.com/fluencelabs/aquamarine/blob/master/interpreter-lib/src/execution/boxed_value/jvaluable.rs#L30). This allows json type key-value access to a desired subset of return values. If you got back to the _ethqlite.clj_ script, you may notice some fancy `$`, `!` operators tucked away in the deep recesses of parenthesis stacking. Below the pertinent snippet:

```text
; ethqlite_rountrip.clj
; <snip>
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
            (call relay ("op" "identity") []) .; coming up next line !! 
            (call sqlite_node (sqlite_service "get_miner_rewards") [select_result_2.$.["block_miner"]!] select_result_3)  ; <=  Here it is !!
        )
        (seq
            (call relay ("op" "identity") [])
            (call %init_peer_id% (returnService "run") [select_result_3])
        )
    )
)
; <snip>
```

Before we dive in, let's review the output from the _get\_reward\_block_ method which is part of the ethqlite service:

```rust
// https://github.com/fluencelabs/examples/blob/c508d096e712b7b22aa94641cd6bb7c2fdb67200/multi-service/ethqlite/src/crud.rs#L139
#[fce]
#[derive(Debug)]
// https://github.com/fluencelabs/examples/blob/c508d096e712b7b22aa94641cd6bb7c2fdb67200/multi-service/ethqlite/src/crud.rs#L89
pub struct RewardBlock {
    pub block_number: i64,
    pub timestamp: i64,
    pub block_miner: String,
    pub block_reward: String,
}
```

and the input expectations of _get\_miner\_rewards_, also an ethqlite service method, with the following [function](https://github.com/fluencelabs/examples/blob/c508d096e712b7b22aa94641cd6bb7c2fdb67200/multi-service/ethqlite/src/crud.rs#L177) signature: `pub fn get_miner_rewards(miner_address: String) -> MinerRewards`.


Basically, _get\_miner\_rewards_ wants an Ethereum address as a `String` and in the context of our AIR script we want to get the value from the _get\_reward\_block_ result. Rather than tightly coupling _get\_miner\_rewards_ to _get\_reward\_block_ in terms of, say, the _RewardBlock_ input parameter, we take advantage of the Fluence capability to turn structs into json strings and then supply the relevant key to extract the desired value. Specifically, we use the `$` operator to access the json representation at the desired index and the `!` operator to flatten the value, if desired.

For example,

```text
(call sqlite_node (sqlite_service "get_miner_rewards") [select_result_2.$.["block_miner"]!]
```

uses the _block\_miner_ key to retrieve the miner address for subsequent consumption. In order to take full advantage of this important feature, developers should return more complex results as FCE structs to prevent tight service coupling. This approach allows for significant reduction of service dependencies, re-writes and re-deployments due to even minor changes in upstream dependencies.

