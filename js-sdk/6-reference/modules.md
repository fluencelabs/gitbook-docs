# @fluencelabs/fluence

## Table of contents

### Classes

- [FluencePeer](js-sdk/6_reference/classes/FluencePeer.md)

### Type aliases

- [AvmLoglevel](js-sdk/6_reference/modules.md#avmloglevel)
- [PeerIdB58](js-sdk/6_reference/modules.md#peeridb58)

### Functions

- [peerIdFromEd25519SK](js-sdk/6_reference/modules.md#peeridfromed25519sk)
- [peerIdToEd25519SK](js-sdk/6_reference/modules.md#peeridtoed25519sk)
- [randomPeerId](js-sdk/6_reference/modules.md#randompeerid)
- [setLogLevel](js-sdk/6_reference/modules.md#setloglevel)

## Type aliases

### AvmLoglevel

Ƭ **AvmLoglevel**: `LogLevel`

Enum representing the log level used in Aqua VM.
Possible values: 'info', 'trace', 'debug', 'info', 'warn', 'error', 'off';

#### Defined in

[internal/FluencePeer.ts:35](https://github.com/fluencelabs/fluence-js/blob/284a59b/src/internal/FluencePeer.ts#L35)

___

### PeerIdB58

Ƭ **PeerIdB58**: `string`

Peer ID's id as a base58 string (multihash/CIDv0).

#### Defined in

[internal/commonTypes.ts:20](https://github.com/fluencelabs/fluence-js/blob/284a59b/src/internal/commonTypes.ts#L20)

## Functions

### peerIdFromEd25519SK

▸ `Const` **peerIdFromEd25519SK**(`sk`): `Promise`<`PeerId`\>

Generates a new peer id from base64 string contatining the 32 byte Ed25519S secret key

#### Parameters

| Name | Type |
| :------ | :------ |
| `sk` | `string` |

#### Returns

`Promise`<`PeerId`\>

- Promise with the created Peer Id

#### Defined in

[internal/peerIdUtils.ts:26](https://github.com/fluencelabs/fluence-js/blob/284a59b/src/internal/peerIdUtils.ts#L26)

___

### peerIdToEd25519SK

▸ `Const` **peerIdToEd25519SK**(`peerId`): `string`

Converts peer id into base64 string contatining the 32 byte Ed25519S secret key

#### Parameters

| Name | Type |
| :------ | :------ |
| `peerId` | `PeerId` |

#### Returns

`string`

- base64 of Ed25519S secret key

#### Defined in

[internal/peerIdUtils.ts:45](https://github.com/fluencelabs/fluence-js/blob/284a59b/src/internal/peerIdUtils.ts#L45)

___

### randomPeerId

▸ `Const` **randomPeerId**(): `Promise`<`PeerId`\>

Generates a new peer id with random private key

#### Returns

`Promise`<`PeerId`\>

- Promise with the created Peer Id

#### Defined in

[internal/peerIdUtils.ts:59](https://github.com/fluencelabs/fluence-js/blob/284a59b/src/internal/peerIdUtils.ts#L59)

___

### setLogLevel

▸ `Const` **setLogLevel**(`level`): `void`

#### Parameters

| Name | Type |
| :------ | :------ |
| `level` | `LogLevelDesc` |

#### Returns

`void`

#### Defined in

[index.ts:23](https://github.com/fluencelabs/fluence-js/blob/284a59b/src/index.ts#L23)
