# Running app in browser

You can use the Fluence JS with any framework (or even without it). The "fluence" part of the application is a collection of pure typescript\javascript functions which can be called withing any framework of your choosing.

## Configuring application to run in browser

To run application in browser you need to configure the server which hosts the application to serve two additional files:

* `avm.wasm` is the execution file of AquaVM. The file is located in `@fluencelabs/avm` package
* `runnerScript.web.js` is the web worker script responsible for running AVM in background. The file is located in `@fluencelabs/avm-runner-background` package

Fluence JS provides a utility script named `copy-avm-public` which locates described above files and copies them into the specified directory. For example if static files are served from the `public` directory you can run `copy-avm-public public` to copy files needed to run Fluence. It is recommended to put their names into `.gitignore` and run the script on every build to prevent possible inconsistencies with file versions. This can be done using npm's `postinstall` script:

```
  ...
  "scripts": {
    "postinstall": "copy-avm-public public",
    "start": "...",
    ..
  },
  ...
```

## Demo applications

See the browser-example which demonstrate integrating Fluence with React: [github](https://github.com/fluencelabs/examples/tree/main/fluence-js-examples/browser-example)

Also take a look at FluentPad. It is an example application written in React: [https://github.com/fluencelabs/fluent-pad](https://github.com/fluencelabs/fluent-pad)
