---
id: 01JH1P0MVCG1B685A6JX87T0YH
modified: 2025-01-07T19:46:25-05:00
---
# Chapter 28. Git Management

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 28th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Understanding Git is one of the best skills you can cultivate. It doesn’t matter if you work on the full stack, specialize on the frontend or backend, or decide to learn a different programming language. You need to know how to manage the changes that happen in the repo. The team will look to you for guidance on how to handle their branches and how changes are merged across the shared branches.

You’ve done a lot of good setup for your team by defining code conventions, using Git hooks to enforce standard commits, adding rules to your branches in GitHub, and following a consistent PR review process. Now you need to go a little further and understand various commands and strategies you can use to minimize conflicts as multiple developers push changes to the remote repo.

In this chapter, I’ll cover:

- Branching strategies
    
- Managing merging with Git commands
    
- Handling merge conflicts
    

One of the trickiest things to do is untangle branches once changes have been merged because you have to make sure to keep the correct version of changes. It always helps to have a few techniques in mind for when conflicts happen, especially around deploy time. I’ll go through some of the ways I normally manage branches and merges from different teams as well as some other methods I’ve been introduced to over the years. Hopefully, these options will help give you ideas for what you might do.

# Branching Strategies

There are a number of ways to handle branches as you have changes being developed concurrently. A few things you want to keep in mind are overlapping file changes, package version updates, how large a feature is, when the feature is supposed to be released, and if there are any app-wide changes. These considerations will help you figure out how you and the team want to handle the changes that are coming in. Some of your branch strategy will be driven by how your deployment pipelines have been defined.

## Common Branches and Merge Flow

As we discussed in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline), there will be different environments that your code gets deployed to. These environments are typically connected to certain branches. A common strategy is to call those branches `main`, `staging`, and `develop`. The names might vary at different organizations, but their uses will be similar. The `main` branch is for production deployments, the `staging` branch is for testing changes before they are released to production, and the `develop` branch is where the dev team will do its own validation before involving QA or other teams. Both the `staging` and `develop` branches are typically connected to a deployment environment.

When [PRs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) are ready to be released to production, the typical flow is to merge a PR to `develop`. There will be some initial dev team checks once the changes are on develop. After develop has been cleared, the changes will go to `staging` for QA and Product approval. Once everything has been validated on staging, then the changes get merged to main, and the production deployment is triggered.

###### TIP

Keep in mind that PRs are a feature of GitHub or Bitbucket, not Git itself. GitLab refers to the same concept as a _merge request._ You can merge changes using Git without ever creating a PR.

