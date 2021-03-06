---
title: "Cloud FAQs"
description: "Questions that we see often when customers are getting started on Astronomer Cloud."
date: 2018-10-12T00:00:00.000Z
slug: "cloud-faq"
---

If you're thinking about Astronomer Cloud, you might have a few questions on your mind. Here are a few FAQ's that might help.

#### What version of Airflow is Astronomer Cloud currently running?

Astronomer Cloud v0.7.5 is officially compatible with Airflow v1.9 and 1.10.1. You'll find this reflected in your Dockerfile:

```
FROM astronomerinc/ap-airflow:0.7.5-2.10.1-onbuild
```

Adjusting between Airflow versions just takes a [quick swap to your Dockerfile](https://forum.astronomer.io/t/how-do-i-run-airflow-1-10-on-astronomer-v0-7/58).

#### How would I give Astronomer Cloud access to my VPC?

To connect Astronomer Cloud to any database in your VPC, you'll just have to whitelist our Static IP: `35.188.248.243`

If you're whitelisting that IP on Amazon Redshift, check out [this forum post](https://forum.astronomer.io/t/how-do-i-whitelist-astronomer-cloud-on-aws-redshift/165).

#### Will I have access to Airflow's underlying database for my deployment?

Yes! Each Astronomer Cloud customer has has an isolated Postgres database per deployment. To access that database and the Ad-Hoc Query feature for a deployment on Astronomer, check out our guidelines in [this forum post](https://forum.astronomer.io/t/how-can-i-set-up-the-ad-hoc-query-feature-in-a-remote-deployment/143).

To set up an Airflow database when you're developing locally, check out [these guidelines](https://forum.astronomer.io/t/how-do-i-set-my-postgres-username-and-password-to-access-the-ad-hoc-query-feature-locally/139).

#### What are my options for monitoring our deployments?

Right now, your monitoring options for Cloud are:

1. Airflow Alerts at the Task or DAG Run level (instructions [here](https://www.astronomer.io/docs/setting-up-airflow-emails/))
2. Deployment Configure page on the Astro UI for all resource configs
3. Flower dashboard in the Astro UI to check on your Celery Workers (whether or not they’re online, how many tasks they’re actively processing, etc.)
4. Scheduler/Webserver/Worker logs coming soon on v0.8 to Astronomer's UI

Access to deployment-level monitoring via Grafana is unfortunately an Enterprise-only feature at the moment since all Cloud deployments live within our wider Astronomer-hosted cluster, but we're actively working to increase exposure to deployment stats for Cloud. Stay peeled.

#### What Airflow Executors do you currently support?

We currently support the Celery and Local Executors, with support for KubernetesExecutor coming up soon in Astronomer v0.9. You can switch between the two freely via the Astronomer UI.

*Not sure which Executor to use?* We generally recommend starting off with the LocalExecutor and moving up from there once you see your deployment is in need of more effective horizontal scaling. Check out [Airflow Executors: Explained](https://www.astronomer.io/guides/airflow-executors-explained/) for a ful analysis on each.

#### Do you have a recommended way to share code amongst my team members?

This is largely dependent on personal preference and your particular use case.

At a Workspace level, we recommend having 1 Astronomer Workspace per team of Airflow users. That way, anyone on each team has access to the same set of deployments under that Workspace (RBAC will soon allow you to adjust that access at a deployment level if you'd like). 

Across deployments, we'd generally recommend one repository/parent directory per project. That way, you leave the door open for CI/CD down the line if that's something you ever want to set up.

As for the code itself, we’ve seen effective organization where external code is partitioned by function and/or business case, so one directly for SQL, one for data processing tasks, one for data validation, etc.

#### What authentication methods does Astronomer Cloud support?

Right now, we offer authentication via Google, Github, and Username/Password in Cloud. Once you've created a Workspace on Astronomer, you'll use that same method to authenticate to the Astronomer CLI.

If you're interested in Enterprise, the options there are more flexible. Check out [this doc](https://www.astronomer.io/docs/ee-integrating-auth-system/) for more info.

#### Where do I log in?

If you're interested in starting a trial,you can head [here](https://trial.astronomer.io/) to kick it off. You can expect a Welcome email from there with next steps on how to create an account on Astronomer Cloud.

If you want to cancel your trial and are having trouble accessing your Astronomer account, reach out to us at support@astronomer.io.

#### How does billing work?

Your first 14 days on Astronomer Cloud are entirely free of cost. When you sign up, you'll trigger a 14-day trial period that allows you to freely spin up a deployment without worrying about an invoice at the end of the month.

After your first 14 days, we'll charge you based on exact resource usage per deployment at the end of the month. You can expect your first invoice on Day 60.

For full pricing details, check out our [Pricing Doc](https://www.astronomer.io/docs/pricing/).

**Did we miss something?** Check out the rest of our docs or our [community forum](https://forum.astronomer.io/) for more product-specific info, or reach out to us at humans@astronomer.io. We'd love to hear from you.