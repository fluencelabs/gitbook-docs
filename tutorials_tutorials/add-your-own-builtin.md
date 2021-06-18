---
description: The Case For Node Native Services
---

# Add Your Own Node Native Service

As discussed in the [Node](../knowledge_knowledge/node/knowledge_node_services.md) section, some service functionalities useful to a large audience. Such services and can be directly deployed to a peer node as a Wasm module. The remainder of this tutorial guides you through the steps necessary to create and submit a Node Native Service candidate. 

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

Now we can build and populate the required directory structure with your service assets. You should put your service files in the corresponding my-_new-super-service_  directory specified in config as `builtins_base_dir`   **TODO: check if that applies to new repo approach.**

Asset Requirements

In order to deploy a builtin service, you need

* the wasm files for each module as the module build
* the blueprint file for the service
* start and schedule scripts

Just to recap, Blueprints capture module names, blueprint name, and blueprint id. -- builtins

```text
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

```javascript
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

`on_start.air` is optional and can contain some startup script and you can specify necessary variables in `on_start.json`. It will be executed only once after service deployment or node restart.

`on_start.json` example:

```javascript
{
    "variable1" : "some_string",
    "variable2" : 5,
}
```

`on_start.air` example:

```text
(seq
    (call relay ("some_service_alias" "some_func1") [variable1] result)
    (call relay ("some_service_alias" "some_func2") [variable2 result])
)
```

