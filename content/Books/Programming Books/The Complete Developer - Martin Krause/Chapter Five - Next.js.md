---
id: 01J9QC2HNS7RARMEAJHHJCAJZC
title: Chapter Five - Next.js
modified: 2024-10-08T20:50:23-04:00
tags:
  - react
  - nextjs
  - javascript
  - programming
  - web-development
  - books
---
In [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml), you used React to create responsive user interface components. But because React is just a library, building a full-stack application requires additional tools. In this chapter, we use Next.js, the leading web application framework built on top of React. To create an app with Next.js, you need to know only a few essential concepts. This chapter covers them.

Next.js streamlines the creation of an application’s frontend, middleware, and backend. On the frontend, it uses React. It also adds native CSS modules to define styles, and custom Next.js modules to perform routing, image handling, and additional frontend tasks. When it comes to the middleware and the backend, Next.js uses a built-in server to provide the entry points for HTTP requests and a clean API in which to work with request and response objects.

We’ll cover its filesystem-based approach to routing, discuss ways to build and render the web pages we deliver to clients, explore adding CSS files to style pages, and refactor our Express.js server to work with Next.js. This chapter uses the traditional _pages_ directory to teach you these basic concepts. To learn about Next.js’s alternative _app_ directory, see [Appendix B](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml).

### Setting Up Next.js

Next.js is part of the npm ecosystem. While you could manually install all of its required modules by running npm install next react react-dom and subsequently create all of your project’s files and folders by yourself, there is a much simpler way to set things up: running the create-next-app command.

Let’s create a sample application to use throughout this chapter. Follow these steps to set up a new empty folder called _sample-next_ and build your first Next.js application inside it. Keep the default answers from the setup wizard, and choose to use the traditional _pages_ directory instead of the _app_ directory:

```
$ mkdir sample-next
$ cd ./sample-next
$ npx create-next-app@latest --typescript --use-npm
--snip--
What is your project named? ... my-app
--snip--
Creating a new Next.js app in /Users/.../my-app.

Installing dependencies:
- react
- react-dom
- next

Installing devDependencies:
- eslint
- eslint-config-next
- typescript
- @types/react
- @types/node
- @types/react-dom
```

We create a new folder, switch to it, and then initialize a new Next.js project. We use the npx command instead of npm because, as you learned in [Chapter 1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter1.xhtml), npx doesn’t require us to install anything as a dependency or development dependency. We mentioned that a typical use case for it is scaffolding, which is precisely what we’re doing here.

The create-react-app command has a few options, of which only two are relevant to us: the --typescript option creates a Next.js project that supports TypeScript, and the --use-npm flag selects npm as a package manager.

We accept the default project name, my-app, and all the other default settings. The script creates a folder based on the project name containing the _package.json_ file and a complete sample project with all necessary files and folders. Finally, it installs the dependencies and development dependencies through npm.

> NOTE

