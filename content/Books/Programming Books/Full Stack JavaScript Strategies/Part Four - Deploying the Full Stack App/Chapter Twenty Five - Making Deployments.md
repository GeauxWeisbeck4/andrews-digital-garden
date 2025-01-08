---
id: 01JH1NY244QSJQY7XTW0GQZ2ER
modified: 2025-01-07T19:44:59-05:00
---
# Chapter 25. Making Deployments

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 25th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Now that you have integration testing in place, it’s time to decide when you want to do deployments and what the strategy behind them should be. Every deployment you do is meaningful to someone, whether it’s the dev working on a change, a QA engineer doing feature and regression testing, stakeholders taking a look at things, or your end users. Each deployment also affects something else. When you push changes to the backend, it’s going to affect the frontend even if it’s indirectly. Pushing frontend changes affects anyone who works with the product through that part of the app.

Having a strategy and understanding the timing around deployments are essential to avoiding surprises. When you roll out changes, that affects the organization and your team’s reputation for quality and reliability. These are some of the things you have to think about. It can be easy to consider your job done as soon as all of the code changes have been merged, but this is the beginning of one of your more visible jobs.

In this chapter, I’ll cover:

- Frontend-only or backend-only updates
    
- Blue-green deploys
    
- Canary deploys
    
- Strategies for doing rollbacks
    

Just like you had to consider a lot of “behind the scenes” things for the initial development across the full stack, you have to do that with deployments, too. This is the planning part of the deployments from a holistic view. When you were building the app, you made sure everything worked with your tools and technology. Now you need to think about the overall impact on the product, the teams involved, and the users. You have to find the balance between speed and risk.

# Deploying Frontend-Only or Backend-Only Updates

Something that’s going to happen regularly is that the backend and frontend will release on different schedules. Not every change will require deployments of both apps while others will take some coordination across the stack. The thing to consider is how the parts of the stack affect each other and the services they interact with. The timing of the deployments is important, especially when you know that you need to wait for an update on one side of the stack or the other.

When you are working on backend features, you need to understand what impact the changes you make will have on other parts of the system. One task is to check if the updates will change the data that the frontend expects. If that’s the case, then you’re introducing a breaking change that will need to be accounted for. Another task is to check if you are changing data being sent to a third-party service. This could be because of an update from the service or because you’re refactoring code to be more manageable. You also need to review if the changes will affect how the app works with infrastructure tools, such as the database.

Determine if your jobs will be affected by the changes. Sometimes a task like refactoring can lead to unexpected side effects. Updating the types or restructuring the folders can break things you aren’t testing for. Unit and integration tests can help you catch many regressions, but sometimes things slip through. The backend usually touches more pieces of the system than the frontend, and that’s when your documentation and architecture diagrams become really useful. So when you make updates, you know exactly how everything connects and what will be impacted.

The frontend can usually be deployed without affecting other parts of the system. What you need to watch for when deploying the frontend is that the backend and other services are ready. If you deploy a frontend change before the backend is deployed, then you risk breaking part of the app for the users. You also have to be aware of potential overlaps with changes from other devs, especially when multiple people are working in the same files. That could lead to functionality or designs getting overwritten.

Sometimes the designs change for one part of the app, and it looks inconsistent with the rest of the app. So you and the team have to be aware of the effects of changing shared components. Updating a component to match designs for one part of the app might break the layout for another part. Check any references for shared components to make sure that your changes don’t have unwanted side effects before you deploy them to users.

