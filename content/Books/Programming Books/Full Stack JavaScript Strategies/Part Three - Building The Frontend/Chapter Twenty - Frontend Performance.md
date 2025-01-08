---
id: 01JH1NS72BQYP5RG9GGXGRRAVG
modified: 2025-01-07T19:42:20-05:00
---
# Chapter 20. Frontend Performance

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 20th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Now you have your frontend in a good place, and you’ve probably done at least a beta release of this product to customers. Going forward, you’ll be adding more features, doing maintenance, and looking for ways to improve the app for both the developer and the user experiences. One way to do both is by improving performance. The faster and more smoothly the app runs, the more the UX improves, and the more streamlined the process becomes for devs to make updates as they develop locally.

It’s frustrating to encounter apps that load slowly, have a jumpy interface as data becomes available, and are difficult for your dev team to update. Once your app is stable and you have all of the main architecture in place, you can start focusing on these concerns. This is your chance to really optimize the code based on patterns you’ve seen from logs and feedback from the dev team and other stakeholders.

In this chapter, I’ll go over:

- Common causes of performance issues
    
- How to measure and improve your performance metrics
    
- Tools to improve your app’s performance
    
- How to keep the code clean for better performance
    

Performance is a topic that tends to come up once an app has been in production for a while. One of the devs on the team will bring up how slow development is because the pages take forever to load code changes. This is something that should be in your mind from the beginning because it directly affects how the app is maintained and developed over time. While many performance issues can be addressed at later phases in the project, the sooner you bring them up, the better.

# Benchmark Metrics

