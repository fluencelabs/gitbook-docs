# Changelog

Fluence JS versioning scheme is the following: `0.BREAKING.ENHANCING`

- `0` shows that Fluence JS does not meet its vision yet, so API can change quickly
- `BREAKING` part is incremented for each breaking API change
- `ENHANCING` part is incremented for every fix and update which is compatible on API level

## [0.14.3](https://github.com/fluencelabs/fluence-js/releases/tag/v0.14.3) – November 10, 2021

FluencePeer:

Extend error handling. Now aqua function calls fail early with the user-friendly error message \([\#91](https://github.com/fluencelabs/fluence-js/pull/98)\)

Compiler support:

Define and export FnConfig interface \([\#91](https://github.com/fluencelabs/fluence-js/pull/97)\) fix issue with incorrect ttl value in config \([\#1001](https://github.com/fluencelabs/fluence-js/pull/100)\)

## [0.14.2](https://github.com/fluencelabs/fluence-js/releases/tag/v0.14.2) – October 21, 2021

FluencePeer: add option to specify default TTL for all new particles \([\#91](https://github.com/fluencelabs/fluence-js/pull/91)\)

## [0.14.1](https://github.com/fluencelabs/fluence-js/releases/tag/v0.14.1) – October 20, 2021

Compiler support: fix issue with incorrect check for missing fields in service registration \([\#90](https://github.com/fluencelabs/fluence-js/pull/90)\)

## [0.14.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.14.0) – October 20, 2021

Compiler support: added support for asynchronous code in service definitions and callback parameters of functions. \([\#83](https://github.com/fluencelabs/fluence-js/pull/83)\)

## [0.12.1](https://github.com/fluencelabs/fluence-js/releases/tag/v0.12.1) – September 14, 2021

- `KeyPair`: add fromBytes, toEd25519PrivateKey \([\#78](https://github.com/fluencelabs/fluence-js/pull/78)\)

## [0.12.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.13.0) – September 10, 2021

- The API to work with the default Fluence Peer has been put under the facade `Fluence`. Method `init` was renamed to `start` and `uninit` renamed to `stop`. `connectionStatus` migrated to `getStatus`.

To migrate from 0.11.0 to 0.12.0

1. `import { Fluence } from "@fluencelabs/fluence"`; instead of `FluencePeer`
2. replace `Fluence.default` with just `Fluence`
3. replace `init` with `start` and `uninit` with `stop`
4. replace `connectionInfo()` with `getStatus()`

\([\#72](https://github.com/fluencelabs/fluence-js/pull/72)\)

## [0.11.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.11.0) – September 08, 2021

- Update JS SDK api to the new version \([\#61](https://github.com/fluencelabs/fluence-js/pull/61)\)
