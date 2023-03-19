---
title: "Azure Dev Ops Boards - How to get and update a Work Item using typed-rest-client"
date: 2023-03-18 20:24
classes: wide
use_math: true
categories: jenkins cicd
---

This post outlines what worked for me to update an Azure Dev Ops (ADO) Boards Work Item. I initially tried the following
prior to settling on [typed-rest-client](https://github.com/microsoft/typed-rest-client):

- [azure-devops-node-api](https://github.com/microsoft/azure-devops-node-api) - This looked promising but ultimately I
    could not get it to work [for updating work
    items](https://stackoverflow.com/questions/75763159/using-azure-devops-node-api-to-update-patch-a-ado-work-item).
    Judging by the silence in my Stackoverflow question it seems updating Work Items via azure-devops-node-api is not
    something anyone has tried?
- [azure-devops-extension-api](https://github.com/microsoft/azure-devops-extension-api) - This one seemed promising but
    has no README nor examples. It was too deep a rabbit hole to go down right now.

Thus I settled on typed-rest-client which seemed relatively straight forward to use.

I'll show some snippets of how I fetched a Work Item and then how to update a custom field (or any field really). For my
use case I just created a toy type to store just what I was interested in rather than try and pull down an official type
from the monster `azure-devops*` repos above (🐇). If I find a more "official" way of doing it then I'll update this
post.

# Step 0 - Minimal Types for Representing Work Item

Given the labynth of types azure-devops-node-api and azure-devops-extension-api have I decided to just make my own as
follows:

```typescript
export interface ADOWorkItemLite {
  readonly id: number,
  readonly fields: ADOWorkItemFields
}

export interface ADOWorkItemFields {
  "Custom.SomeField": string // Can also be another, non-custom field
}
```

# Fetching a Work Item

```typescript
// Note: I can probably get the PAT handler from type-rest-client as well
import * as ado from 'azure-devops-node-api'
import * as restClient from 'typed-rest-client'

const authHandler = ado.getPersonalAccessTokenHandler(process.env.PAT)
const client: restClient.RestClient = new restClient
  .RestClient('example-user-agent', 'https://dev.azure.com', [authHandler])

/**
 * Where:
 * myorg - Your organisation name
 * project - Your URL escaped project name
 * 1234 - The ADO ticket being requested
 * 7.0 - The ADO API version (7.0 as of March 2023)
*/
const location = '/myorg/project/_apis/wit/workitems/1234?api-version=7.0'

return client.get<ADOWorkItemLite>(location).then(response => {
    const workItem = response.result as ADOWorkItemLite
    return workItem
})
```

# Updating a Work Item Field

For this I introduced a new type, the HTTP PATCH update payload:

```typescript
export interface ADOWorkItemUpdatePayload {
  /**
   * What we're updating, usually "add"
   */
  op: string,

  /**
   * The Work Item path e.g. /fields/SomeField
   */
  path: string,

  /**
   * The value given to the item on the path
   */
  value: string
}
```

Then the initial parts are the same as getting a work item above but, after that, we apply the update:

```typescript
import * as ado from 'azure-devops-node-api'
import * as restClient from 'typed-rest-client'

const authHandler = ado.getPersonalAccessTokenHandler(process.env.PAT)
const client: restClient.RestClient = new restClient
  .RestClient('example-user-agent', 'https://dev.azure.com', [authHandler])
const location = '/myorg/project/_apis/wit/workitems/1234?api-version=7.0'

const patchContents: ADOWorkItemUpdatePayload = {
  op: "add",
  path: "/fields/Custom.SomeField",
  value: updatedWorkItem.fields['Custom.SomeField']
}

return client.update<ADOWorkItemUpdatePayload>(location, [patchContents], {
  additionalHeaders: {
    "Content-Type": 'application/json-patch+json'
  }
}).then(results => {
  console.log(`${results.statusCode} - Updated work item ${updatedWorkItem.id}`)
})
```

For me, this worked. As mentioned I should use the auth handler from type-rest-client to remove the dependency on
azure-devops-node-api.
