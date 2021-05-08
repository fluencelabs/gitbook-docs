# Services

## Overview

Each Fluence peer is equipped with a set of "built-in" services that can be called from Aquamarine and fall into the following namespaces:

1. _peer_ - operations related to connectivity or state of a given peer
2. _kad_ - Kademlia API
3. _srv_ – management and information about services on a node
4. _dist_ – distribution and inspection of modules and blueprints
5. _script_ – to manage recurring scripts
6. _op_ – basic operations on data deprecated - namespace for deprecated API Below is the reference documentation for all the existing built-in services. Please refer to the JS SDK documentation to learn how to easily use them from the JS SDK
7. _deprecated_ - namespace for deprecated API

Please note that the [`fldist`](../knowledge_tools.md#fluence-proto-distributor-fldist) CLI tool, as well as the [JS SDK](../knowledge_tools.md#fluence-js-sdk), provide access to node-based services.


## API

### peer is\_connected


Checks if there is a direct connection to the peer identified by a given PeerId

* **Arguments**:
  * PeerId – id of the peer to check if there's a connection with
* **Returns**: bool - true if connected to the peer, false otherwise

Example of a service call:

```scheme
(call node ("peer" "is_connected") ["123D..."] ok)
```

Initiates a connection to the specified peer

* **Arguments**
  * _PeerId_ – id of the target peer
  * [_Multiaddr_](https://crates.io/crates/multiaddr) – an array of target peer's addresses
* **Returns**: bool - true if connection was successful

Example of a service call:

```scheme
(seq 
    (call node ("op" "identity") ["/ip4/1.2.3.4/tcp/7777" "/ip4/1.2.3.4/tcp/9999"] addrs)
    (call node ("peer" "connect") ["123D..." addrs] ok) 
)
```

### peer get\_contact

Resolves the contact of a peer via [Kademlia](https://en.wikipedia.org/wiki/Kademlia)

* **Arguments**
  * _PeerId_ – id of the target peer
* **Returns**: Contact - true if connection was successful

```rust
// get_contact return struct
Contact { 
    peer_id: PeerId,
    addresses: [Multiaddr] 
}
```

Example of a service call:

```scheme
(call node ("peer" "get_contact") ["123D..."] contact)
```

#### peer identify

Get information about the peer

* **Arguments**: None
* **Returns:**  _external address_

```javascript
{ "external_addresses": [ "/ip4/1.2.3.4/tcp/7777", "/dns4/stage.fluence.dev/tcp/19002" ] }
```

Example of service call:

```scheme
(call node ("peer" "identify") [] info) peer timestamp_ms
```

### peer timestamp\_ms


Get Unix timestamp in milliseconds

* **Arguments**: None
* **Returns**: _u128_ - number of milliseconds since 1970

Example of service call:

```scheme
(call node ("peer" "timestamp_ms") [] ts_ms)
```

### peer timestamp\_sec


Get Unix timestamp in seconds

* **Arguments**: None
* **Returns**: _u64_ - number of seconds since 1970

Example of service call:

```scheme
(call node ("peer" "timestamp_sec") [] ts_sec)
```

### kad neighborhood

Instructs node to return the locally-known nodes in the Kademlia neighborhood for a given key

* **Arguments**: _key_ – the peer ID \(PeerId\) of the node
* **Returns**: _peers_ – an array of PeerIds of the nodes that are in the Kademlia neighborhood for the given hash\(key\)

Example of service call:

```scheme
(call node ("dht" "neighborhood") [key] peers)
```

Please note that this service does _not_ traverse the network and may yield incomplete neighborhood.

### srv create

Used to create a service on a certain node.

* **Arguments**:
  * blueprint\_id – ID of the blueprint that has been added to the node specified in the service call by the dist add\_blueprint service. 
* **Returns**: service\_id – the service ID of the created service.

Example of service call:

```scheme
(call node ("srv" "create") [blueprint_id] service_id)
```

### srv list

Used to enumerate services deployed to a peer.

* **Arguments**: None
* **Returns**: a list of services running on a peer

Example of service call:

```scheme
(call node ("srv" "list") [] services)
```

### srv add\_alias

Adds an alias on service, so service could be called not only by service\_id but by alias.

* **Argument**: alias - settable service name service\_id – ID of the service whose interface you want to name.
* **Returns**: alias id

Example of service call:

```scheme
(call node ("srv" "add_alias") [alias service_id])
```

### srv get\_interface

Retrieves the functional interface of a service running on the node specified in the service call.

* Argument: service\_id – ID of the service whose interface you want to retrieve. 
* Returns : an interface object of the following structure:

```typescript
{ 
    interface: { function_signatures, record_types },
    blueprint_id: "uuid-1234...",
    service_id: "uuid-1234..." 
}
```

Example of service call:

```scheme
(call node ("srv" "get_interface") [service_id] interface)
```

### dist add\_module

Used to add modules to the node specified in the service call.

* Arguments:

  * bytes – a base64 string containing the .wasm module to add. 
  * config – an object of the following structure

  ```javascript
  {
    "name": "my_module_name"
  }
  ```

Example of service call:

```scheme
(call node ("dist" "add_module") [bytes config] hash)
```

### dist list\_modules

Get a list of modules available on the node

* Arguments: None
* Returns: an array of objects containing module descriptions

  ```javascript
  [ 
      { 
          "name": "moduleA",
          "hash": "6ebff28c",
          "config": { "name": "moduleA" }
      } 
  ]
  ```

Example of service call:

```scheme
(call node ("dist" "list_modules") [] modules)
```

### dist get\_module\_interface

Get the interface of a module

* Arguments: hash of a module
* Returns: an interface of the module \( see _srv get\_interface \)_ mple of service call:

```scheme
(call node ("dist" "get_interface") [hash] interface)
```

### dist add\_blueprint

Used to add a blueprint to the node specified in the service call.

* Arguments: blueprint – an object of the following structure

  ```javascript
  {
      "name": "good_service",
      "dependencies": [ "hash:6ebff28c...", "hash:1e59875a...", "hash:d164a07..." ] 
  }
  ```

  ```text
    Where module dependencies are specified as [_blake3_](https://crates.io/crates/blake3) hashes of modules
  ```

* Returns: Generated blueprint id

Example of service call:

```scheme
(call node ("dist" "add_blueprint") [blueprint] blueprint_id)
```

### dist list\_blueprints

Used to get the blueprints available on the node specified in the service call.

* Arguments: None
* Returns: an array of blueprint structures.

A blueprint is an object of the following structure:

```javascript
{ 
    "id": "uuid-1234-...", 
    "name": "good_service", 
    "dependencies": [ "hash:6ebff28c...", "hash:1e59875a...", "hash:d164a07..." ] 
}
```

Example of service call:

```scheme
(call node ("dist" "list_blueprints") [] blueprints)
```

#### script add

Adds a given script to a node. That script will be called with a fixed interval with the default setting at approx. three \(3\) seconds.

Recurring scripts can't read variables from data, they must be literal. That means that every address or value must be specified as a literal: \(call "QmNode" \("service\_id-1234-uuid" "function"\) \["arg1" "arg2"\]\).

* Arguments:
  * _script_ – a string containing "literal" script 
  * _interval_ – an optional string containing interval in seconds. If set, the script will be executed periodically at that interval. If omitted, the script will be executed only once. All intervals are rounded to 3 seconds. The minimum interval is 3 seconds.
* Returns: uuid – script id that can be used to remove that script

Example of service call:

* Without an interval parameter value, the script executes once:

  ```text
  (call node ("script" "add") [script] id)
  ```

* With an interval parameter value _k_ passed as a string, the script executes every _k_ seconds \(21 in this case\)

  ```scheme
  (call node ("script" "add") [script "21"] id)
  ```

### script remove

Removes recurring script from a node. Only a creator of the script can delete it

* Arguments: _script id_ \(as received from _script add_\)
* Returns: true if the script was deleted and false otherwise

Example of service call:

```scheme
(call node ("script" "remove") [script_id] result)
```

### script list

* Arguments: None
* Returns: A list of existing scripts on the node. Each object in the list is of the following structure:

  ```javascript
  { 
      "id": "uuid-1234-...",
      "src": "(seq (call ...",  // 
      "failures": 0,
      "interval": "21s",
      "owner": "123DKooAbcEfgh..."
  }
  ```

  Example of a service call:

```scheme
(call node ("script" "list") [] list)
```

### op identity

Acts as an identity function. This service returns exactly what was passed to it. Useful for moving the execution of some service topologically or for extracting some data and putting it into an output variable.

Example of service call:

```scheme
(call node ("op" "identity") [args] result)
```

### deprecated add\_provider

Used in service aliasing. _\*\*_Stores the specified service provider \(provider\) in the internal storage of the node indicated in the service call and associates it with the given key \(key\). After executing add\_provider, the provider can be accessed via the get\_providers service using this key.


* Arguments:

  * key – a string; usually, it is a human-readable service alias. 
  * provider – the location of the service. It is an object of the following structure:

  ```javascript
  { 
      "peer": "123D...", // PeerId of some peer in the network
      "service_id": "uuid-1234-..." // Optional service_id of the service running on the peer specified by peer 
  }
  ```

Example of service call:

```scheme
(call node ("deprecated" "add_provider") [key provider])
```

### deprecated get\_providers

Used in service aliasing to retrieve providers for a given key.

* Arguments: _key_ – a string; usually, it is a human-readable service alias. 
* Returns: an array of objects of the following structure:

  ```javascript

  { 
      "peer": "123D...", // required field
      "service_id": "uuid-1234-..." // optional field
  }
  ```

Example of service call:

```scheme

(call node ("deprecated" "get_providers") [key] providers)
```

