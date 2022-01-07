---
published: true
title: "Using Renovate with Azure DevOps"
excerpt: "Automated dependency scanning using Renovate and Azure DevOps"
date: 2021-11-21T00:00:00-04:00
show_date: true
tags:
  - Azure DevOps
  - Renovate
  - dependencies
  - security
  - .NET
---

Less than a year ago I published an article where you could make use of [Dependabot](https://sanderh.dev/Dependabot-Azure-DevOps/) to manage your package dependencies. That's pretty much exactly what [Renovate](https://docs.renovatebot.com/) does as well:

> Renovate (often referred to as "Renovate Bot") is an Open Source tool designed to automate the process of:
>
> - Detecting dependencies in a repository (including both Open Source as well as private/closed source)
> - Checking if there are newer versions which can be updated to
> - Creating commits and Merge/Pull Requests to apply such changes, and include Release Notes if available

![renovate-logo](https://app.renovatebot.com/images/whitesource_renovate_660_220.jpg){: .align-center}

Renovate is a product of WhiteSource which also has [WhiteSource Bolt](https://marketplace.visualstudio.com/items?itemName=whitesource.whiteSource-bolt-v2) in their product portfolio, which scans for security vulnerabilities, ensures license compliancy, etc. But enough of that, lets dive into setting up Renovate.

# Token

# Setup

config.js

```js
module.exports = {
  platform: 'azure',
  endpoint: 'https://dev.azure.com/sanderholvoet/',
  token: '<PAT>',
  repositories: ['renovate/Bicep'],
  enabledManagers: ['nuget']
};
```

```yaml
schedules:
  - cron: '0 3 * * *'
    displayName: 'Every day at 3am'
    branches:
      include: [main]

trigger: none

pool:
  vmImage: ubuntu-latest

steps:
  - bash: |
      docker run --env LOG_LEVEL=debug --rm -v "${PWD}/config.js:/usr/src/app/config.js" renovate/renovate:latest
```

```docker
services:
  renovate:
    image: renovate/renovate:latest
    volumes: 
      - ${PWD}/config.js:/usr/src/app/config.js
    environment:
      - LOG_LEVEL=debug
```

## Configuration

A couple of handy configuration options:

- `automerge`:
- `azureAutoApprove`:
- `azureWorkItemId`:
- `branchPrefix`: renovate-
- `labels`:

... and *many* (seriously) more, take a look at their extremely detailed 📖 docs [here](https://docs.renovatebot.com/configuration-options/).
