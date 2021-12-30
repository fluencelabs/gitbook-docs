# Changelog

Fluence JS versioning scheme is the following: `0.BREAKING.ENHANCING`

* `0` shows that Fluence JS does not meet its vision yet, so API can change quickly
* `BREAKING` part is incremented for each breaking API change
* `ENHANCING` part is incremented for every fix and update which is compatible on API level

## [0.18.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.18.0) – December 29, 2021

FluencePeer: Update AVM version to 0.20.0 ([#120](https://github.com/fluencelabs/fluence-js/pull/120))

## [0.17.1](https://github.com/fluencelabs/fluence-js/releases/tag/v0.17.1) – December 29, 2021

FluencePeer: Update AvmRunner to 0.1.2 (fix issue with incorrect baseUrl) ([#119](https://github.com/fluencelabs/fluence-js/pull/119))

## [0.17.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.17.0) – December 28, 2021

JS Peer does not embed AVM interpreter any more. Instead [AVM Runner](https://github.com/fluencelabs/avm-runner-background) is used to run AVM in background giving huge performance boost. This is a **breaking change**: all browser applications now not need to bundle `avm.wasm` file and the runner script. See [documentation](4\_run\_in\_browser-1.md) for more info.

([#111](https://github.com/fluencelabs/fluence-js/pull/120))

## [0.16.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.16.0) – December 22, 2021

FluencePeer: Update AVM version to 0.19.3 ([#115](https://github.com/fluencelabs/fluence-js/pull/115))

## [0.15.4](https://github.com/fluencelabs/fluence-js/releases/tag/v0.15.4) – December 13, 2021

FluencePeer: Update AVM version to 0.17.7 ([#113](https://github.com/fluencelabs/fluence-js/pull/113))

## [0.15.3](https://github.com/fluencelabs/fluence-js/releases/tag/v0.15.3) – December 10, 2021

**FluencePeer:**

* Add built-in service to sign data and verify signatures ([#110](https://github.com/fluencelabs/fluence-js/pull/110))
* Update AVM version to 0.17.6 ([#112](https://github.com/fluencelabs/fluence-js/pull/112))

## [0.15.2](https://github.com/fluencelabs/fluence-js/releases/tag/v0.15.2) – November 30, 2021

Add particleId to error message when an aqua function times out ([#106](https://github.com/fluencelabs/fluence-js/pull/106))

## [0.15.1](https://github.com/fluencelabs/fluence-js/releases/tag/v0.15.0) – November 28, 2021

**FluencePeer:**

* Fix timeout builtin error message ([#103](https://github.com/fluencelabs/fluence-js/pull/103))

**Compiler support:**

Issue fixes for `registerService` function

* Throwing error if registerService was called on a non-initialized peer.
* Fix issue with incorrect context being passed to class-based implementations of user services
* Fix typo in JSDoc

([#104](https://github.com/fluencelabs/fluence-js/pull/104))

## [0.15.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.15.0) – November 17, 2021

**FluencePeer:**

* Implement peer.timeout built-in function ([#101](https://github.com/fluencelabs/fluence-js/pull/101))
* Update AVM: add support for the restriction operator ([#102](https://github.com/fluencelabs/fluence-js/pull/102))

## [0.14.3](https://github.com/fluencelabs/fluence-js/releases/tag/v0.14.3) – November 10, 2021

**FluencePeer:**

* Extend error handling. Now aqua function calls fail early with the user-friendly error message ([#91](https://github.com/fluencelabs/fluence-js/pull/98))

**Compiler support:**

* Define and export FnConfig interface ([#97](https://github.com/fluencelabs/fluence-js/pull/97))
* Fix issue with incorrect TTL value in function calls config ([#100](https://github.com/fluencelabs/fluence-js/pull/100))

## [0.14.2](https://github.com/fluencelabs/fluence-js/releases/tag/v0.14.2) – October 21, 2021

FluencePeer: add option to specify default TTL for all new particles ([#91](https://github.com/fluencelabs/fluence-js/pull/91))

## [0.14.1](https://github.com/fluencelabs/fluence-js/releases/tag/v0.14.1) – October 20, 2021

Compiler support: fix issue with incorrect check for missing fields in service registration ([#90](https://github.com/fluencelabs/fluence-js/pull/90))

## [0.14.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.14.0) – October 20, 2021

Compiler support: added support for asynchronous code in service definitions and callback parameters of functions. ([#83](https://github.com/fluencelabs/fluence-js/pull/83))

## [0.12.1](https://github.com/fluencelabs/fluence-js/releases/tag/v0.12.1) – September 14, 2021

* `KeyPair`: add fromBytes, toEd25519PrivateKey ([#78](https://github.com/fluencelabs/fluence-js/pull/78))

## [0.12.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.13.0) – September 10, 2021

* The API to work with the default Fluence Peer has been put under the facade `Fluence`. Method `init` was renamed to `start` and `uninit` renamed to `stop`. `connectionStatus` migrated to `getStatus`.

To migrate from 0.11.0 to 0.12.0

1. `import { Fluence } from "@fluencelabs/fluence"`; instead of `FluencePeer`
2. replace `Fluence.default` with just `Fluence`
3. replace `init` with `start` and `uninit` with `stop`
4. replace `connectionInfo()` with `getStatus()`

([#72](https://github.com/fluencelabs/fluence-js/pull/72))

## [0.11.0](https://github.com/fluencelabs/fluence-js/releases/tag/v0.11.0) – September 08, 2021

* Update JS SDK api to the new version ([#61](https://github.com/fluencelabs/fluence-js/pull/61))
