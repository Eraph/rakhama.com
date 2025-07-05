---
title: "The Joy of Devops"
description: "Or, how I learned to stop worrying and love YAML"
date: 2025-02-28T08:46:17+10:00
draft: true
categories:
  - One
tags:
  - One
series:
  - One
images:
  - "path/to/something.jpg"
---
The project I'm currently on has an Azure Devops build pipeline in place, PR (pull request) builds were taking an hour to run. I, perhaps somewhat foolishly, raised my hand to have a go at optimising it.
<!--more-->
Now I'm not new to optimising Azure DevOps Pipelines, but I still run into my fair share of challenges; not least around which variable syntax I should use for any particular reference. So I've made up this example pipeline YAML to serve as a reference, along with explanations of when to use certain techniques and why. But first, a breakdown of what I was trying to achieve with this.

### The Ultimate Goal
The build and test job was taking well over an hour to complete. While running, that was occupying a build agent, meaning there could often be a long wait for a PR build,  (continuous integration) build or deployment to start running depending on agent availability. The ultimate goal here was to improve the overall availability of build agents primarily by reducing the amount of time they spend on a single job.

### Spreading Tests Across Jobs
By far the longest task in the pipeline run for this project was a task to run all the .NET tests, clocking in at over 50 minutes to complete. The majority of tests in the project are subcutaneous (end-to-end) tests which explains the duration to a degree, and all tests, unit, subcutaneous or otherwise, belong to the same project. Fortunately the `dotnet test` command allows us to run a subset of tests within a project by including or excluding namespaces.

 To take advantage of this, it is important to understand that Azure DevOps Jobs run on their own agent. With multiple agents, multiple jobs may run in parallel, assuming they don't depend on each other. This is a great case for splitting up the test process in the pipeline; by targeting specific namespaces or groups of namespaces, we can split the testing burden across any number of jobs. Bearing in mind that there is overhead for every job (spinning up the agent, checking out or pulling build artifacts, publishing results, etc) and a limited number of agents, consideration must be given to the optimal number of jobs to split into.

 In the case of this project, I found there were four areas with _very_ roughly similar test run times; another thing to bear in mind is that the project is still very active, and run times are likely to increase across any of those areas, so there's little value in establishing a perfectly even set. With only three agents to work with, only three of those areas would be able to run at once, but it was a good case for more agents in the future. The split left me with something like the following namespace groupings:

 - Subcutaneous Import Processes
  - Project.Tests.Subcutaneous.AbcImport
  - Project.Tests.Subcutaneous.DefImport
  - Project.Tests.Subcutaneous.GhiImport
- Subcutaneous Projects Processes
  - Project.Tests.Subcutaneous.Projects
- Subcutaneous External System Processes
  - Project.Tests.Subcutaneous.ExternalSystem
- All Remaining Tests

In the case of the last grouping, "All Remaining Tests", we can provide the list of namespaces already provided as a set to _exclude_ from testing.

### Code Coverage
In the previous pipeline design, building and testing were performed on the same agent as part of the same job. I didn't realise at the time how important it is for code coverage collection to have access to the source files; in hindsight I suppose it makes sense, but after a lot of investigation I found out that it's not _totally_ necessary as the `.pdb` debug files can give the collection process enough information to generate code coverage. I'm pretty sure this is something that's bitten me in the past as well, so it was a relief to finally understand what was going on and how to resolve it. Basically we can tell the coverage collector (Coverlet in this case) to ignore missing source files and rely solely on the `.pdb` files. Did you know `.pdb` files have direct references to the source files? I didn't!

One more thing, code coverage adds a lot of time to test runs, something like 20% from what I saw. Coverage was only ever published with CI builds, so we saved a lot of time by turning off coverage unless it was a CI build.

### Build Batching
Another opportunity for freeing up agents was to batch CI builds. Imagine three PRs are merged in reasonably quick succession, and three CI builds are running in parallel. Doesn't make sense, only the most recent one has value. Batching means that CI builds are queued, bundling all changes committed to the target branch into a single build once the previous one has finished. This means that deployments can continue to happen quickly, but without tying up agents with unnecessary builds.

Batching can be restricted to a branch, typically `main`, ensuring that PR builds will always run as soon as an agent is available.

### Separating Build/Deploy
Pipelines calling pipelines? Oh yes. Not only does this provide a clean separation of purpose for the pipelines, but we can target a source branch (`main` again in this case) as the trigger; only successful CI builds will automatically trigger a deploy run (although they can still be run manually if necessary, targeting any successful build across all branches). This provides its own challenges, with limited access to what was going on in the previous pipeline's run.

### On the Matter of Expressions
Oh, Azure Pipeline [Variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops) and [Expressions](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/expressions?view=azure-devops). I must have committed at least a hundred times trying to get these right. I will attempt to summarise them below.

#### Template Expressions
``` yaml
${{ parameters.someRandomParameter }}
```

These are the first expressions to be applied. They are expanded at compile time, so they only work for variables that are known before execution begins, such as [parameters](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/template-parameters?view=azure-devops) and [statically defined variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops).

Parameters are the only kind of variables that can be defined with a type, the rest are always strings. Supported parameter types include strings, booleans, numbers, objects and arrays.

#### Runtime Expressions
``` yaml
$[eq(variables['Build.SourceBranch'], 'refs/heads/main')]
```

Runtime expressions are expanded during runtime, natch. Expressions must take up the entire right-hand side of a pipeline definition.

#### Macro Variables
``` yaml
$(someRandomVariable)
```

These are expanded before a task is run during runtime. If the variable specified does not exist, the variable is not substitued and appears as-is.

#### Scoping
Variables defined at the root of a pipeline are available everywhere. Variables defined at the root of a stage are available throughout that stage, and so on.

Parameters are available to the task or template that they're provided to.

#### Special Variables
Along with the widely available [Predefined Variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml), there are some special case variables like [Run Number Tokens](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/run-number?view=azure-devops#tokens) that are only available when defining a run's name, and [Pipeline Resources](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/resources?view=azure-devops#pipelines-resource-definition) when a pipeline is triggered by the completion of another pipeline.