When you do separate deployments, always check that the frontend and backend work as expected in the develop environment first. It will help if you version your frontend and backend releases and use slightly different numbers. For example, the backend could be a number ahead on version 2.0.0 while the frontend is on version 1.0.0. Those version numbers go in the [_package.json file_](https://docs.npmjs.com/creating-a-package-json-file#example) for each project. It also helps to specify the version of the backend that the frontend depends on.

You and the dev team are the first line of QA, which makes it crucial that you are paying attention to these details. This is especially true when you’ve been focused on one aspect of the app for a while. Have a checklist or automation in place to help everyone catch things they might not see because they’ve been in the details of one side of the stack.

# Deploy Strategies

As you work on projects across different teams at different organizations, you’ll get exposed to numerous ways of handling deployments. What you do at an enterprise-level organization will be vastly different from what you do at a mid-size organization or a startup. There will be considerations like when to scale up infrastructure resources, how to plan work for new features, and what the priorities are for the dev team. This is another time when you’ll be less focused on code and technical implementations and you’ll be working closer with the Product team.

## Release Dates

One important thing you and the Product team have to coordinate will be actual release dates. These are the deadlines that the dev team needs to meet to have features and fixes in production. Usually, the Product team has talked with a number of stakeholders to come up with a release date, and then they’ll bring it back to your dev team. This is when you need to make sure you and the team have a good understanding of everything needed for the release.

Ask for designs and specs and take some time to discuss them as a team so that you can bring up any questions from a technical standpoint. If there’s anything missing from the Product side, push back and let them know you can’t commit to a release date until you have all the details you need to do the work. Show Product what’s missing and work with them to fill in the missing pieces. Committing to work that hasn’t been thoroughly laid out will set the team up for unnecessary stress because they’ll try to squeeze in a lot of code at the last minute to meet a deadline.

# KEEP CONSTANT COMMUNICATION BETWEEN THE DEV TEAM AND THE PRODUCT TEAM

Now part of your role becomes more about how you communicate with technical and nontechnical teams. That includes considering the reputation of the dev team. That should mostly be handled by your manager or tech lead, but sometimes the responsibility falls on you. That’s especially true if your manager isn’t technical. You don’t want your team to become thought of as always missing deadlines or constantly pushing bad changes or incomplete requirements to production, even if it’s not your fault.

When you’re building your deployment strategy, remember to include some breathing room for the team in case things go wrong for reasons out of your control. Take an optimistically cautious approach to approving the deployment PRs and really test them as much as you can before they get to QA. It is an extra step for you, but it’s one that pays off tremendously when you catch things before anyone else.

The Product team will lean on the dev team heavily to understand what’s needed on the technical side so that they can communicate that to stakeholders. When you discuss the requirements with your team, a good strategy is to have a dev deadline that only you all know about. Always remember that the release date is when Product needs to have the features in production. That means all the code needs to be merged, tested, and working without bugs before then. It can be a good exercise to take the release date and work backward from it to see if you and the team can get all the changes in and tested, as shown in [Figure 25-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#release_timeline_example).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2501.png)

###### Figure 25-1. Release timeline example

The goal is to have a continuous release cycle where every small change goes to production as quickly as it can get through QA. That way, you aren’t as worried about doing large releases at the same time. The size of the organization will affect production release schedules because you’ll have a varied number of teams that need to handle different parts. Larger organizations may involve legal teams in their releases to make sure terms and conditions are well defined for new features. There might be bigger DevOps teams you need to work with to get support for your release date.

So you might end up doing only three to four production releases a year, even though you’ve been working on features and making lots of releases to your staging environment. If you’re at a smaller organization, production releases can happen as often as every few days or weeks. At mid-size organizations and startups, you’ll be able to do a lot more yourself and have considerably fewer teams to talk to. That will speed up the time it takes to release features. But even in these organizations, you’ll reach a point where you might release to production only every few months just because there aren’t enough people to get through all the work as the organization grows.

As you go through the requirements for features, update your dev docs with any conditions that the app needs to account for. This is something that will help you when you’re discussing more features in the future. Also find out what’s on the Product roadmap so that you can work with the team to prioritize tasks. The Product team should stay on top of prioritization, but it’s important for you to check in with them and see if timelines have shifted based on info you’ve given them from the technical side.

Some industries, like advertising and finance, will have certain parts of the year where no releases to production can happen because that’s when customers are spending the most money. These are referred to as “freeze periods,” and they usually happen around major holidays. If you know there’s some tech debt that needs to get worked on and you have feature work to implement, you can try to shift the tech debt to these freeze periods so that you can focus on releases.

## Version Releases

Another strategy that will help you debug issues is versioning the releases of your apps, as mentioned earlier in the chapter. This is especially true on the backend where API changes could break more than the frontend. When you’re doing a versioned release on the backend, you typically keep the current version for a period of time. That gives the frontend and any other consuming apps a chance to upgrade to the newest version without immediately breaking their current functionality. This is a form of graceful degradation.

Once the degradation date comes, you can choose to leave the old version available with a note saying that it’s no longer supported and users need to upgrade. Or you can completely delete the old version. This is a decision you’ll have to work with the Product team and stakeholders on, especially if it causes more work for the dev team outside the normal feature development flow.

Following [semantic versioning](https://semver.org/) is a way to help everyone understand the types of changes they can expect in a new version. Semantic versioning shows when breaking changes, reverse-compatible changes, or bug fixes are released with just a number. Honestly, I’ve seen this followed very loosely around organizations to the point that some teams will release v0.193.36 without ever releasing a v1.0.0 version. Since you want to use best practices and make things better for everyone, follow semantic versioning as much as you can. CHANGELOGs become crucial when you do versioned releases. They provide a quick way to see what the differences between versions are and are one of the ways others can debug their own apps.

There are plenty of tools you can add to your Git hooks or release process to make semantic versioning more automatic. Some of the tools you can use are [release-please](https://github.com/googleapis/release-please), [commit-and-tag-version](https://github.com/absolute-version/commit-and-tag-version), and [release-it](https://github.com/release-it/release-it#readme). By doing [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/), like you’ll do in your Git hooks in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline), you set your team up for an easy transition into semantic versioning. Something else you can do is run the [`npm-version`](https://docs.npmjs.com/cli/v8/commands/npm-version) command to update the package version. All of these update the package version in your _package.json_ and _package-lock.json_. You can also use [Git tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging) so that you have artifacts for each version of your app.

On the frontend, versioning the UI can be a tool for debugging and handling production issues. Sometimes deployments fail silently, and your changes don’t make it to the server like you expected. Without having a version in the UI, it can take a while to figure out that the root cause is a failed deployment. This is also a great tool for the Support team when they have users contact them about issues. The frontend usually doesn’t follow graceful degradation because there should be only one version of the UI for all users.

Frontend versioning usually means showing the app version in a component on the page that is always rendered. In the example app you built, that would be the nav bar for the app. In other apps, you might find a version number in the footer or somewhere else inconspicuous. This isn’t something the user will have to be aware of normally. It should just be easy for them to find if they need to contact Support. One thing to consider is that the version number should be available in the UI regardless of whether a user is logged in.

You’ll see all this versioning done in the wild with third-party services. Many third-party services use semantic versioning and degradation for releasing changes. They announce a deadline to make the upgrades so that users have a chance to do what they need to work with the new version. For example, say the current version of your service is v1.3 and the new version is v2.0. You know this version number means there are breaking changes, so you’ll want to update the service version in your apps and test as soon as you can.

## Blue-Green Deploys

When you know that your app receives a lot of traffic and you can’t risk downtime for all your users, blue-green deployment is a good strategy. “Blue” and “green” refer to the environments the app will run in. For example, the blue environment may run the older version of the app, and the green environment runs the new version, but the color you use for each can switch. This lets you gradually move traffic from the old version of an app to the new one. That means you’ll have two versions of the same app running in production at the same time with different amounts of traffic hitting them.

If one of your key metrics for your apps is uptime, you should discuss this approach with everyone because it will require more effort. You might start by transferring 10% of the traffic to the green environment and see how well your changes work. As you get confirmation that everything works as expected, you can gradually increase the amount of traffic to the green environment. Each time you increase the traffic, take a look at your monitoring tools to see how error rates are doing and how resource usage is adjusting.

Eventually, you’ll reach a point where all your traffic is in the green environment. Then the blue environment can be on standby in case something happens and you need to quickly switch traffic back over. Once you are sure the green environment is really providing the uptime you need, then you can shut off the blue environment to save on costs. Your DevOps team might decide to use this as a template for the next update. That means the blue and green environments switch roles after each deploy, but that’s more for their convenience.

Blue-green deployments are useful for a number of reasons other than uptime. You can do A/B testing easily to see how a percentage of users interact with new features before doing a widespread release. That can give the Product team research data that guides the future of the roadmap. You can monitor your key metrics to see if there’s a significant difference between the two versions of the app. Blue-green deployments also let you test changes in production. All of the teams can do last-minute testing to verify that everything works as expected and then switch the traffic over because the changes are already in production.

As with any strategy, there are downsides to this approach. Setting up environments and resources for blue-green deployments takes a lot of time and effort. It’s a costly strategy because you are working with two production-level environments that have to be provisioned with the same resources. It takes a lot of testing to ensure that both environments really do work the exact same. You can run into issues with keeping data in sync across the environments. That’s especially true if one environment has schema changes that the other doesn’t. Containerization can help with this; we’ll talk about that in [Chapter 26](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch26.html#integration_concerns).

This is true for any external services because you’ll be using the same credentials to connect from different environments. You have to check the license agreements with your services to see if that’s possible. Check for any indirect interactions between the environments through the external services. That can lead to unexpected data or config changes that make both environments unreliable. There’s sometimes a risk that data leaks between the two environments. For example, you might have some cached DNS records that point the blue frontend to the green backend, and that can get even crazier if the DNS records get updated at different times because of geographic location. Blue-green deployments work for both backend and frontend apps, but just be aware of the risks compared with the value.

## Canary Deploys

Canary deployments are similar to blue-green deployments with slight differences. With canary deployments, you don’t need two full production environments. Canary deployments take advantage of a feature-flagged approach of releasing new changes. That way, you can deploy the changes to the production environment, but they are hidden from users until you give them access. This requires some additional setup in your app to handle toggling on and off pieces of code, so you’ll have to work with the team to think through the best way to do that.

Canary deployments allow you to have the new features running in parallel with the existing features instead of separated in a different environment. You can think about these deployments as slowly unveiling a feature in production instead of having the new version in one environment and the old version in another. This is like doing [progressive delivery](https://www.split.io/solutions/progressive-delivery/) or [staged rollout](https://developerexperience.io/articles/staged-rollout).

The dev team will have to make more architectural decisions to handle the feature-flagging functionality. You need a way to be able to manage feature flags by user groups in a way that is sustainable.

It’s easy to forget to fully enable a feature, and then the Support team starts getting issue reports. Or you might deprecate a feature flag too soon and open new features to users before they are fully complete. You can choose to create your own feature-flag management tool, or you can use a tool like [FeatureFlags](https://featureflags.io/), [LaunchDarkly](https://docs.launchdarkly.com/home), [Unleash](https://www.getunleash.io/), and [PostHog](https://github.com/PostHog/posthog). No matter which option you choose, make sure you document all the flags you have in the app, the state of rollout they’re in, and the user groups that have access to them.

Consider having a ticket as a reminder to clean up feature flags as time goes on so that you and the team don’t lose track of what’s still relevant. Using feature flags provides a few advantages, such as more visibility of upcoming features to everyone and getting the dev team used to creating stable PRs, and it encourages more consideration around how you implement a robust feature-flag system.

Canary deployments shouldn’t be confused with canary releases. A _canary release_ is how you can test an early version of an app with users who like to adopt tools early. You’ll see this a lot with open source projects when they have nonstable versions. These will have separate version numbers than normal stable releases. You can also see this with larger organizations, such as Chrome canary releases.

These are a few strategies you can use to get your frontend and backend changes out to users. Once the changes have been deployed, you still have to account for things going wrong and have plans to handle that.

# Strategies for Doing Rollbacks

Despite all your planning and testing, there will be times when things go wrong in production. If everything was working fine before your last deployment, one assumption you can make is that it’s something wrong with the code. While that may not always be the case, this is the area you have the most control over. With that in mind, you need to have contingency plans to be as ready as you can for anything that comes up.

## Deploying Older Versions

A strategy for rolling back a deployment is to redeploy the previous app artifact. This becomes simpler if you use Git tags for your release versions or you have somewhere in the cloud where all your previous versioned builds are stored. If you follow blue-green deployments or canary deployments, that also simplifies this rollback strategy for you. You don’t have to do anything to your code or open new PRs to trigger your pipeline.

This is where you’ll have to work more with the DevOps team. They can do this for you, or maybe there’s a user interface where you can select which version you want to run on production. Either way, this is a fast rollback strategy that can take the pressure off the dev team while they track down the root cause of the issue.

## Reverting or Resetting PRs

Another strategy is to revert the release PR. For our example, this means you remove the code that was merged into the main branch that triggered the deployment in your pipeline. You’ll have to open a new PR that targets the main branch, and that will trigger a new deployment in your pipeline to go back to the previous working version of your code. There are a few ways to do this with some Git commands. The Git commands are only briefly covered here because [Chapter 28](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#git_management) is an in-depth discussion of Git and commands you can use.

One way is to use the [`git revert`](https://git-scm.com/docs/git-revert) command with a specific commit ID, as in [Figure 25-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#example_of_git_revert). You may be tempted to use this command, but unless you understand what this [doc on reverts](https://github.com/git/git/blob/master/Documentation/howto/revert-a-faulty-merge.txt) is talking about, don’t use this. Check out the book [_Version Control with Git_](https://www.oreilly.com/library/view/version-control-with/9781492091189/) by Prem Kumar Ponuthorai and Jon Loeliger (O’Reilly) if you really want to become a Git expert. As a very simple explanation of what happens when you run `git revert`, that removes only the changes in your workspace, not the Git history. It’s like hitting an undo button locally. But again, unless you’re a Git expert, it’s probably best to avoid this command.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2502.png)

###### Figure 25-2. Example of `git revert`

If you need to undo multiple commits, you should consider using the [`git reset`](https://git-scm.com/docs/git-reset) command. This will undo all the changes that happened after the commit you specify. So this command resets commits cumulatively, not just the commit you specify. The reset option you choose is very important here. If you run `git reset –soft`, this will uncommit your changes and leave them as staged, like in [Figure 25-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#example_of_git_reset_en_dashsoft). That way, you can review them one more time and decide how to handle them.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2503.png)

###### Figure 25-3. Example of `git reset –soft`

If you run `git reset –mixed`, this will uncommit your changes and move them out of the staged state, as shown in [Figure 25-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#example_of_git_reset_en_dashmixed). So the reset commits will look like changes you just made locally. The other option you have is running `git reset –hard`. This will uncommit your changes, move them from the staged state, and delete them, as in [Figure 25-5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#example_of_git_reset_en_dashhard). Be careful when you do a hard reset because you completely lose those changes.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2504.png)

###### Figure 25-4. Example of `git reset –mixed`

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2505.png)

###### Figure 25-5. Example of `git reset –hard`

Many developers confuse the `revert` and `reset` commands because you often only have to undo the most recent commit. Both commands work fine in this scenario. But as soon as you need to undo more than one commit, the difference between the commands and the options you use becomes important to understand.

The easiest and simplest approaches are to redeploy an older artifact, switch traffic to your blue or green environment, or rerelease a previous tagged version. You need to be very comfortable with Git commands and the reasons for using them or else you can introduce even more issues. Take some time to play with these commands and see how they change the code.

## Deploying a Hotfix

This is more of a roll-forward strategy because you don’t undo anything. The dev team works as quickly as possible to find the root cause, patch the code, and make a new PR to the main branch. Sometimes this approach makes sense because the infrastructure isn’t set up to handle rollbacks and the repo isn’t set up to handle reverts. So your only choice is to move forward. This can place stress on the team, especially if downtime or bugs are costing users and the organization money.

If you have to work with a hotfix strategy, start by looking at the logs and the last commit because this is likely where the issue is. This is one of those times when your experience will really come in handy. Stay calm and go through the same debugging process you would in any other environment. A hotfix will not go through the same QA process as your normal bug fixes and feature deployments. At this point, it’s up to you and the dev team to test the specific issue that’s happening. You should get on a group call so that several people can verify the fix together. Including QA and Product in that group call will help you verify other functionality so that you have full confidence that the hotfix you’re about to push doesn’t cause more issues.

# HAVE A REALISTIC APPROACH TO HOTFIXES

Ethan Brown made this additional comment on hotfixes:

> It’s reasonable and realistic to create a “hotfix protocol” that may include lowered review, test, and check requirements as long as the last step in the process is “do all the review, test, and check that are part of the normal process as soon as the crisis is over.” In the meantime, getting the hotfix out takes precedence over all other work. When pressures get high, there are two types of people: people who confess to cutting corners and liars. So let’s not shame the corner cutters or let the liars get away with their lies. Make a process that fits the situation and include in that process a way to “uncut” those corners when the fire is out.

# Conclusion

In this chapter, I covered some strategies you can use for your deployments. These are just a few approaches you can take, so feel free to get more creative with them. You may find that a combination of strategies will help your situation the best. Make sure to partner with the DevOps and QA teams closely here. The success of your releases is only as good as the infrastructure the code is run on and the thoroughness of your testing. Reach out to them to make sure everyone knows the plan and is aware of release dates.

Doing deployments shouldn’t be a stressful time for the team when you know that you’ve given yourself space to handle bugs and prep the cloud resources. Constant and frequent communication with the Product team and stakeholders will avoid surprises for everyone. Let them know when you run into issues, even if it doesn’t seem like it will delay their timelines. Keeping everyone updated should be a part of any strategy you and the teams decide to implement, and that will pay off in the form of more trust and freedom for you and your team.