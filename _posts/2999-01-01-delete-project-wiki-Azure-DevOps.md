---
published: false
title: "Delete default project Wiki in Azure DevOps"
excerpt: "A unique line of text to describe this post that will display in an archive listing and meta description with SEO benefits."
date: 2999-01-01T00:00:00-04:00
tags:
  - Azure DevOps
  - wiki
  - repo
  - git
  - postman
---

## Background

Azure DevOps supports 2 types of project Wiki's:
- **provisioned wiki:** users are able to add / edit pages directly within the specified Wiki
- **code wiki:** markdown files are defined in a Git repo and published to the Wiki


Upon creating a new wiki, you're greeted with the following screen:
![image-center](/assets/images/wiki_1.png){: .align-center}


Whenever someone has already created a provisioned (project) Wiki, there seems to be **no way** to delete it going through the web UI. This can get very annoying, especially when your team project contains a code wiki which lives next to the (default) provision wiki. Clicking the **Overview > Wiki** menu will have users ending up in the default project wiki instead of your code wiki, which can get confusing.

Even though it seems like only the latter type of Wiki has a (git) repo backing it, all content of provisioned wikis is stored in a dedicated repo as well. Deleting *this* repo will delete the provisioned wiki in question.

## Solution
We can do this by making use of the Azure DevOps REST API. Call the REST API using PowerShell or specialized tools like for example [Postman](https://www.postman.com/).
Have a look at one of my earlier posts which explains how to do so: [Call Azure DevOps REST API with Postman](https://sanderh.dev/)

{% raw %}{{ site.url }}{{ site.baseurl }}/call-Azure-DevOps-REST-API-Postman{% endraw %}
{% raw %}{{ site.url }}{{ site.baseurl }}call-Azure-DevOps-REST-API-Postman{% endraw %}


**1) Let's first [get a list](https://docs.microsoft.com/en-us/rest/api/azure/devops/wiki/wikis/list?view=azure-devops-rest-5.1) of all Wikis in our team project:**

```http
GET https://dev.azure.com/{organization}/{project}/_apis/wiki/wikis?api-version=5.1
```
Reponse:
```json
{
    "value": [
        {
            "id": "be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "versions": [
                {
                    "version": "master"
                }
            ],
            "url": "https://dev.azure.com/picanolgroup/ebd9248d-6884-4f20-9a7a-ec01dca68160/_apis/wiki/wikis/be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "remoteUrl": "https://dev.azure.com/picanolgroup/ebd9248d-6884-4f20-9a7a-ec01dca68160/_wiki/wikis/be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "type": "codeWiki",
            "name": "Project Wiki",
            "projectId": "ebd9248d-6884-4f20-9a7a-ec01dca68160",
            "repositoryId": "700e36e3-8376-4a1a-9c8f-8b05f0a0a649",
            "mappedPath": "/governance"
        }
    ],
    "count": 1
}
```

**2) [Delete](https://docs.microsoft.com/en-us/rest/api/azure/devops/wiki/wikis/delete?view=azure-devops-rest-5.1) the wiki in question by specifing the `wikiIdentifier`, which maps to our `repositoryId`:**
```http
DELETE https://dev.azure.com/{organization}/{project}/_apis/wiki/wikis/{wikiIdentifier}?api-version=5.1
```
Response:
```json
{
    "value": [
        {
            "id": "be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "versions": [
                {
                    "version": "master"
                }
            ],
            "url": "https://dev.azure.com/picanolgroup/ebd9248d-6884-4f20-9a7a-ec01dca68160/_apis/wiki/wikis/be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "remoteUrl": "https://dev.azure.com/picanolgroup/ebd9248d-6884-4f20-9a7a-ec01dca68160/_wiki/wikis/be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "type": "codeWiki",
            "name": "Project Wiki",
            "projectId": "ebd9248d-6884-4f20-9a7a-ec01dca68160",
            "repositoryId": "700e36e3-8376-4a1a-9c8f-8b05f0a0a649",
            "mappedPath": "/governance"
        }
    ],
    "count": 1
}
```

**3) Double check by re-running our first REST call:**
```http
GET https://dev.azure.com/{organization}/{project}/_apis/wiki/wikis?api-version=5.1
```
Response:
```json
{
    "value": [
        {
            "id": "be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "versions": [
                {
                    "version": "master"
                }
            ],
            "url": "https://dev.azure.com/picanolgroup/ebd9248d-6884-4f20-9a7a-ec01dca68160/_apis/wiki/wikis/be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "remoteUrl": "https://dev.azure.com/picanolgroup/ebd9248d-6884-4f20-9a7a-ec01dca68160/_wiki/wikis/be9827f0-b9c5-4900-9ab5-c0285d235d14",
            "type": "codeWiki",
            "name": "Project Wiki",
            "projectId": "ebd9248d-6884-4f20-9a7a-ec01dca68160",
            "repositoryId": "700e36e3-8376-4a1a-9c8f-8b05f0a0a649",
            "mappedPath": "/governance"
        }
    ],
    "count": 1
}
```