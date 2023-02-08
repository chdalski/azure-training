# Deploying your App to Azure

## Introduction

Now that we've created our Azure Resources, it's time to deploy our local Function App to its Azure counterpart.

Commands in this chapter are to be executed in the _Function App_ root directory unless stated otherwise.

## Deployment best practices

When deploying a Function App, it's important to keep in mind that the unit of deployment for functions in Azure is the Function App.
Meaning, all functions are deployed at the same time, usually from the same [deployment package](https://learn.microsoft.com/en-us/azure/azure-functions/run-functions-from-deployment-package).

Check out the [best practices documentation](https://learn.microsoft.com/en-us/azure/azure-functions/functions-best-practices?tabs=csharp#optimize-deployments) for further insights.

Also, think about if you have added new items to your Function App and if you need to update your [.funcignore](./21-local-function-app.md#the-funcignore) before deploying.

## Deploying the Function App

Our Function App can be deployed with a single command.
However, before we can execute it, we need to know the name of our Function App created by the Bicep template.

Look up the name:

```shell
az resource list --resource-group rg-functions-demo --query "[?kind=='functionapp,linux'].name"
```

Afterwards deploy your Function App with:

```shell
func azure functionapp publish <APP_NAME>
```

<details>
  <summary>Sample output</summary>

```
Getting site publishing info...
Creating archive for current directory...
Uploading 427.34 KB [#############################################################################]
Upload completed successfully.
Deployment completed successfully.
```

</details>

> Note: This might take a minute or two depending on your compute resources and internet connection.

### Testing your deployment

After our deployment completed successfully, we'll now test if everything works as expected.

Query the enabled host names for your Function App with:

```shell
az functionapp show --resource-group rg-functions-demo --name <APP_NAME> --query "defaultHostName"
```

As Azure Websites get a default custom Domain name, where the application name is the third level of the domain name followed by `.azurewebsites.net`.
That is also true for Azure Functions as they are treated as Websites.
Furthermore the domain has a valid wildcard certificate, so we can call our resources via _https_.

From our local tests we also know that our functions are located in the `api` subdirectory followed by the function name.

So, your URL should look like:

```
https://<APP_NAME>.azurewebsites.net/api/greetings
```

> Note: We don't need to specify a port, because the site listens on the default https port.

#### <span class="task">🛠 TASK (optional):</span> Call the greetings API

Use the curl commands introduced in the chapter [Local Functions](./21-local-function-app.md) or any other REST client to test the API.

<details>
  <summary>💡 HINT</summary>

- Look up the function name, as done above, and replace `<function-app-name>` in the url with it
- Make sure to use **https** instead of _http_ for non-local calls

</details>
