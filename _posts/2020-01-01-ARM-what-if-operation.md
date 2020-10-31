---
published: false
title: "ARM what-if operation"
excerpt: ""
date: 2020-10-31T00:00:00-04:00
tags:
  - todo
---

## Introduction

In order to validate your ARM templates you can make use of the [what-if](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-deploy-what-if?tabs=azure-powershell) operation. Similarly to Terraform's `terraform plan` [command](https://www.terraform.io/docs/commands/plan.html), this command will simulate any pending changes to your Azure resources. In other words, it will "predict" the outcome of your deployment *before* you actually deploy anything:

![image-center](/assets/images/what-if_1.png){: .align-center}

## Prerequisites

The example below uses Azure CLI, but you may just as well use Azure PowerShell or the REST API. To follow along with the example below you'll need Azure CLI version **2.5.0** or above:

- Install Azure CLI using [this](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) link
- Check your current version by running `az --version` and run `az upgrade` if your version is too old (< **2.5.0**)

```powershell
az --version

azure-cli                         2.12.0 *
core                              2.12.0 *
telemetry                          1.0.6

Extensions:
azure-devops                      0.17.0
interactive                        0.4.3

...

You have 2 updates available. Consider updating your CLI installation with 'az upgrade'
```

## Example

### Setup

Grab the files from my [ARM what-if](https://github.com/smholvoet/arm-what-if) GitHub repo, or use the following `git clone` command:

```powershell
git clone https://github.com/smholvoet/arm-what-if.git
```

The `az-cli.ps1` file contains all the steps to go through the different examples. I ran the examples below against my MSDN subscription, you'll need to modify the value on line 4.

```powershell
# Login & select subscription
az login
az account list --output table
az account set --subscription "<name or GUID of your Azure subscription>"
az account show
```

The commands above will authenticate to Azure and output a list of your available subscriptions. Select the correct one by running `az account set --subscription` followed by the `Name` or `SubscriptionId` of your subscription. Running `az account show` will show you in which subscription the resource group will eventually be created.

Next up, create a new resource group. This resource group will contain the resource(s) which we'll be creating via the ARM templates:

```powershell
# Create resource group
az account list-locations | findStr 'europe'
az group create --name rg-arm `
                --location westeurope
```

### ARM template

You should find 2 ARM templates:

- `azuredeploy.json`: initial template
- `azuredeploy_update.json`: updated template which contains a modified description

The templates which we'll be using are fairly simply and only contain a [storage account](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {"storageAccountName": {
        "type": "string",
        "metadata": {
            "description": "Storage account name"
        },
        "minLength": 3,
        "maxLength": 24
    }},
    "functions": [],
    "variables": {},
    "resources": [{
        "name": "[parameters('storageAccountName')]",
        "type": "Microsoft.Storage/storageAccounts",
        "apiVersion": "2019-06-01",
        "tags": {
            "displayName": "[parameters('storageAccountName')]"
        },
        "location": "[resourceGroup().location]",
        "kind": "StorageV2",
        "sku": {
            "name": "Premium_LRS",
            "tier": "Premium"
        }
    }],
    "outputs": {}
}
```

**The parameters file:**

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccountName": {
            "value": "<your storage account name>"
        }
    }
}
```

**⚠️ Warning:** Enter a value for the {% raw %}`storageAccountName`{% endraw %} in the {% raw %}`azuredeploy.parameters.json`{% endraw %} file. Make sure you enter a [valid](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/error-storage-account-name) and unique storage account name, which is between 3 and 24 characters in length and uses numbers and lower-case letters only.
{: .notice--warning}

### Create resources

Now that we've created a resource group we can start running our ARM templates.
Execute the *what-if* operation on the empty resource group which we've just created:

```powershell
az deployment group what-if --name 'initialDeploy' `
                            --resource-group rg-arm `
                            --template-file .\azuredeploy.json `
                            --parameters .\azuredeploy.parameters.json
```

**Result:**
![image-center](/assets/images/what-if_2.png)

Thanks to the output of the *what-if* operation we can now safely deploy the initial template:

```powershell
az deployment group create  --name 'initialDeploy' `
                            --resource-group rg-arm `
                            --template-file .\azuredeploy.json `
                            --parameters .\azuredeploy.parameters.json
```

### Mofidy resources

Execute what-if operation again, using the updated template this time. Notice how the `--template-file .\azuredeploy_update.json` parameter points to a different template this time. This template is exactly the same as the one we used earlier, except for the `displayName` property:

```json
...
        "tags": {
            "displayName": "[parameters('storageAccountName')]"
        },
...
```

```json
...
        "tags": {
            "displayName": "sttest01randomname-update"
        },
...
```

Go ahead and run the *what-if* operation. If all goes well this should now reflect the updated `displayName` property.

```powershell
az deployment group what-if --name 'updateStorageSKU' `
                            --resource-group rg-arm `
                            --template-file .\azuredeploy_update.json `
                            --parameters .\azuredeploy.parameters.json
```

**Result:**

![image-center](/assets/images/what-if_3.png)

Looking good 👌. Should we deploy this template, only the `displayName` property will be changed, all other properties of our storage account will remain unchanged.

The steps above showed you how to run the what-if operation, which allows you to simulate pending changes to your ARM template(s). Things get more interesting if you can apply these steps in your CI/CD pipeline(s), have a look at my [test](www.google.com) blog post.
