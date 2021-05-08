# SQLite Service

All our work so far has been about gathering block reward information for the latest block:

```javascript
// Block reward info on Wednesday, March 17, at 2021 7:42:57 PM GMT
// for block 12058131:
"{\"status\":\"1\",\"message\":\"OK\",\"result\":{\"blockNumber\":\"12058131\",
\"timeStamp\":\"1616010177\",\"blockMiner\":\"0x829bd824b016326a401d083b33d092293333a830\",
\"blockReward\":\"6159144598411626490\",\"uncles\":[
{\"miner\":\"0xe72f79190bc8f92067c6a62008656c6a9077f6aa\",\"unclePosition\":\"0\",
\"blockreward\":\"500000000000000000\"}],
\"uncleInclusionReward\":\"62500000000000000\"}}"
```

which [happens about every 13 seconds or so on mainnet](https://etherscan.io/chart/blocktime) and every four seconds on Kovan. Rather than stashing the block reward results in a frontend-based storage solution, we deploy an SQLite service as our peer-to-peer hosted _Ethqlite_ service. Please see the [ethqlite repo](https://github.com/fluencelabs/examples/tree/main/multi-service/ethqlite) for the code.

To get SQLite as a service, we build our service from two modules: the [ethqlite repo](https://github.com/fluencelabs/examples/tree/main/multi-service/ethqlite) and the [Fluence sqlite](https://github.com/fluencelabs/sqlite) Wasm module, which we can build or pickup as a wasm files from the [releases](https://github.com/fluencelabs/sqlite/releases). This largely, but not entirely, mirrors what we did with the cUrl service: build the service by providing an adapter to the binary. Unlike the cUrl binary, we are bringing our own sqlite binary, i.e., _sqlite3.wasm_, with us.

This leaves us to code our _ethqlite_ module with respect to desired CRUD interfaces and security. As [previously](../../quick_start/quick_start_add_persistence/quick_start_persistence_setup.md) discussed, we want writes to the sqlite services to be privileged, which implies that we need to own the service and have the client seed to manage authentication and ambient authorization. Specifically, we can implement a rudimentary authorization system where authentication implies authorization \(to write\). The `is_owner` function in the _ethqlite_ repo does exactly that: if the caller can prove ownership by providing a valid client seed, than we have a true condition equating write-privileged ownership with the caller identity:

```rust
// auth.rs
use fluence::{fce, CallParameters};
use::fluence;
use crate::get_connection;

pub fn is_owner() -> bool {
    let meta = fluence::get_call_parameters();
    let caller = meta.init_peer_id;
    let owner = meta.service_creator_peer_id;

    caller == owner
}

#[fce]
pub fn am_i_owner() -> bool {
    is_owner()
}
```

where the `fluence::get_call_parameters` is a FCE function returning the populated _CallParameter_ struct defined in the [Fluence Rust SDK](https://github.com/fluencelabs/rust-sdk/blob/71591f412cb65879d74e8c38838e827ab74d8802/crates/main/src/call_parameters.rs) provides us with the salient creator and caller parameters at runtime.

While the majority of the CRUD operations in _crud.rs_ are standard fare except, the auth & auth check appears in update\_reward\_blocks:

```rust
// crud.rs
#[fce]
pub fn update_reward_blocks(data_string: String) -> UpdateResult {   
    if !is_owner() {                                // <=  auth & auth check !!
        return UpdateResult { success:false, err_str: "You are not the owner".into()};
    }

    let obj:serde_json::Value = serde_json::from_str(&data_string).unwrap();
    let obj = obj["result"].clone();

    if obj["blockNumber"] == serde_json::Value::Null {
        return UpdateResult { success:false, err_str: "Empty reward block string".into()};
    }

    let conn = get_connection();

    let insert = "insert or ignore into reward_blocks values(?, ?, ?, ?)";
    let mut ins_cur = conn.prepare(insert).unwrap().cursor();
<snip>
```

That is, any non-permissioned call is prevented from write operations and an error message is returned. Please note that in [main.rs](https://github.com/fluencelabs/examples/blob/main/multi-service/ethqlite/src/main.rs) we have a few admin convenience functions that are also protected by the `is_owner` guard.

## Building and Deploying Ethqlite

Our _build.sh_ script should look quite familiar with the possible exception of downloading the already built sqlite3.wasm file:

```bash
# build.sh
#!/bin/sh

fce build --release

rm artifacts/*
cp target/wasm32-wasi/release/ethqlite.wasm artifacts/
wget https://github.com/fluencelabs/sqlite/releases/download/v0.10.0_w/sqlite3.wasm
mv sqlite3.wasm artifacts/
```


Run `./build.sh` and check the artifacts for the expected wasm files

Like all Fluence services, Ethqlite needs a [service configuration](https://github.com/fluencelabs/examples/blob/main/multi-service/ethqlite/Config.toml) file, which looks a little more involved than what we have seen so far.

```text
modules_dir = "artifacts/"

[[module]]
    name = "sqlite3"
    mem_pages_count = 100
    logger_enabled = false

    [module.wasi]
    preopened_files = ["/tmp"]
    mapped_dirs = { "tmp" = "/tmp" }



[[module]]
name = "ethqlite"
    mem_pages_count = 1
    logger_enabled = false

    [module.wasi]
    preopened_files = ["/tmp"]
    mapped_dirs = { "tmp" = "/tmp" }
```

Let's break it down:


* the first \[\[module\]\] section 
  * specifies the _sqlite3.wasm_ module we pulled from the repo, 
  * allocates memory, where each page is about 64KB, and 
  * permissions and maps node file access
* the second section is for our business logic \(CRUD\) adapter module where, again, we allocate the memory and permission and map file access.

We can now fire up `fce-repl`:

```bash
fce-repl Config.toml
Welcome to the FCE REPL (version 0.5.2)
app service was created with service id = 9b923db7-3747-41ab-b1fd-66bd0ccd9f68
elapsed time 916.210305ms

1> interface
Loaded modules interface:
UpdateResult {
  success: I32
  err_str: String
}
RewardBlock {
  block_number: S64
  timestamp: S64
  block_miner: String
  block_reward: String
}
InitResult {
  success: I32
  err_msg: String
}
MinerRewards {
  miner_address: String
  rewards: Array<String>
}
DBOpenDescriptor {
  ret_code: S32
  db_handle: U32
}
DBPrepareDescriptor {
  ret_code: S32
  stmt_handle: U32
  tail: U32
}
DBExecDescriptor {
  ret_code: S32
  err_msg: String
}

ethqlite:
  fn init_service() -> InitResult
  fn get_miner_rewards(miner_address: String) -> MinerRewards
  fn owner_nuclear_reset() -> I32
  fn get_reward_block(block_number: U32) -> RewardBlock
  fn update_reward_blocks(data_string: String) -> UpdateResult
  fn get_latest_reward_block() -> RewardBlock
  fn am_i_owner() -> I32

sqlite3:
  fn sqlite3_reset(stmt_handle: U32) -> S32
  <snip>
  fn sqlite3_column_blob(stmt_handle: U32, icol: S32) -> Array<U8>
```

and see all the public Fluence interfaces including the ones from the _sqlite3.wasm_ module. Let's upload the service to the local network:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 new_service --ms ethqlite/artifacts/sqlite3.wasm:ethqlite/sqlite3_cfg.json ethqlite/artifacts/ethqlite.wasm:ethqlite/ethqlite_cfg.json --name EthQlite
client seed: 7VqRt2kXWZ15HABKh1hS4kvGfRcBA69cYuzV1Rwm3kHv
client peerId: 12D3KooWCzWm4xBv7nApuK8vNLSbKKYV36kvkz3ywqj5xcjscnz9
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
uploading blueprint EthQlite to node 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 via client 12D3KooWCzWm4xBv7nApuK8vNLSbKKYV36kvkz3ywqj5xcjscnz9
service id: fb9ba691-c0fc-4500-88cc-b74f3b281088
service created successfully
```

Now that we crated the service on our local node, let's make sure that we have the necessary owner privileges. First, we create a little AIR script that calls the _am\_i\_owner_ function from thee ethqlite service:

```text
; am_i_owner.clj
(xor
    (seq
        (seq
            (call relay ("op" "identity") [])
            (call node_1 (service "am_i_owner") [] result)
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

and run it with the `fldist` tool:

```bash
 fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air -p air-scripts/am_i_owner.clj -d '{"service":"fb9ba691-c0fc-4500-88cc-b74f3b281088", "node":"12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17"}'
client seed: 3J8BqpGTQ1Ujbr8dvnpTxfr5EUneHf9ZwW84ru9sNmj7
client peerId: 12D3KooW9z5hBDY6cXnkEGraiPFn6hJ3VstqAkVaAM7oThTiWVjL
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: efa37779-e3aa-4353-b63d-12b444b6366b. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  0
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: 'fb9ba691-c0fc-4500-88cc-b74f3b281088',
      function_name: 'am_i_owner',
      json_path: ''
    }
  ]
]
===================
```

As discussed earlier, the service needs some proof that we have owner privileges, which we can provide by adding the client seed, `-s`, to our call parameters:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air -p air-scripts/am_i_owner.clj -d '{"service":"fb9ba691-c0fc-4500-88cc-b74f3b281088", "node":"12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17"}' -s 7VqRt2kXWZ15HABKh1hS4kvGfRcBA69cYuzV1Rwm3kHv
client seed: 7VqRt2kXWZ15HABKh1hS4kvGfRcBA69cYuzV1Rwm3kHv
client peerId: 12D3KooWCzWm4xBv7nApuK8vNLSbKKYV36kvkz3ywqj5xcjscnz9
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: f0371615-7d75-4971-84a9-3111b8263de7. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  1
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: 'fb9ba691-c0fc-4500-88cc-b74f3b281088',
      function_name: 'am_i_owner',
      json_path: ''
    }
  ]
]
===================
```

and all is well. So where does that client seed _7VqRt2kXWZ15HABKh1hS4kvGfRcBA69cYuzV1Rwm3kHv_ come from ? The easy answer is that we copied it from the service creation return values -- line 2 above. But that doesn't really answer the question. The more involved answer is that every developer should have one or more cryptographic key pairs from which the client seed is derived. Moreover, creating a new service, the client seed should be specified but if not, the system creates one instead as above.

The easiest way to get a keypair and seed is from the `fldist` tool:

```bash
fldist create_keypair
client seed: 8LKYUmsWkMSiHBxo8deXyNJD3wXutq265TSTcmmtgQTJ
client peerId: 12D3KooWRtrFyYjis4qQpC4kHcJWbtpM4mZgLYBoDn93eXJEGtVH
relay peerId: 12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb
{
  id: '12D3KooWKphxxaXofYzC2TsN79RHZVubjmutKVdPUxVMHY3ZsVww',
  privKey: 'CAESQO/TcX2DkTukK6XxJUc/2U6gqOLVza5PRWM2FhXfJ1qilKtA6qsHx0Rdibwxsg4Vh7JjTfRfMXSlLJphGCOb7zI=',
  pubKey: 'CAESIJSrQOqrB8dEXYm8MbIOFYeyY030XzF0pSyaYRgjm+8y',
  seed: 'H9BSbZwKmFs93462xbAyfEdGdMXb5LZuXL7GSA4uPK4V'
}
```

So let's re-deploy the Ethqlite service and specify the client seed at creation time:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 new_service --ms ethqlite/artifacts/sqlite3.wasm:ethqlite/sqlite3_cfg.json ethqlite/artifacts/ethqlite.wasm:ethqlite/ethqlite_cfg.json --name EthQliteSecure -s H9BSbZwKmFs93462xbAyfEdGdMXb5LZuXL7GSA4uPK4V
client seed: H9BSbZwKmFs93462xbAyfEdGdMXb5LZuXL7GSA4uPK4V
client peerId: 12D3KooWKphxxaXofYzC2TsN79RHZVubjmutKVdPUxVMHY3ZsVww
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
uploading blueprint EthQliteSecure to node 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 via client 12D3KooWKphxxaXofYzC2TsN79RHZVubjmutKVdPUxVMHY3ZsVww
service id: 470fcaba-6834-4ccf-ac0c-4f6494e9e77b
service created successfully
```

Updating the call parameters to reflect the new service id and client seed confirms our ownership over the service:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air -p air-scripts/am_i_owner.clj -d '{"service":"470fcaba-6834-4ccf-ac0c-4f6494e9e77b", "node":"12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17"}' -s H9BSbZwKmFs93462xbAyfEdGdMXb5LZuXL7GSA4uPK4V
client seed: H9BSbZwKmFs93462xbAyfEdGdMXb5LZuXL7GSA4uPK4V
client peerId: 12D3KooWKphxxaXofYzC2TsN79RHZVubjmutKVdPUxVMHY3ZsVww
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: 6d8c158b-d998-44ca-9d4c-255ce4b9cd21. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  1
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: '470fcaba-6834-4ccf-ac0c-4f6494e9e77b',
      function_name: 'am_i_owner',
      json_path: ''
    }
  ]
]
===================
```

Back to our task at hand: persisting reward block data to our sqlite as a service. Looking over the source code, we know that in order to accomplish persistence, we need to:

* init the database: `pub fn init_service() -> InitResult`
* provide reward data : `pub fn update_reward_blocks(data_string: String) -> UpdateResult`

Initializing Ethqlite for the most part is a one time event, so we'll do it right now and outside of our recurring block discovery and commit workflow with another small AIR script:

```text
; ethqlite_init.clj
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

which we deploy to the node with the `fldist` tool:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air -p air-scripts/ethqlite_init.clj -d '{"service":"470fcaba-6834-4ccf-ac0c-4f6494e9e77b", "node":"12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17"}' -s H9BSbZwKmFs93462xbAyfEdGdMXb5LZuXL7GSA4uPK4V
client seed: H9BSbZwKmFs93462xbAyfEdGdMXb5LZuXL7GSA4uPK4V
client peerId: 12D3KooWKphxxaXofYzC2TsN79RHZVubjmutKVdPUxVMHY3ZsVww
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: 2fb4a366-6f40-46c1-9329-d77c6d03dfad. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  {
    "err_msg": "",
    "success": 1
  }
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: '470fcaba-6834-4ccf-ac0c-4f6494e9e77b',
      function_name: 'init_service',
      json_path: ''
    }
  ]
]
===================
```


If you run the init script again, you will receive an error _"Service already initiated"_, so we can be reasonably confident our code is working and it looks like our Ethqlite service is up and running on the local node.

Due to the security concerns for our database, it is not advisable, or even possible, to use an already deployed Sqlite service from the Fluence Dashboard. Instead, we deploy our own instance with our own \(secret\) client seed. To determine which network nodes are available, run:

```bash
fldist --env testnet env
client seed: Cj4Wpy5y955o2N3T8Hs5myRoFGhBaBhytCdsYeyFLQPw
client peerId: 12D3KooWQg8cyj4z8Bv4rGq1PeXL1XKEQd6Z2CCFguy9D4NnLaKm
relay peerId: 12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb
/dns4/net01.fluence.dev/tcp/19001/wss/p2p/12D3KooWEXNUbCXooUwHrHBbrmjsrpHXoEphPwbjQXEGyzbqKnE9
/dns4/net01.fluence.dev/tcp/19990/wss/p2p/12D3KooWMhVpgfQxBLkQkJed8VFNvgN4iE6MD7xCybb1ZYWW2Gtz
/dns4/net02.fluence.dev/tcp/19001/wss/p2p/12D3KooWHk9BjDQBUqnavciRPhAYFvqKBe4ZiPPvde7vDaqgn5er
/dns4/net03.fluence.dev/tcp/19001/wss/p2p/12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb
/dns4/net04.fluence.dev/tcp/19001/wss/p2p/12D3KooWJbJFaZ3k5sNd8DjQgg3aERoKtBAnirEvPV8yp76kEXHB
/dns4/net05.fluence.dev/tcp/19001/wss/p2p/12D3KooWCKCeqLPSgMnDjyFsJuWqREDtKNHx1JEBiwaMXhCLNTRb
/dns4/net06.fluence.dev/tcp/19001/wss/p2p/12D3KooWKnRcsTpYx9axkJ6d69LPfpPXrkVLe96skuPTAo76LLVH
/dns4/net07.fluence.dev/tcp/19001/wss/p2p/12D3KooWBSdm6TkqnEFrgBuSkpVE3dR1kr6952DsWQRNwJZjFZBv
/dns4/net08.fluence.dev/tcp/19001/wss/p2p/12D3KooWGzNvhSDsgFoHwpWHAyPf1kcTYCGeRBPfznL8J6qdyu2H
/dns4/net09.fluence.dev/tcp/19001/wss/p2p/12D3KooWF7gjXhQ4LaKj6j7ntxsPpGk34psdQicN2KNfBi9bFKXg
/dns4/net10.fluence.dev/tcp/19001/wss/p2p/12D3KooWB9P1xmV3c7ZPpBemovbwCiRRTKd3Kq2jsVPQN4ZukDfy
```

which lists the available testnet peers. Pick one, update the node-id parameter and drop the node-addr parameter in your deployment command-line, upload the new ethqlite service and initiate it. Congrat's, you are now the proud maker of a Fluence testnet Ehqlite service!

Now it is time to get block data into the database.

