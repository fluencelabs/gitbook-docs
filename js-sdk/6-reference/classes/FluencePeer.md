# Class: FluencePeer

This class implements the Fluence protocol for javascript-based environments.
It provides all the necessary features to communicate with Fluence network

## Table of contents

### Constructors

- [constructor](js-sdk/6_reference/classes/FluencePeer.md#constructor)

### Accessors

- [connectionInfo](js-sdk/6_reference/classes/FluencePeer.md#connectioninfo)
- [internals](js-sdk/6_reference/classes/FluencePeer.md#internals)
- [default](js-sdk/6_reference/classes/FluencePeer.md#default)

### Methods

- [init](js-sdk/6_reference/classes/FluencePeer.md#init)
- [uninit](js-sdk/6_reference/classes/FluencePeer.md#uninit)

## Constructors

### constructor

• **new FluencePeer**()

Creates a new Fluence Peer instance. Does not start the workflows.
In order to work with the Peer it has to be initialized with the `init` method

#### Defined in

[internal/FluencePeer.ts:111](https://github.com/fluencelabs/fluence-js/blob/c2e280d/src/internal/FluencePeer.ts#L111)

## Accessors

### connectionInfo

• `get` **connectionInfo**(): `ConnectionInfo`

Get the information about Fluence Peer connections

#### Returns

`ConnectionInfo`

#### Defined in

[internal/FluencePeer.ts:116](https://github.com/fluencelabs/fluence-js/blob/c2e280d/src/internal/FluencePeer.ts#L116)

___

### internals

• `get` **internals**(): `Object`

Does not intended to be used manually. Subject to change

#### Returns

`Object`

| Name | Type |
| :------ | :------ |
| `callServiceHandler` | `CallServiceHandler` |
| `initiateFlow` | (`request`: `RequestFlow`) => `Promise`<`void`\> |

#### Defined in

[internal/FluencePeer.ts:190](https://github.com/fluencelabs/fluence-js/blob/c2e280d/src/internal/FluencePeer.ts#L190)

___

### default

• `Static` `get` **default**(): [`FluencePeer`](js-sdk/6_reference/classes/FluencePeer.md)

Get the default Fluence peer instance. The default peer is used automatically in all the functions generated
by the Aqua compiler if not specified otherwise.

#### Returns

[`FluencePeer`](js-sdk/6_reference/classes/FluencePeer.md)

#### Defined in

[internal/FluencePeer.ts:181](https://github.com/fluencelabs/fluence-js/blob/c2e280d/src/internal/FluencePeer.ts#L181)

## Methods

### init

▸ **init**(`config?`): `Promise`<`void`\>

Initializes the peer: starts the Aqua VM, initializes the default call service handlers
and (optionally) connect to the Fluence network

#### Parameters

| Name | Type | Description |
| :------ | :------ | :------ |
| `config?` | `PeerConfig` | object specifying peer configuration |

#### Returns

`Promise`<`void`\>

#### Defined in

[internal/FluencePeer.ts:130](https://github.com/fluencelabs/fluence-js/blob/c2e280d/src/internal/FluencePeer.ts#L130)

___

### uninit

▸ **uninit**(): `Promise`<`void`\>

Uninitializes the peer: stops all the underltying workflows, stops the Aqua VM
and disconnects from the Fluence network

#### Returns

`Promise`<`void`\>

#### Defined in

[internal/FluencePeer.ts:172](https://github.com/fluencelabs/fluence-js/blob/c2e280d/src/internal/FluencePeer.ts#L172)
