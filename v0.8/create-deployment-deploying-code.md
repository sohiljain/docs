---
title: "Deploying Your Code"
description: "Create your Airflow environment and push up your DAGs"
date: 2018-07-17T00:00:00.000Z
slug: "create-deployment-deploying-code"
---

## Creating a Deployment

The best place to create a deployment right now is directly from the UI.

From your workspace, click New Deployment and Configure your deployment accordingly!


Once that deployment is created, it should show up from your CLI as well (as long as you are pointing at the right workspace):

![Workspace Dashboard](https://s3.amazonaws.com/astronomer-cdn/website/img/guides/workspace_dashboard.png)


```
astro deployment list
 NAME           RELEASE NAME                 ASTRO      DEPLOYMENT ID                            
 demo_cluster     infrared-photon-7780     v0.7.5     c2436025-d501-4944-9c29-19ca61e7f359  
```

## Logging in and Deploying from the CLI

Before you can push your code up, you will need to authenticate.

If you are a Cloud customer:

```
astro auth login astronomer.cloud
```

If you are an Enterprise customer, your login command will be at the domain you chose to deploy on - e.g.

```
astro auth login airflow.MYCOMPANY.com
```

This will take you through our authentication flow:

```
 "Paolas-MBP-2:hello-astro paola$ astro auth login astronomer.cloud
 CLUSTER                             WORKSPACE                           
 astronomer.cloud                    4a6cb370-c361-440d-b02b-c90b07ef15f6

 Switched cluster
Username (leave blank for oAuth):
```

Once you've authenticated, make sure you are in the right workspace:

```
astro workspace switch
```

```
astro deployment list
 NAME           RELEASE NAME                 ASTRO      DEPLOYMENT ID                            
 BI Project     ultraviolet-isotope-9213     v0.7.5     c2436025-d501-4944-9c29-19ca61e7f359  
 ```

To deploy to an Airflow cluster, run:

```
astro airflow deploy
```

As you spin up additional deployments, they'll show up as options when you run `astro airflow deploy`

### What gets deployed?

Everything in your top level directory (and all children directory) that you ran `astro airflow init` in will get bundled into a Docker image and deployed out. We do **not** deploy any of the metadata associated with your the local airflow deployment - only the code.

For more information on what gets built into your image, jump over to the [CLI section](https://www.astronomer.io/docs/customizing-your-image/).


### Deployments, Workspaces, and Kubernetes Namespaces

Each deployment lives in its own Kubernetes [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) and are completely unaware of each other. Each deployment is allocated resources separately, is configured independently, and maintains its own metadata - **the only thing deployments have in common is they run on the same underlying Kubernetes cluster.**

Generally, most use cases will call for a `production` and `dev` deployment that exist within Workspace (so available to the same set of users). As we roll out more permission controls, you will be able to control amounts of access to different deployments.