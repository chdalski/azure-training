# Working with Logs

## Introduction

When it comes to logging, sadly an often undervalued topic, Azure Function Apps have quite a broad scope of different mechanisms available, including [Streaming Logs](https://learn.microsoft.com/en-us/azure/azure-functions/functions-monitoring#streaming-logs), [Diagnostic Logs](https://learn.microsoft.com/en-us/azure/azure-functions/functions-monitoring#diagnostic-logs), [Scale controller logs](https://learn.microsoft.com/en-us/azure/azure-functions/functions-monitoring#scale-controller-logs) and [Azure Monitor metrics](https://learn.microsoft.com/en-us/azure/azure-functions/functions-monitoring#azure-monitor-metrics).

In the last chapter we were already using Azure Monitor metrics to query information about the health check status.

In this chapter, we're going to focus solely on the basics of _Streaming Logs_, which come in two flavours:

- **Built-in log streaming** lets you view a stream of your _application log_ files. This stream is equivalent to the output seen when you debug your functions during local development. This streaming method supports only a single instance and can't be used with an app running on Linux in a Consumption plan.
- **Live Metrics streaming** lets you view log data and other metrics in near real time when your Function App is connected to Application Insights.

We're also going to look into how to run only specific functions, disabling functions and so on.

Commands in this chapter are to be executed in the _Function App_ root directory unless stated otherwise.

## Look under the hood

Before we take a look at the logs, we take a little detour and talk a little about trace levels and log levels, learn the difference and write a little function to write some logs.

### Trace levels

_Trace levels_ define what kind of message we send to our logs stream.

One of the reasons why we're focusing solely on the basics of streaming logs is because trace levels in Azure Functions depend on your [language of choice](https://learn.microsoft.com/en-us/azure/azure-functions/functions-monitoring#writing-to-logs).

Nevertheless, the following four basic levels are available in most languages:

1. **Information**: should be purely informative and not looking into them on a regular basis shouldn’t result in missing any important information. These logs should have long-term value.
2. **Warning**: should be used when something unexpected happens, but the function can continue the work
3. **Error**: should be used when the function hits an issue preventing one or more functionalities from properly executing
4. **Trace / Verbose**: should be used for events considered to be useful during software debugging when more granular information is needed

In [Typescript / Javascript](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-node#trace-levels) we can access the logger via the context as shown in the table below.

| Trace Level | Write with                                            |
| :---------- | :---------------------------------------------------- |
| Information | `context.log.info(message)` or `context.log(message)` |
| Warning     | `context.log.warn(message)`                           |
| Error       | `context.log.error(message)`                          |
| Verbose     | `context.log.verbose(message)`                        |

We've already used `context.log` and if you took a closer look to the logs from the runtime process you should have already seen some of these messages.

> Note: Don't mistake `console.log` for `context.log`.
> Because output from `console.log` is captured at the function app level, it's not tied to a specific function invocation and isn't displayed in a specific function's logs.

### Log Levels

[Log levels](https://learn.microsoft.com/en-us/azure/azure-functions/configure-monitoring#configure-log-levels) define what kind of messages we want to capture in our logs.

While there are only four _trace levels_, there are seven _log levels_.
That's because the _trace levels_ are language-specific, whereas the _log levels_ are runtime-specific.
The Function App runtime is written in .NET, which knows all seven levels for logs and traces.

The six log levels are _Trace_, _Debug_, _Information_, _Warning_, _Error_, _Critical_, _None_, whereas _Trace_ is the highest level and _None_, of course, the lowest.
The level you choose includes all lower levels, except for _None_ which will deactivate logging completely.
So, if you choose _Warning_, for example, you would also include _Error_ and _Critical_ log messages.

Log levels can be configured for different [categories](https://learn.microsoft.com/en-us/azure/azure-functions/configure-monitoring#configure-categories).
The important ones for us right now are only _Function_ and _default_, though.
_Function_ specifies the log level for your functions and _default_ for all categories not configured otherwise.

The log levels can be configured in the _host.json_, so on the scale of the whole Function App.
Nevertheless, we can specify the _log level_ per function and will do so in a later task.

<details>
  <summary>Sample <span class="italic">host.json</span></summary>

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "excludedTypes": "Request"
      }
    },
    "logLevel": {
      "default": "Information",
      "Host.Results": "Warning",
      "Function": "Trace",
      "Host.Aggregator": "Warning"
    }
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.0.0, 5.0.0)"
  }
}
```

</details>

As already mentioned, log levels are runtime-specific, so changing the log level (for functions) will impact all your functions.
We'll learn how to work around that later in this chapter.

Let's add the _log levels_ to the table

| Trace Level | Write with                                            | Log Level   |
| :---------- | :---------------------------------------------------- | :---------- |
| Information | `context.log.info(message)` or `context.log(message)` | Information |
| Warning     | `context.log.warn(message)`                           | Warning     |
| Error       | `context.log.error(message)`                          | Error       |
| Verbose     | `context.log.verbose(message)`                        | Trace       |

The only surprise here is that _Verbose_ correlates to _Trace_, not to _Debug_ as one might have expected.

> Note: The _host.json_ is included when deploying the Function App, so remember to reset your settings before you deploy it.
> Or, as we do later in this chapter, overwrite the _host.json_ settings in your _local.settings.json_ as described [here](https://learn.microsoft.com/en-us/azure/azure-functions/functions-host-json#override-hostjson-values).

## Creating a log function

Now, we want to add a function which automatically writes log data once in a while.

### <span class="task">🛠 TASK:</span> Create a Timer Trigger function

Let's add another function to write some log messages.
But this time we'll create a [Timer Trigger](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?pivots=programming-language-javascript) function rather then an HTTP Trigger.

A _Timer Trigger_ is triggered at scheduled times.
The schedule is defined via a [Cron Expressions](https://en.wikipedia.org/wiki/Cron#Cron_expression) and the expression is configured in the _function.json_.

If you need a verbal interpretation of a cron expression, you can use a [cron expression generator](https://crontab.cronhub.io/) website.

<details>
  <summary>Sample function.json</summary>

```json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 */5 * * * *"
    }
  ],
  "scriptFile": "../dist/logs/index.js"
}
```

</details>
<br/>
<details>
  <summary>🎓 SOLUTION</summary>

The function can be easily created using the _Timer Trigger_ template.
The difference this time is, that we don't need to specify the `authlevel`, because the function is not called by an external source.

Create the function with:

```shell
func new --name logs --template "Timer trigger"
```

</details>

### <span class="task">🛠 TASK:</span> Update the function code

Next, we want to update the function code to write a log message for each trace level.

<details>
  <summary>🎓 SOLUTION</summary>

Update the code (snippet):

```typescript
const timerTrigger: AzureFunction = async function (
  context: Context,
  myTimer: any
): Promise<void> {
  const functionName = context.executionContext.functionName;

  context.log.verbose(`Timer trigger: ${JSON.stringify(myTimer)}`);

  context.log.info(`=> Information from function "${functionName}"`);
  context.log.warn(`=> Warning from function "${functionName}"`);
  context.log.error(`=> Error from function "${functionName}"`);
};
```

</details>

### <span class="task">🛠 TASK:</span> Configure which functions to run

The task is to start the Function App, but let it run the _logs_ function only.

There're two possibilities to do so, you could either [disable](https://learn.microsoft.com/en-us/azure/azure-functions/disable-function#localsettingsjson) all other functions in the _local.settings.json_, or you could define which functions [to run](https://learn.microsoft.com/en-us/azure/azure-functions/functions-host-json#functions) in the _host.json_.
So it's either a [denylist](https://en.wikipedia.org/wiki/Whitelist) or an [allowlist](<https://en.wikipedia.org/wiki/Blacklist_(computing)>).

We want to use a allowlist, but we want to run just some local tests.
So we don't want to change the _host.json_ file because it's deployed to Azure.
Therefore, we need a possibility to [overwrite](https://learn.microsoft.com/en-us/azure/azure-functions/functions-host-json#override-hostjson-values) _host.json_ configuration locally.

Check out the [documentation](https://learn.microsoft.com/en-us/azure/azure-functions/functions-host-json) and create an allowlist, including only the _logs_ function.

Start the function, after changing the settings, to see the log messages.
Depending on how patient, or rather how impatient you are, you might also want to update the cron schedule to run the function more often, or take a little break.

<details>
  <summary>💡 HINT</summary>

- You need overwrite host settings for [functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-host-json#functions)
- Overwritten host settings always start with `AzureFunctionsJobHost`
- Next you need to follow the `json` path to the specific setting and replace every opening curly bracket (`{`) with tow underscores (`__`)
- The settings for `function` is an array and arrays are are numbered starting with _zero_ (`0`)

</details>
<br/>
<details>
  <summary>🎓 SOLUTION</summary>

Update the _local.settings.json_ with:

```shell
func settings add "AzureFunctionsJobHost__functions__0" "logs"
```

For the sake of completion, disabling functions can be done with:

```shell
# Noteworthy is the difference on the first level.
# Function settings are overwritten starting with "AzureWebJobs"."FunctionName"...
func settings add "AzureWebJobs.greetings.Disabled" "true"
```

</details>

> Note: Some types of logging buffer write to the log file, which can result in out of order events in the stream.

### <span class="task">🛠 TASK:</span> Update the log level

As you've seen in the previous task, the logs don't contain the verbose messages yet.

Update the _log level_ to trace for our `logs` function, but not for any other function.

<details>
  <summary>💡 HINT</summary>

- You need overwrite host settings for [logging](https://learn.microsoft.com/en-us/azure/azure-functions/functions-host-json#logging)
- See the answer of the previous task for further details how to do so

</details>
<br/>
<details>
  <summary>🎓 SOLUTION</summary>

Update the _host.json_ (snippet):

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      ...
    },
    "logLevel": {
      "Function.logs": "Trace",
    }
  },
  "extensionBundle": {
    ...
  }
}
```

For the sake of completion, updating the _local.settings.json_ could be done with:

```shell
func settings add "AzureFunctionsJobHost__logging__LogLevel__Function.logs" "Trace"
```

</details>

## Streaming logs

At long last, our logs function automatically writes some logs we can work with.
Let's get into it!

### Built-in log streaming

Built-in log streaming lets you view a stream of your _application log_ files.
This is not yet supported for Linux apps in the Consumption plan.

However, that's not an issue for us, because we're luckily using an [App Service plan](https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans) to run our functions.

Use the [logstream command](https://learn.microsoft.com/en-us/azure/azure-functions/functions-core-tools-reference#func-azure-functionapp-logstream) to show your Function App logs on the command line:

```shell
func azure functionapp logstream <APP_NAME>
```

Wait for a moment, until you get the message:

```
Starting Live Log Stream ---
```

Finally, publish your Function App to Azure.

You'll see some messages like:

```
[INFO]  Starting OpenBSD Secure Shell server: sshd.
[INFO]  Hosting environment: Production
[INFO]  Content root path: /azure-functions-host
[INFO]  Now listening on: http://[::]:80
[INFO]  Application started. Press Ctrl+C to shut down.
No new trace in the past 1 min(s).
```

These logs only show messages written by the Function App itself, but not for a specific function.

So let's stop the execution with `Ctrl+c` and move on.

### Live Metrics streaming

[Live Metrics streaming](https://learn.microsoft.com/en-us/azure/azure-monitor/app/live-stream) lets you view log data and other metrics in near real time.
However, it only works for Function Apps connected to Application Insights.
And, as you might have guessed, we already did that.

You can start the live metrics view with:

```shell
func azure functionapp logstream <APP_NAME> --browser
```

The command will open the the live metrics view for your function app in your browser and you can see your incoming requests as well as the messages from our `logs` function.

> Note: The browser view might fail with [Data is temporarily inaccessible](https://learn.microsoft.com/en-us/azure/azure-monitor/app/live-stream#data-is-temporarily-inaccessible-status-message).
> Try to deactivate your ad blocker, cookie blocker and so on for `portal.azure.com` and it should work as expected.

### Query Application Insights

As final step in this chapter we want to query our log messages from [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview).

You can use either the [query](https://learn.microsoft.com/en-us/cli/azure/monitor/app-insights#az-monitor-app-insights-query) command, or the [events show](https://learn.microsoft.com/en-us/cli/azure/monitor/app-insights/events#az-monitor-app-insights-events-show) command to receive messages for your instance.

Let's look at both options in detail.

> Note: New messages are not instantly displayed in Application Insights.
> You need to wait a couple of minutes for messages to be available.

#### <span class="task">🛠 TASK (optional):</span> Query your instance name

You can skip this one if you didn't change the name of your Application Insights instance when you created our Azure resources.

If you changed the name, query it and replace `ains-training-demo` with your `<INSTANCE_NAME>` in the commands below.

<details>
  <summary>🎓 SOLUTION</summary>

Query the name of your Application Insights instance:

```shell
az resource list --resource-group rg-functions-demo --query "[?type=='Microsoft.Insights/components'].name"
```

</details>

#### Using query

The command uses the [Kusto Query Language (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/), which you can use in [Azure Portal](https://portal.azure.com) for Azure Monitor as well as Azure Data Explorer.

Take the latest 2 results within the last 5 minutes with:

```shell
az monitor app-insights query --resource-group rg-functions-demo --app ains-training-demo --analytics-query 'traces | sort by timestamp desc | take 2' --offset 5m
```

The output is one or more tables represented as [JSON](https://www.json.org).
It's very powerful, but the output on the command line is not optimal for further processing.

When it comes to our trace levels, you can find the messages for a specific level by querying for the specific [severityLevel](https://learn.microsoft.com/en-us/dotnet/api/microsoft.applicationinsights.datacontracts.severitylevel).

We can query our verbose messages during the last 10 minutes with:

```shell
az monitor app-insights query --resource-group rg-functions-demo --app ains-training-demo --analytics-query 'traces | where severityLevel == 0 and operation_Name has "logs" | project message, timestamp | sort by timestamp desc' --offset 10m
```

You will see some of your `Timer trigger` messages written with `context.log.verbose(message)`.
If you updated the _host.json_ to log verbose messages for the `logs` function, that is.

#### Using events show

The command can be used with the global `--query` parameter to filter the messages.
It uses a [JMESPath query string](https://jmespath.org/) to filter the messages.

List messages from the last 5 minutes with:

```shell
az monitor app-insights events show --resource-group rg-functions-demo --app ains-training-demo --type traces --offset 5m
```

We can queries our verbose messages, during the last 10 minutes with:

```shell
az monitor app-insights events show --resource-group rg-functions-demo --app ains-training-demo --type traces --offset 10m --query "value[?operation.name=='logs'].[trace, timestamp]"
```

> Note: A recommended alternative, especially if you work with more CLIs using JSON as output format, is using [jq](https://github.com/stedolan/jq) and [bat](https://github.com/sharkdp/bat).
> It's more fun with these two extraordinary tools.
