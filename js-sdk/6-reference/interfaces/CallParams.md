# Interface: CallParams<ArgName\>

Additional information about a service call

## Type parameters

| Name | Type |
| :------ | :------ |
| `ArgName` | extends `string` \| ``null`` |

## Table of contents

### Properties

- [initPeerId](js-sdk/6_reference/interfaces/CallParams.md#initpeerid)
- [particleId](js-sdk/6_reference/interfaces/CallParams.md#particleid)
- [signature](js-sdk/6_reference/interfaces/CallParams.md#signature)
- [tetraplets](js-sdk/6_reference/interfaces/CallParams.md#tetraplets)
- [timeStamp](js-sdk/6_reference/interfaces/CallParams.md#timestamp)
- [ttl](js-sdk/6_reference/interfaces/CallParams.md#ttl)

## Properties

### initPeerId

• **initPeerId**: `string`

The peer id which created the particle

#### Defined in

[internal/commonTypes.ts:36](https://github.com/fluencelabs/fluence-js/blob/480d630/src/internal/commonTypes.ts#L36)

___

### particleId

• **particleId**: `string`

The identifier of particle which triggered the call

#### Defined in

[internal/commonTypes.ts:31](https://github.com/fluencelabs/fluence-js/blob/480d630/src/internal/commonTypes.ts#L31)

___

### signature

• **signature**: `string`

Particle's signature

#### Defined in

[internal/commonTypes.ts:51](https://github.com/fluencelabs/fluence-js/blob/480d630/src/internal/commonTypes.ts#L51)

___

### tetraplets

• **tetraplets**: { [key in string]: SecurityTetraplet[]}

Security tetraplets

#### Defined in

[internal/commonTypes.ts:56](https://github.com/fluencelabs/fluence-js/blob/480d630/src/internal/commonTypes.ts#L56)

___

### timeStamp

• **timeStamp**: `number`

Particle's timestamp when it was created

#### Defined in

[internal/commonTypes.ts:41](https://github.com/fluencelabs/fluence-js/blob/480d630/src/internal/commonTypes.ts#L41)

___

### ttl

• **ttl**: `number`

Time to live in milliseconds. The time after the particle should be expired

#### Defined in

[internal/commonTypes.ts:46](https://github.com/fluencelabs/fluence-js/blob/480d630/src/internal/commonTypes.ts#L46)
