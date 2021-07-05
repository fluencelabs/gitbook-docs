# Security

In the Fluence network, an application consists of one or more services composed with Aquamarine. Services expose actions in the form of functions, and these actions may require authorization. In this section, we present the concept of Security Tetraplets: Verifiable origins of the function arguments in form of \(peer\_id, service\_id, function\_name, data\_getter\) tetraplets. This concept enables the secure composition of function calls with AIR scripts.

## Decouple the Authorization Service

Aquamarine, as a composability medium, needs to take care of many aspects of security to enable composing services of different vendors in a safe way. Let's consider the example of authorization service – a service that verifies permission:

```text
// Pseudocode of a service interface
service Auth:
    // Works only for the service creator 
    def grant_permission(to_peer: PeerId) 
    def check_permission(): bool
```

The service contains all the data necessary to check that permission was granted to a given peer. That is, we have authentication and authorization logic.

Consider a simple Blog service with an authorization argument for writes, i.e. adding posts.

```text
service Blog:
    def add_post(text: string, is_permitted: bool)
    def list_posts(): Post[]
```

By decoupling the storage of posts from the user and permissions model, we add a lot of flexibility to our Blog service. Not only can we use it for, say,both personal and corporate blogs but also as a building block for more complex social interactions. Just remember, the blog service itself doesn't care about security guards, it just stores posts, that's all.

Let's write an AIR script that checks permissions and adds a new post where authNode is the peer running the auth service, authSrvId and blogNde is the peer hosting the blog service:

```text
;; Script is slightly simplified for better readability
;; Setting data is omitted
(seq
  (call authNode (authSrvId "check_permission") [] token)
  (call blogNode (blogSrvId "add_post") [text token])
)
```

This is what we want to have but now let's see if we can poke holes in our security.

### First Try: Person in the Middle \(PITM/MITM\) Attack

In case check\_permission\(\) returns false, a PITM attacker intercepts the outgoing network package, takes the function output and attempts tp replace false with true. This attempt fails, however, as in Aquamarine every peer's ID is derived from its public key and every response is signed with the corresponding private key:

```rust
let resp_signature = sign(particle.signature, srvId, fnName, argsHash, responseHash)
```

Since only the private key holders can verifiably sign the output of a function call. Hence, attackers' attempts to change a function output or replay the output of a function call from another particle leads to particle rejection on blogNode.

### Second Try: Using The Wrong Service

Consider the following script where we set the token to true so that add\_post may assume that permission was actually given.

```text
(seq
  (call %init_peer_id% ("" "get_true") [] token)
  (call blogNode (blogSrvId "add_post") [text token])
)
```

How could we overcome this potential breach? On blog service host, blogNode, the entire AIR script execution flow is verified. That is, the Aquamarine interpreter visits each instruction and checks whether the particle's data has the result of the execution of this instruction and, if it does, checks that it was done by the expected peer, service, function and with the expected arguments. This is verified by the argsHash signed within _resp\_signature_. So when the token is set to a value inside the Aquamarine interpreter, we know the origin of this data: a triplet of peerId, serviceId, functionName.

In our case, the data triplet is %init\_peer\_id%, "", "get\_true" but we expect authNode, authSrvId, "check\_permission" with some known constants for authNode, authSrvId as we know where we deployed the service. As the add\_post function checks this triplet along with the token argument, and will reject the particle. Hence, we failed to trick the system by fakking the argument's origin as only the Auth service is considered a valid source of truth for authorization tokens.

Our attack got thwarted again but we have a few more tricks up our sleeves.

### Third Try: Using The Wrong Piece Of Data

Let's make a more sophisticated AuthStatus service that provides more data associated with the current peer id:

```text
struct Status:
    is_admin: bool
    is_misbehaving: bool

service AuthStatus:
    def get_status(): Status
```

If this peer misbehaves, we set a special flag as follows:

```text
;; Script is slightly simplified for better readability
;; Setting data is omitted
(seq
  (call authNode (authSrvId "get_status") [] status)
  (call blogNode (blogSrvId "add_post") [text status.$.is_admin])
)
```

So we pass an _is\_admin_ flag to the blogNode, as we now have a permissioned blog and all is well. Maybe.

The problem is that we can also pass the _is\_misbehaving_ flag to fake admin permissions and add a post. Consider other possible scenarios, where, for example, you could have a role in the status, as well as a nickname, and you need to distinguish the two, even though both are strings.

Recall that the origin of the result is stated with three values _peerId_, _serviceId_, _functionName_, while the origin of the argument is extended with one more attribute: the data getter. This forms a structure of four fields – the **tetraplet**:

```text
 struct SecurityTetraplet:
    peer_id: string
    service_id: string
    fn_name: string
    getter: string
```

The Aquamarine interpreter provides this tetraplet along with each argument during the function call, which are checked by the service if deemed necessary. In fact, tetraplets are present for every argument as a vector of vectors of tetraplets:

```text
pub tetraplets: Vec<Vec<SecurityTetraplet>>
```

which is possible due to the use of accumulators in AIR and produced with the fold instruction. Usually, you don't need to care about them, and only the first, i.e. origin, tetraplet is set.

## Limitations Of The Authentication Approach

This strategy positions that only arguments should affect function behavior by decoupling the service from the AIR script and its input data. That is, the \(public\) service API is safe only by relying on exogenous permissions checking ascertaining that the security invariants have no access to the AIR script or input data.

### Only Arguments Affect The Function Execution

This API cannot be used safely:

```text
service WrongAuth:
    def get_status_or_fail() // does not return if not authorized
```

as _WrongAuth_ service cannot be used to provide the expected checks:

```text
(seq
  (call authNode (authSrv "get_status_or_fail") []) ;; no return
  (call blogNode (blogSrv "add_post") [text]) ;; no data
)
```

In the above script, if _get\_status\_or\_fail_ fails, _add\_post_ never executes. But nothing prevents a user from calling _add\_post_ directly, so this design cannot be considered secure. That's why there must be an output from a security service to be provided as an argument later.

### Only Direct Dependencies Are Taken Into Account

Consider the modified WrongAuth, which takes the peer id as an argument:

```text
service WrongAuth:
    def get_status(peer_id) // Status of the given peer
```

In this case, a tetraplet can easily be verified that the input arguments are not compromised. However, what data is it? As arguments of _get\_status_ function are not a part of a tetraplet, we can't check that the right peer\_id was provided to the function. So from a design perspective, it is preferable for _get\_status_ to not have arguments, so that input cannot be altered.

What if we want to make the system secure in terms of tracking the data origin by taking the arguments into account? In this case, the verifier function _add\_post_ needs to know not only the name of the provider but also its structure, i.e., what inputs it has and, even worse, what the constraints of these inputs are and how to verify them. Since we cannot perform garbage collection easily, we need to express the model of the program, i..e., auth service and AIR script, on the verifier side.

This makes decomposition a pain: why decouple services if we need them to know so much about each other? That's why function calls in Aquamarine depend on the direct inputs, and direct inputs only.

**References**

* [Tetraplet implementation in the Aquamarine interpreter](https://github.com/fluencelabs/aquamarine/blob/master/crates/polyplets/src/tetraplet.rs)
* [Example of checking tetraplets for authorization in Fluent Pad](https://github.com/fluencelabs/fluent-pad/blob/main/services/history-inmemory/src/service_api.rs#L91)
* [Getting tetraplets with Rust SDK](https://github.com/fluencelabs/rust-sdk/blob/master/crates/main/src/call_parameters.rs#L35)

