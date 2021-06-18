# Add Your Own Builtins

As discussed in the [Node](../knowledge_knowledge/node/knowledge_node_services.md) section, some service functionalities useful to a large audience and such services and can be directly deployed to a peer node. The remainder of this tutorial guides you through the steps necessary to create and deploy a Builtin service. 

In order to have a service available out-of-the-box with the necessary startup and scheduling scripts,  we can take advantage of the Fluence [deployer feature](https://github.com/fluencelabs/fluence/tree/master/deploy) for Node native Services. This feature  handles the complete deployment process including  

* module uploads,
* service deployment, 
* script initialization and scheduling

Note that the deployment process is a fully automated workflow requiring you to merely submit your service assets in the appropriate structure as a PR to the appropriate GitHub repository. At this point you should have a solid grasp of creating service modules and their associated configuration files. See the [Developing Modules And Services](../development_development/) section for details.

Our first step is fork  the ??? repo by clicking on the Fork button, upper right of the repo webpage, and follow the instructions to create a local copy. In your local repo copy, checkout a new branch with a new, unique branch name:

```text
git checkout -b MyBranchName 
```

In your new branch create a new directory with the service name in the _builtin_ directory:

```text
cd builtins 
mkdir my-new-super-service
cd new-super-service
```

 Replace my-_new-super-service_ with your service name. 

Now we can build and populate the required directory structure with your service assets. You should put your service files in the corresponding my-_new-super-service_  directory specified in config as `builtins_base_dir`   

**TODO: check if that applies to new repo approach.**

#### Requirements

In order to deploy a builtin service, you need

* the Wasm file for each module required for the service
* the blueprint file for the service
* start and scheduling scripts

Just to recap, blueprints capture the service name and dependencies. For example:

```javascript
// example_blueprint.json
{
  "name": "my-new-super-service", 
  "dependencies": [
    "name:my_module_1",
    "name:my_module_2"
  ]
}
```

where name specifies the service's name and _my\_module\_i_ refers to ith module created when you compiled your service code, i.e. _my\_module\_i.wasm_. Please note that dependencies may also be specified as hashes:

```javascript
// example_blueprint.json with hashes
{
  "name": "aqua-dht",
  "dependencies": [
    "hash:558a483b1c141b66765947cf6a674abe5af2bb5b86244dfca41e5f5eb2a86e9e",
    "name:aqua-dht"
  ]
}
```



Start Script



Scheduling Script



Putting it all together:

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
        -- {module2_name}.wasm
        -- {module2_name}_config.json
        ...
```

In blueprint you can specify dependencies either with name or hashes but .wasm files and config should have corresponding names. `blieprint.json` example:

Just to recap, blueprints specify the service name and module dependencies:



In blueprint you can specify dependencies either with name or hashes but .wasm files and config should have corresponding names. `blieprint.json` example:

```javascript
// blueprint.json
{
  "name": "my-new-super-service",
  "dependencies": [
    "name: module_1",
    "name: module_2"
  ]
}
```

where _module\_i_ is the name of the Wasm module. Note that dependencies can be specified as string literals or hashes:

```javascript
// blueprint.json
{
  "name": "aqua-dht",
  "dependencies": [
    "hash:558a483b1c141b66765947cf6a674abe5af2bb5b86244dfca41e5f5eb2a86e9e",
    "name:aqua-dht"
  ]
}
```

So modules and configs names should look like this:

```text
-- aqua-dht.wasm
-- aqua-dht_config.json
-- 558a483b1c141b66765947cf6a674abe5af2bb5b86244dfca41e5f5eb2a86e9e.wasm
-- 558a483b1c141b66765947cf6a674abe5af2bb5b86244dfca41e5f5eb2a86e9e_config.json
```

**Start Script**

Start scripts, which are optional, execute once after service deployment or node restarts and are submitted as _air_  files and may be accompanied by a json file containing the necessary parameters.

```text
;; on_start.air
(seq
    (call relay ("some_service_alias" "some_func1") [variable1] result)
    (call relay ("some_service_alias" "some_func2") [variable2 result])
)
```

and the associated data file:

```javascript
// on_start.json
{
    "variable1" : "some_string",
    "variable2" : 5,
}
```



**Scheduling Script**

TBD

see previous reference in [additional concepts](../development_development/development_reward_block_app/development_additional_concepts.md)

Scheduling scripts allows us to decouple service execution from the client and instead can rely on a cron-like scheduler to independently trigger our service\(s\).

#### Directory Structure

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
        -- {module2_name}.wasm
        -- {module2_name}_config.json
        ...
```

can we call service _alias just service name._

