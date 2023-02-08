# Basic Function Security

## Introduction

The aim of this chapter is to give you an understanding of the most fundamental security mechanisms of Azure Functions, or more specific, on how to use [Function Access Keys](https://learn.microsoft.com/en-us/azure/azure-functions/security-concepts#function-access-keys).

Azure Function security in general is a much broader topic, which we can't possibly explain in its entirety in this training.
That said, I strongly encourage you to read the whole documentation of Azure Functions [security concepts](https://learn.microsoft.com/en-us/azure/azure-functions/security-concepts) after completing this chapter.

Commands in this chapter are to be executed in the _Function App_ root directory unless stated otherwise.

## Function Access Keys

As the name implies, Function Access Keys can be used to secure your functions in a manner where your requests must include an API access key in the request.
Unless the HTTP access level on an HTTP triggered function is set to anonymous, that is.

They're good enough for trainings, demos, or development purposes but shouldn't be used in [production](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook-trigger#secure-an-http-endpoint-in-production).

Nevertheless they are the default security mechanisms Azure Functions provide and that's why we're learning about them.
In fact, we already used Function Access keys back in [Chapter 2.5](./25-working-with-app-settings.md#use-app-settings-in-functions), were we created a function with authorization level `function`, and they're also the reason why we created most of our functions with authorization level `anonymous`.

So, as you might have already guessed, authorization levels are coupled to access keys.

Possible authorization levels are:

- `anonymous`: functions can be called without providing an API access key.
- `function`: functions can be called providing an API access key of scope _Function_ or _Host_.
- `admin`: functions can be called providing an API access key of scope _Admin_.

Possible key scopes are:

- **Function**: These keys apply only to the specific functions under which they're defined.
- **Host**: These keys apply to all functions within the Function App (host-level).
- **Admin**: Each Function App also has an admin-level host key. In addition to providing host-level access to all functions in the app, the master key also provides administrative access to the runtime REST APIs.
- **System**: These keys are required by certain function extensions and the scope of system keys is determined by the extension, but it generally applies to the entire function app (host-level).

As a developer you'll most likely use authorization level `function` with either key scope `Function` or `Host`.

## List Function App keys

Let's take a look at the keys our Function App provides:

```shell
az functionapp keys list --resource-group rg-functions-demo --name <APP_NAME>
```

<detail>
    <summary>Sample output</summary>

```json
{
  "functionKeys": {
    "default": "<some-default-key>"
  },
  "masterKey": "<some-master-key>",
  "systemKeys": {}
}
```

</detail>

As you can see, there's only one function key and the master key so far.

## Calling functions using keys

Let's call our `settings` function with using anonymous and the different keys.

Anonymous:

```shell
curl 'https://<APP_NAME>.azurewebsites.net/api/settings' -v
```

Using a key:

```shell
curl 'https://<APP_NAME>.azurewebsites.net/api/settings?code=<KEY>'
```

Using a anonymous call, the function response is `401 Unauthorized`, whereas it works as expected if we provide the key.

### <span class="task">🛠 TASK (optional):</span> Verify the admin key access

Update the _function.json_ of your `settings` function, and set the authorization level to `admin` instead of `function`.

Redeploy the Function App and try using the `function` key and the `admin` key for the call.

What're the results?

<details>
  <summary>🎓 SOLUTION</summary>

Update the authLevel in the _function.json_ (snippet):

```json
{
  "bindings": [
    {
      "authLevel": "Admin",
      "type": "httpTrigger",
      "direction": "in",
      ...
    }
    ...
  ]
}
```

Redeploy with:

```shell
func azure functionapp publish <APP_NAME>
```

Summary: The function key, doesn't work anymore, whereas the admin key works as expected.

</details>

## Function-specific keys

So far, we've used a host key or the admin key, where both can access all APIs of our Function App.
But, as we've learned above, we can also create keys for specific functions.

Let's create one for our `greetings` function:

```shell
az functionapp function keys set --resource-group rg-functions-demo  --function-name greetings --key-name GreetingsKey --name <APP_NAME>
```

<details>
    <summary>Sample output</summary>

Output (snippet):

```json
{
  ...
  "name": "GreetingsKey",
  "resourceGroup": "rg-functions-demo",
  "type": "Microsoft.Web/sites/functions/keys",
  "value": "<your-new-key>"
}
```

</details>

If we list our keys again, we won't find the key we just created, because it's function-specific.

We need to use the function specific [command](https://learn.microsoft.com/en-us/cli/azure/functionapp/function/keys#az-functionapp-function-keys-list) to list the keys for a function.

```shell
az functionapp function keys list --resource-group rg-functions-demo --function-name greetings --name <APP_NAME>
```

> Note: We use `az functionapp keys list ...` for the Function App, but `az functionapp function keys list ...` for a specific function.
> The keys using the first command have the _Host_ scope, the keys using the second command have the _Function_ scope.

### <span class="task">🛠 TASK (optional):</span> Verify the function scope access

This task contains of three easy questions.

1. Using the function key, we just created, can you access the `greetings` function with it?
2. Using the function key, we just created, can you access the `settings` function with it?
3. Can you still access the `greetings` function with the function key if you set its authorization level to `admin`?

<details>
  <summary>🎓 SOLUTION</summary>

**Question 1**: Calling the greetings function with the key it works as expected and results in a 200 OK.
The function should be anonymous right now, so it doesn't care about the key at all.

**Question 2**: As one would expect it results in 401 Unauthorized.
Again nothing unexpected because the key is not valid for the settings function.

**Question 3**: This results in 401 Unauthorized because a function with authorization level admin can only be called using the master key.

</details>
