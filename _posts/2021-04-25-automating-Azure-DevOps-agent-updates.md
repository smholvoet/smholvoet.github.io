---
published: true
title: "Automating Azure DevOps agent updates"
excerpt: "Automating Azure DevOps agent updates"
date: 2021-04-25T00:00:00-04:00
show_date: true
tags:
  - Azure DevOps
  - automation
  - updates
  - REST
  - API
---

## Introduction

As you might know, Azure Pipelines agents come in 2 flavors: [Microsoft-hosted](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser#microsoft-hosted-agents) and [self-hosted](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser#install) ones. Both types come with their own (dis)advantages and will be more applicable in certain use cases, but one of the main advantages of the Microsoft-hosted agents comes down to:

> ...  
> With Microsoft-hosted agents, **maintenance and upgrades are taken care of for you**. Each time you run a pipeline, you get a fresh virtual machine.  
> ...

Besides running a whole host of different tools, runtimes, package managers et cetera  (e.g. [`windows-latest`](https://github.com/actions/virtual-environments/blob/main/images/win/Windows2019-Readme.md))
which are maintained in the [actions/virtual-environments](https://github.com/actions/virtual-environments) repo, all agents will also run the actual [Azure Pipelines 🚀](https://github.com/microsoft/azure-pipelines-agent) agent.
No matter whether you're using the Microsoft-agents, or choose to run a self-hosted agent as a service or an interactive process, it's *this* piece of software which allows you to run build or release jobs on a self-hosted machine.

## Agent version update policy

This part is actually pretty well documented, see 👉 [Agent version and upgrades](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser#agent-version-and-upgrades).

In summary:

- **Microsoft-hosted agents** are kept up-to-date automatically. They will always run the latest stable version as the newer agent version just gets added to the [VM images](https://github.com/actions/virtual-environments/blob/main/docs/create-image-and-azure-resources.md) which are used to provision new agents.
- **Self-hosted agents** on the other hand are *not* kept up-to-date automatically,  but can be updated through the [web UI](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser#to-update-self-hosted-agents). According to the docs, an agent should automatically updates itself when it runs a task that requires a newer version of the agent.

The agent software receives [updates](https://github.com/microsoft/azure-pipelines-agent/releases) every few weeks. If you'd like to get the latest version which contains the latest features and bug fixes, you would have to manually update all of your self-hosted agent pools...

![aint-nobody-got-time-for-that](/assets/images/aint-nobody-got-time-for-that.jpeg){: .align-center}

...so let's 🤖 automate it!

## Automatic updates using PowerShell

Below is a fairly simple YAML pipeline:

```yaml
trigger:
- none

schedules:
- cron: "0 2 * * 4"
  displayName: Weekly check for new Azure Pipelines release
  branches:
    include:
    - master

pool:
  vmImage: windows-latest

steps:
- task: PowerShell@2
  displayName: 'run Update-Agents.ps1'
  inputs:
    filePath: 'scripts/Update-Agents.ps1'
  env:
    PAT_SECRET: $(PAT)
```

The pipeline is scheduled to run once a week (every Thursday @ 2 AM in this case) and contains just a single task: running a [`Update-Agents`](https://github.com/smholvoet/azure-devops-toolbox/blob/main/src/agent-updates/Update-Agents.ps1) PowerShell script where the actual updates are triggered.
It  consumes a single environment variable `PAT`, containing the personal access token which is needed to authenticate.

![yaml-PAT-secret](/assets/images/yaml-PAT-secret.png)

**⚠️ Info:** In theory you could leave out this "custom" variable and simply refer to `$env:SYSTEM_ACCESSTOKEN` which is one of the many [predefined system variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml#systemaccesstoken) that holds the security token used by the build agent. However, I kept running into authentication issues upon running the script, so I opted to use a dedicated PAT for now.
{: .notice--info}

The PowerShell script makes use of the [Azure DevOps REST API](https://sanderh.dev/call-Azure-DevOps-REST-API-Postman/) as well as the [GitHub REST API](https://docs.github.com/en/rest), and goes through the following steps:

1. Get the [latest](https://github.com/microsoft/azure-pipelines-agent/releases/latest) release of the Azure Pipelines agent, using the [GitHub API](https://docs.github.com/en/rest/reference/repos#get-the-latest-release).
2. Get the list of [agent pools](https://docs.microsoft.com/en-us/rest/api/azure/devops/distributedtask/pools/get) of the current organization.
3. Loop through all [agents](https://docs.microsoft.com/en-us/rest/api/azure/devops/distributedtask/agents/get) within a specific pool and compare its current Azure Pipelines agent version with the *latest* one.
4. Trigger the update process for any agent which is running an outdated version, using the same REST call which runs in the background should you update the agent via the web UI.

![agent-updates.png](/assets/images/agent-updates.png){: .align-center}

The meat and bones of the PowerShell script are contained within the last 2 steps as you can see below.

👉 Comparing the current agent version with the *latest* one:

```powershell
$Response.value | ForEach-Object {
        if ($_.version -eq $LatestReleaseVersion)
        {
              Write-Host "$($AgentPoolName)/$($_.name) is up to date: current = $($_.version) -> latest = $LatestReleaseVersion"
        }
        else 
        {
              Write-Host "$($AgentPoolName)/$($_.name) is outdated: current = $($_.version) -> latest = $LatestReleaseVersion"
              if($_.status -eq 'offline') {
                      Write-Host "$($AgentPoolName)/$($_.name) is currently offline, unable to update"
              }
              # if an agent is currently running a request, they'll have an 'assignedRequest' object
              elseif($null -ne $_.assignedRequest) {
                      Write-Host "$($AgentPoolName)/$($_.name) is currently running request $($_.assignedRequest.requestId), unable to update"       
              }
              else {
                      Write-Host "Updating $($AgentPoolName)/$($_.name)..."

                      Update-AgentVersion -PoolId $AgentPoolId `
                                          -AgentId $_.id    
              }
        }
}
```

👉 Triggering the updates using the `Update-AgentVersion` cmdlet:

```powershell
function Update-AgentVersion {
  param (
          $PoolId,
          $AgentId
  )
  
  $Uri = "$($Organization)_apis/distributedtask/pools/$($PoolId)/messages?agentId=$($AgentId)&api-version=$($ApiVersion)"

  try 
  {
      $Response = Invoke-RestMethod -Uri $Uri `
                                    -Method Post `
                                    -ContentType "application/json" `
                                    -Headers $Header
  } 
  catch 
  {
      Write-Host "URI": $Uri
      Write-Host "Status code:" $_.Exception.Response.StatusCode.value__ 
      Write-Host "Exception reason:" $_.Exception.Response.ReasonPhrase
      Write-Host "Exception message:" $_.Exception.Message
  }   
}
```

## Results

Below is what the output will look like whenever the pipeline encounters any outdated agents. It will obviously skip any agents which are offline or are in the process of running any jobs. As this already being used in production, any agent names as well as the name of my Azure DevOps org have been blurred:

![agent-updates-results](/assets/images/agent-updates-results.png)

You can grab the Azure Pipelines definition (`azure-pipelines.yml`) as well as the PowerShell script (`Update-Agents.ps1`) from my *azure-devops-toolbox* repo [here](https://github.com/smholvoet/azure-devops-toolbox/blob/main/src/agent-updates).
