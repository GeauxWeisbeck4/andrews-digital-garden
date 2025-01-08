---
id: 01JH1NZR47P5G8N822759XTVNQ
modified: 2025-01-07T19:45:56-05:00
---
# Chapter 27. Building a CI/CD Pipeline

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 27th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Now that you’ve gone through the process of integrating the frontend and backend along with all the infrastructure and services you need, you can automate your deployments with a CI/CD (continuous integration and continuous deployment) pipeline. That way, you can ensure that all your deployments to production or other environments run the same way each time. This is a standard for just about every organization you might work at.

Having an automated CI/CD pipeline will encourage your dev team to make small and frequent releases. You’ll partner with the DevOps team to build and maintain this pipeline. Your contribution will be having a deep understanding of how the app works and how it should run. That way, you can help set environment variables and runtime versions, and you can make sure the correct commands are being executed to get the app in a deployable state.

In this chapter, I’ll cover:

- Using CircleCI to set up a CI/CD pipeline
    
- Selecting tools to simplify your pipelines
    
- Setting up different environments and their purposes
    
- Using GitHub as part of your process
    

A lot of options and tools are available to set up your pipeline, and I’ll go through some that will get you going quickly. This is another area that crosses over with DevOps quite a bit, so you don’t have to be the expert here. Work with them to understand enough to keep your team unblocked; you can decide to go deeper from there or not. Once you’ve created your own pipeline, you can modify it as the project’s needs change.

# Creating Your Own Pipeline

Deploying an app to any environment involves running a number of commands in a terminal on the server. There are steps to download all the packages, build the app or the container, set environment variables, handle any other setup, and run the app on the server. There can be any number of steps to deploy the app smoothly. That’s where automation comes in because it’s easy to skip steps or run them out of order when the process is manual.

Your CI/CD pipeline will help you with automation because it ensures consistency throughout your process. Your pipeline will have steps and stages. A _step_ is an individual command you run to do a task like create a build or invalidate a cache. A _stage_ comprises multiple steps to complete a specific goal, such as doing testing or deploying an artifact to a service.

