# Working with App Settings

## Introduction

In the previous step we deployed our Function App to Azure.
We'll now take a look at the [App settings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-app-settings).

Application settings in a function app contain configuration options that affect all functions for that function app.
By default, you store connection strings and secrets used by your function app and bindings as application settings and access them as environment variables.
However, secrets not needed by Azure to be stored as application settings should rather be stored in [Azure KeyVault](https://learn.microsoft.com/en-us/azure/key-vault/general/) instead of the application settings.

Commands in this chapter are to be executed in the _Function App_ root directory unless stated otherwise.

## Fetch the settings from Azure

We learned in chapter [2.1 Local Function App](./1-1-local-function-app.md#the-localsettingsjson) that our local Function App settings are stored in a file called _local.settings.json_.
When we created our local Function App, the file was added with the minimal set of entries the runtime needs to work properly.

Take a look at your local settings first:

```shell
func settings list
```

And compare it to the settings in Azure:

```shell
az functionapp config appsettings list --resource-group rg-functions-demo --name <APP_NAME>
```

As you can see, there are already more settings in Azure, even if we just created the app and deployed it once.
For example, the settings for [Application Insights](./1-3-azure-function-app.md#creating-the-demo-resources) were automatically set when we created the resources via Bicep.

To fetch the settings from Azure we can execute:

```shell
func azure functionapp fetch-app-settings <APP_NAME>
```

After executing the command, the _local.settings.json_ is updated and matches the settings in Azure.
If the file is, for whatever reason, not found on your machine, you can still execute the command and the file will be created for you.

## Encrypting and decrypting

App settings and connection strings are stored encrypted in Azure.
They're decrypted only before being injected into your app's process memory when the app starts.

The function runtime offers the possibility to encrypt and decrypt your local settings as well, and as these settings contain sensitive information (connection strings, secrets and so on) the simple advice is to encrypt them and only decrypt them if needed.

However, secrets not needed by Azure to be stored as application settings should rather be stored in [Azure KeyVault](https://learn.microsoft.com/en-us/azure/key-vault/general/) instead of the application settings.

Encrypt the settings with:

```shell
func settings encrypt
```

Decrypt the settings with:

```shell
func settings decrypt
```

## Use app settings in functions

Application settings can be accessed in functions like every other environment variable and are accessible in every function in the same Function App.
Let's add a new function that returns all environment variables when it's called.

Add the function (use authlevel `function`):

```shell
func new --name settings --authlevel function --template "HTTP Trigger"
```

Update the code to return all environment variables (snippet):

```typescript
const httpTrigger: AzureFunction = async function (
  context: Context,
  req: HttpRequest
): Promise<void> {
  context.res = {
    // this will return all environment variables
    body: process.env,
  };
};
```

Now start the Function App and call the settings endpoint.
Interestingly you'll see not just the settings from the _local.settings.json_ but also quite a lot that the runtime added by itself.
Don't use environment variables not configured in _local.settings.json_ in your function, however, because they aren't guaranteed.

> Note: As application settings contain sensitive information you usually wouldn't return them like we did here and just use what's necessary where it's necessary in your functions.

### <span class="task">🛠 TASK (optional):</span> Deploy the Function App

Deploy the function app like you did in the [previous chapter](./1-4-deploying-local-to-azure.md#deploying-the-function-app) and call the new settings endpoint.
You'll see that you won't get any output, but instead a [401 Unauthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401) response.
That's because we configured the authorization level to be _function_.

We'll learn how to call that function in a later chapter.
For now we can just leave it like that as we made sure nobody besides us can call it.

> Note: If you missed to create the function with authorization level set to `function`, just update the `authLevel` to `function` in the _function.json_ and redeploy the Function App.

## <span class="quiz">Quiz</span>

<details>
  <summary>Can the functions still be started if you delete the <span class="italic">local.settings.json?</span></summary>

The Function App can still be executed, even though you'll get a warning:

```
Can't determine project language from files. Please use one of [--csharp, --javascript, --typescript, --java, --python, --powershell, --custom]
```

Using the specific language parameter the warning disappears:

```shell
npx tsc && func start --typescript
```

But only the basic environment variables will be available like that.
Refetch them from Azure before you continue.

</details>
<br/>
<details>
  <summary>What alternate secrets store should you consider for sensitive information instead of application settings?</summary>

[Azure KeyVault](https://learn.microsoft.com/en-us/azure/key-vault/general/) is a secure alternative if your secrets are not required to be stored as application settings by Azure.

</details>