_Instead of setting up a new project, you can use the online playgrounds at_ [https://codesandbox.io/s/](https://codesandbox.io/s/) _or_ [https://stackblitz.com](https://stackblitz.com/) _to run the Next.js code examples from this chapter. Just opt for the_ pages _directory setup instead of_ app _there as well._

#### Project Structure

Let’s explore the boilerplate Next.js app’s project structure. Enter the following commands to run it:

```
$ cd my-app
$ npm run dev

> my-app@0.1.0 dev
> next dev

ready - started server on 0.0.0.0:3000, url: http://localhost:3000
```

Visit the provided URL in your browser. You should see a default page similar to the one in [Figure 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#fig5-1) (this welcome page could change depending on your Next.js version).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure5-1.jpg)

Figure 5-1: The boilerplate Next.js app viewed in a browser

Now open the _my-app_ folder that the scaffolding script created, and look around. The _my-app_ folder contains a lot of folders, but only three are currently important to you: _public_, _styles_, and _pages_.

The _public_ folder holds all static assets, such as custom font files, all images, and files the app makes available for download. We’ll link to these assets from the app’s HTML and CSS files. The _pages_ folder contains all of the app’s routes. Each of its files is an endpoint belonging to a page route or an API route (in the _api_ subfolder).

> NOTE

_Recent versions of Next.js additionally include an_ app _directory that you can choose to use for routing as an alternative to the_ pages _directory. Because the_ app _directory uses more advanced concepts, this chapter covers the simpler_ pages _architectural style. However, you can learn more about the_ app _directory in [Appendix B](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml), where we’ll cover its use in detail._

In the _my-app_ folder, we also find the __app.tsx_ file, which is Next.js’s equivalent to the _App.tsx_ file we used in [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml). This is the entry point for the whole application and the place where we’ll add our global styles, components, and context providers. Finally, the _styles_ folder contains the global CSS files and modules for locally scoped, component-specific files.

#### Development Scripts

The technologies our app uses, including TypeScript, React, and JSX, don’t run directly in the browser. They all require a build pipeline with a reasonably complex transpiler. Next.js provides four command line scripts to simplify development.

The npx next dev and npm run dev commands starts the application at _http://localhost:3000_ in development mode. As a result, Next.js rebuilds and reloads the rendered application in the browser window as soon as we change a file. In addition to this _hot-code_ reloading, the development server also displays errors and warning messages to aid the application’s development. The installation wizard adds the server to _package.json_’s script section, so we can start it with npm run dev as well.

The npx next build and npm run build commands create the optimized build of our application. They remove unused code and reduce the file size of our scripts, styles, and all other assets. You’ll use them for the live deployment. The npx next start and npm run start commands run the optimized application at _http://localhost:3000_ in production mode on the built-in server or in a serverless environment. This production build relies on a previously created build. Hence, we must have first run the build command.

Finally, npx next export and npm run export commands create a stand-alone version of your application that is independent of the built-in Next.js server and can run on any infrastructure. This version of your app won’t be able to use features that require Next.js on the server side, however. Consult the official Next.js documentation at [_https://nextjs.org_](https://nextjs.org/) for a guide to using it.

### Routing the Application

When we built our sample Express.js server, we created an explicit routing file that mapped each of the app’s endpoints to distinct functions that performed corresponding behavior. Next.js offers a different, perhaps simpler, routing system; it automatically creates the app’s routes based on the files in the _pages_ directory. If a file in this folder exports a React component (in the case of a web page) or an async function (in the case of an API), it becomes a valid endpoint, as either an HTML page or an API.

In this section, we’ll revisit the routes we created in our Express.js server and remake them using Next.js’s routing technique.

#### Simple Page Routes

For our Express.js server, we manually created a _/hello_ route in the _index.ts_ file. When visited, it returned Hello World! Let’s convert this route to a page-based one in Next.js. The simplest kind of page route consists of a file placed directly in the _pages_ directory. For example, the _pages/index.tsx_ file, created by default, maps to _http://localhost:3000_. To create a simple _/hello_ route, make a new file, _hello.tsx_, in that directory. Now add to it the code from [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1).

```
import type {NextPage} from "next";

const Hello: NextPage = () => {
    return (<>Hello World!</>);
}

export default Hello;
```

Listing 5-1: The pages/hello.tsx file

Our Express.js server used the routeHello function to return the Hello World! string. Here we need to add a little more code to export a React component. First we import the custom type NextPage from the Next.js module and use it to create a constant, Hello. We assign the constant a fat arrow function that returns a NextPage, which is nothing but a custom wrapper for React components. In this case, we return JSX that renders the Hello World! string. Finally, we export the NextPage as the file’s default export.

Run the server and navigate to _http://localhost:3000/hello_. The page you see should show Hello World! as its content.

The page looks different from the one in the sample Express.js server. That’s because Next.js automatically renders all global styles defined in the __app.tsx_ file to each page. Hence, the font looks different, even though we didn’t explicitly define any styles in the _hello.tsx_ file.

#### Nested Page Routes

_Nested_ routes, such as _/components/weather_ from the sample Express.js server, are logical subroutes of other routes. In other words, _weather_ is nested inside the _components_ entry point. You’ve probably already guessed how we create a nested route with Next.js’s page-routing pattern. Yes, we merely create a subfolder, and Next.js maps the folder structure to the URL schema.

Create a new folder, _components_, inside the _pages_ folder and add a new file, _weather.tsx_, there. [Figure 5-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#fig5-2) depicts the relationship between the URL _components/weather_ and the file structure _pages/components/weather.tsx_.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure5-2.jpg)

Figure 5-2: The relationship between the URL components/weather and the file structure pages/components/weather.tsx

Our _pages_ folder is the root of the URL, and each nested folder becomes a URL segment. The file that exports the NextPage is the _leaf segment_, or the final part of the URL. For this file, we reuse the weather component code we wrote in [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml), shown in [Listing 5-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-2).

```
import type {NextPage} from "next";
import React, {useState, useEffect} from "react";

const PageComponentWeather: NextPage = () => {

    interface WeatherProps {
        weather: string;
    }

    const WeatherComponent = (props: WeatherProps) => {

        const [count, setCount] = useState(0);
        useEffect(() => {
            setCount(1);
        }, []);

        return (
            <h1 onClick={() => setCount(count + 1)}>
                The weather is {props.weather},
                and the counter shows {count}
            </h1>
        );
    };

    return (<WeatherComponent weather="sunny" />);
};

export default PageComponentWeather;
```

Listing 5-2: The pages/components/weather.tsx file

The only difference from the functional component created in [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml) is that we wrap the code in a function that returns a NextPage, which we then export as the default export. This is consistent with the page we created in [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1) and follows Next.js’s pattern requirement.

Visit the new page at _http://localhost:3000/components/weather_ in the browser. It should look similar to [Figure 5-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#fig5-3).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure5-3.jpg)

Figure 5-3: The pages/components/weather.tsx file is rendered at the /components/weather URL in the browser.

You should recognize the click-handler functionality you saw in [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml).

#### API Routes

In addition to a user-friendly interface, a full-stack application might also need a machine-readable interface. For example, the Food Finder application you’ll create in [Part II](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/part2.xhtml) will provide an API to external services so that a mobile app or a third-party widget can display our wish list. As JavaScript-driven full-stack developers, the most common API formats we’ll use are GraphQL and REST, and we talk about these in depth in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml). Here we will create REST APIs, which we like for their simplicity.

With Next.js, we can design and create APIs via the same patterns we use for pages. Each file in the _pages/api/_ folder is a single API endpoint, and we can define nested API routes in the same way we define nested page routes. However, unlike page routes, API routes are not React components. Instead, they are async functions that take two parameters, NextApiRequest and NextApiResponse, and return a NextApiResponse and JSON data.

There are two caveats you need to remember when it comes to API routes. First, they do not specify a _Cross-Origin Resource Sharing (CORS)_ header by default. This set of HTTP headers, most notably the Access -Control-Allow-Origin header, lets a server define the origins from which client-side scripts can request resources. If you want client-side scripts running in websites on third-party domains to access your API endpoints, you’ll need to add additional middleware to enable CORS directly in the Next.js server. Otherwise, external requests will prompt a CORS error. This isn’t specific to Next.js; Express.js and most other server frameworks require you to do the same.

The second caveat is that static exports done by running next export do not support API routes. They rely on the built-in Next.js server and cannot run as static files.

We used one API route, _api/names_, in the Express.js server. Now let’s refactor the code and convert it to a Next.js API route. As before, create a new file, _names.ts_, and place it in the _api_ folder. Because API routes return an async function instead of JSX, we use the _.ts_ extension, not the _.tsx_ extension used for JSX code. Paste the code from [Listing 5-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-3) into the file.

```
import type {NextApiRequest, NextApiResponse} from "next";

type responseItemType = {
    id: string;
    name: string;
};

export default async function handler(
    req: NextApiRequest,
    res: NextApiResponse
): Promise<NextApiResponse<responseItemType[]> | void> {
    const url = "https://www.usemodernfullstack.dev/api/v1/users";
    let data;
    try {
        const response = await fetch(url);
        data = (await response.json()) as responseItemType[];
    } catch (err) {
        return res.status(500);
    }
    const names = data.map((item) => {
        return {id: item.id, name: item.name};
    });
    return res.status(200).json(names);
}
```

Listing 5-3: The pages/api/names.ts file

First we import the custom types for the API request and response from the Next.js package. Then we define the custom type for the API response. In [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml), we created the same type for typing the await call in the _routes.ts_ file. We’re using the same code and await call here, so we’ve reused the type as well. We then create and directly export the API handler function mentioned earlier. You learned in [Chapter 2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml) that async functions need to return a promise as their return type. Therefore, we wrap this API response in a promise.

The code in the function’s body is similar to the code in the routeAPINames function from [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml). It makes an API request to fetch the user data, converts the received data into the desired return format, and finally returns the data. However, we need to make a few modifications. First, instead of returning an error string, we return an API response with no content and a generic status code of _500_, for an _Internal Server Error_.

The second adjustment involves the data mapping. Previously, we returned a string that rendered in the browser. Now, instead of this string, we return a JSON object. Therefore, we modify the array.map function to create an array of objects. Finally, we change the return statement to return the API response with the names object as JSON and a status code of _200: OK_.

Now open the new API route in the browser at _http://localhost:3000/api/names_. You should see the API response shown in [Figure 5-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#fig5-4).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure5-4.jpg)

Figure 5-4: The pages/api/names.ts file rendered from /api/names in the browser

#### Dynamic URLs

You now know how to create page and API routes, which are the foundation of any full-stack application. However, you might be wondering how to create _dynamic_ URLs, which change based on input. We often use dynamic URLs for profile pages, where the user’s name becomes part of the URL. In fact, we implemented a dynamic URL in the Express.js server’s weather API when we defined the route _/api/weather/:zipcode_ in the _index.ts_ file. There, _zipcode_ was a dynamic parameter, or a dynamic leaf segment, whose value was provided by the req.params.zipcode function.

Next.js uses a slightly different pattern for dynamic URLs. Because it creates the routes based on folders and files, we need to define dynamic segments through their filenames by wrapping the variable portion in square brackets ([]). The dynamic route _/api/weather/:zipcode_ from the Express.js server would thus translate to the file _/api/weather/[zipcode].ts_.

Let’s create a dynamic route in our sample Next.js application that mimics the _/api/weather/:zipcode_ route from the Express.js server. Make a new folder, _weather_, in the _api_ folder, and place a file named _[zipcode].ts_ in it. Then paste the code from [Listing 5-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-4) into the file.

```
import type {NextApiRequest, NextApiResponse} from "next";

type WeatherDetailType = {
    zipcode: string;
    weather: string;
    temp?: number;
};

export default async function handler(
    req: NextApiRequest,
    res: NextApiResponse
): Promise<NextApiResponse<WeatherDetailType> | void> {

    return res.status(200).json({
        zipcode: req.query.zipcode,
        weather: "sunny",
        temp: 35
    });

}
```

Listing 5-4: The api/weather/[zipcode].ts file

This code should be familiar to you, as it follows the basic outline of an API route in Next.js. We import the necessary types, then define a custom type, WeatherDetailType, and use it as the type for the data returned by the function. (By the way, this is the same type definition we created in [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml).) In the function’s body, we return the response with a status code of _200: OK_ and a JSON object. We fill the zipcode property with the ZIP code from the dynamic URL parameter, retrieved with req.query .zipcode.

When you run the server, the browser should show the JSON response with the dynamic URL parameter in the response type. If you navigate to _http://localhost:3000/api/weather/12345_, you should see the API response. If you change the “12345” part of the URL and request the data again, the response data should change accordingly.

Note that the dynamic route _/api/weather/[zipcode].ts_ matches _/api/weather/12345_ and _/api/weather/54321_ but not sub-paths of those routes, such as _/api/weather/location/12345_ or _/api/weather/location/54321_. For this, you’ll need to use a _catch all_ API route, which includes all paths that are inside the current path. You can create a catch all route by adding three dots (...) in front of the filename. For example, the catch all route _/api/weather/[...zipcode].ts_ could handle all four API endpoints mentioned in this paragraph.

### Styling the Application

To add styles to our Next.js application, we create regular CSS files, written without the vendor prefixes used in other frameworks. Later, Next.js’s postprocessor will add necessary properties to generate backward-compatible styles. While CSS syntax is beyond the scope of this book, this section describes how to use Next.js’s two kinds of CSS styles: global styles and locally scoped component styles, defined in CSS modules.

#### Global Styles

Global styles affect all pages of an app. We stumbled across this behavior when we rendered the _hello.tsx_ file; the page used CSS even though we hadn’t added any style information ourselves.

Practically speaking, global styles are just regular CSS files. They aren’t modified during the build, and their class names are guaranteed to stay the same. Therefore, we can use them as regular CSS classes across the application. We import these CSS files in the app’s entry point, the _pages/_app.tsx_ file. Take a look at those in the boilerplate project. You should see a line of code similar to [Listing 5-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-5).

```
import "@/styles/globals.css";
```

Listing 5-5: Importing global styles in the _app.tsx file

Of course, you can adjust the filename and location of the imported styles or import multiple files. Try playing around by adding a few styles in the _global.css_ file and some regular CSS classes to the HTML elements in the _hello.tsx_ file. Then visit the page at _htttp://localhost:3000/hello_ to see how it changed.

#### Component Styles

In [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml), you saw that React lets us create user interfaces out of independent, reusable components. Global styles aren’t the best approach for styling independent components, as they require us to keep track of the names we’ve already used in various components, and if we import components from a previous project, we risk having the CSS classes collide with one another.

We need the CSS classes to be scoped to individual modules to work efficiently with modularized components. There are multiple architectural patterns for implementing this. For example, using the Block Element Modifier methodology, you can manually scope the styles to a component or a user interface block.

Luckily, we don’t need to bother with such a clumsy solution. Next.js lets us use CSS modules that are scoped during the build process. These CSS modules follow the naming convention <_component_>._module.css_. The compiler automatically prefixes each CSS class name inside the module with the component’s name and a unique identifier. This enables you to use the same style names for multiple components without issue.

The actual CSS you write won’t have these prefixes. For example, look at the _Home.module.css_ file inside the _styles_ folder, shown in [Listing 5-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-6).

```
.container {
    padding: 0 2rem;
}
```

Listing 5-6: Regular CSS code in styles/Home.module.css

One problem is that, because the build process modifies the class names and prefixes them, we can’t directly use these styles in our other files. Instead, we must import the styles and treat them like a JavaScript object. Then we can refer to them as a property of the styles object. For example, the _pages/index.tsx_ file in [Listing 5-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-7) uses the container class from [Listing 5-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-6), providing an example of how to use scoped styles.

```
import styles from "../styles/Home.module.css"
--snip--
const Home: NextPage = () => {
    return (
        <div className={styles.container}>
        --snip--
        </div>
    );
};
```

Listing 5-7: Using styles from the CSS module styles/Home.module.css in the index.tsx file

This code imports the CSS file into a constant called styles. Now all the CSS class names will be available as properties of the styles object. In JSX, we use variables wrapped in curly brackets ({}), so we add a reference to the container class as {styles.container}.

You can now build APIs and custom-styled pages out of React components. The next section introduces useful custom components that Next.js provides to enhance your full-stack application.

### Built-in Next.js Components

Next.js provides a set of custom components. Each of these addresses one specific use case: for example, accessing internal page properties such as the page title or SEO metadata (next/head), improving the app’s overall rendering performance and user experience (next/image), or enabling the application’s routing (next/link). We’ll use the Next.js components covered in this chapter in the full-stack application in [Part II](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/part2.xhtml), where you can see them applied in practice. For additional attributes and niche use cases, refer to the Next.js documentation.

#### The next/head Component

The next/head component exports a custom Next.js-specific Head component. We use it to set a page’s HTML title and meta elements, which are found inside an HTML head component. To improve SEO ranking and enhance usability, each page should have its own metadata. [Listing 5-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-8) shows an example of the _hello.tsx_ page from [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1) with a customized title and meta element.

It is important to remember that the Head elements are not merged across pages. Next.js’s client-side routing removes the content of the Head element during the page transition.

```
import type {NextPage} from "next";
import Head from "next/head";

const Hello: NextPage = () => {
    return (
        <div>
            <Head>
                <title>Hello World Page Title</title>
                <meta property="og:title" content="Hello World" key="title" />
            </Head>
            <div>Hello World!</div>
        </div>
    );
};

export default Hello;
```

Listing 5-8: The pages/hello.tsx file with a customized title and meta element

We import the Head element from the next/head component and add it to the returned JSX element, placing it above the existing content and wrapping both in another div element because we need to return one element instead of two.

#### The next/link Component

The next/link component exports a Link component. This component is built on top of the React Link element. We use it instead of an HTML anchor tag when we want to link to another page in the application, enabling client-side transitions between pages. When clicked, the Link component updates the browser DOM with the new DOM, scrolls to the top of the new page, and adjusts the browser history. Furthermore, it provides built-in performance optimizations, prefetching the linked page and its data as soon as the Link component enters the _viewport_ (the currently visible part of the website). This background prefetch enables smooth page transitions. [Listing 5-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-9) adds the Next.js Link element to the page from the previous listing.

```
import type {NextPage} from "next";
import Head from "next/head";
import Link from "next/link";

const Hello: NextPage = () => {
    return (
        <div>
            <Head>
                <title>Hello World Page Title</title>
                <meta property="og:title" content="Hello World" key="title" />
            </Head>
            <div>Hello World!</div>
            <div>
                Use the HTML anchor for an
                <a href="https://nostarch.com" > external link</a>
                and the Link component for an
                <Link href="/components/weather"> internal page
                </Link>
                .
            </div>
        </div>
    );
};

export default Hello;
```

Listing 5-9: The pages/hello.tsx file with an external link and an internal next/link element

We import the component, then add it to the returned JSX element. For comparison purposes, we use a regular HTML anchor to link to the No Starch Press home page and the custom Link to connect to the weather component page in our Next.js application. In the app, try clicking both links to see the difference.

#### The next/image Component

The next/image component exports an Image component used to display images. This component is built on top of the native HTML <img> element. It handles common layout requirements, such as filling all available space and scaling images. The component can load modern image formats, such as _AVIF_ and _WebP_, and serve the image with the correct size for the client’s screen. Furthermore, you have the option to use blurred placeholder images and lazy-load the actual image as soon as it enters the viewport; this enforces the visual stability of your website by preventing _cumulative layout shifts_, which occur when an image renders after the page, causing the page content to shift down. Cumulative layout shifts are considered a bad user experience, and they can make the user lose their focus. [Listing 5-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-10) provides a basic example of the next/image component.

```
import type {NextPage} from "next";
import Head from "next/head";
import Link from "next/link";
import Image from "next/image";

const Hello: NextPage = () => {
    return (
        <div>
            <Head>
                <title>Hello World Page Title</title>
                <meta property="og:title" content="Hello World" key="title" />
            </Head>
            <div>Hello World!</div>
            <div>
                Use the HTML anchor for an <a href="https://nostarch.com">
                external link</a> and the Link component for an
                <Link href="/components/weather"> internal page</Link>.
                <Image
                    src="/vercel.svg"
                    alt="Vercel Logo"
                    width={72}
                    height={16}
                />
            </div>
        </div>
    );
};
export default Hello;
```

Listing 5-10: The pages/hello.tsx file using the next/image element

Here we display the Vercel logo from our application’s _public_ folder. First we import the component from the _next/image_ package. Then we add it to the page content. The syntax and the properties of our minimal example are similar to the HTML img element. You can read more about the component’s advanced properties in the official documentation at [_https://nextjs.org/docs/api-reference/next/image_](https://nextjs.org/docs/api-reference/next/image).

### Pre-rendering and Publishing

While you can start building full-stack Next.js applications with the information you’ve learned so far, you’ll find it useful to know one more advanced topic: the different ways to render and publish your application and their implications for its performance.

Next.js provides three options for pre-rendering your app with its built-in server. The first, _static site generation (SSG)_, generates the HTML at build time. Thus, each request will always return the same HTML, which remains static and is never re-created. The second option, _server-side rendering (SSR)_, generates new HTML files on each request, and the third, _incremental static regeneration (ISR)_, combines both approaches.

Next.js lets us choose our pre-rendering option on a per-page basis, meaning the full-stack application can contain pages with SSG, SSR, and ISR, as well as client-side rendering for some React components. You can also create a complete static export of your site by running next export. The exported application will run independently on all infrastructures, as it doesn’t need the built-in Next.js server.

To gain experience with these rendering approaches, we’ll create a new page that displays the data from our names API for each rendering option. Create a new folder, _utils_, next to the _pages_ folder and add an empty file, _fetch-names.ts_, to it. Then add the code in [Listing 5-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-11). This utility function calls the remote API and returns the dataset.

```
type responseItemType = {
    id: string;
    name: string;
};

export const fetchNames = async () => {
    const url = "https://www.usemodernfullstack.dev/api/v1/users";
    let data: responseItemType[] | [] = [];
    let names: responseItemType[] | [];
    try {
        const response = await fetch(url);
        data = (await response.json()) as responseItemType[];
    } catch (err) {
        names = [];
    }
    names = data.map((item) => {return {id: item.id, name: item.name}});

    return names;
};
```

Listing 5-11: The async utility in utils/fetch-names.ts

After defining a custom type, we create a function and directly export it. This function contains the code from the previously created _names.ts_ file, with two adjustments: first we need to define the data array as possibly empty; next, we return an empty array instead of an error string if the API call fails. This change means that we don’t need to verify the type before iterating over the array when we generate the JSX string.

#### Server-Side Rendering

Using SSR, Next.js’s built-in Node.js server creates an application’s HTML in response to each request. You should use this technique if your page depends on fresh data from an external API. Unfortunately, SSR is slower in production, because the pages aren’t easily cacheable.

To use SSR for a page, export an additional async function, getServerSideProps, from that page. Next.js calls this function on every request and passes the fetched data to the page’s props argument to pre-render it before sending it to the client.

Try this out by creating a new file, _names-ssr.tsx_, in the _pages_ folder. Paste the code from [Listing 5-12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-12) into the file.

```
import type {
    GetServerSideProps,
    GetServerSidePropsContext,
    InferGetServerSidePropsType,
    NextPage,
    PreviewData
} from "next";
import {ParsedUrlQuery} from "querystring";
import {fetchNames} from "../utils/fetch-names";

type responseItemType = {
    id: string;
    name: string;
};

const NamesSSR: NextPage = (props: InferGetServerSidePropsType<typeof getServerSideProps>) => {

    const output = props.names.map((item: responseItemType, idx: number) => {
        return (
            <li key={`name-${idx}`}>
                {item.id} : {item.name}
            </li>
        );
    });

    return (
        <ul>
            {output}
        </ul>
    );
};

export const getServerSideProps: GetServerSideProps = async (
    context: GetServerSidePropsContext<ParsedUrlQuery, PreviewData>
) => {

    let names: responseItemType[] | [] = [];
    try {
        names = await fetchNames();
    } catch(err) {}
    return {
        props: {
          names
        }
    };
};

export default NamesSSR;
```

Listing 5-12: A basic page that displays data with SSR, page/names-ssr.tsx

To use Next.js’s SSR, we export the additional async function, getServerSideProps. We also import necessary functionality from the _next_ and _querystring_ packages and the fetchNames function we created earlier. Then we define the custom type for the response to the API request. It’s the same custom type we used in [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml).

Next, we create the page and store the export as the default one. The page returns a NextPage and takes the default properties for this page type. We iterate over the props parameter’s names array and create a JSX string that we render and return to the browser. Then we define the getServerSideProps function, which gets the data from the API. We return the created dataset from the async function and pass it to the NextPage inside the page properties.

Navigate to the new page at _http://localhost:3000/names-ssr_. You should see the list of the usernames.

#### Static Site Generation

SSG creates the HTML files only once and reuses them for every request. It is the recommended option, because pre-rendered pages are easy to cache and fast to deliver. For example, a content delivery network can easily pick up your static files.

Usually, SSG applications have a lower _time to first paint_, or the time it takes after a user requests the page (by, for example, clicking a link) until the content appears in the browser. SSG also reduces _blocking time_, or the time it takes until the user can actually interact with the page’s content. Good scores in these metrics indicate a responsive website, and they are part of Google’s scoring algorithm. Hence, these pages have increased SEO rankings.

If your page relies on external data, you can still use SSG by exporting an additional async function, getStaticProps, from the page’s file. Next.js calls this function at build time, passes the fetched data to the page’s props argument, and pre-renders the page with SSG. Of course, this works only if the external data isn’t dynamic.

Try creating the same page as in the SSR example, this time with SSG. Add a new file, _names-ssg.tsx_, in the _pages_ folder and then paste in the code shown in [Listing 5-13](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-13).

```
import type {
    GetStaticProps,
    GetStaticPropsContext,
    InferGetStaticPropsType,
    NextPage,
    PreviewData,
} from "next";
import {ParsedUrlQuery} from "querystring";
import {fetchNames} from "../utils/fetch-names";

type responseItemType = {
    id: string,
    name: string,
};

const NamesSSG: NextPage = (props: InferGetStaticPropsType<typeof getStaticProps>) => {
     
    const output = props.names.map((item: responseItemType, idx: number) => {
        return (
            <li key={`name-${idx}`}>
                {item.id} : {item.name}
            </li>
        );
    });
     
    return (
        <ul>
            {output}
        </ul>
    );
};

export const getStaticProps: GetStaticProps = async (
    context: GetStaticPropsContext<ParsedUrlQuery, PreviewData>
) => {

    let names: responseItemType[] | [] = [];
    try {
        names = await fetchNames();
    } catch (err) {}
     
    return {
        props: {
            names
        }
    };
};

export default NamesSSG;
```

Listing 5-13: A page that displays data with SSG, page/names-ssg.tsx

The code is mostly identical to [Listing 5-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-9). We just need to change the SSR-specific code to use SSG. Therefore, we export getStaticProps instead of getServerSideProps and adjust the types accordingly.

When you visit the page, it should look similar to the SSR page. But instead of requesting fresh data on each visit to _http://localhost:3000/names-ssg_, the data is requested only once, on page build.

#### Incremental Static Regeneration

ISR is a hybrid of SSG and SSR that runs purely on the server side. It generates the HTML on the server during the initial build and sends this pre-generated HTML the first time a page is requested. After a specified time has passed, Next.js will fetch the data and regenerate the page on the server in the background. In the process, it invalidates the internal server cache and updates it with the new page. Every subsequent request will receive the up-to-date page. Like SSG, ISR is less costly than SSR and increases a page’s SEO ranking.

To enable ISR in SSG pages, we need to add a property to revalidate getStaticProp’s return object. We define the validity of the data in seconds, as shown in [Listing 5-14](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-14).

```
return {
        props: {
        names,
        revalidate: 30
    }
};
```

Listing 5-14: Changing getServerSideProps to enable ISR

We add the revalidate property with a value of 30. As a result, the custom Next.js server will invalidate the current HTML 30 seconds after the first page request.

#### Client-Side Rendering

A completely different approach, _client-side rendering_ involves first generating the HTML with SSR or SSG and sending it to the client. The client then fetches additional data at runtime and renders it in the browser DOM. Client-side rendering is a good choice when working with highly flexible, constantly changing datasets, such as real-time stock market or currency prices. Other sites use it to send a skeleton version of the page to the client and later enhance it with more content. However, client-side rendering lowers your SEO performance, as its data can’t be indexed.

[Listing 5-15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-15) shows the page we created earlier, configured for client-side rendering. Create a new file, _names-csr.tsx_, in the _pages_ folder and then add the code to it.

```
import type {
    NextPage
} from "next";
import {useEffect, useState} from "react";
import {fetchNames} from "../utils/fetch-names";

type responseItemType = {
    id: string,
    name: string,
};

const NamesCSR: NextPage = () => {
    const [data, setData] = useState<responseItemType[] | []>();
    useEffect(() => {
        const fetchData = async () => {
            let names;
            try {
                names = await fetchNames();
            } catch (err) {
                console.log("ERR", err);
            }
            setData(names);
        };
        fetchData();
    });

    const output = data?.map((item: responseItemType, idx: number) => {
        return (
            <li key={`name-${idx}`}>
                {item.id} : {item.name}
            </li>
        );
    });

    return (
        <ul>
            {output}
        </ul>
    );

};

export default NamesCSR;
```

Listing 5-15: The client-side rendered page, page/names-csr.tsx

This code differs significantly from the previous examples. Here we import the useState and useEffect hooks. The latter one will fetch the data after the page is already available. As soon as the fetchNames function returns the data, we use the useState hook and the reactive data state variable to update the browser DOM.

We cannot declare the useEffect hook as an async function, because it returns either an undefined value or a function, whereas an async function returns a promise, and therefore TSC would throw an error. To avoid this, we need to wrap the await call in an async function, fetchData, and then call that function.

The page configured for client-side rendering should look similar to the other versions. But when you visit _http://localhost:3000/names-csr_, you might see a white flash. This is the page waiting for the asynchronous API request.

To get a better feel for the different rendering types, modify the code for each example in this section to use the API _[https://www.usemodernfullstack.dev/api/v1/now](https://www.usemodernfullstack.dev/api/v1/now)_, which returns an object with the timestamp of the request.

#### Static HTML Exporting

The next export command generates a static HTML version of your web application. This version is independent of the built-in Node.js-based Next.js web server and can run on any infrastructure, such as an Apache, NGINX, or IIS server.

To use this command, your page must implement getStaticProps, as in SSG. This command won’t support the getServerSideProps function, ISR, or API routes.

Exercise 5: Refactor Express.js and React to Next.js

Let’s refactor the React and Express.js applications from the previous chapters into a Next.js application that we’ll expand in the upcoming chapters. As a first step, we’ll summarize the functionality we need to build. Our application has an API route, _api/names_, that returns usernames, and another API route, _api/weather/:zipcode_, that returns a static JSON object and the URL parameter. We used it to understand dynamic URLs. In addition, we created pages at _/hello_ and _component/weather_.

Throughout this chapter, we’ve already refactored these various elements to work with Next.js’s routing style. In this exercise, we’ll put it all together. Follow the steps in “Setting Up Next.js” on page 70 to initialize the Next.js application. Within the _sample-next_ folder, name your application _refactored-app_.

#### Storing Custom Interfaces and Types

We create a new file, _custom.d.ts_, in the root of the project to store our custom interface and type definitions ([Listing 5-16](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-16)). It is similar to the one we used in [Chapters 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml) and [4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml). The main difference is that we add custom types for the Next.js application.

```
interface WeatherProps {
    weather: string;
}

type WeatherDetailType = {
    zipcode: string;
    weather: string;
    temp?: number;
};

type responseItemType = {
    id: string;
    name: string;
};
```

Listing 5-16: The custom.d.ts file

We’ll use the custom interface WeatherProps for the props argument of the page that displays the weather components, _components/weather_. The WeatherDetailType is for the API route _api/weather/:zipcode_, which uses a dynamically fetched ZIP code. Finally, we use responseItemType in the API route _api/names_ to type the fetch response.

#### Creating the API Routes

Next, we re-create the two API routes from the Express.js server. Earlier sections of this chapter showed this refactored code. For the _api/names_ route, create a new file, _names.ts_, in the _api_ folder, then add the code from [Listing 5-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-3). Refer to that section for a detailed explanation of the code.

Migrate the dynamic route _api/weather/:zipcode_ from the Express.js server to the Next.js application by creating a _[zipcode].js_ file in the _api_ folder and adding the code from [Listing 5-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-4), shown in “Dynamic URLs” on page 77. You can refer to that section for more details.

#### Creating the Page Routes

Now we work on the pages. First, for the simple page _hello.tsx_, we create a new file in the _pages_ folder and add the code from [Listing 5-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-10). This code renders the Hello World! example and uses the custom Next.js components Head, Link, and Image, all of which are explained in detail in “Built-in Next.js Components” on page 80.

The second page is the nested route _pages/components/weather.tsx_. As before, we create a new file, _weather.tsx_, in a folder called _components_, within the _pages_ folder. Add the code from [Listing 5-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-2). This listing uses the useState and useEffect hooks to create a reactive user interface. We can remove the custom interface definition for the WeatherProps from this file. The _custom.d.ts_ file already adds them to TSC.

#### Running the Application

Start the application with the npm run dev command. Now you can visit the same routes we created for the Express.js server and see that they are functionally the same. Congratulations! You created your first Next.js-based full-stack application. Play around with the code and try using global and component CSS to style your pages.

### Summary

Next.js adds the missing functionality needed to create full-stack applications with React. After scaffolding a sample project and exploring the default file structure, you learned how to create page and API routes in the framework. You also learned about global- and component-level CSS, Next.js’s four built-in command line scripts, and its most useful custom components.

We also discussed the different ways to render content and pages with Next.js and when to choose each option. Finally, you used the code from this chapter to quickly migrate the Express.js application you built in the previous chapters to Next.js. To continue your adventures in this useful framework, I recommend the official tutorials at [_https://nextjs.org_](https://nextjs.org/).

In the next chapter, we’ll explore two types of web APIs: the standard RESTful APIs and modern GraphQL.