Before you dive into optimizations, you need to know what metrics your optimizations are based on. If you aren’t sure which metrics to measure, a good place to start is with the [Core Web Vitals](https://web.dev/articles/vitals). Then you can research the particular metrics that are most applicable to your app. Is bundle size affecting load times? How are the response times on slow networks? What about the [_largest contentful paint_](https://web.dev/articles/lcp) (LCP)? It’s OK to focus on a few metrics at a time and incrementally make changes to your app. Some of the main metrics to start with are LCP, _first input delay_ (FID), and _cumulative layout shift_ (CLS).

LCP measures how long it takes the largest image or content block to render relative to when the user navigated to the page. The goal is to have an LCP within 2.5 seconds of when the page first starts loading. It’s recommended to keep it at least under 4 seconds. Anything longer than that provides poor UX and is something the team should improve.

FID measures how long it takes the browser to process event handlers that get triggered from a user interaction—in other words, how long it takes something to happen on the page in response to a click or type from a user. The optimal score for this metric is 100 ms or less; anything over 300 ms needs to be improved. Nobody wants to click on something and then experience nothing.

CLS is very important for layout development. This metric measures how jumpy the page is while it’s loading. You want to limit how much the layout of a page shifts while the user interacts with the page. That’s where things like skeleton loading components come in handy because they preserve the layout as content is loaded. A good score is less than 0.1, while anything over 0.25 needs improvement. Keep an eye out for this metric in particular because it’s something that the dev team has direct control over.

You can use performance-testing tools to give you insights into other metrics to consider as well as to find out where your app is already doing well. If you’re curious about any of the sites you visit regularly, [WebPageTest](https://www.webpagetest.org/) and [Google PageSpeed Insights](https://pagespeed.web.dev/?utm_source=psi&utm_medium=redirect) will let you enter the URL for the website you want to test and return a performance report. The rest of this section will introduce you to two more tools: [Lighthouse](https://github.com/GoogleChrome/lighthouse?tab=readme-ov-file#lighthouse) and [sitespeed.io.](https://github.com/sitespeedio/sitespeed.io)

## Lighthouse Tools

You may have already used Lighthouse in your Chrome DevTools while doing local development. This audit doesn’t take long to run, so do it periodically. That way, you can give a little report to the dev team about things they should watch out for as they make decisions. This is one way you can mentor everyone with real examples that have impactful solutions. You can also demonstrate the results to the Product team and any other stakeholders so that they understand when you ask for time to work on tech debt.

###### TIP

While it’s fine to check your Lighthouse results locally, keep in mind that the results in development can be drastically different than in production for reasons such as assets not being compressed, code not being [minified](https://www.cloudflare.com/learning/performance/why-minify-javascript-code/), and less optimal configs. One way to handle this is to do a production build locally and then run Lighthouse on that build.

If you run the current project locally and open the Chrome DevTools, you can run a Lighthouse performance test. Your results will look similar to [Figure 20-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#lighthouse_report_overview) and [Figure 20-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#lighthouse_report_details).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2001.png)

###### Figure 20-1. Lighthouse report overview

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2002.png)

###### Figure 20-2. Lighthouse report details

Now you have a list of metrics that you can focus on improving, and you’ll also get some suggestions for improvements in the report.

You can use the [Lighthouse npm package](https://github.com/GoogleChrome/lighthouse?tab=readme-ov-file#using-the-node-cli) to create automated performance reports as part of your CI/CD testing. There are a number of configs you can set to generate the reports and save them in JSON or HTML format. Just to check it out, you can install and run Lighthouse on your project locally and save the results in JSON format with the following commands:

npm install -g lighthouse
lighthouse http://localhost:5173/ --output-path`=`./report.json --output json

You can find the full JSON report in the [project GitHub repo](https://github.com/flippedcoder/dashboard-ui/blob/main/report.json), but here’s a snippet of what the results look like:

```
“
```

They’re the same results you’d see on the HTML page, but now you can do more with the results programmatically. Maybe there are some metrics that you want to pay close attention to, so you configure the system to throw some kind of error if the metric isn’t above a specified threshold. That might include core web vitals like LCP, FID, and CLS since these have a huge impact on user experience.

## Sitespeed.io

Let’s look at an open source tool you can use for more flexibility over what you can modify and test during performance testing. Sitespeed.io is a really versatile option because you can [create dashboards](https://github.com/sitespeedio/sitespeed.io?tab=readme-ov-file#performance-monitoring-dashboard) to monitor your app’s performance over time, get reports for individual pages, and test with different browsers. It has lots of configs you can customize to test for almost any performance metric you want as well as accessibility testing. You can even run sitespeed.io in a standalone Docker container.

If you want, you can have sitespeed.io send messages to a Slack channel to alert the team when any thresholds have been exceeded. It can record videos for your performance tests. You can easily integrate it into your CI/CD pipelines. I recommend reading the sitespeed.io [docs](https://www.sitespeed.io/documentation/sitespeed.io/) to learn all this package can do, but here’s an example of what a test can look like with this tool:

npm i -g sitespeed.io
sitespeed.io http://localhost:5173/ --browser safari -n `2` --summary-detail

`[``2024`-02-17 `19`:32:04`]` INFO: Versions OS: darwin `23`.1.0 nodejs: v21.2.0 sitespeed.io: `33`.0.0 browsertime: `21`.2.1 coach: `8`.0.2
`[``2024`-02-17 `19`:32:05`]` INFO: Running tests using Safari - `2` iteration`(`s`)`
`[``2024`-02-17 `19`:32:06`]` INFO: Testing url http://localhost:5173/ iteration `1`
`[``2024`-02-17 `19`:32:12`]` INFO: Take after page `complete` check screenshot
`[``2024`-02-17 `19`:32:15`]` INFO: http://localhost:5173/ TTFB: 3ms DOMContentLoaded: 196ms FCP: 243ms Load: 197ms 
`[``2024`-02-17 `19`:32:16`]` INFO: Testing url http://localhost:5173/ iteration `2`
`[``2024`-02-17 `19`:32:23`]` INFO: Take after page `complete` check screenshot
`[``2024`-02-17 `19`:32:26`]` INFO: http://localhost:5173/ TTFB: 3ms DOMContentLoaded: 201ms FCP: 248ms Load: 201ms 
`[``2024`-02-17 `19`:32:26`]` INFO: http://localhost:5173/ TTFB: 3ms `(`σ0.00ms `0`%`)`, FCP: 246ms `(`σ3.00ms `1`.0%`)`, DOMContentLoaded: 199ms `(`σ3.00ms `1`.3%`)`, CPUBenchmark: `61`.5ms `(`σ13.50ms `22`.0%`)`, Load: 199ms `(`σ2.00ms `1`.0%`)` `(``2` runs`)`
`[``2024`-02-17 `19`:32:27`]` INFO: HTML stored `in` /Repos/dashboard-ui/sitespeed-result/localhost/2024-02-17-19-32-04
`1` page analysed `for` http://localhost:5173/ `(``2` runs, Safari/desktop/`[`object Object`])`
   Score / Metric             Median
   -------------              ------
√  First Contentful Paint     `246` ms
   Page Load Time             `199` ms
   TTFB                       `3` ms
!  Coach Overall Score        `84`
✗  Coach Best Practice Score  `68`
!  Coach Privacy Score        `80`
√  Coach Performance Score    `99`

[Figure 20-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#sitesepeddotio_results) summarizes the results.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2003.png)

###### Figure 20-3. Siteseped.io results

Click on any of these metrics to get more detail about them and suggestions for how to improve them. If you want to check out the reports more, go to the [project GitHub repo](https://github.com/flippedcoder/dashboard-web/tree/ch-23/sitespeed-result/localhost/2024-02-17-19-32-04) and open the HTML files in your browser.

There are also commercial products available for performance measuring, such as Sentry and New Relic. I discussed those in [Chapter 12](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#monitoringcomma_loggingcomma_and_incide).

Now that you know some of the tools you can use to analyze your site, it’s time to look at areas where you can improve your app and actively guide the rest of the team to watch. You can include these as items in your PR review process or quarterly audits. Some will fundamentally change the way that you and the team develop code, so make sure to review them with the team—and don’t forget about demos.

# Areas for Improvement

After you’ve run the tests, you need to decide how to fix any performance issues. There are a few ways to update the frontend to speed things up. Some of these can be incrementally updated as you add new features; others will be a larger undertaking because they fundamentally change the way the app works. Let’s start by looking at the packages you have in your project.

## Bundle Size Analysis

Bundle size is one of the less obvious things that can slow your app down. There’s a package for just about any functionality you can think of, but that doesn’t mean you should reach for packages immediately. Every package you install increases your bundle size, which increases the page load and response times. Many packages also depend on other packages, so you might have things in your bundle that you don’t directly use.

One way to determine which packages add the most to your bundle size is by analyzing your build with a tool like [source-map-explorer](https://github.com/danvk/source-map-explorer) or [vite-bundle-visualizer](https://www.npmjs.com/package/vite-bundle-visualizer#vite-bundle-visualizer). These analysis tools will help you find out which project files are adding the most to your bundle size and where to trim things. I’ll use vite-bundle-visualizer here, since you’re using the Vite tool for your project. [Figure 20-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#source_map_for_bundle_size_with_vite_bu) shows what the vite-bundle-visualizer results will look like for your project if you run these commands:

npm i vite-bundle-visualizer
npx vite-bundle-visualizer

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2004.png)

###### Figure 20-4. Source map for bundle size with vite-bundle-visualizer

You can click on each of the squares to see more detail about which file is adding the most to the size. This can help you decide if it’s time to choose an alternative package or if you need to refactor certain files in your repo.

###### NOTE

One project I worked on relied on a [micro frontend architecture](https://martinfowler.com/articles/micro-frontends.html). The different micro frontend projects often bloated the consuming app, and source-map-explorer made it easy to clearly see which one was causing a given issue. You may sometimes be surprised to find that it isn’t a package that’s increasing your bundle size, but rather a component you and the team have written.

After you’ve analyzed your build to see where anything can be decreased in size, it’s time to turn your attention to the build itself.

## Build Configurations

Fine-tuning your build configs isn’t easy, but it’s a skill that you should have and that can really let you shine as a senior dev. Take the time to learn some of the ins and outs of the concept. Sometimes you can optimize your build configurations to decrease bundle size or even optimize the artifact you deploy to production. You might find yourself working on a project that was built on Webpack and need to update [those configs](https://webpack.js.org/configuration/).

Thankfully, for this project you started from scratch with a modern tool, Vite. If you’re on a real-world app that is a Create React App (CRA) project, a tool like [CRACO](https://craco.js.org/docs/configuration/getting-started/) can help you access configuration settings that improve performance. Since this project is written in TypeScript, what you have in your [_tsconfig.json_](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) file will determine how your app is compiled: which files are included in the production artifact, where the files are located, how they are minimized or compressed, and with which browsers your app will be most compatible. Since browsers don’t use TypeScript, your _.ts_ files will get compiled to JavaScript at some point.

When you’re trying to speed up your app, check your configs for files that _don’t_ need to be included in your deployment. Tests, reports, and any files used strictly for development can probably be left out of the production artifact. You should also check to see which [JavaScript modules](https://esbuild.github.io/content-types/) you are targeting because this will affect how you handle [polyfills](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill) for functionality that isn’t supported in certain browser versions.

JavaScript’s functionality has changed over the years, leading to modules like ES2016 and ES8 that have allowed it to include things like [async functions](https://github.com/tc39/proposal-async-await), [the spread operator](https://github.com/tc39/proposal-object-rest-spread), [optional chaining](https://github.com/tc39/proposal-optional-chaining), and [nullish coalescing](https://github.com/tc39/proposal-nullish-coalescing). To stay up to date with changes, visit the [TC39 GitHub repo](https://github.com/tc39/ecma262) occasionally to read the latest proposals.

Browsers don’t make updates in sync with JavaScript updates, so your code might need polyfills to work with older browsers, depending on your build configs. That can slow the app by bloating the code with conditions and packages to get it to work for specific browser versions.

Build configs in general can get tricky. Honestly, it can be one of the more frustrating parts of development because there aren’t always straightforward answers or strategies. It takes some experimentation to figure out what works the best. You just have to keep a level of grit that won’t let you give up. I’ve definitely been there a few times.

## Caching Configuration

Trying out different caching strategies can drastically improve performance. _Caching_ is when you decide what resources and data should be stored in the browser to improve loading times. After a certain period of time, data in the cache is considered _stale,_ which means that it needs to be refreshed from the server. You get to set that time period, so you have to balance between improving performance (by keeping data in cache longer) and consistency (cached data is out of date with respect to the server). Fonts and images tend to be stored in the browser cache for longer periods because they don’t change often.

There’s an art to deciding how long to cache data before making a new request for the most current data. You should research how often your data should be updated. Some data, like bank account info or order info, should be fetched with every page load. Other data, like product lists and event calendars, doesn’t need to be updated as often. Work with your team to figure out what to do here.

The tricky thing about caching is that it can make debugging harder. When the user (or even the developer) has data cached locally, the app can appear to be having issues that may or may not exist. When you run into any issues where data isn’t showing as expected, make sure you suggest that people clear their cache or try with a different browser. If the app is using outdated data from the cache that has values that don’t exist anymore, that could crash the app.

You can implement any number of [strategies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) to customize how caching works on the frontend. You can target certain endpoints to become stale after a specific amount of time, or you can select endpoints that should never be cached. Just make sure you clearly document how the cache is being handled. That documentation could be as simple as commenting the code so that the dev team understands what’s happening for each cache implementation.

There are so many tools available now that have made this better. You’re working with React Query, which handles caching out of the box, so take some time to read through the [default configs](https://tanstack.com/query/v4/docs/framework/react/guides/important-defaults) and the [React docs](https://react.dev/reference/react/cache) before you start changing things. Even the built-in React hooks like [`useContext`](https://react.dev/reference/react/useContext), [`useMemo`](https://react.dev/reference/react/useMemo), and [`useCallback`](https://react.dev/reference/react/useCallback) can help cache some data for you. You have many options to choose from, so I suggest you read through the documentation.

You also need a mechanism to perform _cache busting_, where you force-reload pages or delete the user’s current cached data. You might need to do that if API updates change the response in ways that break the frontend or if you need to push updated data to your users to meet some third-party upgrade deadline. You can do this by writing code that force-reloads the page and changes the cache strategy temporarily, or you might use a [content delivery network (CDN)](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/) that lets you push changes to all users .

###### NOTE

CDNs can boost your app’s performance. They essentially cache snapshots of your app on servers that are geographically closer to the user so that the user gets the page content faster. This can add a wrinkle to debugging production issues, so if you get reports that some but not all users have issues and you’re using a CDN, that could be the cause. Clear out the CDN using the service provider’s dashboard or CLI and see if that fixes the problem.

With React Query, the code implementation for caching will involve setting a few parameters on your endpoint requests. Here’s an example of setting a stale time for a particular endpoint in your project to clear the cache and refetch the data:

```
const
```

The main thing you and the team have to understand is what config values actually do with your data. This is when you should turn to the [docs](https://tanstack.com/query/v5/docs/framework/react/reference/useQuery) and get a good understanding of the package you’re using. Once you’ve got a caching strategy in place, users will notice the difference in their experience. Just remember to watch out for the cache when it’s time to debug issues.

## Lazy Loading

_Lazy loading_ is a way to speed up your page load times by having the app load important content first and the remaining content when the user needs it. That way, the user doesn’t have to wait for all the page content before they see any content at all. You can use this technique on any page that implements some type of load-on-scroll functionality.

As the user needs to see more content, lazy loading makes it available; in the meantime, they see some type of loading element. This can have a huge impact on your CLS score because if you don’t style the loading elements correctly, lazy loading can cause drastic page shifts.

Fortunately, there are a couple of built-in ways to handle this in React. You can lazy load with the [`<Suspense />`](https://react.dev/reference/react/Suspense) [component](https://react.dev/reference/react/Suspense) or the [`lazy`](https://react.dev/reference/react/lazy) method. This lets you create a loading state for any components that depend on data loading with the flexibility to create things like page scroll loading, and it lets you handle different components on the page with custom loading components. These custom loading components are sometimes referred to as “skeleton components” because they have the same dimensions as the component that will eventually load.

For example, you might have a `<div />` that’s styled to match the order table in your app. It doesn’t have to be fancy, but it will help you avoid that content layout shift issue. Here’s what that might look like when it’s implemented with `<Suspense />`:

```
…
```

[Figure 20-5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#skeleton_loading_component_in_suspense) shows the resulting page.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2005.png)

###### Figure 20-5. Skeleton loading component in Suspense fallback

Now if there’s any delay with the component loading, you can target it specifically with its own loading component. Remember to keep at least the height and width of the loading component about the same as the real component. An added benefit of this method is that you might see ways to make your components smaller, so you can manage the code better over time and even improve testability.

###### TIP

You can do something similar to lazy loading by conditionally rendering components based on loading state, but this is an alternative to that if you want to keep the code in a certain format. This is another decision to discuss with the team because it can come down to preference.

You might also want to try lazy loading packages or imported components. That way, you call even less content until you need it on the page. Here’s an example of how you can do a lazy import of the `Header` component in the _UserInfo.Container_ file:

```
…
```

These are a few ways to improve your page load time and your CLS score at the same time.

## Prefetching

_Prefetching_ is when you get the data or components that you expect the user to need next and have them waiting in the cache. React Query helps you do this with the [`prefetchQuery`](https://tanstack.com/query/v4/docs/framework/react/guides/prefetching) method. [_Server-side rendering_](https://www.joshwcomeau.com/react/server-components/) (SSR) also comes up when you bring up prefetching data because you can prefetch entire pages from the server if you’re working with something like Next.js or [React Server Components](https://react.dev/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components), which is relatively new to the framework.

SSR is a completely different paradigm than what’s currently used on the frontend, and not every app will implement it. I recommend [reading up on SSR](https://www.sanity.io/glossary/server-side-rendering) before you try to use it. It’s an effective tool that can speed up your app a lot, but it has its own drawbacks: for example, the server-side-rendered components never rerender. If you aren’t expecting that, it can be hard to debug. These components won’t update when `useState` or `useEffect` hooks are called, and the code likely won’t compile anyway. The upside to using server components is that you can write backend functionality the same way you write your frontend code, like database queries, and it isn’t exposed to users.

Scenarios where you would want to use SSR include any static content that doesn’t change when a page is loaded or functionality that isn’t user specific, such as shared event calendars or product lists. The performance benefits may be limited if all the content is specific to a user, as with user account info or dashboards with specific user data. I encourage you to [look through the docs](https://2023.stateofjs.com/en-US/usage/#js_app_patterns) and form your own opinion. Every technology has its uses, but you have to discern when and why.

## CSS, Images, and Fonts

The last thing I’ll cover in this chapter is optimizing your assets. Images in particular have a huge effect on how fast your pages load because they can be large and take time to download. Doing things like compressing images, using images with the correct dimensions from the server, and using formats like SVG can improve those load speeds. There are also the [WebP](https://developers.google.com/speed/webp) or [AVIF](https://web.dev/learn/images/avif) formats for images, but there isn’t wide browser support for these formats yet, so you might still have to fall back on SVGs or PNGs. Remember, you can always check browser support with [CanIUse](http://caniuse.com/).

When it comes to CSS files, whether you’re using a package or your own custom CSS, you need to minify it and check for unused styles, which contribute to your bundle size and page speed. CSS can handle quite a few things you might be doing with JavaScript, like dark mode, gradient animations, and parallax effects. Make sure you check out the [latest CSS rules](https://developer.mozilla.org/en-US/docs/Web/CSS/Syntax) before you jump into a JavaScript solution because CSS is constantly changing, especially for responsive layouts and animations.

Fonts are often overlooked because they tend to come from some other service, like the [Google Fonts CDN](https://fonts.google.com/). Make sure you’re loading only the fonts you need and remove any unused fonts or icons. If you can, self-hosting your fonts can speed up your pages because you don’t have to rely on a connection to a third-party CDN. This also goes for things like the MUI library. You can import the specific components you need on a page instead of the entire library, decreasing the amount of content the page needs to load.

# Conclusion

In this chapter, you learned about testing your app’s performance and how to spot where to make improvements. Never forget that poor performance leaves people out. People who live in rural areas, people who only use cell phones or older hardware, areas that have slow internet connections, and people who pay for cell plans by data use could be excluded if site performance is bad. Often, we as developers are used to having nice hardware with high-speed internet connections, so this can be easily overlooked. But don’t forget that not all users have this access.

As you learn more about how users interact with your app, you’ll be able to make these decisions with your team based on the data you see. It’s great to keep best practices in mind early in the feature-development process, but you never know what users will do or ask for until you show them the app.

Watch those key performance metrics as they change over time, and you’ll learn more about their behavior so that you can focus on the areas that really matter. The best ways to handle things like browser support and the cache will reveal themselves over time. Do your best in the beginning and don’t hesitate to make changes when the data shows you it’s time to do so. What’s important to users will change over time, too. As long as you know the main areas to focus on improving, you can discuss how to do this with the team and make new decisions over the life of the product.