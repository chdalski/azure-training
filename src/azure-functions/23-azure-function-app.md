# Azure Function App

## Introduction

After successfully creating a local Function App and get our first function up and running it is now time to create a Function App in Azure.

The Azure resources will be created using a [Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) template (you don't need to install any additional software).

## Login to your Azure account

First we need to login to our Azure account with the Azure CLI.

Login the Azure CLI:

```shell
az login
```

<details>
  <summary>Sample output</summary>

```
A web browser has been opened at https://login.microsoftonline.com/organizations/oauth2/v2.0/authorize.
Please continue the login in the web browser.
If no web browser is available or if the web browser fails to open, use device code flow with `az login --use-device-code`.
Opening in existing browser session.
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "some-tenant-uuid",
    "id": "your-subscription-uuid",
    "isDefault": true,
    "managedByTenants": [],
    "name": "you@sample.com",
    "state": "Enabled",
    "tenantId": "some-tenant-uuid",
    "user": {
      "name": "you@sample.com",
      "type": "user"
    }
  }
]
```

</details>

> Note: If you have more than one subscription use `az account` (list / set) to switch to the one you want to use.

## Create a new Resource Group

A [resource group](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview#resource-groups) is a container that holds related resources for an Azure solution.

You might want to change the location for the created resources to something close to you.

Check what locations are eligible with:

```shell
az account list-locations --query "[].name"
```

Create one with:

```shell
az group create --name rg-functions-demo --location westeurope
```

> Note: It might take a while for the resources to show up in Azure Portal.

## Create the demo resources

The resources will be created in the same location you used for the resource group.
However, not all resources are available for every resource.
You can lookup which products are available in which location [here](https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/).

Download the template file from [here](./functions-demo.bicep).

Create the resources with:

```shell
az deployment group create --resource-group rg-functions-demo --template-file functions-demo.bicep
```

The following resources will be created:

- A [Function App](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) is a serverless solution that allows you to implement your system's logic into readily available blocks of code.
- A [Storage Account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview) contains all of your Azure Storage data objects, including blobs, file shares, queues, tables, and disks. In our case it's providing storage for our Function App.
- An [Application Insights instance](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) is an extension of Azure Monitor and provides Application Performance Monitoring (also known as “APM”) features. We'll use it to as log solution for our functions.
- An [App Service plan](https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans) defines a set of compute resources for a web app to run. In our case it provides the execution environment for our functions.

> Note: If you use an [alternate language](https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages) you need to update the Bicep template accordingly.
