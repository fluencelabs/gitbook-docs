# Add Your Own Builtin



If you want to have a service that available out-of-the-box with startup script and scheduled scripts, you can use `builtins deployer` feature. It will upload modules, deploy service, run init script and schedule others.  
You should put your service files in the corresponding folder specified in config as `builtins_base_dir`.

### Builtins directory structure

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

