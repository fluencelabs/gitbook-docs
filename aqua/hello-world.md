---
description: WIP
---

# Hello World

_**Hello World**_, the obligatory introduction to just about any programming language, ... TODO  

In order to write our first peer-to-peer application, we need a network, a service and a workflow to coordinate the service into an application. For the purposes of this introduction to Aqua, we reuse an already deployed service



ToDo: Dashboard -- no testnet



```text
-- greeter.aqua                                        1
service Local("returnService"):                        2
  run: string -> ()

service Greeting("service-id"):                        3
    greeting: string, bool -> string

func greeting(name: string, greeter: bool, node: string, greeting_service: string) -> string:
  on node:
    Greeting greeting_service
    res <- Greeting.greeting(name, greeter)
  Local.run(res)
  <- res



```

1. `--` is a comment with the Aqua file name
2.  We define a local services ...
3. We define the interface to the deployed service ...
4. we define our workflow by specifying node, service 



