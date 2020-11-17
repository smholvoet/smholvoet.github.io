---
published: true
title: "Delete default project Wiki in Azure DevOps"
excerpt: "Deleting the default project Wiki in Azure DevOps"
date: 2020-10-23T00:00:00-04:00
show_date: true
tags:
  - Azure DevOps
  - VSTS
  - wiki
  - repo
  - git
  - postman
---

## Background

Azure DevOps supports 2 types of project Wikis:

- [**project wiki**](https://docs.microsoft.com/en-us/azure/devops/project/wiki/wiki-create-repo?view=azure-devops): users are able to add / edit pages directly within the specified Wiki
- [**code wiki**](https://docs.microsoft.com/en-us/azure/devops/project/wiki/publish-repo-to-wiki?view=azure-devops&tabs=browser#publish-a-git-repository-to-a-wiki-1): markdown files are defined in a Git repo and published to the Wiki

Upon creating a new wiki, you're greeted with the following screen:
![image-center](/assets/images/wiki_1.png){: .align-center}

Whenever someone has already created a project Wiki, there seems to be **no way** to delete it going through the web UI 🙉 . This can get very annoying, especially when your team project contains a code wiki which lives next to the (default) project wiki. Clicking the **Overview > Wiki** menu will have users ending up in the default project wiki (which will be selected by default) instead of your code wiki, which can get confusing.

![image-center](/assets/images/wiki_2.png){: .align-center}

Even though it seems like only the code wikis (obviously) have a git repo backing it, all content of provisioned wikis is stored in a dedicated repo as well. Deleting *this* repo will delete the project wiki in question.

## Solution

We can do this by making use of the Azure DevOps REST API. Call the REST API using PowerShell or specialized tools like for example [Postman](https://www.postman.com/).
Have a look at one of my earlier posts which explains how to do so: [Call Azure DevOps REST API with Postman](https://sanderh.dev/call-Azure-DevOps-REST-API-Postman/)

**1) Let's first [get a list](https://docs.microsoft.com/en-us/rest/api/azure/devops/wiki/wikis/list?view=azure-devops-rest-5.1) of all Wikis in our team project:**

```http
GET https://dev.azure.com/{organization}/{project}/_apis/wiki/wikis?api-version=6.0
```

**Response:**

```json
{
    "value": [
        {
            "id": "1318cf4b-2653-4567-af6b-f8028d7d9e33",
            "versions": [
                {
                    "version": "wikiMaster"
                }
            ],
            "url": "https://dev.azure.com/fabrikamfiber/7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4/_apis/wiki/wikis/1318cf4b-2653-4567-af6b-f8028d7d9e33",
            "remoteUrl": "https://dev.azure.com/fabrikamfiber/7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4/_wiki/wikis/1318cf4b-2653-4567-af6b-f8028d7d9e33",
            "type": "projectWiki",
            "name": "first-wiki.wiki",
            "projectId": "7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4",
            "repositoryId": "1318cf4b-2653-4567-af6b-f8028d7d9e33",
            "mappedPath": "/"
        },
        {
            "id": "5cfeda59-ec80-4e85-8600-9d99cbf54ebb",
            "versions": [
                {
                    "version": "master"
                }
            ],
            "url": "https://dev.azure.com/fabrikamfiber/7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4/_apis/wiki/wikis/5cfeda59-ec80-4e85-8600-9d99cbf54ebb",
            "remoteUrl": "https://dev.azure.com/fabrikamfiber/7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4/_wiki/wikis/5cfeda59-ec80-4e85-8600-9d99cbf54ebb",
            "type": "codeWiki",
            "name": "second-wiki",
            "projectId": "7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4",
            "repositoryId": "6bb93d89-e257-4385-8aa0-675fb4830d9c",
            "mappedPath": "/"
        }
    ],
    "count": 2
}
```

You can see both of our wikis are listed, each having a different `type` property. The first one being the project wiki (`"type": "projectWiki"`), and the second one being or code wiki (`"type": "codeWiki"`). Note the `repositoryId` of the project wiki (`1318cf4b-2653-4567-af6b-f8028d7d9e33` in this example), which we'll be using in the next step.

**2) [Delete](https://docs.microsoft.com/en-us/rest/api/azure/devops/git/repositories/delete?view=azure-devops-rest-6.0) the project wiki in question by specifing the `repositoryId` (`1318cf4b-2653-4567-af6b-f8028d7d9e33` in this example). This will delete the underlying repository and get rid of the wiki:**

```http
DELETE https://dev.azure.com/{organization}/{project}/_apis/git/repositories/{repositoryId}?api-version=6.0
```

**Response:**

```json
{
    "id": "1318cf4b-2653-4567-af6b-f8028d7d9e33",
    "versions": [
        {
            "version": "wikiMaster"
        }
    ],
    "url": "https://dev.azure.com/fabrikamfiber/7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4/_apis/wiki/wikis/1318cf4b-2653-4567-af6b-f8028d7d9e33",
    "remoteUrl": "https://dev.azure.com/fabrikamfiber/7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4/_wiki/wikis/1318cf4b-2653-4567-af6b-f8028d7d9e33",
    "type": "projectWiki",
    "name": "first-wiki.wiki",
    "projectId": "7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4",
    "repositoryId": "1318cf4b-2653-4567-af6b-f8028d7d9e33",
    "mappedPath": "/"
}
```

**3) Double check by re-running our first REST call:**

```http
GET https://dev.azure.com/{organization}/{project}/_apis/wiki/wikis?api-version=5.1
```

**Response:**

```json
{
    "value": [
        {
            "id": "5cfeda59-ec80-4e85-8600-9d99cbf54ebb",
            "versions": [
                {
                    "version": "master"
                }
            ],
            "url": "https://dev.azure.com/fabrikamfiber/7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4/_apis/wiki/wikis/5cfeda59-ec80-4e85-8600-9d99cbf54ebb",
            "remoteUrl": "https://dev.azure.com/fabrikamfiber/7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4/_wiki/wikis/5cfeda59-ec80-4e85-8600-9d99cbf54ebb",
            "type": "codeWiki",
            "name": "second-wiki",
            "projectId": "7c964ac3-ee1f-1234-9bfb-3bb5c5aff2d4",
            "repositoryId": "6bb93d89-e257-4385-8aa0-675fb4830d9c",
            "mappedPath": "/"
        }
    ],
    "count": 1
}
```

The above steps should have deleted your project wiki, leaving you with just the code wiki intact 🎉
