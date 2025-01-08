---
id: 01JH1NK9SN4F8HXEDWK81D999R
modified: 2025-01-07T19:39:08-05:00
---
# Chapter 13. Setting Up the Frontend

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 13th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

The core functionality of the backend is in place, so now it’s time to focus on building the frontend. This is how your customers will be able to use the product and interface with all the backend functionality you have. The frontend is the face of the product your company is building and growing. No matter how cool your data is or how fast your APIs are, if the user experience is bad, then it will be hard for customers to want to use the product.

The frontend requires a different type of thinking from the backend because some user research is involved to see how designs and layouts make people feel about the product. Usually, the Product team or other stakeholders will get the direct user feedback and help translate it into designs, which get passed to you. Then it’s time for you to start your magic.

In this chapter, I’ll cover:

- How to choose a frontend framework
    
- How to choose other packages for your framework
    
- Some long-term decisions you can make now
    
- Why this frontend project will use React
    

As you start thinking through how you want to set up this new frontend repo, study the designs you have and ask about the Product roadmap so that you can consider the direction in which the app is intended to grow. Ask every little question that comes to mind as you go through the designs and look at them from a user perspective. You’ll be surprised how many conditions you can come up with!

Just like with every project, though, you have to get started somewhere. Before you dive into the code, take a look at some of the framework options you have. This is a huge decision for the lifetime of this product because the framework you choose will lock you into the packages and services you can use. Migrating frontend frameworks usually means completely rebuilding the app in a different framework, so do _a lot_ of research.

# Frontend Architecture Decisions

The decisions you make on the frontend will have to be consistent because if there are differences, your users can _see_ them. Just like you did on the backend, you’ll create code conventions with your team. On the frontend, those little differences can directly affect the UX because they are visual and interactive.

