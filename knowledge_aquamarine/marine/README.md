# Marine

[Marine](https://github.com/fluencelabs/marine) is a general purpose WebAssembly runtime favoring Wasm modules based on the [ECS](https://en.wikipedia.org/wiki/Entity_component_system) pattern or plugin architecture and uses Wasm [Interface Types](https://github.com/WebAssembly/interface-types/blob/master/proposals/interface-types/Explainer.mdhttps://github.com/WebAssembly/interface-types/blob/master/proposals/interface-types/Explainer.md)  \( IT\) to implement a [shared-nothing](https://en.wikipedia.org/wiki/Shared-nothing_architecture) linking scheme. Fluence [nodes](https://github.com/fluencelabs/fluence) use Marine to host the Aqua VM and execute hosted Wasm services.

The  [Marine Rust SDK](https://github.com/fluencelabs/marine-rs-sdk) allows to hide the IT implementation details behind a handy procedural macro `[marine]` and provides the scaffolding for unit tests.









