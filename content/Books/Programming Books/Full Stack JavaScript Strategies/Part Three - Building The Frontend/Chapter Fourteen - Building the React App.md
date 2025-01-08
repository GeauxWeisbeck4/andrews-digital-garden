---
id: 01JH1NM617WPCZQ1BZ3D1FFKE7
modified: 2025-01-07T19:39:35-05:00
---
# Chapter 14. Building the React App

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 14th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Now that you’ve done your research and know the tools you’re going to use, it’s time to initialize your frontend project! You’ll set up the React app and all the tooling you need to build your frontend.

By now, you’ve reviewed the designs multiple times, had discussions with the Product team and your dev team, and talked to the DevOps team about getting the infrastructure for the frontend ready. You have a few diagrams and documents to guide you and the team through building all the components. You also have plenty of tickets to split up among the team. When you’re starting a greenfield project, though, you’ll be looked to as the person who will initialize the repo and make the structure of the project. That way, your team can start working on different parts of the app, and everyone has confidence that development can proceed without issues.

In this chapter, I’ll go over:

- Setting up the React app with some of the packages
    
- Building the first feature
    
- Writing the first test
    

You don’t need to have everything in place, just enough for the team to start work. You may be tasked with background development while the rest of the team does feature work. This is one way you act as a multiplier for the team: by working on conventions, docs, and implementing tools that help others get up to speed and get through their tasks more smoothly.

# Set Up the Initial React App

