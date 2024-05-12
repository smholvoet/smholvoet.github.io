---
published: true
title: "Dynamically adding firewall rules to Azure App Service"
excerpt: "Dynamically adding firewall rules to Azure App Service"
date: 2024-05-12T00:00:00-04:00
show_date: true
tags:
  - Azure
  - App Service
  - networking
  - Azure DevOps
---

If you have a use case where you need to allow inbound connection from Azure DevOps Microsoft-hosted agents to your App Service, you're in for quite the ride... 🎢

## Azure DevOps networking and weekly JSON files...

Having a look at the [Inbound connections](https://learn.microsoft.com/en-us/azure/devops/organizations/security/allow-list-ip-url?view=azure-devops&tabs=IP-V4#inbound-connections) section of the Azure DevOps documentation will make you feel like you've come to the right place. Just get the IP ranges for your specific region and add those, right?

![right wrong gif](/assets/images/right-wrong-gif.gif){: .align-center}

Scroll down to find the following caveat:

> The Service Tag or previously mentioned inbound IP addresses don't apply to Microsoft Hosted agents. Customers are still required to **allow the entire geography for the Microsoft Hosted agents**. If allowing the entire geography is a concern, we recommend using the Azure Virtual Machine Scale Set agents.

Not exactly what we're looking for as:

- This pretty much means anyone (and anything 🤖) who shares the same Azure region as you could in theory access your App Service. This obviously *is* a concern, Microsoft suggests you to set up [Azure Virtual Machine Scale Set agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/scale-set-agents?view=azure-devops), which is not exactly something you'll be doing in a just a couple of minutes, let alone hours.
- If you're brave enough to ignore the obvious red flag mentioned above, you would *still* need a way to automate -something- which whitelists the correct IP ranges for your region. These IP ranges are delivered through the infamous [weekly JSON file](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#networking). At the time of writing (May 2024), this JSON monstrosity contains 118,313 lines 🙃...

## The magic 🪄

We can do better than that. Why not dynamically add the IP of our Azure DevOps agent during the actual pipeline run, using Azure CLI?

Let's first find the IP of the Azure Pipelines agent:

```bash
AGENT_IP=$(curl --silent ifconfig.me)
echo "🌍 Public IP: $AGENT_IP"
```

Once we have the IP we can then go ahead and add it to the App Service network configuration:

```bash
echo "✅ Adding public IP to App Service network configuration..."
az webapp config access-restriction add \
    --subscription <...> \
    --resource-group <...> \
    --name <...> \
    --rule-name azdo-ms-hosted-agent \
    --action Allow \
    --ip-address $AGENT_IP \
    --priority 250 \
    --scm-site true \
    --output none
```

This will add a firewall rule named `azdo-ms-hosted-agent` to the networking configuration of your App Service.

Once the deployment has successfully completed, we can then go ahead and remove the firewall rule from our network configuration:

```bash
echo "❌ Removing public IP from App Service network configuration..."
az webapp config access-restriction remove \
    --subscription <...> \
    --resource-group <...> \
    --name <...> \
    --rule-name azdo-ms-hosted-agent \
    --scm-site true \
    --output none
```

And this is what it looks like in the actual YAML pipeline (Ubuntu agent):

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: '...'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
    AGENT_IP=$(curl --silent ifconfig.me)
    echo "🌍 Public IP: $AGENT_IP"
    echo "✅ Adding public IP to App Service network configuration..."
    az webapp config access-restriction add \
        --subscription <...> \
        --resource-group <...> \
        --name <...> \
        --rule-name azdo-ms-hosted-agent \
        --action Allow \
        --ip-address $AGENT_IP \
        --priority 250 \
        --scm-site true \
        --output none
    echo "💤 Sleeping for 30 seconds..."
    sleep 30
displayName: 'Add firewall rule'

- task: AzureWebApp@1
  inputs:
    azureSubscription: '<...>'
    appName: '<...>'
    package: '$(Pipeline.Workspace)/**/*.zip'
    deploymentMethod: 'auto'
displayName: 'Deploy to App Service'

- task: AzureCLI@2
  inputs:
    azureSubscription: '<...>'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
    echo "❌ Removing public IP from App Service network configuration..."
    az webapp config access-restriction remove \
        --subscription <...> \
        --resource-group <...> \
        --name <...> \
        --rule-name azdo-ms-hosted-agent \
        --scm-site true \
        --output none
displayName: 'Remove firewall rule'
```

## Conclusion

This is what the end result looks like in our pipeline run:

![fw-rules-log](/assets/images/fw-rules-log.png)

```shell
🌍 Public IP: 108.143.112.123
✅ Adding public IP to App Service network configuration...
💤 Sleeping for 30 seconds...
...
❌ Removing public IP from App Service network configuration...
```

As you can see this adds two extra steps to our pipeline: one to add the firewall rule and one to remove said firewall rule once we've finished deploying. If you leave out the 30 second timeout which I've added to give the App Service some time to "catch up" on its networking configuration, these additional steps take less than a minute.

## References

- [az webapp config access-restriction](https://learn.microsoft.com/en-us/cli/azure/webapp/config/access-restriction)