You can think of `main` as the source of truth for the app as the users are currently experiencing it. If there’s ever a question about which features are available or what changes are the correct ones, `main` is usually the branch you want to refer to because of how it affects the users. The `staging` branch is the source of truth for all the changes that are ready to be released to users. It probably contains fewer changes than the `develop` branch, and it has functionality the `main` branch doesn’t. The `develop` branch is usually the source of truth for the latest features that have been approved by the dev team. So when you’re starting a new task, this is typically the branch you want to work from. [Figure 28-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#pr_branch_merging_flow) is an example of this flow.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2801.png)

###### Figure 28-1. PR branch merging flow

You’ll want to treat the `main` and `staging` branches carefully. They usually have rules around them to prevent any direct pushes except for in the case of hotfixes, which was discussed in [Chapter 25](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#making_deployments). These branches are connected to environments that shouldn’t experience downtime, especially `main` since it’s connected to the production environment. You want to protect the `develop` branch as well, but maybe to a lesser degree.

Ensuring that the unit tests and build are successful is fine here because someone may need to validate their changes in a cloud environment before they’re ready to get the code reviewed. There are times when you’ll want to merge smaller changes to a shared branch to help unblock someone else’s work. That can include adding new components to the frontend like buttons or modals or updating response values from an endpoint.

## PR Reviews

The reasons we review each other’s code include:

- Ensuring that quality code gets merged to the shared branches
    
- Checking that there aren’t any obvious bugs getting to production
    
- Helping to share and spread knowledge across the team
    
- Fostering communication and collaboration on the team
    

Remember that the purpose of the code review isn’t to criticize one another or make the comments personal. You won’t have the same context for a code change as the person who made it, so be humble and ask questions if there’s something that seems odd to you.

There are multiple things you’ll want to check for when you do a PR review. Here’s a checklist of items you may want to include:

- Require at least one approval from another dev to confirm that someone other than the author has seen the new code.
    
- Make sure the unit tests pass and the build is successful when pushing changes to `main` and `staging`.
    
- Check for anything that goes against the code conventions the team has in place, such as naming or function formatting.
    
- Pull down the branch and make sure the app runs.
    
- Run the app locally to see if the functionality works as expected.
    
- Go through the changes line by line and make sure you understand them.
    
- If there’s something you don’t understand, ask questions about it because you might not be the only one.
    
- Try not to impose your own coding style on another dev because there are always multiple ways to approach the same task.
    
- Be mindful of how conditions are written and how third-party services are called.
    
- Double-check that data is sent and returned as expected.
    

You can add [any number of tasks](https://github.com/cfpb/development/blob/main/guides/code-reviews.md) to the PR review, and you can [make a PR template](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository) that’s automatically available on all PRs. This checklist has a short number of items to check for, so work with your team to determine what’s best for your reviews. One way to make reviews smoother is deciding how to handle branches.

## Branches for Smaller Functionality

Often each branch you create will be associated with a ticket, especially with bug fixes and smaller one-off tasks. Every time you need to make a new branch, you should pull down the latest changes from `develop`. I usually pull changes every day and sometimes multiple times throughout the day just to make sure I’m working with the latest code. This will ensure you aren’t working with older code that could cause merge conflicts later.

With these branches, try to make small commits when you can. Let’s take the user sign-up form task as an example. This will require new types, a new component, an API call, and maybe some new packages. Making each of these changes its own commit can help you during debugging, but you don’t have to do this. Once you’re ready to merge to `develop`, it’s a good practice to squash all the commits for your individual branch; we’ll talk about that in a bit.

Since you and the team already have conventions for branch names and commit messages from [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline), the intent of your changes is clearer. Having that intent is really helpful when you need to update packages as part of a task because that’s one of the changes that can lead to cascading issues once the branch is merged. You can pinpoint the exact feature or bug fix that holds the updates and undo that one if it starts blocking other developers. So if you’ve merged in multiple tickets and squashed the commits, you’ll still have a flow that looks like [Figure 28-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#multiple_tickets_merged_to_develop).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2802.png)

###### Figure 28-2. Multiple tickets merged to `develop`

As soon as these smaller branches are approved, they should be merged quickly. This will help keep changes flowing to the `develop` branch and prevent branches from becoming stale. A _stale branch_ is one that has sat long enough for `develop` to have updates after it’s already been approved. Small, quick merges will help mitigate conflicts and keep the dev team in sync with changes.

## Feature Branches for Larger Implementations

Sometimes a larger feature will be under development. This type of work will add significant new functionality to the app. It may or may not have anything to do with existing code, so this work can usually be implemented in isolation from other changes in progress.

###### TIP

Remember that feature flags, which we discussed in [Chapter 25](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#making_deployments), are also an option for this. You can use feature flags and feature branches if you want to do a more fine-grained rollout, or you can choose one over the other. When you’re developing a large feature that needs multiple small parts to work, then a feature branch may be preferred. If you’re developing a smaller, high-impact feature, then feature flags may be the way to go.

When the team is working on multiple large features, it’s a good idea to consider creating separate branches for them that you merge smaller changes into as you build more of the functionality. That way, you aren’t blocking the `develop` branch, and the team can review the feature in smaller chunks. When it’s time to merge to `develop`, the individual pieces have been thoroughly reviewed, and the feature can be evaluated at a higher level. Having a separate feature branch also makes testing easier because you can focus on one particular feature without worrying about any other changes. The flow for this strategy is shown in [Figure 28-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#feature_branch_flow).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2803.png)

###### Figure 28-3. Feature branch flow

In this example, you’re working on a feature called “onboarding,” so you’ve created a feature branch called `feature/onboarding` based on the `develop` branch. After you create the `feature/onboarding` branch, then every task you work on related to onboarding will have its own branch based on it. The `task/sign-up-form` branch was based on `feature/onboarding` just like the other task branches. These smaller units of work will get reviewed and address any feedback, then get merged to `feature/onboarding`. That way, when you finish all the tasks related to onboarding, the feature branch PR review won’t be as tedious for the team to go through.

This strategy helps you keep track of all the smaller development work needed to make the full feature, which will let you focus on one thing at a time and keep the code quality up with things like types and tests. It helps with release time because you have multiple steps of validation that the full feature is complete. Merging the task work directly into `develop` can lead to a weird user experience because all the functionality isn’t available yet unless you use feature flags. Using a feature branch can give you more time to think through edge cases and ask questions that come up as you develop because you have to break things down into smaller pieces, which can reveal areas that don’t have all the requirements defined.

## Squashing Commits

The most important thing when you merge branches, big or small, is to communicate when things are merged. Most of the time, this will happen in the PR reviews, but when you know a change is going to affect shared parts of the app or core functionality, it helps to explicitly alert the team. That way, you all can work together to make sure there are minimal conflicts between changes.

There are a few ways you and the team might decide to merge branches. The first is by squashing commits from individual ticket branches that get merged to `develop`. When you squash the commits for a PR, that means you take all the commits you’ve made on that branch and roll them together under one commit. You usually don’t need to see all the small commits that went into a bug fix or even a feature. Something to remember is that your small commit messages can be informal, as in the example in [Figure 28-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#squashing_commits_from_an_individual_br).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2804.png)

###### Figure 28-4. Squashing commits from an individual branch to `develop`

When you squash commits from individual branches to `develop`, it helps take the noise out of your Git history so that you can see the changes as an overall feature or bug fix. This is useful when you need to debug because it’s unlikely you would undo a single commit without the context of the other changes associated with it. Squashing the small commits is like keeping the history organized by the tasks and features or by the ticket number, not by the work that went into them.

Usually, you want to merge commits that go from `develop` to `staging` and from `staging` to `main` without squashing them. Because you’ve already squashed the small commits from the individual branches to `develop`, you’re left with the most relevant info in your Git history so that you don’t lose track of those features and tickets that are being tested or released. [Figure 28-5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#squashing_commits_from_develop_to_stagi) is an example of what would happen if you squashed all your commits from `develop` to `staging` and why it makes it harder to know what features are included. You don’t usually want to do this.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2805.png)

###### Figure 28-5. Squashing commits from `develop` to `staging`

Now if you have a bug, there’s no way to go back in the Git history and figure out what feature introduced it. If you saw this entry in the history for the `develop` branch, the only way you could figure out what changes were included is to go back and look at the original PR. That might leave you with missing context. So when you’re merging changes to the shared branches like `develop`, do not squash commits.

###### NOTE

A super important thing to note here is that you need to keep feature branches in sync with all changes that get merged to `develop`. Since feature branches indicate that the work being done is in progress, it’s easy for these branches to get out of sync, depending on how often changes get merged to `develop`. Once your feature branch diverges from `develop`, you end up with merge conflicts that get messy to sort out. At least once a week, you should rebase your feature branch with the changes from `develop`. We’ll go over how to handle that in the next section.

As an exercise, take some time to look through the current Git history at your organization. An example of what that might look like on GitHub is shown in [Figure 28-6](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#project_git_history_in_github).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2806.png)

###### Figure 28-6. Project Git history in GitHub

You can use the `git rebase` command to handle squashing commits, and it would be great if you understood [how that works](https://github.com/flippedcoder/dashboard-web/blob/main/docs/git-rebase.md). Realistically though, you’ll be using the buttons on your PRs to squash your commits. You can see what the button options look like in [Figure 28-7](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#squash_button_in_github).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2807.png)

###### Figure 28-7. Squash button in GitHub

By clicking the “Squash and merge” button, you can bundle all the commits in your PR into a single commit with a message that summarizes the changes. This cleans up your history and can help you focus during debugging efforts. Hopefully, this will also show you why you don’t want to squash commits to your shared branches. It removes the ability to see separate commits. If you really need to get the original history, there are ways to do it, although it can be difficult.

## Rebasing and Merging Branches

Since you know how to squash commits from your individual branches to make a develop-ready commit, you can turn your attention to how you keep feature branches up to date with an upstream branch like `develop`. This will make merging your changes easier when they’re ready for testing. Any new changes on `develop` could introduce merge conflicts that are tedious to fix if you wait too long to update your feature branch. As a rule of thumb, I always pull down the latest changes from `develop` every day and rebase my local feature branches. That way, if there are any conflicts, they will be smaller and more manageable.

This does leave you and the team with a choice to make. Do you all agree on rebasing feature branches, or would you rather merge the changes from `develop`? Let’s take a look at the difference between the two and how they affect your Git history.

When you rebase your branch, that means you are moving the beginning of the history of the feature branch to the end of a source branch, like `develop`. This results in the Git history for your feature branch starting where the latest changes from `develop` begin, as shown in [Figure 28-8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#feature_branch_when_it_is_behind_curren) and [Figure 28-9](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#feature_branch_after_it_has_been_rebase).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2808.png)

###### Figure 28-8. Feature branch when it is behind current changes on `develop`

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2809.png)

###### Figure 28-9. Feature branch after it has been rebased with current changes on `develop`

If you and the team go with the rebase option, be mindful that this can make reverting changes difficult, it can lead to making similar but different commits during merge conflict resolution, and it can break intermediate commits if you perform them incorrectly. You never want to rebase a shared branch, like `develop` or `main`, onto a feature branch for these reasons. Rebasing is very useful for your individual branches because you’re the only one working on them. But if you rebase a shared branch, it can cause major problems, such as removing a number of existing commits. Only rebase the feature branches with the shared branches to avoid this problem.

When you merge `develop` into your feature branch, that means you’re making a commit of the changes from `develop` on your feature branch. That adds the full history of the changes from `develop` to the head of your feature branch instead of moving the beginning of the feature branch history to the latest commit on `develop` as with a rebase. So every time you merge a commit from `develop`, it gets added as the latest change of your feature branch, shown in [Figure 28-10](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#merging_a_branch). That can be nice because rebasing can be a more involved process whereas merging is just one step. It does become an issue when you need to roll back or revert changes, though.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2810.png)

###### Figure 28-10. Merging a branch

###### NOTE

Something else to be mindful of is what you do with your branches after they’ve been merged. It’s good technical hygiene to delete branches after they’ve been merged so that you and the team can manage Git better. Deleting merged branches removes a lot of noise from the repo, and it should be fine since all the changes are already in a shared branch and are recorded in the PR that got merged.

Merging is usually the easier option because it just adds the changes to the end of the feature branch, so you don’t have to decide which parts of individual commits to keep in the history as with rebasing. I encourage you to get familiar with rebasing, though, because it can actually be easier to handle merge conflicts and it can produce better results. There are trade-offs for both approaches, and you might use a combination of both to manage all your branches. You will find a comparison of the methods in [Table 28-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#comparison_of_rebase_and_merge). If you want more details on how rebasing and merging compare with each other, check out [this Atlassian tutorial](https://www.atlassian.com/git/tutorials/merging-vs-rebasing), the book [_Version Control with Git_](https://oreilly.com/library/view/version-control-with/9781492091189/) by Prem Kumar Ponuthorai and Jon Loeliger (O’Reilly), or this article on one of the [revised ways of managing branching](https://nvie.com/posts/a-successful-git-branching-model/).

Table 28-1. Comparison of rebase and merge
|Rebase|Merge|
|---|---|
|More linear and less cluttered history|History cluttered with merge commits|
|Resolving conflicts can be multistep process|Resolving conflicts is a single-step process|
|Not recommended for shared branches|Good for shared branches|
|Rewrites commit hashes|Preserves commit hashes|

You’ll want to work with your team to establish a pattern for when to merge and rebase. On most of the teams I’ve worked with, we used this general flow:

- If you’re developing on an individual branch or a feature branch, you should _rebase_ with `develop` to keep your code up to date.
    
- Once your individual or feature branch has been reviewed, the branch should be _merged_ to `develop`.
    
- After a change is on `develop`, those changes should be _merged_ to `staging`.
    
- From `staging`, the changes should be _merged_ to `main`.
    

That way, you keep the Git process consistent and avoid situations where you may need to undo or cherry-pick changes, which can be a difficult process. The goal is to be able to understand which commits contain each feature. As long as you can go through the Git history and understand when features were deployed and what code was associated with them, you have a good strategy.

# Handling Merge Conflicts

No matter what strategies you implement, eventually you will have to resolve merge conflicts. It’s one of your skills that will be tested many times, and it will expand your knowledge of Git commands, communication, and the project itself. Some conflicts are simple, such as a couple of lines in a single file as shown in [Figure 28-11](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#merge_conflict_in_the_git_resolution_to). Others will span multiple files with large changes. It will require some creativity to handle them without losing important changes or breaking the app. I’ll go over some ways you can approach merge conflicts, so you’ll have a few more tools to work with.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2811.png)

###### Figure 28-11. Merge conflict in the Git resolution tool

## Discuss with the Dev Who Made Changes

Usually, the dev with the conflicted branch is the one who has to fix it. You should reach out to the dev and get them to try to fix the conflicts first because they are the ones who know how a feature is supposed to work. While you will probably be able to solve the conflicts, you may be missing context on which changes to keep. That’s why it’s important to encourage everyone to solve their own branch conflicts before they bring you in. You can make suggestions like merging changes from the shared branch into their branch or doing a rebase.

If it gets to the point where they are stuck, see if you can hop on a screen-sharing call with them and try to work through it. Something you can do to prevent large conflicts with feature branches is to encourage the team to update their branches every day with any changes from `develop`. The conflicts tend to be smaller or at least more focused when they do this. If the feature branch is very out of sync with `develop` and a rebase doesn’t work, it may be a good idea to open a new branch with the latest changes and then re-add the feature code as an absolute last resort.

This is a great opportunity for you to mentor another dev on how to handle merge conflicts and the process you have to go through. Always try to include them in the process of cleaning up their branch so that this task doesn’t turn into something only you do. If you find yourself going through each commit on a shared branch, loop in the dev who added the changes. Remember the goal isn’t to blame them. You’re just trying to get an understanding of what happened so that it can be fixed.

## Use git bisect to Find the Affected Files

Sometimes a merge conflict can be hard to track through the Git history. You might check out a really early commit and see if the bad code is there. Then you check out the next commit in the history and look for the conflicting code there. And you do this process for a while until you find something.

That’s where `git bisect` comes in. This is a tool that does this for you, allowing you to go through commits and mark them as good or bad until you narrow down to the commit that contains the conflict.

Let’s go through an example of this. Start by looking through your Git history to find a good commit hash (one before the merge conflict) and a bad commit hash (one after the merge conflict). Now you can start the process with the following command:

git bisect start

Now you’re in the interactive menu for the bisect process. Keep in mind that you can exit this menu at any time with the command `git bisect reset`. But to continue with the process, enter the following command with your good commit hash:

git bisect good b29cafce7

Then you need to enter the bad commit hash:

git bisect bad f0dd190b8

Based on what you’ve entered, `git bisect` will check out a commit halfway between these two, as in [Figure 28-12](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#selecting_a_commit_between_the_good_and). That’s how this command works: it will continue checking out commits that are halfway between the ones you label good and bad until you find the bad one.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2812.png)

###### Figure 28-12. Selecting a commit between the good and bad ones

At this point, you need to examine the files that have the merge conflicts to determine if the changes are what you expect. If they aren’t, then you’ll enter this command and get an output similar to what’s shown in [Figure 28-13](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#labeling_the_next_bad_commit):

git bisect bad

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2813.png)

###### Figure 28-13. Labeling the next bad commit

This will tell the bisect tool that the commit you currently have checked out is bad, and it will open a new one. This new commit will be halfway between your initial good commit and the latest bad commit. You’ll continue this process until you find the next good commit. Once you have a good commit, you’ll run this command:

git bisect good

Then the bisect tool will give you the hash for the bad commit, and you can do whatever you need to resolve it, as in [Figure 28-14](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#showing_the_bad_commit_and_the_files_th). Doing this can save you hours of time manually going through commits, and it’ll pinpoint bad commits with more accuracy. I encourage you to try this out next time you run into a conflict!

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2814.png)

###### Figure 28-14. Showing the bad commit and the files that were changed

## Manually Compare File Changes

Sometimes merge conflicts are so big and complicated that you and the other devs can’t figure out which changes need to stay and what effect they will have. This is when you might need to resort to a manual method of comparison. You can look at the code that’s already on the `main` and `develop` branches to make sure you aren’t breaking anything. Then you can start looking at changes that were introduced on the feature branches and add them.

This can involve making a separate branch and copying and pasting code from multiple branches into it. Sometimes you run into more confusion when you use the Git resolution tool if you’re facing large conflicts with little context. Usually, when I see a conflict with more than five lines of code in a single file, that’s when I start evaluating my options for resolving them.

When you fix complex merge conflicts like this, make sure to run the app locally. Sometimes things can look fine in the code, but there are still runtime issues. This is where your tests will come in handy because you’ll be able to find some of those unexpectedly affected areas. All the tools and conventions you’ve implemented early in the project are going to start paying off when you get to issues like this. You’ll also want to double-check that the features you’ve merged are still working as expected. This is when you should bring in the devs who worked on the features to get validation.

# Conclusion

Understanding the ways you can use your Git skills to help the team is a subtle thing you should do. The branching strategy you help shape is going to affect everything from the CI/CD pipeline to the way the team does their everyday work. It’s going to affect how you debug larger issues, and it will create the history for the entire project. You also have to be aware that as the team grows, there will be conflicts between branches, and the team will often turn to you to handle the complex ones.

Stay on top of the common Git commands like the ones I covered in this chapter, such as `rebase`, `merge`, and `bisect`. You’ll find new commands as you go through merge conflicts and discover ways to more efficiently manage the team’s branches. As you solve any conflicts, get creative with your approaches and document them. When you learn new things, try to document them and show them to the team, and encourage the rest of the team to do the same. That’s one way you can help everyone learn more about how Git works so that you aren’t the only dev who feels comfortable with any scenarios that come up.