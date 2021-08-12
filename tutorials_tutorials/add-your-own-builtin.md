# Add Your Own Builtins

As discussed in the [Node]() section, some service functionalities have ubiquitous demand making them suitable candidates to be directly deployed to a peer node. The [Aqua distributed hash table](https://github.com/fluencelabs/fluence/tree/master/deploy/builtins/aqua-dht) \(DHT\) is an example of builtin service. The remainder of this tutorial guides you through the steps necessary to create and deploy a Builtin service.

In order to have a service available out-of-the-box with the necessary startup and scheduling scripts, we can take advantage of the Fluence [deployer feature](https://github.com/fluencelabs/fluence/tree/master/deploy) for Node native services. This feature handles the complete deployment process including

* module uploads,
* service deployment and
* script initialization and scheduling

Note that the deployment process is a fully automated workflow requiring you to merely submit your service assets, i.e., Wasm modules and configuration scripts, in the appropriate format as a PR to the [Fluence](https://github.com/fluencelabs/fluence) repository.

At this point you should have a solid grasp of creating service modules and their associated configuration files.

Our first step is fork the [Fluence](https://github.com/fluencelabs/fluence) repo by clicking on the Fork button, upper right of the repo webpage, and follow the instructions to create a local copy. In your local repo copy, checkout a new branch with a new, unique branch name:

```text
cd fluence
git checkout -b MyBranchName
```

In our new branch, we create a directory with the service name in the _deploy/builtin_ directory:

```text
cd deploy/builtins 
mkdir my-new-super-service
cd my-new-super-service
```

Replace _my_-_new-super-service_ with your service name.

Now we can build and populate the required directory structure with your service assets. You should put your service files in the corresponding _my_-_new-super-service_ directory.

## Requirements

In order to deploy a builtin service, we need

* the Wasm file for each module required for the service
* the blueprint file for the service
* the optional start and scheduling scripts

### Blueprint

Blueprints capture the service name and dependencies:

```javascript
// example_blueprint.json
{
  "name": "my-new-super-service", 
  "dependencies": [
    "name:my_module_1",
    "name:my_module_2",
    "hash:Hash(my_module_3.wasm)"
  ]
}
```

Where

* name specifies the service's name and 
* dependencies list the names of the Wasm modules or the Blake3 hash of the Wasm module

In the above example, _my\_module\_i_ refers to ith module created when you compiled your service code

{% hint style="info" %}
The easiest way to get the Blake3 hash of our Wasm modules is to install the [b3sum](https://crates.io/crates/blake3) utility:

```text
cargo install b3sum
b3sum my_module_3.wasm
```
{% endhint %}

If you decide to use the hash approach, please use the hash for the config files names as well \(see below\).

### **Start Script**

Start scripts, which are optional, execute once after service deployment or node restarts and are submitted as _AIR_ files and may be accompanied by a _json_ file containing the necessary parameters.

```text
;; on_start.air
(seq
    (call relay ("some_service_alias" "some_func1") [variable1] result)
    (call relay ("some_service_alias" "some_func2") [variable2 result])
)
```

and the associated data file:

```javascript
// on_start.json data for on_start.air
{
    "variable1" : "some_string",
    "variable2" : 5,
}
```

### **Scheduling Script**

Scheduling scripts allow us to decouple service execution from the client and instead can rely on a cron-like scheduler running on a node to trigger our service\(s\). 

### Directory Structure

Now that we got our requirements covered, we can populate the directory structure we started to lay out at the beginning of this section. As mentioned above, service deployment as a builtin is an automated workflow one our PR is accepted. Hence, it is imperative to adhere to the directory structure below:

```text
-- builtins
    -- {service_alias}
        -- scheduled
            -- {script_name}_{interval_in_seconds}.air [optional]
        -- blueprint.json
        -- on_start.air [optional]
        -- on_start.json [optional]
        -- {module1_name}.wasm
        -- {module1_name}_config.json
        -- Hash(module2_name.wasm).wasm
        -- Hash(module2_name.wasm)_config.json
        ...
```

For a complete example, please see the [aqua-dht](https://github.com/fluencelabs/fluence/tree/master/deploy/builtins/aqua-dht) builtin:

```text
fluence
    --deploy
        --builtins
            --aqua-dht
                 -aqua-dht.wasm
                 -aqua-dht_config.json
                 -blueprint.json
                 -scheduled
                 -sqlite3.wasm # or 558a483b1c141b66765947cf6a674abe5af2bb5b86244dfca41e5f5eb2a86e9e.wasm 
                 -sqlite3_config.json # or 558a483b1c141b66765947cf6a674abe5af2bb5b86244dfca41e5f5eb2a86e9e_config.json
```

which is based on the [eponymous](https://github.com/fluencelabs/aqua-dht) service project.

