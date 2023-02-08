# Local Function

## Introduction

In this chapter we will create a local function, look at its details and then start and call it.

Make sure you have created a [Local Function App](./1-2-local-function.md).

Commands in this chapter are to be executed in the _Function App_ root directory unless stated otherwise.

## Add a function to our Function App

A function is the primary concept in Azure Functions.

A function contains two important pieces:

- your code, which can be written in a variety of languages
- its config file called _function.json_

For scripting languages, you must provide the config file yourself.
For compiled languages, the config file is generated automatically from annotations in your code.

We can add a new one to our _Function App_ with the Azure Functions Core Tools – take a look at the [command reference](https://learn.microsoft.com/en-us/azure/azure-functions/functions-core-tools-reference) for insights on the parameters:

```shell
func new --name greetings --authlevel anonymous --template "HTTP Trigger"
```

That's it!
We've successfully added a new function 🎉!

When listing the files in our _Function App_ root directory, you'll see a new directory, named like our function ("greetings"), which contains all the files for the function itself.
So if you want to delete a function, it's enough to remove the directory.

> Note: We won't go into the details of different authorization levels yet. For now use _anonymous_ when you add new functions.

### The _function.json_

The _function.json_ file defines the function's trigger, bindings, and other configuration settings.
The runtime uses this config file to determine the events to monitor and how to pass data into and return data from a function execution.

#### Triggers

[Triggers](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings) cause a function to run.
They define how a function is invoked and a function must have exactly one trigger.
Triggers have associated data, which is often provided as the payload of the function.

In our case we created an HTTP Trigger, so our function is triggered via HTTP requests.

#### Bindings

[Binding](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings) to a function are a way of declaratively connecting another resource to the function.
Bindings may be connected as input bindings, output bindings, or both.
Data from bindings is provided to the function as parameters.

Bindings are optional and a function might have one or multiple input and/or output bindings.

#### Details

Browsing the contents of our _function.json_ file reveals we currently have two bindings.

<details>
  <summary>Sample function.json</summary>

```json
{
  "bindings": [
    {
      "authLevel": "Anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ],
  "scriptFile": "../dist/greetings/index.js"
}
```

</details>

- An `in`-binding (input) named `req` of type `httpTrigger`
- An `out`-binding (output) named `res` of type `http`

### The index.ts

The _index.ts_ was created for us from the HTTP Trigger template we specified when adding the new function.
It contains a sample function implementation with which we'll play around in a moment.

But first take a closer look at the constant called httpTrigger:

```typescript
const httpTrigger: AzureFunction = async function (context: Context, req: HttpRequest): Promise<void> {...}
```

There are some noteworthy things here:

First, the function is asynchronous.
Review the language-specific details if that has any implications for you (i. e. [TypeScript](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-node#contextdone-method) or [Python](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python#async))

Second, the function expects two parameters:

- The first one (`context`) is language-specific, but other languages have equivalents for it. It's passed to every function and is used for receiving and sending binding data, logging, and communicating with the runtime.
  - The structure of the context object **depends on** the selected trigger and bindings.
- The second one (`req`) is the http request object.
  - The name of the request object **must match** the name defined for the input binding in your `function.json`

## Start the function

Now we can finally start our new function with:

```shell
npx tsc && func start
```

The command triggers the TypeScript compiler and starts the function afterwards.

The output contains details on the Core Tools version and the runtime version, but more importantly we get an overview over the functions we provide, their URL and the accepted http methods.

<details>
  <summary>Sample output</summary>

```text
Azure Functions Core Tools
Core Tools Version:       4.0.4915 Commit hash: N/A  (64-bit)
Function Runtime Version: 4.14.0.19631


Functions:

        greetings: [GET,POST] http://localhost:7071/api/greetings

For detailed output, run func with --verbose flag.
```

</details>

### Shutdown the function

To shutdown your Function App, press `ctrl+c`.

### Verbose flag

The _verbose flag_ can be helpful if you want to get insights what the runtime is doing under the hood.

Give it a try and start the function with the _verbose flag_:

```shell
npx tsc && func start --verbose
```

> Note: The verbose flag toggles only the runtime log level, but not the log level of your functions.
> We'll learn how to toggle the function log levels later, though.

## Calling the function

After starting the function app you can easily test it with any Rest client or open it in your [browser](http://localhost:7071/api/greetings):

```shell
curl http://localhost:7071/api/greetings
```

Or with parameters ([browser](http://localhost:7071/api/greetings?name=codecentric)):

```shell
curl http://localhost:7071/api/greetings -d '{"name":"codecentric"}'
```

## <span class="quiz">Quiz</span>

<details>
  <summary>What file specifies the accepted http methods for a function of type HTTP Trigger?</summary>

Every function has it's dedicated settings file called _function.json_.

</details>
<br/>
<details>
  <summary>What error code does the Function send if you called it with an unspecified http method?</summary>

It responds with [404 Not Found](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404).

Test command:

```shell
curl -X OPTION http://localhost:7071/api/greetings -v
```

</details>
<br/>
<details>
  <summary>Can you see error messages in the Function console output? Does the output change if you use the verbose flag to start the Function?</summary>

The console output does indeed not show unsuccessful attempts to call the function.
That changes, however, if we restart the Function with the verbose flag.

Test command:

```shell
curl -X OPTION http://localhost:7071/api/greetings -v
```

</details>
<br/>
<details>
  <summary>Are there function bindings for Kafka? If so, which runtime version is supported? Are in, out or both binding types supported?</summary>

Take a look at the [documentation](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings#supported-bindings).

As you can see Kafka is supported since runtime version 2.x.
Furthermore, only output bindings are supported.

</details>
