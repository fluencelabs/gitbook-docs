# Marine Rust SDK

The [marine-rs-sdk](https://github.com/fluencelabs/marine-rs-sdk) empowers developers to write services suitable for peer hosting in peer-to-peer networks using the Marine Virtual Machine by enabling the wasm32-wasi compile target for Marine. 

### API

The procedural macros `[marine]` and `[marine_test]` are the two primary features provided by the SDK. The `[marine]` macro can be applied to a function, external block or structure. The `[marine_test]` macro, on the other hand, allows the use of the familiar `cargo test` to execute tests over the actual Wasm module generated from the service code.

#### Function Export

Applying the `[marine]` macro to a function results in its export, which means that it can be called from other modules or AIR scripts. For the function to be compatible with this macro, its arguments must be of the `ftype`, which is defined as follows:

`ftype` = `bool`, `u8`, `u16`, `u32`, `u64`, `i8`, `i16`, `i32`, `i64`, `f32`, `f64`, `String`  
`ftype` = `ftype` \| `Vec`&lt;`ftype`&gt;  
`ftype` = `ftype` \| `Record`&lt;`ftype`&gt;

In other words, the arguments must be one of the types listed below:

* one of the following Rust basic types: `bool`, `u8`, `u16`, `u32`, `u64`, `i8`, `i16`, `i32`, `i64`, `f32`, `f64`, `String`
* a vector of elements of the above types
* a vector composed of vectors of the above type, where recursion is acceptable, e.g. the type `Vec<Vec<Vec<u8>>>` is permissible
* a record, where all fields are of the basic Rust types
* a record, where all fields are of any above types or other records 

The return type of a function must follow the same rules, but currently only one return type is possible.

See the example below of an exposed function with a complex type signature and return value:

```rust
// export TestRecord as a public data structure bound by 
// the IT type constraints
#[marine]
pub struct TestRecord {
    pub field_0: i32,
    pub field_1: Vec<Vec<u8>>,
}

// export foo as a public function bound by the 
// IT type contraints 
#[marine] # 
pub fn foo(arg_1: Vec<Vec<Vec<Vec<TestRecord>>>>, arg_2: String) -> Vec<Vec<Vec<Vec<TestRecord>>>> { 
    unimplemented!() 
}
```



{% hint style="info" %}
Function Export Requirements

* wrap a target function with the `[marine]` macro
* function arguments must by of `ftype`
* the function return type also must be of `ftype` 
{% endhint %}

#### Function Import

The `[marine]` macro can also wrap an [`extern` block](https://doc.rust-lang.org/std/keyword.extern.html). In this case, all functions declared in it are considered imported functions. If there are imported functions in some module, say, module A, then:

* There should be another module, module B, that exports the same functions. The name of module B is indicated in the `link` macro \(see examples below\).
* Module B should be loaded to `Marine` by the moment the loading of module A starts. Module A cannot be loaded if at least one imported function is absent in `Marine`.

See the examples below for wrapped `extern` block usage:

{% tabs %}
{% tab title="Example 1" %}
```rust
#[marine]
pub struct TestRecord {
    pub field_0: i32,
    pub field_1: Vec<Vec<u8>>,
}

// wrap the extern block with the marine macro to expose the function
// as an import to the Marine VM
#[marine]
#[link(wasm_import_module = "some_module")]
extern "C" {
    pub fn foo(arg: Vec<Vec<Vec<Vec<TestRecord>>>>, arg_2: String) -> Vec<Vec<Vec<Vec<TestRecord>>>>;
}
```
{% endtab %}

{% tab title="Example 2" %}
```rust
[marine]
#[link(wasm_import_module = "some_module")]
extern "C" {
  pub fn foo(arg: Vec<Vec<Vec<Vec<u8>>>>) -> Vec<Vec<Vec<Vec<u8>>>>;
}
```
{% endtab %}
{% endtabs %}



{% hint style="info" %}


#### Function import requirements

* wrap an extern block with the function\(s\) to be imported with the `[marine]` macro
* all function\(s\) arguments must be of the `ftype` type
* the return type of the function\(s\) must be `ftype`
{% endhint %}

#### 

#### Structures

Finally, the `[marine]` macro can wrap a `struct` making possible to use it as a function argument or return type. Note that 

* only macro-wrapped structures can be used as function arguments and return types
* all fields of the wrapped structure must be public and of the `ftype`.
* it is possible to have inner records in the macro-wrapped structure and to import wrapped structs from other crates

See the example below for  wrapping `struct`:

{% tabs %}
{% tab title="Example 1" %}
```rust
#[marine]
pub struct TestRecord0 {
    pub field_0: i32,
}

#[marine]
pub struct TestRecord1 {
    pub field_0: i32,
    pub field_1: String,
    pub field_2: Vec<u8>,
    pub test_record_0: TestRecord0,
}

#[marine]
pub struct TestRecord2 {
    pub test_record_0: TestRecord0,
    pub test_record_1: TestRecord1,
}

#[marine]
fn foo(mut test_record: TestRecord2) -> TestRecord2 { unimplemented!(); }
```
{% endtab %}

{% tab title="Example 2" %}
```rust
#[fce]
pub struct TestRecord0 {
    pub field_0: i32,
}

#[fce]
pub struct TestRecord1 {
    pub field_0: i32,
    pub field_1: String,
    pub field_2: Vec<u8>,
    pub test_record_0: TestRecord0,
}

#[fce]
pub struct TestRecord2 {
    pub test_record_0: TestRecord0,
    pub test_record_1: TestRecord1,
}

#[fce]
#[link(wasm_import_module = "some_module")]
extern "C" {
  fn foo(mut test_record: TestRecord2) -> TestRecord2;
}
```
{% endtab %}

{% tab title="Example 3" %}
```rust
mod data_crate {
    use fluence::marine;
    #[marine]
    pub struct Data {
        pub name: String,
        pub data: f64,
    }
}

use data_crate::Data;
use fluence::marine;

fn main() {}

#[marine]
fn some_function() -> Data {
    Data {
        name: "example".into(),
        data: 1.0,
    }
}

```
{% endtab %}
{% endtabs %}



{% hint style="info" %}


> #### Structure passing requirements
>
> * wrap a structure with the `[marine]` macro
> * all structure fields must be of the `ftype`
> * the structure must be pointed to without preceding package import in a function signature, i.e`StructureName` but not `package_name::module_name::StructureName`
> * wrapped structs can be imported from crates
{% endhint %}

#### 

#### Call Parameters

There is a special API function `fluence::get_call_parameters()` that returns an instance of the [`CallParameters`](https://github.com/fluencelabs/marine-rs-sdk/blob/master/src/call_parameters.rs#L35) structure defined as follows:

```rust
pub struct CallParameters {
    /// Peer id of the AIR script initiator.
    pub init_peer_id: String,

    /// Id of the current service.
    pub service_id: String,

    /// Id of the service creator.
    pub service_creator_peer_id: String,

    /// Id of the host which run this service.
    pub host_id: String,

    /// Id of the particle which execution resulted a call this service.
    pub particle_id: String,

    /// Security tetraplets which described origin of the arguments.
    pub tetraplets: Vec<Vec<SecurityTetraplet>>,
}
```

CallParameters are especially useful in constructing authentication services:

```text
// auth.rs
use fluence::{marine, CallParameters};
use::marine;

pub fn is_owner() -> bool {
    let meta = marine::get_call_parameters();
    let caller = meta.init_peer_id;
    let owner = meta.service_creator_peer_id;

    caller == owner
}

#[marine]
pub fn am_i_owner() -> bool {
    is_owner()
}
```

#### 

#### MountedBinaryResult

Due to the inherent limitations of Wasm modules, such as a lack of sockets, it may be necessary for a module to interact with its host to bridge such gaps, e.g. use a https transport provider like _curl_. In order for a Wasm module to use a host's _curl_  capabilities, we need to provide access to the binary, which at the code level is achieved through the Rust `extern` block:

```rust
// Importing a linked binary, curl, to a Wasm module
#![allow(improper_ctypes)]

use fluence::marine;
use fluence::module_manifest;
use fluence::MountedBinaryResult;

module_manifest!();

pub fn main() {}

#[marine]
pub fn curl_request(curl_cmd: Vec<String>) -> MountedBinaryResult {
    let response = curl(curl_cmd);
    response
}

#[marine]
#[link(wasm_import_module = "host")]
extern "C" {
    fn curl(cmd: Vec<String>) -> MountedBinaryResult;
}
```

The above code creates a "curl adapter", i.e., a Wasm module that allows other Wasm modules to use the the `curl_request` function, which calls the imported _curl_  binary in this case, to make http calls. Please note that we are wrapping the `extern` block with the `[marine]`macro and introduce a Marine-native data structure [`MountedBinaryResult`](https://github.com/fluencelabs/marine/blob/master/examples/url-downloader/curl_adapter/src/main.rs) as the linked-function return value.

Please not that if you want to use `curl_request` with testing, see below, the curl call needs to be marked unsafe, e.g.:

```rust
    let response = unsafe { curl(curl_cmd) };
```

since cargo does not access to the marine macro to handle unsafe.

MountedBinaryResult itself is a Marine-compatible struct containing a binary's return process code, error string and  stdout and stderr as byte arrays:

```rust
#[marine]
#[derive(Clone, PartialEq, Default, Eq, Debug, Serialize, Deserialize)]
pub struct MountedBinaryResult {
    /// Return process exit code or host execution error code, where SUCCESS_CODE means success.
    pub ret_code: i32,

    /// Contains the string representation of an error, if ret_code != SUCCESS_CODE.
    pub error: String,

    /// The data that the process wrote to stdout.
    pub stdout: Vec<u8>,

    /// The data that the process wrote to stderr.
    pub stderr: Vec<u8>,
}

```

MountedBinaryResult then can be used on a variety of match or conditional tests.

#### Testing

Since we are compiling to a wasm32-wasi target with `ftype` constrains, the basic `cargo test` is not all that useful or even usable for our purposes. To alleviate that limitation, Fluence has introduced the [`[marine-test]` macro ](https://github.com/fluencelabs/marine-rs-sdk-test/tree/master/crates/marine-test-macro)that does a lot of the heavy lifting to allow developers to use `cargo test` as intended. That is,  `[marine-test]` macro generates the necessary code to call Marine, one instance per test function, based on the Wasm module and associated configuration file so that the actual test function is run against the Wasm module not the native code.

To use the `[marine-test]` macro please add `marine-rs-sdk-test` crate to the `[dev-dependencies]` section of `Config.toml`:

```rust
[dev-dependencies]
marine-rs-sdk-test = "0.2.0"
```

 Let's have a look at an implementation example:

```rust
use marine_rs_sdk::marine;
use marine_rs_sdk::module_manifest;

module_manifest!();

pub fn main() {}

#[marine]
pub fn greeting(name: String) -> String {    // 1  
    format!("Hi, {}", name)
}

#[cfg(test)]
mod tests {
    use marine_rs_sdk_test::marine_test;   // 2

    #[marine_test(config_path = "../Config.toml", modules_dir = "../artifacts")] // 3
    fn empty_string(greeting: marine_test_env::greeting::ModuleInterface) {
        let actual = greeting.greeting(String::new());  // 4 
        assert_eq!(actual, "Hi, ");
    }

    #[marine_test(config_path = "../Config.toml", modules_dir = "../artifacts")]
    fn non_empty_string(greeting: marine_test_env::greeting::ModuleInterface) {
        let actual = greeting.greeting("name".to_string());
        assert_eq!(actual, "Hi, name");
    }
}
```

1. We wrap a basic _greeting_ function with the `[marine]` macro which results in the greeting.wasm module
2. We wrap our tests as usual with `[cfg(test)]` and import the marine _test crate._ Do **not** import _super_ or the _local crate_. 
3. Instead, we apply the `[marine_test]` macro to each of the test functions by providing the path to the config file, e.g., Config.toml, and the directory containing the Wasm module we obtained after compiling our project with `marine build`. Moreover, we add the type of the test as an argument in the function signature. It is imperative that project build precedes the test runner otherwise the required Wasm file will be missing.
4. The target of our tests is the `pub fn greeting` function. Since we are calling the function from the Wasm module we must prefix the function name with the module namespace -- `greeting` in this example case as specified in the function argument.

Now that we have our Wasm module and tests in place, we can proceed with `cargo test --release.` Note that using the `release`flag vastly improves the import speed of the necessary Wasm modules.

The same macro also allows testing data flow between multiple services, so you do not need to deploy anything to the network and write an Aqua app just for basic testing. Let's look at an example:

{% tabs %}
{% tab title="test.rs" %}
```rust
fn main() {}

#[cfg(test)]
mod tests {
    use marine_rs_sdk_test::marine_test;
    #[marine_test( // 1
        producer(
            config_path = "../producer/Config.toml", 
            modules_dir = "../producer/artifacts"
        ),
        consumer(
            config_path = "../consumer/Config.toml",
            modules_dir = "../consumer/artifacts"
        )
    )]
    fn test() {
        let mut producer = marine_test_env::producer::ServiceInterface::new(); // 2
        let mut consumer = marine_test_env::consumer::ServiceInterface::new();
        let input = marine_test_env::producer::Input { // 3
            first_name: String::from("John"),
            last_name: String::from("Doe"),
        };
        let data = producer.produce(input); // 4
        let result = consumer.consume(data);
        assert_eq!(result, "John Doe")
    }
}

```
{% endtab %}

{% tab title="producer.rs" %}
```rust
use marine_rs_sdk::marine;
use marine_rs_sdk::module_manifest;

module_manifest!();

pub fn main() {}

#[marine]
pub struct Data {
    pub name: String,
}

#[marine]
pub struct Input {
    pub first_name: String,
    pub last_name: String,
}

#[marine]
pub fn produce(data: Input) -> Data {
    Data {
        name: format!("{} {}", data.first_name, data.last_name),
    }
}

```
{% endtab %}

{% tab title="consumer.rs" %}
```rust
use marine_rs_sdk::marine;
use marine_rs_sdk::module_manifest;

module_manifest!();

pub fn main() {}

#[marine]
pub struct Data {
    pub name: String,
}

#[marine]
pub fn consume(data: Data) -> String {
    data.name
}

```
{% endtab %}
{% endtabs %}

1. We wrap the `test` function with the `marine_test` macro by providing named service configurations with module locations. Based on its arguments the macro defines a `marine_test_env` module with an interface to the services.
2. We create new services. Each `ServiceInterface::new()` runs a new marine runtime with the service.
3. We prepare data to pass to a service using structure definition from `marine_test_env`. The macro finds all structures used in the service interface functions and defines them in the corresponding submodule of  `marine_test_env` . 
4. We call a service function through the `ServiceInterface` object.
5. It is possible to use the result of one service call as an argument for a different service call. The interface types with the same structure have the same rust type in `marine_test_env`. 

The full example is [here](https://github.com/fluencelabs/marine/tree/master/examples/multiservice_marine_test).

The `marine_test` macro also gives access to the interface of internal modules which may be useful for setting up a test environment. This feature is designed to be used in situations when it is simpler to set up a service for a test through internal functions than through the service interface. To illustrate this feature we have rewritten the previous example:

```rust
fn main() {}

#[cfg(test)]
mod tests {
    use marine_rs_sdk_test::marine_test;
    #[marine_test(
        producer(
            config_path = "../producer/Config.toml",
            modules_dir = "../producer/artifacts"
        ),
        consumer(
            config_path = "../consumer/Config.toml",
            modules_dir = "../consumer/artifacts"
        )
    )]
    fn test() {
        let mut producer = marine_test_env::producer::ServiceInterface::new();
        let mut consumer = marine_test_env::consumer::ServiceInterface::new();
        let input = marine_test_env::producer::modules::producer::Input { // 1
            first_name: String::from("John"),
            last_name: String::from("Doe"),
        };
        let data = producer.modules.producer.produce(input); // 2
        let consumer_data = marine_test_env::consumer::modules::consumer::Data { name: data.name } // 3;
        let result = consumer.modules.consumer.consume(consumer_data); 
        assert_eq!(result, "John Doe")
    }
}

```

1. We access the internal service interface to construct an interface structure. To do so, we use the following pattern: `marine_test_env::$service_name::modules::$module_name::$structure_name`.
2. We access the internal service interface and directly call a function from one of the modules of this service. To do so, we use the following pattern: `$service_object.modules.$module_name.$function_name` .
3. In the previous example, the same interface types had the same rust types. It is limited when using internal modules: the property is true only when structures are defined in internal modules of one service, or when structures are defined in service interfaces of different services. So, we need to construct the proper type to pass data to the internals of another module. 

### Features

The SDK has two useful features: `logger` and `debug`.

#### Logger

Using logging is a simple way to assist in debugging without deploying the module\(s\)  to a peer-to-peer network node. The `logger` feature allows you to use a special logger that is based at the top of the [log](https://crates.io/crates/log) crate.

To enable logging please specify the `logger` feature of the Fluence SDK in `Config.toml` and add the [log](https://docs.rs/log/0.4.11/log/) crate:

```rust
[dependencies]
log = "0.4.14"
fluence = { version = "0.6.9", features = ["logger"] }
```

The logger should be initialized before its usage. This can be done in the `main` function as shown in the example below.

```rust
use fluence::marine;
use fluence::WasmLogger;

pub fn main() {
    WasmLogger::new()
        // with_log_level can be skipped,
        // logger will be initialized with Info level in this case.
        .with_log_level(log::Level::Info)
        .build()
        .unwrap();
}

#[marine]
pub fn put(name: String, file_content: Vec<u8>) -> String {
    log::info!("put called with file name {}", file_name);
    unimplemented!()
}
```

In addition to the standard log creation features, the Fluence logger allows the so-called target map to be configured during the initialization step. This allows you to filter out logs by `logging_mask`, which can be set for each module in the service configuration. Let's consider an example:

```rust
const TARGET_MAP: [(&str, i64); 4] = [
    ("instruction", 1 << 1),
    ("data_cache", 1 << 2),
    ("next_peer_pks", 1 << 3),
    ("subtree_complete", 1 << 4),
];

pub fn main() {
  use std::collections::HashMap;
    use std::iter::FromIterator;
  
    let target_map = HashMap::from_iter(TARGET_MAP.iter().cloned());
    
  fluence::WasmLogger::new()
        .with_target_map(target_map)
        .build()
        .unwrap();
}

#[marine]
pub fn foo() {
    log::info!(target: "instruction", "this will print if (logging_mask & 1) != 0");
    log::info!(target: "data_cache", "this will print if (logging_mask & 2) != 0");
}
```

Here, an array called `TARGET_MAP` is defined and provided to a logger in the `main` function of a module. Each entry of this array contains a string \(a target\) and a number that represents the bit position in the 64-bit mask `logging_mask`. When you write a log message request `log::info!`, its target must coincide with one of the strings \(the targets\) defined in the `TARGET_MAP` array. The log will be printed if `logging_mask` for the module has the corresponding target bit set.

{% hint style="info" %}
REPL also uses the log crate to print logs from Wasm modules. Log messages will be printed if`RUST_LOG` environment variable is specified.
{% endhint %}

#### Debug

The application of the second feature is limited to obtaining some of the internal details of the IT execution. Normally, this feature should not be used by a backend developer. Here you can see example of such details for the greeting service compiled with the `debug` feature:

```bash
# running the greeting service compiled with debug feature
~ $ RUST_LOG="info" fce-repl Config.toml
Welcome to the Fluence FaaS REPL
app service's created with service id = e5cfa463-ff50-4996-98d8-4eced5ac5bb9
elapsed time 40.694769ms

1> call greeting greeting "user"
[greeting] sdk.allocate: 4
[greeting] sdk.set_result_ptr: 1114240
[greeting] sdk.set_result_size: 8
[greeting] sdk.get_result_ptr, returns 1114240
[greeting] sdk.get_result_size, returns 8
[greeting] sdk.get_result_ptr, returns 1114240
[greeting] sdk.get_result_size, returns 8
[greeting] sdk.deallocate: 0x110080 8

result: String("Hi, user")
 elapsed time: 222.675µs
```

The most important information these logs relates to the `allocate`/`deallocate` function calls. The `sdk.allocate: 4` line corresponds to passing the 4-byte `user` string to the Wasm module, with the memory allocated inside the module and the string is copied there. Whereas `sdk.deallocate: 0x110080 8` refers to passing the 8-byte resulting string `Hi, user` to the host side. Since all arguments and results are passed by value, `deallocate` is called to delete unnecessary memory inside the Wasm module.

#### Module Manifest

The `module_manifest!` macro embeds the Interface Type \(IT\), SDK and Rust project version as well as additional project and build information into Wasm module.  For the macro to be usable, it needs to be imported and initialized in the _main.rs_ file:

```rust
// main.rs
use fluence::marine;
use fluence::module_manifest;    // import manifest macro

module_manifest!();              // initialize macro

fn main() {}

#[marine]
fn some_function() {}
}
```

Using the Marine CLI, we can inspect a module's manifest with `marine info`:

```rust
mbp16~/localdev/struct-exp(main|…) % marine info -i artifacts/*.wasm
it version:  0.20.1
sdk version: 0.6.0
authors:     The Fluence Team
version:     0.1.0
description: foo-wasm, a Marine wasi module
repository:
build time:  2021-06-11 21:08:59.855352 +00:00 UTC
```





























