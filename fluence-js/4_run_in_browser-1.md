# Running app in browser

You can use the Fluence JS with any framework (or even without it). The "fluence" part of the application is a collection of pure typescript\javascript functions which can be called withing any framework of your choosing.

## Configuring application to run in browser

{% hint style="warning" %}
**Breaking change!**\
****\
****Starting from **v0.23.0**  Fluence JS uses different file structure:

* **copy-avm-public** script has been replaced with **copy-marine**
* **runnerScript.web.js** has been replaced with **`marine-js.web.js`**
* additional dependency added: **`marine-js.wasm`**



Make sure to migrate when switching to **v0.23.0**
{% endhint %}

To run application in browser you need to configure the server which hosts the application to serve Marine JS dependencies. Fluence JS provides a utility script named `copy-marine` which copies all the necessary files into specified folder. For example if static files are served from the `public` directory you can run `copy-marine public` to copy files needed to run Fluence. It is recommended to put their names into `.gitignore` and run the script on every build to prevent possible inconsistencies with file versions. This can be done using npm's `postinstall` script:

```
  ...
  "scripts": {
    "postinstall": "copy-marine public",
    "start": "...",
    ..
  },
  ...
```

In case you want to distribute the files by hand here is the list of dependencies required to run Fluence:

* `marine-js.wasm` is the Marine JS runtime. The file is located in `@fluencelabs/marine-js` package.
* `marine-js.web.js` is the web worker script responsible for running Marine JS in background. The file is located in `@fluencelabs/marine-js` package.
* `avm.wasm` is the AquaVM execution file. The file is located in `@fluencelabs/avm` package.

## Demo applications

See the browser-example which demonstrate integrating Fluence with React: [github](https://github.com/fluencelabs/examples/tree/main/fluence-js-examples/browser-example)

Also take a look at FluentPad. It is an example application written in React: [https://github.com/fluencelabs/fluent-pad](https://github.com/fluencelabs/fluent-pad)
