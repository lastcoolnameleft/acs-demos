# Create a resource group

Here we will create an ACR instance

```
az acr create -g $SIMDEM_RESOURCE_GROUP -n $SIMDEM_ACR_NAME --sku Managed_Standard --admin-enabled
```

# validation

```
az acr show -g $SIMDEM_RESOURCE_GROUP -n $SIMDEM_ACR_NAME -o table
```

Results:

```
NAME        RESOURCE GROUP    LOCATION       SKU               LOGIN SERVER           CREATION DATE                     ADMIN ENABLED
----------  ----------------  -------------  ----------------  ---------------------  --------------------------------  ---------------
sandboxacr  sandbox-acr       eastus         Managed_Standard  sandboxacr.azurecr.io  2017-08-10T02:16:10.147879+00:00  True
```