First, initialize Git in this repo or connect it to a remote repo so that you have version control in place at the very beginning. Then initialize the app with [Vite](https://vitejs.dev/guide/). Run the command below and follow along with the prompts. Keep in mind that the following example has the current prompts, which may be different when you run the command. Make sure to choose the React and TypeScript options when they appear:

npm create vite@latest

? Project name: › dashboard-ui
? Select a framework: › - Use arrow-keys. Return to submit.
    Vanilla
    Vue
❯   React
    Preact
    Lit
    Svelte
    Solid
    Qwik
    Others
? Select a variant: › - Use arrow-keys. Return to submit.
❯   TypeScript
    TypeScript + SWC
    JavaScript
    JavaScript + SWC

Your project is called `dashboard-web` and will be scaffolded with some files to get you started. One of the cool things about Vite is that it’s not very opinionated. So you can set your file structure up for the components and screens any way you like. For now, it’s good to finish installing and configuring some of the core developer tools you need.

## Set Up Linters and Formatters

As the team commits code and pushes changes when deadlines get tight, you might write code that doesn’t stick to the conventions for the sake of time. This leads to messy code that’s hard to read and manage. That’s why you’re implementing a linter and a formatter. The formatter will be [Prettier](https://prettier.io/docs/en/configuration.html) to ensure that the code format stays consistent. It will check for things like spacing, commas, and line lengths.

The linter will be [ESLint](https://eslint.org/) to make sure you don’t let bugs in like unused variables, deeply nested ternaries, or magic numbers. These tools are usually paired with [husky](https://github.com/typicode/husky#readme) to plug in to your [Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks), such as commits and pushes. ESLint was configured as part of the Vite initialization, so you can leave that config file as is for now, but you can check out some of the [rules in their docs.](https://eslint.org/docs/latest/rules/#suggestions) Install the other packages with this command:

npm install --save-dev husky prettier

###### TIP

Make sure you don’t have conflicting rules in your Prettier and ESLint configs. I’ve seen this be a problem for Git hooks and CI pipelines. Just be sure to run the `lint` command a few times to ensure that you’ve handled any issues before committing the changes. You won’t implement this, but if you need to run linters on specific files or in a specific order, [lint-staged](https://github.com/lint-staged/lint-staged#readme) is a great tool to get Prettier and ESLint to work well together.

Now you’ll need to add a new file to the root of your project called _.prettierrc_. Remember, the configurations that you’re setting up are a matter of opinion and will reflect the formatting preferences of the team. There will be some formatting that everyone agrees with and some that people disagree with, but eventually, the team gets used to it. In the _.prettierrc_ file, you can add the following code, which are some of my preferences:

```
{
```

This is some basic formatting you may want to keep consistent in your codebase—the little things that can keep a codebase tidy and readable. You can check out all the options available in the [Prettier docs](https://prettier.io/docs/en/options). You’ll also need a way to ignore files that don’t need to be formatted, like your _package-lock.json_ or some of the other config files. So add another file to the root of your project called _.prettierignore_ to handle this:

# Ignore artifacts:
build

# Ignore all HTML files:
**/*.html

# Ignore other config files:
**/*.json
.prettierrc
.eslintrc.cjs
src/vite-env.d.ts
vite.config.ts

This is similar to how your _.gitignore_ file works. You put the files that you don’t want Prettier to format in here. A good practice is to add a command to automatically format the code when a developer wants to. This will edit the code directly in the file to make it meet the formatting rules. So in your _package.json,_ add the following code to the `scripts` section:

```
“scrip
```

You can also encourage the devs on the team to enable formatting on save in their code editor. VS Code has this in its [settings](https://stackoverflow.com/questions/39494277/how-do-you-format-code-on-save-in-vs-code), so no one has to deal with last-minute formatting changes when they’re ready to push a commit. Now you can set up husky to run different commands when you get ready to commit changes or push to the remote repo. To do that, run the following command:

npx husky-init `&&` npm install

Take a look in your repo; you should find that a _.husky_ directory has been created. This has the husky script needed to interface with the different Git hooks you target. It also has the first Git hook script for pre-committing a change. You’ll need to create another file in this directory called _pre-push._ This is where you’ll put commands to run before code can be pushed to the remote repo. Update the _pre-commit_ file with this code:

```
#!/usr/bin/env sh
```

Then add the following code to the _pre-push_ file:

```
#!/usr/bin/env sh
```

This is a personal preference, but I like to do the linting and formatting for pre-commit and save the testing for pre-push. You can do any combination of these things if you like something better. See if anyone on your team has strong opinions about linting and formatting rules. That way, you are including everyone in these initial decisions. Do a test to see if your husky configs are working by committing and pushing your changes so far. Note that the `test` command is commented out right now. We’ll set that up a little later.

That’s all for the linters right now. So you can turn your attention to how the app is actually built with Vite as your bundler.

## Set Up the Build Configs

This is important because it will determine how the project is compiled into the code that gets run in the browser. This can change the way you import packages and components, the syntax you use, and how big your bundles are. You usually make this choice based on the environments your app will run in. Not every browser supports the latest features in JavaScript.

First, to compile your TypeScript to JavaScript, take a look at the _tsconfig.json,_ where you’ll find the `target`, `module`, and `lib` values. This is one of those files that just gets copy-pasted across new projects, but it’s good to at least understand what the values do for your build. These determine the way your TypeScript code gets compiled to JavaScript.

The `target` value determines the versions of [ECMAScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/JavaScript_technologies_overview) your app will be compatible with. This is important because it determines what browsers and Node versions the app can run on. This can affect things like the Node version your server needs to use. The `lib` value tells the compiler which JavaScript features are available when the compiled code is executed. This is usually the same as the `target` value because it affects the output of your build. You might have a different `lib` value if you are working in a [polyfilled environment](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill). The `module` value is how the compiler resolves the modules in your app. For example, using import statements will fail in old versions of Node. This value affects how the modules in your project are found, so that’s what makes it important.

You can leave the config values unchanged for now since the defaults should run in most modern browsers. I encourage you to glance through the [TSConfig docs](https://www.typescriptlang.org/tsconfig#lib) to learn more about these values and deepen your knowledge about builds.

###### NOTE

Choosing the values for `lib`, `target`, and `module` will make a huge difference in your bundle size and how you write your code. The values you choose will affect how you import functionality in different files and how your code interacts with other code. This is usually more noticeable when you’re making packages that are used by other projects, but it’s something that you should be aware of for all projects you work on.

When you’re ready to bundle all your code into the build artifact that gets deployed to servers, you can change the build options in your _vite.config.ts_ file. Vite uses [Rollup](https://rollupjs.org/) under the hood to create builds, and the default options are usually good enough. If you need to do anything more specific to change the bundle size or work with browser compatibility, you can add [build options](https://vitejs.dev/config/build-options) to your Vite config.

## Set Up Styles

Let’s move on to the styles. You’ll use [styled-components](https://styled-components.com/) to handle any custom styling and avoid inline styles. When you have a component for your styles, you have more control over how the styles work, and you can do things like conditionally update styles with props.

You’ll also be using [Material UI](https://mui.com/material-ui/) (MUI) for the component library. This will make it easier for you to focus on functionality because your components will already have accessibility and responsiveness built in. Combined with styled-components, MUI can be customized to have any theme you want. MUI can also be extended to include its [Material Icons](https://mui.com/material-ui/material-icons/) if you know you’ll need these for buttons or other visuals.

# BE CAREFUL WHEN USING COMPONENT LIBRARIES

I’ve been on a number of teams across organizations of different sizes, and I never really saw any of them implement a custom component library well unless they had a team dedicated to that. It always starts out with the best intentions, but over time an internal component library becomes hard to maintain without a dedicated team, especially if it’s used by multiple frontend teams with different component needs.

Ethan Brown also had a few things to say about this:

> I think component libraries are changing thanks to projects like Tailwind UI, Headless UI, shadcn/ui. The downside of traditional component libraries is that they’re great until you need to do something slightly different. If you’re using MUI, and you need to do something that isn’t the “MUI way,” it can be incredibly time-consuming and frustrating.
> 
> I personally feel the future lies in something like shadcn/ui and Headless UI where you say, “Generate a button component for me,” and it drops a button component into your project, which you then control and customize. Time will tell. I will say that every UI library I’ve used—including my favorite, Mantine—is eventually as much hindrance as help. It’s a real “choose your poison” decision.

Often, the organization will already have a preferred choice for how to handle styles and components because the Design team will have created a design system to have a consistent look and feel for apps across the organization. So regardless of the component library you work with, there will be scenarios where you have to override the existing styles.

To get started, go ahead and install the packages you need with this command:

npm install @mui/material @mui/icons-material @emotion/react @emotion/styled styled-components

Now you can import MUI components and icons and create your own components with styled-components. The next thing you can do is initialize the theme for your app. Every app will have custom colors, spacing, fonts, and other details. To prepare for this, you can delete _App.css_, _index.css_, and the _assets_ folder from the _src_ directory. You’ll need to update the _main.tsx_ file by deleting `import ‘./index.css’` from the file. Then create a new file called _theme.tsx_ and add the following code:

```
import
```

This sets a couple of colors in the theme object. This theme will definitely change as you start building views and getting the colors you need from the Figma designs, but you have a good starting point here. You should take a look at the [MUI docs on theming](https://mui.com/material-ui/customization/theming/) to understand all the values you can set, including dark mode.

For the theme to be applied throughout your app, you need to add a `ThemeProvider` to _App.tsx_. A _provider_ is a common pattern in React development to provide top-level functionality that should be available to all the components in your application and is commonly used for functionality like themes, modals, and state management. It’s usually based on [React Context](https://react.dev/learn/passing-data-deeply-with-context). Open the file and update the contents with this code:

```
import
```

You’ve imported the `theme` you just created and applied it with the `ThemeProvider` component. Now every MUI component inside this provider will have the theme you defined. This sets the foundation for all your styles, so you can move on to the other tasks to get this repo developer-ready.

## Set Up Testing

Frontend testing, just like backend testing, is going to save you from regressions all across the app. It gets trickier on the frontend because you have dynamically rendered components and API requests happening, which changes how you have to write and think about tests. Since you’re using Vite as the tool to run and build the app, you’ll use [Vitest](https://vitest.dev/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) to write the unit tests for the app. You’ll need to install them in your project with the following command:

npm install -D jsdom vitest @testing-library/react

Then in your _package.json,_ you need to add the following script to run the tests:

```
…
```

You also need to update the _vite.config.ts_ file so that your tests will run correctly. You can overwrite all of the code in this file with the following:

```
/// <reference types=“vitest” />
```

Now when you write tests for the components you build, you’ll have the packages and script ready. This will also be used in your CI pipeline, and when you try to push changes, husky will run the tests for your pre-push Git hook. We’ll go into deeper detail on tests in [Chapter 21](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#frontend_testing) and [Chapter 24](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch24.html#integration_testing).

You now have the core of the app set up. There are just a couple more things to do to make sure you set the code up for long-term maintainability.

## Set Up CHANGELOG and README

The README and CHANGELOG files are some subtle files that will really help over the lifetime of the project. Think of the README as the how-to file for the project. This should tell any developer how to set up the project, any environment variables, and any quirks that you know exist. The CHANGELOG keeps the history of all the changes in each release. Changes should be recorded so that you always know what’s been released in each version. It’s good to link tickets in the CHANGELOG so you can refer to them later.

# ANOTHER VIEW ON CHANGELOGS

Depending on the release process at your organization, you will run into different ways to document changes. Here’s another view on them from Ethan Brown:

> My thinking on CHANGELOGs has shifted. I used to use this approach where there’s just a Markdown file that you update as you make changes. The problem with that is that it usually just duplicates work done in the release process. The release process usually involves examining all PRs merged since the last release and writing a summary for the release. And yes, the CHANGELOG can help facilitate this, but it can also introduce confusion and be one more thing that has to be maintained and referenced as part of the release process.
> 
> I’ve found it to be a much better approach to focus on issue/PR documentation. If the description of each PR contains all the relevant information (as it should), constructing release notes becomes much easier, and it concentrates the documentation effort where it belongs, on well-defined issues and PRs.

You can find an example of the project [README](https://github.com/flippedcoder/dashboard-web/blob/main/README.md) and [CHANGELOG](https://github.com/flippedcoder/dashboard-web/blob/main/CHANGELOG.md) in the GitHub repo. This is something you’ll want to discuss with the team because keeping these files up to date is the only way they stay relevant. Work with the team to incorporate these updates as part of your normal PR reviews so that you don’t miss them when you have a breaking change.

## Run the App Locally

You’ve set up all the tools, so now it’s time to make sure this app runs with all your configs! Start by checking the version of Node you’re using. I’ll be running this app locally with [Node 20.10.0](https://nodejs.org/en). If you don’t have a way to switch between Node versions, try out [nvm](https://github.com/nvm-sh/nvm), [Volta](https://volta.sh/), or [n](https://github.com/tj/n). Once you have the correct version ready in your terminal, navigate to the project directory and install all of the packages with this command:

npm i

Then you should be able to run the app with this command and get the same output as follows:

npm run dev

 VITE v5.0.5  ready `in` `174` ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show `help`

When you navigate to the localhost URL, you should see something like [Figure 14-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch14.html#ui_app_running_locally) in your browser.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1401.png)

###### Figure 14-1. UI app running locally

Next, you need to try out all the other commands for your build, tests, linting, and formatting. They should all run without errors. The test script won’t execute because there aren’t any test files right now. The build script should generate a new _dist_ directory in the root of the project. Double-check that this directory is in your _.gitignore_ since you don’t want to commit builds. The lint and format scripts should run and not generate any errors.

With all these checks complete, you should commit the changes you have so far because this is a good foundation for the app. Anything you add after this will be feature work or some improvements or additions to the tools the project uses. But you have the bare bones to get started on the first feature.

# Build the First Feature

With this repo set up and ready for development, it’s time to add some functionality! You can start with any part of the designs you want, but I like to pick something relatively small yet big enough to highlight potential issues. In this example, you’ll get the folder structure in place, set up the initial routing, and build the container for the different app pages.

## Project Structure

We’ll go with a take on the simple React approach. In your _src_ directory, add two new folders named _components_ and _pages_. The _components_ folder will have the smaller, reusable functionality, like search bars and shared styled components, as well as things like the navbar and header. The _pages_ folder will have the larger functionality that makes up a whole page, like the user info and user actions pages.

###### NOTE

This is some info you should add to the README. It tells future devs how the code is organized so that they can quickly figure out where to look for things. When you get the README in a good place, you should have the team test out the steps and add more details when they run into issues.

Add some boilerplate code in the _pages_ folder for the user info page. The folder structure will look like this:

…
|__ pages
|____ UserInfo
|_______ index.tsx
|_______ UserInfo.Container.tsx
|_______ UserInfo.Container.test.tsx
…

For now, you can put the following code in the _UserInfo.Container.tsx_ file:

```
import
```

Then you can add code in the _index.tsx_ file in the _UserInfo_ directory to create the module for this directory. The reason you’re using this module file is to streamline imports and exports across the app. That way, you don’t have to import all your functionality from separate files as the app grows. Take a deeper dive into how modules work in [this MDN doc](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) because it’s something you should know about and be able to help others on the team understand. Here’s the code to handle the module:

```
import
```

The last thing to do is write at least one test to make sure tests aren’t overlooked during development. This sets the precedent that the project will have tests so that the team knows not to skip writing them. In _UserInfo.Container.test.tsx_, add this code:

```
import
```

This is something to get you and the team going on actual development, so don’t worry about making this pretty or super functional at this point. You’ll follow this pattern of writing code for every part of the app. so all the components will have this structure and these three files as the minimum.

## Set Up Routing

Based on the architecture diagram we made earlier, the main container is composed of two parts: the navbar and the current screen. Let’s set up the initial routing for the current screen. You’ll need to install [React Router](https://reactrouter.com/en/main/start/tutorial). There are a couple of alternatives to this: [TanStack Router](https://tanstack.com/router/v1) and [Next.js](https://nextjs.org/). To install React Router, run this command:

npm i react-router-dom

This is where you’ll handle what the app renders based on the URL. Remember, at this point in the app you aren’t trying to get everything feature-complete. You’re trying to set up the skeleton to enable everyone to work on different parts. It can be tempting to try to do everything yourself and dive into the details, but you have to know when you have enough in place to let the team do their magic, too.

Now you need to create a new file in the _src_ directory called _routes.tsx_. This will give you two routes that direct users to the different screens available. The reason you want to implement routing now is because the project will have different URLs for the pages you’re making so that you can set up some initial linking. Add this code to the file:

```
import
```

## Update the Root of the App

Now you can update _App.tsx_ to have the base UI layout with the following code:

```
import
```

Now you should run the app to make sure everything is still working as expected. You should see something like [Figure 14-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch14.html#skeleton_of_the_app_with_the_user_info).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1402.png)

###### Figure 14-2. Skeleton of the app with the user info screen

You can check the [code for this chapter in this PR](https://github.com/flippedcoder/dashboard-web/pull/1/files) to make sure you have everything set up correctly. Now you have more code that you can commit to the repo. This can be the example the other devs use to start building more of the core functionality of the project. These first components give you a chance to really try out the new repo and see if any adjustments need to be made to the tools or the configs.

###### NOTE

Remember to make commits often and keep them small. It will help with code reviews as well as make sure you don’t lose a huge number of changes. Encourage the team to do the same and include it in your conventions for PR reviews.

Also remember that at this point in the project, you aren’t responsible for single-handedly creating everything. That’s why these components are essentially placeholders and you haven’t dived into the designs yet. I know I’ve said this a few times, but it’s important not to get hung up on making one part perfect before the team has a chance to actually look at the project structure, understand it, ask questions, and give their own input. Things will change.

All your scripts should still run without errors. This is a good time to test any pipeline things the DevOps team has in place, such as deploying to different environments. That way, the DevOps team can address any issues as early as possible so that development flows smoothly as the team ramps up. I’ll get into the CI things you can handle for the frontend and backend in [Chapter 25](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#making_deployments) and [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline) because once the infrastructure is in place, you and the team will likely handle more of that.

# Conclusion

In this chapter, you set up all the initial tools and configs for the React app. You worked on the first components to make sure the project works as expected. The code you have so far is going to enable the rest of your team to get started on other functionality. You don’t have _all_ the core functionality in place, like state management and data handling, but that’s something other devs can add as they go.

You and the team will install more packages and set up more config files as you need different functionality. As you and the team dive deeper into the designs and functionality, you need to have conversations and maybe even small demos to compare packages. With the repo in this state, you should definitely have a meeting to go over how everything works so that there isn’t confusion once everyone starts working on different pieces.