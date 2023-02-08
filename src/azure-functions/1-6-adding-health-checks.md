# Adding health checks

## Introduction

In this chapter we're going to add basic health check functionality to our Function App, which you can use to implement [Azure Service Health alerts](https://learn.microsoft.com/en-us/azure/service-health/resource-health-overview) (not part of this training).

The hosting infrastructure for Azure Functions is provided by [Azure App Service](https://learn.microsoft.com/en-us/azure/app-service/overview).
Therefore the documentation for [health checks](https://learn.microsoft.com/en-us/azure/app-service/monitor-instances-health-check) is found in the Azure App Service documentation rather then for Azure Functions.

The Health Check feature can be used to monitor Function Apps on the Premium (Elastic Premium) and Dedicated (App Service) plans only.
It's not an option for the Consumption plan, however, as the runtime for these Function Apps is only available when functions are called.

Commands in this chapter are to be executed in the _Function App_ root directory unless stated otherwise.

## Creating a health check endpoint

Reading the [documentation](https://learn.microsoft.com/en-us/azure/app-service/monitor-instances-health-check#what-app-service-does-with-health-checks), we learn that creating an health check endpoint is rather simple.

- our Function App needs an endpoint that returns HTTP status code 200 if everything is fine and 500 otherwise
- the default path for that endpoint is called _health_

We also learn that health checks are usually used as an interface to probe other services our Function App depends on, like databases and so on.
As we have no database available we need to substitute it.

But let's take one step at a time.

### <span class="task">🛠 TASK:</span> Create the endpoint

As you have done before, create a new function called `health` with authorization level `anonymous`.

<details>
  <summary>💡 HINT</summary>

We already create a function like that [before](./1-2-local-function.md#add-a-function-to-our-function-app).

</details>
<br/>
<details>
  <summary>🎓 SOLUTION</summary>

```shell
func new --name health --authlevel anonymous --template "HTTP Trigger"
```

</details>

### <span class="task">🛠 TASK:</span> Update the implementation

As of now we don't have a bad path option, so let's just implement the [happy path](https://en.wikipedia.org/wiki/Happy_path) first.
We want to return HTTP status code 200.

<details>
  <summary>🎓 SOLUTION</summary>

Update the code (snippet):

```typescript
const httpTrigger: AzureFunction = async function (
  context: Context,
  req: HttpRequest
): Promise<void> {
  context.res = {
    status: 200,
  };
};
```

</details>

### <span class="task">🛠 TASK:</span> Deploy the Function App

Let's deploy the Function App before we continue.

<details>
  <summary>🎓 SOLUTION</summary>

Look up the name:

```shell
az resource list --resource-group rg-functions-demo --query "[?kind=='functionapp,linux'].name"
```

Afterwards deploy your Function App with:

```shell
func azure functionapp publish <APP_NAME>
```

</details>

### <span class="task">🛠 TASK:</span> Update the health check settings

Azure Functions won't automatically probe the health endpoint just because there's an implementation.
So we need to update our Function App in order to make use of it.

Update the health check settings with:

```shell
az webapp config set --resource-group rg-functions-demo --name <APP_NAME> --generic-configurations '{"healthCheckPath": "/api/health/"}'
```

> Note: In a real world scenario we wouldn't execute the command like that.
> Instead we would update our IaC template (bicep, terraform, you name it).
> However, you might have changed your Function App because you tested something or you're using an alternate language and updated the template and we might end up breaking your Function App if we would use an updated template – that's why we don't.

### <span class="task">🛠 TASK (optional):</span> Monitor the health check status

Before you can start monitoring the health check status of your Function App, you should take a little break, because it'll take a while before any metric is available.

So wait for at least 5 Minutes before you execute:

```shell
az monitor metrics list --resource-group rg-functions-demo --resource-type "Microsoft.Web/sites" --metric "HealthCheckStatus" --interval 5m --output table --resource <APP_NAME>
```

<details>
  <summary>Sample output</summary>

The Average column might be empty if no metric was recorded before.

```
Timestamp            Name                 Average
-------------------  -------------------  ---------
2023-01-31 11:17:00  Health check status
2023-01-31 11:22:00  Health check status
2023-01-31 11:27:00  Health check status
2023-01-31 11:32:00  Health check status
2023-01-31 11:37:00  Health check status
2023-01-31 11:42:00  Health check status
2023-01-31 11:47:00  Health check status
2023-01-31 11:52:00  Health check status
2023-01-31 11:57:00  Health check status
2023-01-31 12:02:00  Health check status  33.333333333333336
2023-01-31 12:07:00  Health check status  100.0
2023-01-31 12:12:00  Health check status  100.0
```

</details>

> Note: The output from the monitor command depends on the interval and the execution time.
> So depending on what interval you choose and what time you execute it you might get different results.
> If you want results based on a specific start time, you can set the `--start-time` parameter.
> Take a look at the [az monitor metrics](https://learn.microsoft.com/en-us/cli/azure/monitor/metrics?view=azure-cli-latest#az-monitor-metrics-list) documentation for further options.

### <span class="task">🛠 TASK:</span> Prepare the unhappy path

As already mentioned, we don't have any database or other alternate service available which we could use to test our health check against.

So we're going to implement a function that allows us to toggle an environment variable to either true or false.
Afterwards we can use that environment variable in our health check to toggle its state.

Create a function called `toggles` and implement it with the following traits:

- the API expects either the _query_ or the _body_ to contain a variable called `toggle`
- create a switch for that toggle
- the switch either knows the toggle and toggles its value or it returns an HTTP status code 400
- the toggle for the health state is called `isHealthy` and will toggle an environment variable called `TOGGLE_IS_HEALTHY`
- valid values for the environment variable are `"true"` or `"false"`
- if the environment variable is not undefined, default to `"false"`
- respond with HTTP status code 200 and a message which value was toggled

<details>
  <summary>🎓 SOLUTION</summary>

Create the function with:

```shell
func new --name toggles --authlevel anonymous --template "HTTP Trigger"
```

Update the code (snippet):

```typescript
const httpTrigger: AzureFunction = async function (
  context: Context,
  req: HttpRequest
): Promise<void> {
  const toggle = req.query.toggle || (req.body && req.body.toggle);
  var responseStatus = 200;
  var responseMessage = "";

  context.log(`Toggling: ${toggle}`);

  switch (toggle) {
    case "isHealthy":
      if (process.env.TOGGLE_IS_HEALTHY == "false") {
        process.env.TOGGLE_IS_HEALTHY = "true";
      } else {
        process.env.TOGGLE_IS_HEALTHY = "false";
      }
      responseMessage = `Toggled ${toggle} to ${process.env.TOGGLE_IS_HEALTHY}`;
      break;
    default:
      context.log.error(`Unknown toggle: ${toggle}`);
      responseStatus = 400;
      break;
  }

  context.res = {
    status: responseStatus,
    body: responseMessage,
  };
};
```

</details>

### <span class="task">🛠 TASK:</span> Update the health check function

Now that we can toggle our environment variable using a simple API called, it's time to use that variable in our health check function.

Update the function to respond with HTTP status code 500 if `process.env.TOGGLE_IS_HEALTHY` is `"false"`, otherwise respond with HTTP status code 200.

<details>
  <summary>🎓 SOLUTION</summary>

Update the code (snippet):

```typescript
const httpTrigger: AzureFunction = async function (
  context: Context,
  req: HttpRequest
): Promise<void> {
  const responseStatus = process.env.TOGGLE_IS_HEALTHY == "false" ? 500 : 200;
  context.res = {
    status: responseStatus,
  };
};
```

</details>

### <span class="task">🛠 TASK (optional):</span> Deploy, Toggle and Monitor the health status

You can now deploy your Function App as well as toggle and monitor the health status.
Remember, though, it might take a while till the status is available.

You can also use [Azure Service Health alerts](https://learn.microsoft.com/en-us/azure/service-health/resource-health-overview) to monitor your Function App in Azure Portal.
As this is not part of our training, remember to clean up afterwards.

> Note: Toggle the health state back to `"true"` when you're done or Azure will try to restart your Function after a while.

## <span class="quiz">Quiz</span>

<details>
  <summary>What's the application setting <span class="italic">WEBSITE_HEALTHCHECK_MAXPINGFAILURES</span> used for and what's its default value (hint: App Service documentation)?</summary>

The default value is **10** and it's used to determine how many failed requests to the health check endpoint are valid, before the service is deemed unhealthy.

</details>
