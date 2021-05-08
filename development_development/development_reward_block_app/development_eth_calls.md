# Ethereum Request Service

The source code for this section can be found [here](https://github.com/fluencelabs/examples/tree/main/multi-service) and is pretty straight forward with the two Etherscan api endpoints wrapped as public, FCE marked functions:

```rust
use crate::curl_request;
use fluence::fce;
use fluence::MountedBinaryResult;


fn result_to_string(result:MountedBinaryResult) -> String {
    if result.is_success() {
        return String::from_utf8(result.stdout).expect("Found invalid UTF-8");
    }
    String::from_utf8(result.stderr).expect("Found invalid UTF-8")
}

#[fce]
pub fn get_latest_block(api_key: String) -> String {
    let url = f!("https://api.etherscan.io/api?module=proxy&action=eth_blockNumber&apikey={api_key}");
    let header = "-d \"\"";

    let curl_cmd:Vec<String> = vec![header.into(), url.into()];
    let response = unsafe { curl_request(curl_cmd) };
    let res = result_to_string(response);
    let obj = serde_json::from_str::<serde_json::Value>(&res).unwrap();
    serde_json::from_value(obj["result"].clone()).unwrap()
}

#[fce]
pub fn get_block(api_key: String, block_number: u32) -> String {
    let url = f!("https://api.etherscan.io/api?module=block&action=getblockreward&blockno={block_number}&apikey={api_key}");
    let header = "-d \"\"";

    let curl_cmd:Vec<String> = vec![header.into(), url];
    let response = unsafe { curl_request(curl_cmd) };
    result_to_string(response)
}
```

Of course, both functions need to be able to make https calls, which is accomplished by calling \(unsafe\) `curl_request`:

```rust
// main.rs
#[macro_use]
extern crate fstrings;

use fluence::{fce, WasmLoggerBuilder};
use fluence::MountedBinaryResult as Result;

mod eth_block_getters;

fn main() {
    WasmLoggerBuilder::new().build().ok();
}

#[fce]
#[link(wasm_import_module = "curl_adapter")]
extern "C" {
    pub fn curl_request(curl_cmd: Vec<String>) -> Result;
}
```

Since we are dealing with Wasm modules, we don't have access to sockets at the module level but may be permissioned to call cUrl at the node level. In order to do that, we need to provide an adapter module. The code from the [cUrl adapter](https://github.com/fluencelabs/examples/tree/main/multi-service/curl_adapter) project illustrates how we're mounting the binary and expose the fce-marked interface for consumption, like above.

```bash
// main.rs
#![allow(improper_ctypes)]

use fluence::fce;
use fluence::MountedBinaryResult as Result;

fn main() {}

#[fce]
pub fn curl_request(curl_cmd: Vec<String>) -> Result {
    let response = unsafe { curl(curl_cmd.clone()) };
    log::info!("curl response for {:?} : {:?}", curl_cmd, response);
    response
}

// mounted_binaries are available to import like this:
#[fce]
#[link(wasm_import_module = "host")]
extern "C" {
    pub fn curl(cmd: Vec<String>) -> Result;
}
```

From both modules, we can now create a service configuration which specifies the name for each module and the permission specification for the mounted binaries:

```text
// Block-Getter-Config.toml
modules_dir = "artifacts/"

[[module]]
    name = "curl_adapter"

    [module.mounted_binaries]
    curl = "/usr/bin/curl"


[[module]]
    name = "block_getter"
```


If you haven't done so already, run `./scripts/build.sh` to compile the projects. Once we have _wasm_ files and the service configuration, we can check out our accomplishments with the REPL:

```bash
fce-repl Block-Getter-Config.toml
```

which gets us in the REPL to call the _interface_ command:

```bash
Welcome to the FCE REPL (version 0.5.2)
app service was created with service id = 15b9c3ee-ffbc-4464-bb7f-675a41acf81a
elapsed time 111.573048ms

1> interface
Loaded modules interface:
Result {
  ret_code: S32
  error: String
  stdout: Array<U8>
  stderr: Array<U8>
}

curl_adapter:
  fn curl_request(curl_cmd: Array<String>) -> Result

block_getter:
  fn get_block(api_key: String, block_number: U32) -> String
  fn get_latest_block(api_key: String) -> String

2>
```

Checking the available interfaces, shows the **public** interfaces to our respective Wasm modules, which are ready for calling:

```bash
> call curl_adapter curl_request [["-sS", "https://google.com"]]
result: Object({"error": String(""), "ret_code": Number(0), "stderr": Array([]), "stdout": Array([Number(60), Number(72), Number(84), Number(77), Number(76), Number(62), Number(60), Number(72), Number(69), Number(65), N
<snip>
, Number(72), Number(84), Number(77), Number(76), Number(62), Number(13), Number(10)])})
 elapsed time: 328.965523ms
```

As implemented, the raw cUrl call returns a [MountedBinaryResult](https://github.com/fluencelabs/rust-sdk/blob/c2fec5939fc17dcc227a78c7c8030549a6ff193f/crates/main/src/mounted_binary.rs) and we can see the corresponding _struct_ at the top of our `fce-repl` interfaces output. Looking through the return object, we see the standard pipe approach in place and find our query result in the stdout pipe. Of course, we are mostly interested in using cUrl from other modules as part of our service, such as getting the most recently produced block and its corresponding data:

```bash
3> call block_getter get_latest_block ["MC5H2NK6ZIPMR32U7D4W35AWNNVCQX1ENH"]
result: String("0xb7eeb3")
 elapsed time: 559.991486ms
```

and with some cognitive gymnastics we convert 0xb7eeb3 to 12054195:

```bash
4> call block_getter get_block ["MC5H2NK6ZIPMR32U7D4W35AWNNVCQX1ENH", 12054195]
result: String("{\"status\":\"1\",\"message\":\"OK\",\"result\":{\"blockNumber\":\"12054195\",\"timeStamp\":\"1615957734\",\"blockMiner\":\"0x99c85bb64564d9ef9a99621301f22c9993cb89e3\",\"blockReward\":\"2000000000000000000\",\"uncles\":[],\"uncleInclusionReward\":\"0\"}}")
 elapsed time: 578.485579ms
```

All good but please note that your latest block data is going to be significantly different from what's use here. Regardless, manual conversions are really not all that productive and that's why we implemented a [hex\_converter](https://github.com/fluencelabs/examples/tree/main/multi-service/hex_converter) module. Let's update our service config to:

```bash
// Block-Getter-With-Converter-Config.toml
modules_dir = "artifacts/"

[[module]]
    name = "curl_adapter"

    [module.mounted_binaries]
    curl = "/usr/bin/curl"


[[module]]
    name = "block_getter"

[[module]]
    name = "hex_converter"
```

and running `fce-repl` with _Block-Getter-With-Converter-Config.toml_ lists the interface for the _hex\_converter_ module. So far, so good. Using the previously generated hex string, yields the expected conversion result:

```bash
Welcome to the FCE REPL (version 0.5.2)
app service was created with service id = 09bfcff0-67dd-44c2-a677-de5a7a0c6383
elapsed time 176.472631ms

1> interface
Loaded modules interface:
Result {
  ret_code: S32
  error: String
  stdout: Array<U8>
  stderr: Array<U8>
}

hex_converter:
  fn hex_to_int(data: String) -> U64

block_getter:
  fn get_latest_block(api_key: String) -> String
  fn get_block(api_key: String, block_number: U32) -> String

curl_adapter:
  fn curl_request(curl_cmd: Array<String>) -> Result

2> call hex_converter hex_to_int ["0xb7eeb3"]
result: Number(12054195)
 elapsed time: 120.34µs

3>
```

Before we review the SQLite code, let's deploy our two services to the local node with the `fldist` tool. Make sure you got the node id and address for **your** local Fluence node:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 new_service --ms artifacts/curl_adapter.wasm:config/curl_cfg.json artifacts/block_getter.wasm:config/block_getter_cfg.json --name EthGetters
client seed: 4mp3sXX5FR9heeuqFtfRkq5GRqNJFQ8TvGCZ94PoSvQr
client peerId: 12D3KooWBdvur9HwahxMaGN2yrYDiofVD4GDBHivLtJwxwBuyzcr
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
uploading blueprint EthGetters to node 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 via client 12D3KooWBdvur9HwahxMaGN2yrYDiofVD4GDBHivLtJwxwBuyzcr
service id: ca0eceb3-871f-440e-aff1-0a186321437d
service created successfully
```

and for the hex conversion service:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 new_service --ms artifacts/hex_converter.wasm:config/hex_converter_cfg.json  --name HexConverter
client seed: BGvUGBvYifJf8oHS6rA7UmBc7Cs8EeaJxie8eFyP7YmY
client peerId: 12D3KooWJLXYiXwmmWPEv7kdQ8nYb646L96XyyTgkrrMAXen3FQy
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
uploading blueprint HexConverter to node 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 via client 12D3KooWJLXYiXwmmWPEv7kdQ8nYb646L96XyyTgkrrMAXen3FQy
service id: 36043704-4d40-4c74-a1bd-3abbde28305d
service created successfully
```

Our first service, _EthGetters_, is comprised of two modules and the the second service, _HexConverter_, of one module. With those two services available, we have everything we need to get the block reward information for the most recently block. In order to get us there, we write a small AIR script to coordinate the services into an app:

```text
; latest_block_reward.clj
(xor
    (seq
        (seq
                (seq
                    (seq
                        (call relay ("op" "identity") [])
                        (call node_1 (service_1 "get_latest_block") [api_key] hex_result)
                    )
                    (seq
                        (call relay ("op" "identity") [])
                        (call %init_peer_id% (returnService "run") [hex_result])
                    )
                )
                (seq
                    (seq
                        (call relay ("op" "identity") [])
                        (call node_2 (service_2 "hex_to_int") [hex_result] int_result)
                    )
                    (seq
                        (call relay ("op" "identity") [])
                        (call %init_peer_id% (returnService "run") [int_result])
                    )
                )
        )
        (seq

                (seq
                    (call relay ("op" "identity") [])
                    (call node_1 (service_1 "get_block") [api_key int_result] block_result)
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

As always, we use the `fldist` _run\_air_ command:

```bash
fldist --node-id 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17  --node-addr /ip4/127.0.0.1/tcp/9999/ws/p2p/12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17 run_air -p air-scripts/latest_reward_block.clj -d '{"service_1": "ca0eceb3-871f-440e-aff1-0a186321437d", "service_2": "36043704-4d40-4c74-a1bd-3abbde28305d", "node_1":"12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17", "node_2": "12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17", "api_key":"your-api-key"}'
client seed: 9xfs3P1r5QmBxCohcA4xmpE448Q64c14jmYn4XNJZEiz
client peerId: 12D3KooWNfA3Za3bvfHutWhvtZxC5NWdbaujoFZkR8bh2WVTZzw3
relay peerId: 12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17
Particle id: 930ea13f-1474-4501-862a-ca5fad22ee42. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  "0xb7fe13"
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: 'ca0eceb3-871f-440e-aff1-0a186321437d',
      function_name: 'get_latest_block',
      json_path: ''
    }
  ]
]
===================
===================
[
  12058131
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: '36043704-4d40-4c74-a1bd-3abbde28305d',
      function_name: 'hex_to_int',
      json_path: ''
    }
  ]
]
===================
===================
[
  "{\"status\":\"1\",\"message\":\"OK\",\"result\":{\"blockNumber\":\"12058131\",\"timeStamp\":\"1616010177\",\"blockMiner\":\"0x829bd824b016326a401d083b33d092293333a830\",\"blockReward\":\"6159144598411626490\",\"uncles\":[{\"miner\":\"0xe72f79190bc8f92067c6a62008656c6a9077f6aa\",\"unclePosition\":\"0\",\"blockreward\":\"500000000000000000\"}],\"uncleInclusionReward\":\"62500000000000000\"}}"
]
[
  [
    {
      peer_pk: '12D3KooWQQYXh78acqBNuL5p1J5tmH4XCKLCHM21tMb8pcxqGL17',
      service_id: 'ca0eceb3-871f-440e-aff1-0a186321437d',
      function_name: 'get_block',
      json_path: ''
    }
  ]
]
===================
```


Right on! Our two services coordinate into the intended application returning the reward data for the latest block. Before we move on, locate the corresponding services on the Fluence testnet via the [ dashboard](https://dash.fluence.dev/), update your command-line with the appropriate service and node ids and run the same AIR script. Congratulations, you just run an app coordinated by distributed services!


