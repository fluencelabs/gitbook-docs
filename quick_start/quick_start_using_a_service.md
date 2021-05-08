# Using a Service

Let's dive right into peer-to-peer awesomeness by harnessing a distributed curl service, which pretty much keeps with its namesake: pass it a url and collect the response. Instead of developing our service from scratch, we reuse one already deployed to the \[Fluence testnet\]\([https://dash.fluence.dev/nodes](https://dash.fluence.dev/nodes)\).

The [Fluence Dashboard](https://dash.fluence.dev/) facilitates the discovery of available services, such as the [Curl Adapter](https://dash.fluence.dev/blueprint/b7d2454e-2a75-408c-a23a-fe35de3beeb9) service, which allows us to harness http\(s\) requests as a service. Drilling down on the metadata provides a few useful parameters such as _service id_, _node id_ and _ip address_, which we need to execute our distributed curl service.

In order to execute the cUrl service and collect the result, i.e., response, we call upon our composition and coordination medium Aquamarine via an Aquamarine Intermediate Representation \(AIR\) script.

```scheme
;; handle possible errors via xor
(xor
    (seq
        ;; call function 'service_id.request' on node 'relay'
        (call relay (service_id "request") [url] result)

        ;; return result back to the client
        (call %init_peer_id% (returnService "run") [result])
    )
    ;; if error, return it to the client
    (call %init_peer_id% (returnService "run") [%last_error%])
)
```

Without going too deep into Aquamarine and AIR, this script specifies that we call a public peer-to-peer relay \(node\) on the network, ask to run the \(curl\) _request_ function with data parameter _url_  and _service\_id_ parameter, and collect the _result_ **xor** the _error message_ in case of execution failure. We also promise to pass the _service\_id_ and _url_ parameters to the scripts. 

The "magic" happens by handing the script to the `fldist` CLI tool, which then sends the script for execution to the specified p2p network and locally shadows the execution. Please note that Instead of developing full-fledged frontend applications, we use the `fldist` CLI tool. However, a [JS SDK](https://github.com/fluencelabs/fluence-js) is available to accelerate the development of more complex frontend applications.

{% hint style="info" %}
Throughout the document, we utilize service and node ids, which in most cases may be different for you.
{% endhint %}

With the service id parameter obtained from the dashboard lookup above, e.g., "f92ce98b-1ed6-4ce3-9864-11f4e93a478f", and some Fluence goodness at both the local and remote levels enables us to:

1. find the p2p node hosting the curl service with above service id ,
2. execute the service and
3. collect the response

In your directory of choice, save the above script as _curl\_request.clj_ and run:

```bash
$ fldist run_air -p curl_request.clj -d '{"service_id": "f92ce98b-1ed6-4ce3-9864-11f4e93a478f", "url":"https://api.duckduckgo.com/?q=homotopy&format=json"}'
```

and voila, book-ended by process and network metadata, we see our result in the stdout pipe ready for further processing.

```bash
client seed: 3qrmrkDUkXRu5tNawBz2wgvhaS6pgUSRVUdcG6hZgzjx
client peerId: 12D3KooWLu13eNDTfvBeH92a7o4nzLGgjrbVnot85etMKPfzoQyK
node peerId: 12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb
Particle id: 005a2c5a-3cde-48ed-92eb-3ce814281ba0. Waiting for results... Press Ctrl+C to stop the script.
===================
[
  {
    "error": "",
    "ret_code": 0,
    "stderr": "",
    "stdout": "{\"Abstract\":\"In topology, a branch of mathematics, two continuous functions from one topological space to another are called homotopic if one can be \\\"continuously deformed\\\" into the other, such a deformation being called a homotopy between the two functions. A notable use of homotopy is the definition of homotopy groups and cohomotopy groups, important invariants in algebraic topology. In practice, there are technical difficulties in using homotopies with certain spaces. Algebraic topologists work with compactly generated spaces, CW 
    <snip>
  }
]
[
  [
    {
      peer_pk: '12D3KooWBUJifCTgaxAUrcM9JysqCcS4CS8tiYH5hExbdWCAoNwb',
      service_id: '56f1a984-b719-45e9-baa8-f99ed0a9810b',
      function_name: 'request',
      json_path: ''
    }
  ]
]
===================
^C
```

Feel free to experiment with the AIR script not only with different url's but also different curl services already available from the [Fluence Dashboard](https://dash.fluence.dev/); maybe even force an error or two.

To recap, we:

* discovered a cUrl service on the Fluence Dashboard,
* wrote a short AIR script to coordinate the service into an application
* submitted the script with the Fluence distributor tool `fldist` to the Fluence p2p testnet
* executed the \(remote\) curl service request, and
* collected the result

With essentially a two line script and a couple of parameters we executed a search request as a service on a peer-to-peer network. Even this small example should impress the ease afforded by Aquamarine to compose applications from portable, reusable and distributed services not only taken serverless to the next level by greatly reducing devops requirements but also empowering developers with a composition and coordination medium second to none.

In the next section, we build an Ethereum block getter application by coordinating multiple services into an application.

