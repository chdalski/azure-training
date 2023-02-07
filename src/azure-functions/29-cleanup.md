# Cleanup

This is the last chapter for this course and without further ado, we'll just clean up the resources.

Make sure you're in the right subscription:

```shell
az account show
```

Change the subscription if needed with:

```shell
az account list
```

```shell
az account set --subscription <SUBSCRIPTION_ID>
```

Drop it like it's hot (this might take a while):

```shell
az group delete --name rg-functions-demo --yes
```

That's it, well done!

I hope you enjoyed the training.
