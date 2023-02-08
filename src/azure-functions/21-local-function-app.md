# Local Function App

## Introduction

In this chapter we will create a local function app and take a closer look at it's files.

Before we create our first _Function App_ let me point out an important distinction between the terms _Azure Functions_, _Functions_, _Function App_ and _function_:

_Azure Functions_, _Functions_ and _Function App_, are used interchangeable in the Documentation and refer to Azure's serverless compute service or more specific it's runtime environment, whereas a _function_ or _functions_ (lower case) refer to a block of code executed in the runtime environment.

## Install the Azure Function Core Tools

[Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools) lets you develop and test your functions on your local computer from the command prompt or terminal.
Your local functions can connect to live Azure services, and you can debug your functions on your local computer.
You can even deploy a Function App to your Azure subscription.

Install the Azure Function Core Tools via the node package manager (npm):

```shell
npm install -g azure-functions-core-tools@4 --unsafe-perm true
```

Check if you can execute the `func` command:

```shell
func --version
```

> Note: If you can't execute the `func` command check the output from the installation command and your _PATH_ environment variable.

## Create a Function App

To create a new Function App in a directory called `function-app-demo` type:

```shell
func init function-app-demo --worker-runtime node --language typescript
```

Change to the newly created directory and browse the `devDependencies` section in the package.json file.
If the package `@types/node` is not installed in version _18_ update it with:

```shell
npm install -D @types/node@18
```

> Note: We're using the types for `@types/node@18`, because the version should match with the Node.js version mentioned in [prerequisites](./README.md#prerequisites).

## The Function App's files

Let's take a look at the Function App specific files in our new app.

### The _host.json_

The [host.json](https://learn.microsoft.com/en-us/azure/azure-functions/functions-host-json) metadata file contains configuration options that affect all functions in a function app instance.

Browsing the file contents you will notice to different version statements.
The first one (root level) defines the _Function Runtime_ version and the second one defines the _Extension Bundle_ version.

[Function Runtime](https://learn.microsoft.com/en-us/azure/azure-functions/functions-versions?tabs=v4&pivots=programming-language-typescript), as the name implies, specifies the runtime version for the function.
The runtime version defined in the host.json can be misleading though because it's specified as either version **1** or version **2** where the later effectively means version **2 and above**.
The actual runtime version is defined by the _Azure Functions Core Tools_ we're using.

[Extension bundles](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-register#extension-bundles) are a way to add a pre-defined set of compatible binding extensions to your function app.

#### <span class="task">🛠 TASK:</span> Update the extension bundle version

Update the extensionBundle version to version **4**.

<details>
  <summary>💡 HINT</summary>

Check out the [documentation](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-register#extension-bundles).

</details>
<br/>
<details>
  <summary>🎓 SOLUTION</summary>

The extensionBundle definition in the host.json should now look like:

```json
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.0.0, 5.0.0)"
  }
```

</details>

### The _local.settings.json_

The _local.settings.json_ represents the [App settings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-app-settings) for your local environment, which contain configuration options that affect all functions for that function app.
These settings are accessed as environment variables.

As of now the file doesn't contain too many settings.
However, there is a setting for our language worker runtime (`FUNCTIONS_WORKER_RUNTIME`) which corresponds to the language runtime being used in your application (in our case node).

### The _.funcignore_

A function app may contain language-specific files and directories that shouldn't be [published](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local#publish).
Excluded items are listed in the _.funcignore_ file in the root directory.

> Note: The file already includes the directories of the dev dependencies in our package.json file and a test directory.
> If you add further dependencies you should make sure they are listed in the .funcignore file.
> This is something to keep in mind as it might slow down the publishing of new versions considerably if you upload test frameworks or other unwanted (and possibly large) dependencies to Azure.
