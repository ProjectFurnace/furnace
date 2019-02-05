# Project Furnace

![Furnace Logo](https://github.com/ProjectFurnace/furnace/raw/master/assets/furnace-logo-horiz-black.png)

A streaming data processing framework built on Serverless (or FaaS) and GitOps that allows you to build complex pipelines rapidly.

> This is currently a community release, we're looking for feedback and there are currently a number of limitations.

Furnace aims to simplify the creation and deployment of streaming data applications for things like Security Information and Event Management, Internet of Things, Clickstream Analysis and anywhere you need to process larges amounts of data and:

- Parse, cleanse, normalise before further processing
- Enrich, filter, calculate by assembling pipelines
- Store in a Data Lake for querying and performing analysis
- Create action workflows to respond to events in your data

## Serverless

We do this in an abstracted way that isn't attached to any particular cloud or platform however the key being that we try to run **natively** on the chosen underlying platform.  That means, when running on AWS we make use of its Serverless resources such as Lambda, Kinesis and a Data Lakes such as Redshift, S3 and Elasticsearch.  This approach allows Furnace to provide numerous benefits that we believe make it a compelling framework for applications of this kind:

- Dramatic cost savings, only pay for the processing power you require
- No servers or clusters to manage, also saving time and money
- Scale upwards and back to zero if necessary as your traffic flow varies
- Avoid cloud/vendor lock in

## DevOps Lite

Furnace is an opinionated but open framework using best practices in terms of efficiency and security that enables teams to focus on building upon the intelligence they have in their data with the least amount of pain possible.  Simply commit configuration and code to git and Furnace will build and deploy into multiple environments (eg. Dev, Staging, Production).  We've provided tooling that gets out of your way allowing you to be productive using the tools and languages you're used to.

## GitOps

We're advocates of the emerging GitOps methodology, not because it's trendy but because we believe it has some real benefits both in its general sense and in its application in Furnace.  Furnace uses Git as a single source of truth, everything is described in configuration and code, which makes auditing and change control easy.  Using a series of YAML files and packaging your code in the form of well described modules, Furnace is declarative in a similar way that was made popular by platforms such as Ansible and Kubernetes.  Today we have tight integration with GitHub providing a rich and effective developer experience.

## Furnace Constructs

**Source** - Defines the source of data, usually this is stream from Apache Kafka or AWS Kinesis but can be anything you define. We provide some standard sources out of the box.

**Tap** - Taps connect to a source. Their job is to parse and normalise data into a common format. A default set of Taps are provided and you can create new Taps by creating a module which maps to a serverless function.

**Pipeline** - Pipelines create a linear path for data to flow through a set of operations. Pipelines are connected to Taps and a Tap can feed multiple Pipelines.

**Sink** - A Sink is where your data arrives after it exits a Pipeline. A Sink could be a data lake or object storage. Multiple Pipelines can feed into a Sink and a Pipeline can feed into multiple Sinks.

**Resource** - Resources are used to initiate resources native to the environment in which Furnace is being deployed such as data stores (eg. Redshift on AWS)

**Actions** - Once data has been processed by your Pipelines, Actions make use of the structured data react and automate tasks in real-time.

**Stack** - A Stack is comprised of one or more end to end data flows into a logical container. A Stack can have multiple Environments (Dev, Staging, Production).

## Architectural Components

To provide these capabilities Furnace assembles a number of components into an autonomous system.

**Furnace CLI** - Responsible for 'Igniting' Furnace, creating new stacks and many helpful commands to keep you productive.

**Bootstrap Template** - Provisions the minimal components you need to create an instance of Furnace.  For AWS, this is a CloudFormation Template.

**Stack Template** - Provides a starting point for your stack with a set of configuration files, we provide a starter template and you can create your own.

**Module Template** - Boilerplate used to create your own modules.

**Function Template** - Wraps around a module for various platforms and languages.

**Module Repository** - Contains a library of prebuilt modules that can be added to a Stack.  We provide a set of 'core' modules and you can create your own repository.

**Stack Repository** - The various configuration files and modules for a specific Stack.

**State Repository** - Since everything is stored in Git, each stack requires a State Repository to store current state of a Stack and Environment.

**Deployment Service** - Responsible for receiving deployment events (via `git push` or `furnace promote`), building artifacts and provisioning the associated infrastructure on the chosen platform.

**Artifact Repository** - Stores module artifacts together with hashes and versions them.

**Log Repository** - Contains various logs files created as part of the Deployment process.

## Getting Started

Our aim is to have you up and running in less than 5 minutes.  You'll need to meet the following prerequisites (see current limitations):

- Node 8.x LTS and NPM 6.x (https://nodejs.org)
- AWS CLI and a configured profile (https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- A GitHub account and Personal Access Token (https://github.com/settings/tokens)

The process starts with installing the Furnace CLI and running `furnace ignite`

```bash
npm install @project-furnace/furnace-cli -g
furnace ignite
```
You'll be asked a series of questions relating to your environment.  This will take a few minutes to complete.  On AWS a CloudFormation template is being deployed that contains and ECS Cluster to execute the Deployment Service when needed, a series of Lambda functions to react to various events and a Simple Queue Service (SQS) to route messages between components.

Once this command has completed you're ready to create a new stack.
```bash
furnace new my-stack
cd my-stack
```
Again you'll be prompted to answer a series of questions about your Stack and will attempt to initialise various of the required parts including GitHub repositories and webhooks.  You'll now have a new repository created that you can start to build on.  You can learn more about working with Furnace Stacks with the 5 Minute Walkthrough.

The `stack.yaml` contains a list of environments, by default you have `dev`, `staging`, `production`.  Once you're ready deploy your first stack, we do so using a standard `git` flow.
```bash
git commit -am "initial commit"
git push origin master
```
This pushes to the first environment defined in your Stack, you can see the deployment status in `environments` in the Stack repository in GitHub or using the `furnace status` command inside the Stack directory.

Once you're happy that your Stack is functioning how you expected in your initial environment you can `promote` it to an upstream environment eventually towards its final/production environment.  For example, keeping with the default environments, to promote the `dev` environment to `staging` you would simply:
```bash
furnace promote dev
```

## Limitations

As this is currently a community preview, there are some limitations to be aware of.

**AWS** - Since a key goal is for Furnace to run natively on cloud providers, the initial release targets AWS exclusively since it has the largest user base.  More platforms to follow shortly including Kubernetes for on-premise deployments.
**GitHub** - Furnace aims to provide an end-to-end development and deployment flow therefore tight integration with Git providers is necessary, we initially support GitHub but GitLab and regular Git repository support will quickly follow.
**Node 8.10**- Another key goal of Furnace its to allow developers to write in any language they wish.  In this initial release we provide helpers and wrappers for Node 8.10 which enables you to write modules that are not tied to a specific platform.  We'll be creating support for a wide range of languages in the coming weeks.
**ECS** - Furnace uses a container to perform the deployment of your stacks to the underlying infrastructure since running this in a serverless function is impractical for several reasons.  Currently we're spinning up a single node Elastic Container Service as part of the bootstrap process.  The preferred approach will be to use Fargate for this however it is not currently supported in many regions and thus we took the decision to use ECS until further support is reached.
**GitHub Auth/HTTPS** - Currently only HTTPS based repositories are supported together with Personal Access Tokens however we'll look to support other forms of integration in the coming weeks.

# Repositories

The Furnace platform spans across several repositories.

| Name                    | Repository                                                   |
| ----------------------- | ------------------------------------------------------------ |
| Furnace CLI             | https://github.com/ProjectFurnace/furnace-cli                |
| Deployment Service      | https://github.com/ProjectFurnace/deploy                     |
| Module Templates        | https://github.com/ProjectFurnace/module-templates           |
| Core Modules            | https://github.com/ProjectFurnace/modules-core               |
| Platform Bootstrap      | https://github.com/ProjectFurnace/bootstrap                  |
| Example Security Stack  | https://github.com/ProjectFurnace/AWS-FlowLogs-Security-Example |
| NodeJS Helper Libraries | https://github.com/ProjectFurnace/lib-nodejs                 |
| Minimum Stack Template  | https://github.com/ProjectFurnace/starter-template           |

# Support
For general help and assistance, join our [Slack Community](https://join.slack.com/t/projectfurnace/shared_invite/enQtNTA1NTg3NDkwMzU4LWNmZDllMjk5ZTU1NzFjNTgzNDA2NzBjMmUxNmFiNGVhMjE2MjM5MDhiYmJkMjhiYTIwODAwOGIxN2I0MjZkMWE)

For general Furnace related issues or bugs, create an issue in [this repository](https://github.com/ProjectFurnace/furnace/issues)

For issues or bugs related to a specific repository, please ensure you create it there.

# License

Copyright (c) 2019 Furnace Ignite Ltd.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
