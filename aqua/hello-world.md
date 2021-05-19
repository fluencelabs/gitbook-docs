---
description: WIP
---

# Hello World

In order to write our first peer-to-peer application, we will take advantage of an already deployed greeting, aka hello world, service. 

ToDo: Dashboard -- no testnet

```text
-- greeter.aqua                                    1
service Echo("service-id"):                        2
    echo: []string -> []string


service Greeting("service-id"):                    3
    greeting: string, bool -> string

func seq_echo_greeter(data: []string, name: string, greeter: bool) -> []string:    4
    big_res: []string
    echo_res <- Echo.echo(data)
    for s <- echo_res:
        big_res.append(Greeting.greeting(s, greeter))
    <- big_res

func echo_greeter_seq(data: []string, name: string, greeter: bool) -> []string:    5
    echo_res <- Echo.echo(data)
```





