# cUrl as a Service


## Overview

[Curl](https://curl.se/) is a widely available and used command-line tool to receive or send data using URL syntax. Chances are, you probably just used it when you set up your Fluence development environment. For Fluence services to be able to interact with the world, cUrl is one option to facilitate https calls. Since Fluence modules are Wasm IT modules, cUrl cannot not be a service intrinsic. Instead, the curl command-line tool needs to be made available and accessible at the node level. And for Fluence services to be able to interact with Curl, we need to code a cUrl adapter taking care of the mounted \(cUrl\) binary.

## Adapter Construction

The work for the cUrl adapter has been fundamentally done and is exposed by the Fluence Rust SDK. As a developer, the task remaining is to instantiate the adapter in the context of the module and services scope. The following code [snippet](https://github.com/fluencelabs/fce/tree/master/examples/url-downloader/curl_adapter) illustrates the implementation requirement.

```rust
use fluence::fce;

use fluence::WasmLoggerBuilder;
use fluence::MountedBinaryResult;

pub fn main() {
    WasmLoggerBuilder::new().build().unwrap();
}

#[fce]
pub fn download(url: String) -> String {
    log::info!("get called with url {}", url);

    let result = unsafe { curl(vec![url]) };
    String::from_utf8(result.stdout).unwrap()
}

#[fce]
#[link(wasm_import_module = "host")]
extern "C" {
    fn curl(cmd: Vec<String>) -> MountedBinaryResult;
}
```

with the following dependencies necessary in the Cargo.toml:

```rust
fluence = { version = "=0.4.2", features = ["logger"] }
log = "0.4.8"
```

We are basically linking the [external](https://doc.rust-lang.org/std/keyword.extern.html) cUrl binary and are exposing access to it as a FCE interface called download.

### Code References

* [Mounted binaries](https://github.com/fluencelabs/fce/blob/c559f3f2266b924398c203a45863ebf2fb9252ec/fluence-faas/src/host_imports/mounted_binaries.rs)
* [cUrl](https://github.com/curl/curl)


### Service Construction

In order to create a valid Fluence service, a service configuration is required.

```text
modules_dir = "target/wasm32-wasi/release"

[[module]]
    name = "curl_adapter"
    logger_enabled = true

    [mounted.mounted_binaries]
    curl = "/usr/bin/curl"
```

We are specifying the location of the Wsasm file, the import name of the Wasm file, some logging housekeeping, and the mounted binary reference with the command-line call information.

### Service Creation

```bash
cargo new curl-service
cd curl-service
# copy the above rust code into src/main
# copy the specified dependencies into Cargo.toml
# copy the above service configuration into Config.toml

fce build --release
```

You should have the Fluence module curl\_adapter.wasm in `target/wasm32-wasi/release` we can stest our service with `fce-repl`.

### Service Test

Running the REPL, we use the `interface` command to list all available interfaces and the `call` command to run a method. Fr our purposes, we furnish the [https://duckduckgo.com/?q=Fluence+Labs](https://duckduckgo.com/?q=Fluence+Labs) url to give the the curl adapter a workout.

```bash
fce-repl Config.toml
Welcome to the FCE REPL (version 0.5.2)
app service was created with service id = 8ad81c3a-8c5c-4730-80d1-c54cd177725d
elapsed time 40.312376ms

1> interface
Loaded modules interface:

curl_adapter:
  fn download(url: String) -> String

2> call curl_adapter download ["https://duckduckgo.com/?q=Fluence+Labs"]
result: String("<!DOCTYPE html><html lang=\"en-US\" class=\"no-js has-zcm  no-theme\"><head><meta name=\"description\" content=\"DuckDuckGo. Privacy, Simplified.\"><meta http-equiv=\"content-type\" content=\"text/html; charset=utf-8\"><title>Fluence Labs at DuckDuckGo</title><link rel=\"stylesheet\" href=\"/s1963.css\" type=\"text/css\"><link rel=\"stylesheet\" href=\"/r1963.css\" type=\"text/css\"><meta name=\"robots\" content=\"noindex,nofollow\"><meta name=\"referrer\" content=\"origin\"><meta name=\"apple-mobile-web-app-title\" content=\"Fluence Labs\"><link rel=\"preconnect\" href=\"https://links.duckduckgo.com\"><link rel=\"preload\" href=\"/font/ProximaNova-Reg-webfont.woff2\" as=\"font\" type=\"font/woff2\" crossorigin=\"anonymous\" /><link rel=\"preload\" href=\"/font/ProximaNova-Sbold-webfont.woff2\" as=\"font\" type=\"font/woff2\" crossorigin=\"anonymous\" /><link rel=\"shortcut icon\" href=\"/favicon.ico\" type=\"image/x-icon\" /><link id=\"icon60\" rel=\"apple-touch-icon\" href=\"/assets/icons/meta/DDG-iOS-icon_60x60.png?v=2\"/><link id=\"icon76\" rel=\"apple-touch-icon\" sizes=\"76x76\" href=\"/assets/icons/meta/DDG-iOS-icon_76x76.png?v=2\"/><link id=\"icon120\" rel=\"apple-touch-icon\" sizes=\"120x120\" href=\"/assets/icons/meta/DDG-iOS-icon_120x120.png?v=2\"/><link id=\"icon152\" rel=\"apple-touch-icon\"s
<snip>
ript\">DDG.index = DDG.index || {}; DDG.index.signalSummary = \"\";</script>")
 elapsed time: 334.545388ms

3>
```

