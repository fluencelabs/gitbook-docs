# Marine-RS-SDK

The marine-rs-sdk empowers developers to write services suitable for peer hosting in peer-to-peer networks using the Marine Virtual Machine by enabling the wasm32-wasi compile target for Marine. For an introduction to writing services with the marine-rs-sdk, see the [Developing Modules And Services](../../../development_development/) section. 

### API

The procedural macros `[marine]` and `[marine_test]` are the two primary features provided by the SDK. The `[marine]` macro can be applied to a function, external block or structure. The `[marine_test]` macro, on the other hand, allows the use of the familiar `cargo test` to execute \(unit\) tests over the actual Wasm module generated from the service code.

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



See the example below of a wrapped `extern` block:

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
* it is possible to have inner records in the macro-wrapped structure

See the example below for a wrapped `struc`t:

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



{% hint style="info" %}


> #### Structure passing requirements
>
> * wrap a structure with the `[marine]` macro
> * all structure fields must be of the `ftype`
> * the structure must be pointed to without preceding package import in a function signature, i.e`StructureName` but not `package_name::module_name::StructureName`
{% endhint %}



#### Call Parameters

There is a special API function `fluence::get_call_parameters()` that returns an instance of the `CallParameters` structure defined as follows:

```rust
pub struct CallParameters {
    pub call_id: String,
    pub user_name: String,
    pub application_id: String,
}
```

Where

* **call\_id**  is the id, or nonce, of the current call to the service. This number is unique across the calls.
* **user\_name** is the  user name of the caller
* **application\_id**  is the  id of the application that this service belongs to



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

since cargo does not have access to the magic in place in the marine rs sdk to handle unsafe.

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

Since we are compiling to a wasm32-wasi target with `ftype` constrains, the basic `cargo test` is not all that useful or even usable for our purposes. To alleviate that limitation, Fluence has introduced the [`[marine-test]` macro ](https://github.com/fluencelabs/marine-rs-sdk/tree/master/crates/marine-test-macro)that does a lot of the heavy lifting to allow developers to use `cargo test` as intended. That is,  `[marine-test]` macro generates the necessary code to call Marine, one instance per test function, based on the Wasm module and associated configuration file so that the actual test function is run against the Wasm module not the native code.

Let's have a look at an implementation example:

```rust
use fluence::marine;
use fluence::module_manifest;

module_manifest!();

pub fn main() {}

#[marine]
pub fn greeting(name: String) -> String {    # 1  
    format!("Hi, {}", name)
}

#[cfg(test)]
mod tests {
    use fluence_test::marine_test;   # 2

    #[marine_test(config_path = "../Config.toml", modules_dir = "../artifacts")] # 3
    fn empty_string() {
        let actual = greeting.greeting(String::new());  # 4 
        assert_eq!(actual, "Hi, ");
    }

    #[marine_test(config_path = "../Config.toml", modules_dir = "../artifacts")]
    fn non_empty_string() {
        let actual = greeting.greeting("name".to_string());
        assert_eq!(actual, "Hi, name");
    }
}
```

1. We wrap a basic _greeting_ function with the `[marine`\] macro which results in the greeting.wasm module
2. We wrap our tests as usual with `[cfg(test)]` and import the fluence_test crate._ Do **not** import _super_ or the _local crate_. 
3. Instead, we apply the `[marine_test]` to each of the test functions by providing the path to the config file, e.g., Config.toml, and the directory containing the Wasm module we obtained after compiling our project with `marine build`. It is imperative that project compilation proceeds the test runner otherwise there won't be the required Wasm file.
4. The target of our tests is the `pub fn greeting` function. Since we are calling the function from the Wasm module we must prefix the function name with the module namespace -- `greeting` in this example case.

Now that we have our Wasm module and tests in place, we can proceed with `cargo test --release.` Note that using the `release`vastly improves the import speed of the necessary Wasm modules.  

### Features

The SDK has two useful features: `logger` and `debug`.

#### Logger

Using logging is a simple way to assist in the debugging without deploying the module\(s\)  to a peer-to-peer network node. The `logger` feature allows you to use a special logger that is based at the top of the [log](https://crates.io/crates/log) crate.

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

#[fce]
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

