Take a look at the designs for the app you’ll be building in [this Figma file](https://www.figma.com/file/Be3O95Dt7bLIyzmNMc61yo/Example-Dashboard?type=design&node-id=1-119&mode=design&t=CoYReahpG7yCMg5g-0). Note where you’ll need to handle data and API calls, what parts of the UI might be reusable, and any other patterns you can find. This will give you some initial thoughts on how to separate parts of the app and what architecture pattern you should go with.

Patterns to look at before you dive into the project include [Model-View-Controller (MVC)](https://developer.mozilla.org/en-US/docs/Glossary/MVC), [Model-View-View Model (MVVM)](https://www.geeksforgeeks.org/introduction-to-model-view-view-model-mvvm/), [component based](https://marutitech.com/guide-to-component-based-architecture/), and [micro frontends](https://micro-frontends.org/). No one architecture is better than any other; it just depends on the approach you want for the app and the tools you’re going to work with. For this project, you’ll go with component-based architecture.

The reason we’re using a component-based architecture is because it’s a common architecture for large-scale apps; you’re building the app with React, which encourages component usage. We’ll also use the [atomic design methodology](https://atomicdesign.bradfrost.com/chapter-2/#atoms), but don’t get too caught up on the terminology. Use names that make the code structure make sense to the team. This methodology breaks down the frontend into the smallest pieces and then slowly builds the pages from there. You’ll likely see some version of this implemented in different projects, so familiarize yourself with it. It may even give you some ideas for refactors you can currently do.

###### NOTE

This is a great teaching moment! As you research the different architectures and make decisions based on the designs and roadmap, share what you find with the team. You don’t have to make a fancy presentation to share your findings. You can just talk through the tabs and editors you have open from your research. That way, the team can see some of your thoughts behind how you make decisions, which will give them insight into what they can look for in their own decisions.

On all the projects I’ve worked on, the PR review process for frontend changes has taken longer than for the backend. It can also get a little nitpicky. Things like naming conventions matter because variables get passed around different components in different states, and you need to make sure you’re referencing the right values. Make a template for things to check in PR reviews, especially on the frontend. Here’s one that I made in the [frontend repo](https://github.com/flippedcoder/dashboard-ui/blob/main/docs/pr-review-checks.md#pr-review-checklist), and here’s one from the [Google engineering practices docs](https://google.github.io/eng-practices/review/reviewer/looking-for.html).

Technical history gets lost as developers come and go, so unexpected breaking changes can happen. I’ve seen this a lot with style changes breaking mobile designs and props getting changed in one component while breaking 11 other components. Having a PR review template will make the team aware of what needs to be checked without having to remember it every time.

Security is going to be huge on the frontend, and I’ll go over that in detail in [Chapter 19](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch19.html#frontend_security_considerations). I mention it now because that’s going to be a driver behind some of the decisions you make for architecture patterns. For example, you need to decide what tools you’ll use to validate inputs and user tokens. You’ll also need to know how to prevent common attacks. This will get repeated numerous times throughout your app, so it will have lasting impacts on how you build forms and make API requests.

As far as how you create and organize components, you can do anything you can imagine. That’s why you need to be strict about code conventions in the beginning. (Sound familiar?) Here are a few things you need to consider as you start building components:

Follow a modular approach and keep business logic separated from components when possible

Having modular components helps you keep track of the different business areas and lets you reuse code across the app. That might look like having files or custom hooks that call endpoints instead of doing it directly in components.

Work with the Design team on building a standard for components and the terminology you use to refer to elements

Using the same phrases will help everyone understand what’s being used and where changes need to be made. In a more mature organization, the Design team will likely have a [design system](https://www.figma.com/blog/design-systems-101-what-is-a-design-system/) in place, so you need to take that into consideration.

Implement separation of concerns and single responsibility principles as soon as you start building

Keeping a separation of concerns separates design and business logic so that components are truly reusable. With single responsibility, you keep components from having too much code bloat. If a component manages a lot of props, it’s probably responsible for too many things, which makes it harder to work with.

Since you know what architecture you’ll be building the app with, it’s time to make an architecture diagram based on the designs. This will help you figure out how to break down the design into your components and views in a reusable way. [Figure 13-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch13.html#project_architecture_diagram) shows what the skeleton of the architecture might look like based on [the designs](https://www.figma.com/file/Be3O95Dt7bLIyzmNMc61yo/Example-Dashboard?type=design&node-id=1-119&mode=design&t=QuG07DIRA82txtkf-0) you got at the beginning of the project.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1301.png)

###### Figure 13-1. Project architecture diagram

As you fill in more details as you work on the app, you can include where API calls will be made and how state should flow between components. This diagram gives you an idea of how you might break the pages up into different containers and the components that go in those containers. This will let you find some of those common components, like the tables, search bars, and buttons.

Once you have this rough draft of the architecture, you can share it with the team and get their input on if the app should be broken up in a different way. Encourage some of the mid-level devs to take a container and add details like states, API calls, and conditional rendering. Now you and the team have a path forward so that you can start making code decisions.

# Choosing a Frontend Framework

The frontend landscape is constantly and rapidly changing. I remember when jQuery used to be the closest thing we had to a JavaScript framework. Then at some point, the number of frameworks exploded. You have [React](https://react.dev/), [Angular](https://angular.io/), [Vue.js](https://vuejs.org/), [Astro](https://astro.build/), [SolidJS](https://www.solidjs.com/), [Svelte](https://svelte.dev/), [Next.js](https://nextjs.org/), [Remix](https://remix.run/), and a ton of other frameworks to choose from now.

###### WARNING

Because new frameworks are constantly coming out, it can be easy to get caught up in the hype. Be careful with choosing newer frameworks over existing ones. There are some great new tools out there, but not everything is ready for production. Some developers have been burned by investing their time heavily into a new tool just for it to lose community interest and support. Sometimes the best way to try out new frameworks is on smaller, personal projects rather than full production apps.

It can be daunting to look through all the possible frameworks and choose one, so here are some key things you should look for during your evaluation:

- Look at community support and how issues are handled on GitHub and Stack Overflow.
    
- Go through the documentation to see how quickly you can find answers.
    
- Check how often the framework is updated.
    
- Check how long the framework has been around.
    
- Research if any companies support, sponsor, or own the framework.
    
- Research accompanying packages, such as data visualization, bundling, and state management, as well as third-party service SDKs.
    
- Check developer surveys like [State of JavaScript](https://stateofjs.com/en-US) or [Stack Overflow’s Annual Developer Survey](https://survey.stackoverflow.co/) to see what people are using in production and what the trends are.
    
- Read through job descriptions to find out what tech stacks companies are using in production.
    
- Find out if there are any geographical restrictions on where the framework is developed due to world politics.
    
- Check the license type for the framework.
    
- Make sure security vulnerabilities get patched regularly and quickly.
    
- Check if there are docs on how your cloud platform handles deployments for your framework.
    
- Look at the tools other projects in your organization are using.
    

Pick no more than five frameworks and decide on your five minimum requirements from this list. Five is an arbitrary number, and you can choose more or less, but this will give you a good breadth and depth to explore. As you go through your evaluation list, start eliminating the frameworks that are falling behind. There’s no need to keep researching them if they don’t meet your minimum requirements.

Then you can take deeper dives into the frameworks’ details with more of the items from the list. It does get hard to decide between the top two, so bring in the team and see what everyone feels the most confident with. This might not be the same as what everyone’s the most interested in, but you have to keep the product’s maintainability as the top priority. Keep in mind what everyone’s expertise is because this will be a huge factor in the final decision. If you have several engineers who work with React, it could be difficult to get adoption of Vue or Astro even if they make sense for the product.

###### NOTE

There’s nothing wrong with using the tried and tested frameworks that are out there. For production apps, that’s usually a better choice because you know there’s support as you add features. It’s tempting to pick up the shiny new thing for a greenfield project. To help strike a balance, you can build a prototype in the more stable framework and the same prototype in a newer framework. Let the team test-drive them and see which one gets developed faster and with a better DX.

This project will use React and TypeScript because these are tried and tested tools for production apps. React checks many of the boxes on the list, making it a solid choice. The market currently still heavily favors React, but doing all the research will ensure that you’re using the most up-to-date production framework.

###### WARNING

Just because React is currently the most used framework doesn’t mean it’s necessarily the best choice. Other frameworks I mentioned earlier, like Svelte and Solid, have great DX and are just as good—if not better in some cases—than React. I very much encourage you to try some of these frameworks and try to get them adopted at your organization. The only way our profession evolves is when people bring in changes and others start to see how good they are. Be one of those people who helps keep us moving forward to a better future!

## App Setup Options

Regardless of the framework you choose, you need a way to run the frontend app. You’ll need to do some initial setup that will determine how the app is built and where and how it can run. This is when you’ll start looking into build tools like [Vite](https://vitejs.dev/), [Rollup](https://rollupjs.org/), [esbuild](https://esbuild.github.io/), and [Webpack](https://webpack.js.org/) (if you have to). The build tool you use will affect the bundle size, other tools you can interface with, app performance, and how fast you can deploy the app.

## Common Components

Every frontend app has some similar functionality. Your project will include error handling, modals, tables, forms, search, filtering, and toast messages. Go to any website you use and see how many of these features you can find on each one. Do a walkthrough of the designs and start looking for common functionality across pages. If you have some questions about UX, bring them up early and often. Making design changes as the app grows gets tricky because that could have unintended side effects on page layouts and consistency across functionality.

## Choosing Packages

There’s a trade-off between having total control over your code through internally developed packages and speeding up feature development by using existing packages from [npm](https://www.npmjs.com/). The other thing you have to watch with packages is bloat. This is another art form for you to get comfortable with. You need to be able to make a call on when it’s worth developing an internal package over using an existing one to balance bundle size and load time.

Here are some common tools for all kinds of frontend functionality:

Data visualization

- [Chart.js](https://www.chartjs.org/)
    
- [D3](https://d3js.org/)
    
- [Three.js](https://threejs.org/)
    
- [Recharts](https://recharts.org/en-US)
    
- [VictoryChart](https://formidable.com/open-source/victory/docs/victory-chart)
    
- [Highcharts](https://github.com/highcharts/highcharts-react)
    

Form handling

- [React Final Form](https://final-form.org/react)
    
- [React Hook Form](https://react-hook-form.com/)
    
- [Formik](https://formik.org/)
    

Component library

- [Material UI](https://mui.com/material-ui)
    
- [Chakra UI](https://chakra-ui.com/)
    
- [Materialize](https://materializecss.com/)
    
- [Semantic UI](https://semantic-ui.com/)
    
- [Mantine](https://ui.mantine.dev/)
    
- [React Bootstrap](https://react-bootstrap.netlify.app/)
    
- [Radix UI](https://www.radix-ui.com/)
    

State management

- [Redux](https://redux.js.org/)
    
- [MobX](https://mobx.js.org/README.html)
    
- [Zustand](https://github.com/pmndrs/zustand)
    
- [XState](https://xstate.js.org/)
    
- [Jotai](https://jotai.org/)
    
- [Valtio](https://valtio.dev/docs/introduction/getting-started)
    

Data fetching

- [TanStack Query](https://tanstack.com/query/latest/docs/react/overview)
    
- [SWR](https://swr.vercel.app/)
    
- [Apollo Client](https://www.apollographql.com/docs/react/get-started)
    
- [RTK-Query](https://redux-toolkit.js.org/rtk-query/overview)
    
- [React Router](https://reactrouter.com/en/main/guides/data-libs#loading-data)
    

Styling

- [styled-components](https://styled-components.com/)
    
- [Emotion](https://emotion.sh/docs/styled)
    
- [CSS Modules](https://github.com/css-modules/css-modules)
    
- [Tailwind CSS](https://tailwindcss.com/)
    

Accessibility

- [i18n](https://github.com/mashpie/i18n-node)
    
- [React Aria](https://react-spectrum.adobe.com/react-aria)
    

Utilities

- [Lodash](https://lodash.com/)
    
- [date-fns](https://date-fns.org/)
    
- [Day.js](https://day.js.org/)
    
- [jwt-decode](https://github.com/auth0/jwt-decode)
    

Linters/formatters

- [ESLint](https://eslint.org/)
    
- [Babel](https://babeljs.io/)
    
- [Prettier](https://prettier.io/)
    
- [js-beautify](https://github.com/beautify-web/js-beautify)
    
- [Biome](https://biomejs.dev/)
    
- [dprint](https://dprint.dev/overview)
    

Package managers

- [npm](https://www.npmjs.com/)
    
- [pnpm](https://pnpm.js.org/)
    
- [Yarn](https://yarnpkg.com/)
    

Testing

- [Jest](https://jestjs.io/)
    
- [Vitest](https://vitest.dev/)
    
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)
    
- [Mocha](https://mochajs.org/)
    
- [Cypress](https://cypress.io/)
    
- [Puppeteer](https://pptr.dev/)
    
- [Playwright](https://playwright.dev/)
    

Now that you have an outline of how the app works and the tools you want to use, take some time to think through the architecture pattern you want to follow.

###### NOTE

Keep in mind that these package options are current suggestions. The ecosystem changes fast, so it’s important that you do your due diligence and find the latest and greatest tools being used. At some point, all tools get replaced with newer, hopefully better versions. So don’t stop learning and looking for what different teams are using for their production apps.

# Working with Other Teams

As you review the designs, you and your team will have questions about functionality and UX. Go through the designs as a team and write up some tickets for each screen. Write out the [acceptance criteria](https://theproductmanager.com/general/how-to-write-excellent-acceptance-criteria-with-examples/) in terms of how the screen should work. Think about how inputs should look and how they should handle errors and validation. You also have to consider what happens when a user submits a form or pushes a button that triggers an API call or a UI event.

Look for a few edge cases and add all questions to the tickets. This is a great way to work with the Product team to help guide them from the technical side. The questions you ask will have an impact on the way Product thinks about features and other ideas they have in mind. You’ll go through plenty of iterations with questions as development starts, but it’s good to get an initial round in before starting.

There might be a QA team you need to work with as well. Even if there isn’t, you should still get a test plan in place. You’ll work with QA to decide the scenarios that should be tested and which environments will be used to test different states of deployed changes. You can bring up any new questions to the Product team. If you don’t have QA, this can also be a group activity for the dev team to get a different perspective on the app.

Talk with the DevOps team to start getting deployment pipelines set up for multiple environments. This can be something they work on in parallel with you as you get the codebase ready. You’ll probably need some type of storage for assets, like images, so make sure that’s a part of their setup. We’ll get into some of the operations (Ops) tasks you’ll handle as a full stack dev starting in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline), so for now we’re not going to worry about pipelines and the tools to integrate with the DevOps infrastructure.

# Conclusion

In this chapter, I went over some considerations you should have as you start building your frontend. The type of complexity you’ll run into will make you think about implementation details differently than the backend. Your changes can propagate in unintended ways without this first round of thinking about the architecture. As the app grows, you’ll be able to add things and adapt, so don’t feel like you have to have it all figured out now.

This project is built with React because it is one of the most popular, well-supported frameworks at the time of writing, but please try out some of the others! I’d encourage you to at least initialize this project in two other frameworks to truly see the differences between them. While React may be the common go-to, the others are also very popular and well supported by their communities. Go through some of those differences with the rest of the team as a teaching moment!