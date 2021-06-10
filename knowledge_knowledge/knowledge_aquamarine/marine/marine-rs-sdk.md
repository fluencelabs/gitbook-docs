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

Below an example of an exposed function with a complex type signature and return value:

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

```text
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