At a minimum, you will have a build stage, a test stage, and a deploy stage, although a [number of others](https://zeet.co/blog/cicd-pipeline-stages#11-most-important-cicd-pipeline-stages) may be included. During the _build stage_, you’ll have steps to get the current version of the code from the team repo. All the individual features and fixes you and the team have been working on will get combined during this stage of the pipeline.

Common steps in the build stage include:

- Spin up the environment
    
- Prepare your environment variables
    
- Check out the code from the repo
    
- Install all your packages
    
- Run the build command(s)
    

The _test stage_ is when you run unit tests, integration tests, and maybe some security tests on the version of the app you’re ready to deploy. Teams can be tempted to skip this stage in crunch time because it can take a while for the tests to run, depending on which tools you use. Fight the temptation to skip the test stage in favor of faster deployment times. Your tests have been written to save you from regressions, bugs, and security vulnerabilities getting to users.

Common steps in the test stage include:

- Run code-quality tests
    
- Run unit tests
    
- Run integration tests
    
- Run security tests
    
- Report test results to the devs
    

The last stage is the _deploy stage,_ when you decide which environment the changes are released to. You can use the deploy stage to manage other things, like rollbacks, and you can set the stage to run on a schedule or trigger based on certain events. This stage differs greatly based on the organization where you work because the strategy you’ll use depends on the industry, the size of the teams, and the agreements between engineering and business. Some organizations will never automate deployments to production while others expect that to happen once the pipeline has been triggered under the right conditions.

Common steps in the deploy stage include:

- Set up any secrets or connections to the cloud service that will serve the build artifact
    
- Upload the artifact to the cloud service
    
- Move the build artifact to the server location
    
- Reset the cache service for the app
    
- Send notification that the deploy has been successful or has failed
    

Although the steps in each stage will depend on the organization where you work and the tools being used, the steps listed are ones you can implement if you have to set up your own pipeline.

## Speed Considerations

As you look at these stages, keep in mind that you want them to be fast without sacrificing quality. I’ve seen some CI/CD pipelines take more than an hour to get through all the stages. That slows your team substantially, leads to frustration for everyone involved, and creates a situation where you can’t release small, incremental changes. Each of these stages can be broken down into smaller stages, with some of them running in parallel to speed up the pipeline.

Keep the execution time in mind as you build the pipeline. The first place to start and get instant feedback is in your PRs. One sign of a good pipeline is that you get failures as early in the pipeline as possible. That way, the dev team can address issues quickly, and you haven’t wasted a lot of time watching scripts run just for something to fail at the last stage because of a code problem.

One strategy I’ve seen work is running the build and test stages on PRs that target certain branches. For example, you might open a draft PR before it’s ready for review. It can be helpful to check that the tests are still passing and the build is successful at this point because you can address issues in the PR before you even get a review. It’s a way to get feedback early in your development process so that it doesn’t hold up reviews or releases later. You can get feedback even earlier in your development process when you have some of these checks implemented with Git hooks.

One of the purposes of automating deployments is to have confidence that the code will work consistently in any environment you use. That’s why you should take time to think out the steps for each stage and document them. One of the best things you can do is stay on top of documentation for things like this. That way, you can get feedback and validate your ideas while making the strongest decisions based on everyone’s experience. Documenting your CI/CD pipeline will also help you think through most of the process, so when it’s time to write the code and use the tools, you aren’t still figuring out the big picture.

Now that you know what your pipeline consists of, you can start adding some of the initial automation checks.

## Git Hooks

Since your project uses Git, you can set a lot of checks to help you catch issues long before it’s time for a deployment. You already have a couple of [Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) in your [frontend repo](https://github.com/flippedcoder/dashboard-ui/blob/main/.husky/pre-commit) for `pre-commit` and `pre-push` actions using [Husky](https://typicode.github.io/husky/how-to.html). The commands you currently have are to enforce code quality before any changes leave a dev’s machine. Right now in your repo, you have these commands in your `pre-commit` hook:

```
#!/usr/bin/env sh
```

You can add more code-quality and developer-hygiene checks here to watch for commit messages, merge conflicts with the target branch, and restrict commits to protected branches locally. Let’s add a new hook to check the commit messages that developers write. Create a new file in the _.husky_ directory called _commit-msg_ in your frontend repo and write the following script to check the format of the commit messages:

```
#!/usr/bin/env bash
```

This will make it easier for you to quickly look through merged PRs and see what changes are present on the shared branches. This is particularly helpful when you’re debugging issues that come up in your environments after a deployment. You and the team should decide what the commit messages should look like, but at the minimum, they should let anyone glance through the PRs and see what each one implements. You might go with ticket numbers or identifying code changes by type, such as fixes, bugs, features, and other types defined in the [conventional commits spec](https://www.conventionalcommits.org/en/v1.0.0/#summary). You might even look at using AI tools like [ai-commit](https://github.com/insulineru/ai-commit) or [aicommits](https://github.com/Nutlope/aicommits) to further automate this task.

Another Git hook you can use is the `pre-push` hook. This runs a script any time a developer tries to push changes to a remote branch, even if it’s just their own branch. Right now, you have this hook configured to run your unit tests before any code leaves a dev’s computer. Here’s the script you currently have:

```
#!/usr/bin/env sh
```

###### WARNING

Be careful of the commands you run in the `pre-commit` and `pre-push` hooks because sometimes they can hinder the team from sharing work in progress. This could lead to a situation where the devs comment out all the `pre-push` checks, which results in lower-quality code getting to the shared branches. Be selective with the commands you run in these hooks to keep them useful. Remember, you still have plenty of checks that will happen in the pipeline.

I want to mention that you don’t have to use a tool like Husky for Git hooks. When you’re working in a repo that uses Git, you can look in the _.git/hooks_ directory of the project and find examples for all the possible hooks you can use, as shown in [Figure 27-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#git_hooks_in_the_dotgit_directory). Remember that the _.git_ directory is probably hidden in your file explorer in case you don’t find it immediately. The reason some devs prefer using Husky over the built-in hooks directory is because it handles much of the configuration for you in the background.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2701.png)

###### Figure 27-1. Git hooks in the .git directory

Here’s an example of how you could implement `pre-commit` and `pre-push` hooks in pure Git. This script already exists; you just have to remove the _.sample_ ending from the filename of the hook you want to use.

This is the `pre-commit` hook:

```
#!/bin/sh
```

This is the `pre-push` hook:

```
#!/bin/sh
```

Now you’re using the Git lifecycle to handle some common checks before anything gets to the CI/CD pipeline, which will save you a lot of time on PR reviews, triggering the pipeline, and waiting for it to report issues that could have been found much sooner. You can add more hooks if you and the team think it will help save time or keep things consistent.

## GitHub Configs

You’ve added tools to make sure you catch as much as you can before the code moves from a developer’s computer, so now it’s time to move to the next layer. That’s going to be on GitHub in your PRs. This is where you can do more to enforce code quality and do some initial testing to make sure the code works as expected and doesn’t break anything obvious. The conventions you set early in the project are going to be great here because you’ve already accounted for what should be checked in the PR reviews.

There are a few things you should set up in your repo on GitHub to ensure that the branches are protected from less thorough reviews. Consider restricting direct pushes to your `main` and `develop` branches since these are the ones that all the devs typically branch and deploy from. You should also restrict which branches can be deleted and by whom as well as other actions that can be taken. You do this in the settings for the repo, which will look something like [Figure 27-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#github_branch_settings).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2702.png)

###### Figure 27-2. GitHub branch settings

Explore the settings you have available in GitHub, and you’ll find something suitable for just about every rule you want to enforce. I’ll go into more detail on Git branch strategies in [Chapter 28](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#git_management) because this will hugely impact how you handle things in your dev, deploy, and incident processes. For now, we’ll focus on having a `main` branch that is used for production deploys, a `develop` branch used for staging or preproduction deploys, and dev branches that the team works on.

A good rule of thumb is to require at least one PR approval before merging a developer’s branch into `develop`. That way, you and the team know that someone else verified that the code meets the agreed-upon conventions and that improvements can be suggested and handled. A bare minimum check in any PR review is to pull the dev branch to your machine and make sure the app runs and meets the acceptance criteria for the ticket.

Another rule of thumb for your GitHub configs is to block any direct PRs or pushes to the `main` branch. This is your source of truth for the code that runs in production, and you don’t want anyone to be able to directly change that code without going through the release process. Keep this in mind when hotfixes come up. Someone may need to temporarily disable that rule to get a fix to production quickly.

One more rule is to run some of your pipeline checks directly on the PR. For example, making sure that the tests pass and a build is successfully created will catch issues before the PR gets merged and potentially blocks the team while the dev figures out what’s wrong.

With these rules and restrictions in place, you can have even more confidence that the code you and your team are writing will work in production, and you can keep better code hygiene so that you don’t have files with different formatting or with methods and variables written differently. You’ve done everything to catch issues at the PR-review and code-merging levels, so now you can start on your pipeline.

## CircleCI Configs

There are plenty of CI/CD pipeline tools, such as [Jenkins](https://jenkins.io/), [CircleCI](https://circleci.com/), [Bamboo](https://www.atlassian.com/software/bamboo), [GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions), and [TeamCity](https://www.jetbrains.com/teamcity/). I’ll use CircleCI to set up the pipeline for this app because it takes very little configuration and it integrates well with GitHub, Bitbucket, and others. To use CircleCI, you can [sign up](https://circleci.com/signup/) for a free account and connect it to the GitHub repo you want to set up a pipeline for. Once you are signed up, I encourage you to go through the Quickstart Guide to connect to your repo and create your initial CircleCI config file. You’ll see something like[Figure 27-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#initial_circleci_pipeline_setup) once you’re logged in and you have to choose which GitHub repo to set up a new pipeline project in.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2703.png)

###### Figure 27-3. Initial CircleCI pipeline setup

I selected the `main` branch to set up the pipeline, and I chose the Fastest option. Your pipeline config file will be in the new _.circleci_ directory at the root of your project in the _config.yml_ file. Now if you go to your GitHub repo, you should see a new branch called `circleci-project-setup`, which will have your initial CI/CD pipeline config file. Open a PR to merge this to your `main` branch, and you’ll see the new file, as in [Figure 27-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#github_pr_to_add_cisoliduscd_config_gen).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2704.png)

###### Figure 27-4. GitHub PR to add CI/CD config generated by CircleCI

You’ll make plenty of changes to this file to set up your own stages and steps, but for now, you want to make sure your pipeline works from your repo. Open a PR with any changes, even just an update to the README, and you should see something like [Figure 27-5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#circleci_status_on_github_pr) in GitHub.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2705.png)

###### Figure 27-5. CircleCI status on GitHub PR

Now you have this check that will run on every PR you and the team create. To find out what exactly happens with the checks, go to your CircleCI dashboard, and you should see something like [Figure 27-6](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#circleci_dashboard).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2706.png)

###### Figure 27-6. CircleCI dashboard

From here, you should click on the say-hello-workflow and then click the say-hello job within it. This will show you the steps that have been run for this job, as shown in [Figure 27-7](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#say_hello_job_steps).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2707.png)

###### Figure 27-7. Say-hello job steps

Since you have confirmation that the pipeline is connected to your repo, you can start building your own stages and make a pipeline that deploys the code to production and other environments. It’s a good idea to do this one stage at a time so that you can make sure each works like you expect. This will save you time on debugging your pipeline configs.

Let’s start with the build and test stages we outlined earlier. In your _config.yml_, add the following code:

```
version
```

This is a very basic setup, and you should work with your DevOps team to make something more robust than this, but it’s enough to get a pipeline in place to run a few checks. Since CircleCI is only one of the tools you might use, I’ll give a brief overview of what the different parts of this file mean, but you should read about the [concepts that CircleCI uses](https://circleci.com/docs/concepts/) to get a deeper understanding. For example, you will likely have [sequential jobs](https://circleci.com/docs/workflows/) run based on the results of previous jobs, and there are a few of those present in this config.

The first thing you will notice in the config is the `version`. This is the version of CircleCI your pipeline will use, and it determines what [keys](https://circleci.com/docs/reusing-config/) you have available to use in your pipeline. Then there is the `orbs` section. [Orbs](https://circleci.com/docs/orb-concepts/) are shared packages that can contain jobs and commands that simplify your config file. Many third-party services will have orbs available, so you don’t have to write all of your commands from scratch. That’s how you’re using Cypress for your integration testing and Snyk for your security scan in the pipeline without having to set up anything.

Next, you have the `jobs` section. [Jobs](https://circleci.com/docs/jobs-steps/) contain all the steps you want to run at various stages in your pipeline. They will be one of the most detailed sections in your config because jobs are the building blocks for everything you do in the pipeline. Last, you have the `workflows` section. [Workflows](https://circleci.com/docs/workflows/) are how you put the jobs together in the order you want them to run in your pipeline based on the requirements you set.

All of these sections can be used in more advanced ways where you write custom scripts for jobs and have more granular requirements in your workflows. You can eventually get to a place where you have workflows and jobs for container orchestration with [Kubernetes](https://kubernetes.io/docs/home/), but that’ll be something for the DevOps team to handle. For now, if you push the changes to your config to your repo you’ll see something like [Figure 27-8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#circleci_with_new_workflows) in CircleCI.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2708.png)

###### Figure 27-8. CircleCI with new workflows

You can go into the build-and-tests workflow to see the details of the jobs being run in it, as shown in [Figure 27-9](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#jobs_in_the_build_and_tests_workflow).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2709.png)

###### Figure 27-9. Jobs in the build-and-tests workflow

Take note of how long each workflow and their jobs take so that you’re aware of how long your pipeline stages take to run as a whole. This information will help guide you as you add more jobs and steps. This is how you’ll be able to find places to improve your pipeline.

You now have a CI pipeline in place, and the only thing left is to work with the DevOps team to handle the deploy stage where your build artifact is passed to your cloud services to run on the servers they’ve set up. This last bit with the DevOps team is the _CD_ part of the CI/CD pipeline.

# Environment Pipelines

The last part of your pipeline setup is handling multiple environments for your apps. For both backend and frontend apps, it’s common to have development, staging, and production environments. There might be additional feature environments for the frontend because those changes may need to be tested in isolation before being promoted to the next environment. If you do have feature environments for the frontend, this is usually the first environment that changes get deployed to. Let’s take a look at these different environments in a little more detail.

## Feature Environment

The _feature environment_ is a place where larger frontend changes get tested first before they are merged into the rest of the code. When you are refactoring a large piece of the frontend app and it changes views or data handling substantially, this is a good place to make sure the changes don’t break everything. Having feature environments will keep the rest of the team unblocked as well because they can still merge their smaller changes to the development environment and do their testing in a larger system.

## Development Environment

The _development environment_ is usually a place where the dev team can do preliminary deployments to test how their changes work with the overall system. The deploys here can happen in an ad hoc manner as long as the team communicates with one another when changes are being deployed. You might even bring the QA team in here to do some quick tests or just to give them a demo of what to expect soon. The development environment for the frontend and backend are usually connected to each other so that you can test how your full stack changes will work. Often, the feature environments for the frontend are connected to the development environment for the backend. This is also a good place to make sure your feature flags are working as expected, like we discussed in [Chapter 25](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#making_deployments).

## Staging Environment

The _staging environment_ is where you’ll check that the changes you’ve made work as expected. You’ll work with the QA team here to do their feature and regression testing. You can also give the Product and Design teams access to this environment so that they can do user acceptance testing and make any final requests. This is where your frontend and backend changes should be in sync and you are connected to any third-party services to test a fully integrated app. It should be as close to production as possible. You’ll likely have test credentials for your services, and you may decide to use them only in the staging environment or in development as well.

## Production Environment

The _production environment_ is the one where users are interacting with the app. After all the testing in the previous environments, you should be fully aware of what to expect from a user’s perspective. This is usually where the best server resources are, and it’s where you want to make sure you update your environment variables for third-party services and cloud services.

## Environment Variables

You can use environment variables to control the deployments to your different environments and set up connections in different environments. For example, it’s very likely that your DevOps team will have an entirely separate infrastructure in place for the development environment compared to the production one. That includes things like separate databases and event managers.

There are a few approaches you can take to manage the env vars in your pipelines. You can set up variables for each environment in CircleCi in the Project Settings and then reference them in the _config.yml_. Now anywhere in your apps that reference the env var will use the value you’ve set. Here are the new jobs and workflow in the _config.yml:_

```
…
```

[Figure 27-10](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#environment_variable_in_circleci) shows what this will look like in CircleCI.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2710.png)

###### Figure 27-10. Environment variable in CircleCI

These new jobs will require close work with the DevOps team to make sure you connect to the infrastructure as expected. In this example, note that you connect the app to different versions of the `API_URL` environment variable and that your deploy workflow is dependent on which branch changes have been merged to. Another thing to note is that in this workflow, the build and test stages are required to pass first before you deploy to production, but not the other environments. This check is bypassed for the sake of the example, but it’s something you should have in place in your real pipeline. The new pipeline is shown in [Figure 27-11](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#production_deployment_in_circleci).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2711.png)

###### Figure 27-11. Production deployment in CircleCI

Another approach to handling env vars is to have separate _.env_ files for each environment. This is a risker approach that can be done in a private repo or with a script that updates these values in memory when the app passes through the pipeline. Otherwise, you risk everyone having access to your production and test credentials. Keep in mind that one of the reasons env vars are considered a best practice is because there’s no persistent record of them anywhere except in your cloud provider and they have a secure infrastructure to handle them. Any other time, your env vars are loaded in memory and don’t persist outside of it.

With the different deployments you have in your pipeline, all that’s left is working with your DevOps team to connect everything to services and infrastructure and testing that the environments are working like you expect.

# Conclusion

In this chapter, I went over building your own CI/CD pipeline. You learned about the different stages and jobs that are necessary for a simple pipeline. I encourage you to take a look at the pipelines at your organization. There are a number of environments that you’ll deploy to, and you have to account for the differences between them in your pipelines.

You were able to build a pipeline with CircleCI, which is just one of the many tools you can use to deploy your code. I recommend checking out some of the other pipeline tools to learn how they work compared to what we did here. As long as you have the stages we’ve covered here, you can work with your DevOps team to create more advanced functionality.