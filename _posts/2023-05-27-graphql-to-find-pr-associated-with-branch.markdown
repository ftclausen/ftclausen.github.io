---
title: "GraphQL to find PR associated with branch"
date: 2023-05-27 11:14
classes: wide
use_math: true
categories: jenkins cicd
---

This about a mini adventure to find which PR(s) belong to which branch using GitHub GraphQL for a GitHub Action project.

### Introduction to GraphQL

GraphQL is a query language.

See the [quick introduction](https://docs.github.com/en/graphql/guides/introduction-to-graphql) in the docs but, essentially, we are working with

- Connection: Like SQL these let you query related objects in the same call. E.g. `associatedPullRequests` is a connection between a branch (ref) and a pull request.
- Nodes: Generic term for an object. We are interested in the `pullrequest` object
- Edge: Represents connections between nodes e.g. pull request(s) that are associated with a PR

Here is the query we are using to find which PR(s) are associated with a given commit (only using the first one found so far):

```graphql
{
  repository(name: "example-repo", owner: "ftclausen") {
    ref(qualifiedName:"topic/some_branch") {
      associatedPullRequests(first: 1) {
        edges {
          node {
            number
            title
            baseRef {
              name
            }
          }
        }
      }
    }
  }
}
```

This then returns the following data

```json
{
  "data": {
    "repository": {
      "ref": {
        "associatedPullRequests": {
          "edges": [
            {
              "node": {
                "number": 55,
                "title": "This is an example pull request",
                "baseRef": {
                  "name": "main"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

As best I understand `edges` will always be an array. Finally this can be imagined graphically which I find useful:

![/images/graphql_gh.png](/images/graphql_gh.png)

We then pull the above data in and use it as needed


### How to represent types for GraphQL data

Given GraphQL data varies per type I followed [this very helpful
guide](https://benlimmer.com/2020/05/16/adding-typescript-types-github-graphql-api/) guide to generate types for GraphQL
responses instead of manually doing it. Please see that for a concrete example.

### A note on the concept of a Pull Request

We have three different ways pull requests are represented depending on the context:

- Pull Request Event: When a new PR is created we get a [pull_request event payload](https://docs.github.com/en/webhooks-and-events/webhooks/webhook-events-and-payloads#pull_request)
- [REST API Pull Request](https://docs.github.com/en/rest/pulls/pulls?apiVersion=2022-11-28#get-a-pull-request) response
- [Pull Request from GraphQL](https://docs.github.com/en/graphql/reference/objects#pullrequest) response

So that is something to be aware of when working with TypeScript. I had to make my own meta-object to unify this for my
purposes; this meta-object extracted just what I needed out of each one of the above types.
