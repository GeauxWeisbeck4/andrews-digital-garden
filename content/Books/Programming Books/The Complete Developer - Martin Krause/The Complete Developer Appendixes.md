---
id: 01J9QDWGSC6DR1393AZTRC3S3V
modified: 2024-10-08T21:25:50-04:00
---
# Appendix A - TypeScript Compiler Options

Pass any of these options to the _tsconfig.json_ file’s compilerOptions field to configure TSC’s transpilation of TypeScript code to JavaScript. For more information about this process, see [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml).

Here we look at the most common options. You can find more information and the complete list in the official documentation at [_https://www.typescriptlang.org/tsconfig_](https://www.typescriptlang.org/tsconfig).

allowJs  A Boolean that specifies whether the project can import JavaScript files.

baseUrl  A string that defines the root directory to use for resolving module paths. For example, if you set it to "./", TypeScript will resolve file imports from the root directory.

esModuleInterop  A Boolean that specifies whether TypeScript should import CommonJS, AMD, or UMD modules seamlessly or treat them differently from ES.Next modules. In general, this is necessary if you use third-party libraries without ES.Next module support.

forceConsistentCasingInFileNames  A Boolean that specifies whether file imports are case sensitive. This can be important when some developers are working on case-sensitive filesystems and others are not, to ensure file-loading behaviors are consistent for everyone.

incremental  A string that defines whether the TypeScript compiler should save the last compilation’s project graph, use incremental type checks, and perform incremental updates on consecutive runs. This can make transpiling faster.

isolatedModules  A Boolean that specifies whether TypeScript should issue warnings for code not compatible with third-party transpilers (such as Babel). The most common cause for those warnings is that the code uses files that are not modules; for example, they don’t have any import or export statements. This value doesn’t change the behavior of the actual JavaScript; it only warns about code that can’t be correctly transpiled.

jsx  A string that specifies how TypeScript handles JSX. It applies only to _.tsx_ files and how the TypeScript compiler emits them; for example, the default value react transforms and emits the code by using React .createElement, whereas preserver does not transform the code in your component and emits it untouched.

lib  An array that adds missing features through polyfills. In general, _polyfills_ are snippets of code that add support for features and functions the target environment does not support natively. We need to emulate modern JavaScript features when we target less-compliant systems, such as older browsers or node versions. The compiler adds the polyfills defined in the lib array to the generated code.

module  A string that sets the module syntax for the transpiled code. For example, if you set it to commonjs, TSC will transpile this project to use the legacy CommonJS module syntax with require for importing and module.exports for exporting the code, whereas with ES2015 the transpiled code will use the import and export keywords. This is independent of the target property, which defines all available language features except the module syntax.

moduleResolution  A string that specifies the module resolution strategy. This strategy also defines how TSC locates definition files for modules at compile time. Changing the approach can resolve fringe problems with the importing and exporting of modules.

noEmit  A Boolean that defines whether TSC should produce files or only check the types in the project. Set it to false if you want third-party tools such as webpack, Babel.js, or Parcel to transpile the code instead of TSC.

resolveJsonModule  A Boolean that specifies whether TypeScript imports JSON files. It generates type definitions based on the JSON inside the file and validates the types on import. We need to manually enable JSON imports as TypeScript can’t import them by default.

skipLibCheck  A Boolean that defines whether the TypeScript compiler performs type checks on all type declaration files. Setting it to false decreases compilation time and is your escape hatch for working with untyped third-party dependencies.

target  A string that specifies the language features to which the TypeScript code should be transpiled. For example, if you set it to es6, or the equivalent ES2015, TSC will transpile this project to ES2015-compatible JavaScript, which, for example, uses let and const.

# Appendix B - The Next.js App Directory

In version 13, Next.js introduced a new routing paradigm that uses an _app_ directory instead of the _pages_ directory. This appendix discusses this new feature so that you can explore it further on your own. As there are no plans to deprecate the _pages_ directory, you can continue using the routing approach you learned in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). You can even use both directories simultaneously; just be careful not to add to both directories folders and files that would create the same route, as this could cause errors.

Both the _app_ and _pages_ directories use folders and files to create routes. However, the _app_ directory distinguishes between server and client components. In the _pages_ folder, everything is a _client component_, meaning that all the code is part of the JavaScript bundle Next.js sends to the client. But every file in the _app_ directory is a _server component_ by default, and its code is never sent to the client.

This appendix takes a look at the basic concepts of the new approach and then initializes a Next.js application using the new structure.

### Server Components vs. Client Components

The terms _client_ and _server_ in this context refer to the environments in which the Next.js runtime renders a component. The client environment is the user’s environment (usually the browser), whereas the server refers to the Next.js server that receives the request from the client, whether it runs on your local host or in a remote location.

With the introduction of server components, Next.js no longer purely uses client-side routing. In _server-centric_ routing, the server renders components and then sends the rendered code to the client. This means the client doesn’t download a routing map, which reduces the initial page size. Additionally, the user doesn’t have to wait until all resources have loaded before the page becomes interactive. Next.js server components leverage React’s streaming architecture to progressively render each component’s content. With this model, the page becomes interactive before it has finished loading.

#### Server Components

Next.js server components build upon the React server components that have been available since React version 18. Because the server renders these components, they don’t add anything to the JavaScript sent to the client, reducing the overall page size and increasing page performance scores. Also, the JavaScript bundle is cacheable, so the client won’t redownload it when we add new additional server components, only when we add new client-side scripts through additional client components.

In addition, because these components are rendered completely on the server, they can contain sensitive server information, such as access tokens and API keys. (To add an additional layer of protection, Next.js’s rendering engine replaces with an empty string all environment variables that are not explicitly prefixed with NEXT_PUBLIC.) Finally, we can use large-scale dependencies and additional frameworks without bloating the client-side scripts and access backend resources directly, increasing the application’s performance.

[Listing B-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-1) shows the basic structure of a server component.

```
export default async function ServerComponent(props: WeatherProps): Promise<JSX.Element> {

    return (
      <h1>The weather is {props.weather}</h1>
    );
}
```

Listing B-1: A basic server component

In [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml), you learned that a React component is a JavaScript function that returns a React element; Next.js server components follow that same structure, except that they’re asynchronous functions, so we can use the async/await pattern with fetch. Thus, instead of returning the React element, it returns a promise of it. The code in [Listing B-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-1) should remind you of the WeatherComponent created in the previous chapters, except it doesn’t contain any client-side code.

#### Client Components

By contrast, a client component is a component rendered by the browser rather than by the server. You already know how to write client components, because all React and Next.js components were traditionally client components.

To render these components, the client needs to receive all required scripts and their dependencies. Each component increases the bundle size, decreasing the application’s performance. For that reason, Next.js offers options to optimize the application’s performance, such as server-side rendering (SSR), which pre-renders the pages on the server, then lets the client add interactive elements to the page.

All components in the _app_ directory are server components by default. Client components, however, can reside anywhere (for example, in the _components_ directory we’ve used previously). [Listing B-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-2) shows the WeatherComponent created in [Listing 5-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-4) refactored into a client component that works with the _app_ directory.

```
"use client";

import React, {useState, useEffect} from "react";

export default function ClientComponent (props: WeatherProps): JSX.Element {

    const [count, setCount] = useState(0);
    useEffect(() => {setCount(1);}, []);

    return (
        <h1
          onClick={() => setCount(count + 1)} >
          The weather is {props.weather},
          and the counter shows {count}
        </h1>
    );
}
```

Listing B-2: A basic client component that is similar to the WeatherComponent created in [Listing 5-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-4)

We export the component as the default function with the name ClientComponent. Because we’re using the client-side hooks useEffect and useState as well as the onClick event handler, we need to declare the component as a client component with the "use client" directive at the top of the file. Otherwise, Next.js will throw an error.

### Rendering Components

In [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml), we performed server-side rendering with the getServerSideProps function and used static site generation (SSG) with the getStaticProps function. In the _app_ directory, both functions are obsolete. If we want to optimize an application, we can instead use Next.js’s built-in fetch API, which controls data retrieval and rendering at the component level.

#### Fetching Data

The new asynchronous fetch API extends the native fetch web API and returns a promise. Because server components are just exported functions that return a JSX element, we can declare them as asynchronous functions and then use fetch with the async/await pattern.

This pattern is beneficial because it allows us to fetch data for only the segment that uses the data rather than for an entire page. This lets us leverage React features to automatically display loading states and gracefully catch errors, as discussed in “Exploring the Project Structure” on page 269. If we follow this pattern, a loading state will block the rendering of only a particular server component and its user interface; the rest of the page will be fully functional and interactive.

> NOTE

_Client components shouldn’t be asynchronous functions, because the way JavaScript handles asynchronous calls can easily lead to multiple re-renders and slow down the whole application. Next.js developers have discussed adding a generic use hook that lets us use asynchronous functions in client components by caching the results, but this hook is not yet finalized. If you absolutely need client-side data fetching, I recommend using a specialized library such as SWR, which you can find at_ [https://swr.vercel.app](https://swr.vercel.app/)_._

You might worry that, when each server component loads its own data, you’ll end up with a massive number of requests. How do these numbers impact the overall page performance? Well, Next.js’s fetch comes with multiple optimizations to speed up the application. For example, it automatically caches the response data for GET requests sent from a server component to the same API, reducing the number of requests.

However, POST requests aren’t usually cacheable, as the data they contain might change, so fetch won’t automatically cache them. This is a problem for us because GraphQL typically uses POST requests. Fortunately, React exposes a cache function that memorizes the result of the function it wraps. [Listing B-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-3) shows an example of using cache with a GraphQL API.

```
import {cache} from 'react';

export const getUserFromGraphQL = cache(async (id:string) => {
    return await fetch("/graphql," {method: "POST", body: "query":" "});
});
```

Listing B-3: A simple outline of a cached POST API call

We wrap the API call in the cache function we imported from React and return the API’s response object. Note that the cached arguments can use only primitive values because the cache function doesn’t perform a deep comparison for the arguments.

Another optimization we can implement is to leverage the asynchronous nature of fetch to request data for the server component in a parallel fashion instead of sequentially. Here, the most common pattern is to use Promise.all to start all requests at the same time and block the rendering until all requests have been completed. [Listing B-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-4) shows us the relevant code for this pattern.

```
const userPromiseOne = getUserFromGraphQL ("0001");
const userPromiseTwo = getUserFromGraphQL ("0002");

const [userDataOne, userDataTwo] = await Promise.all([userPromiseOne, userPromiseTwo]);
```

Listing B-4: Two parallel API calls with Promise.all

We set up two requests, both of which return a promise user object. Then we await the result of both promises and call Promise.all with an array of the previously created asynchronous API calls. The Promise.all function resolves as soon as both promises return their data, and then the server component’s code continues.

#### Static Rendering

Static rendering is the default setting for both server and client components. It resembles static site generation, which we used with getStaticProps in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). This rendering option pre-renders both client and server components in the server environment at build time. As a result, requests will always return the same HTML, which remains static and is never re-created.

Each component type is rendered slightly differently. For client components, the server pre-renders the HTML and JSON data; the client then receives the pre-rendered data, including the client-side script, to add interactivity to the HTML. For server components, the browser receives only the rendered payload to hydrate the component. They neither have client-side JavaScript nor use JavaScript for hydration; hence they do not send any JavaScript to the client and, in turn, don’t bloat the bundled scripts.

[Listing B-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-5) shows how to statically render the _utils/fetch-names.ts_ file from [Listing 5-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-8).

```
export default async function ServerComponentUserList(): Promise<JSX.Element> {
    const url = "https://www.usemodernfullstack.dev/api/v1/users";
    let data: responseItemType[] | [] = [];
    let names: responseItemType[] | [];
    try {
        const response = await fetch(url, {cache: "force-cache"});
        data = (await response.json()) as responseItemType[];
    } catch (err) {
        throw new Error("Failed to fetch data");
    }
    names = data.map((item) => {
        return {id: item.id, name: item.name};
    });

    return (
        <ul>
            {names.map((item) => (
                <li key="{item.id}">{item.name}</li>
            ))}
        </ul>
    );
}
```

Listing B-5: A server component that uses static rendering

First we define a server component as an asynchronous function that directly returns a JSX.Element wrapped in a promise.

In [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml), we returned the page’s data and then used the page props to pass it the NextPage function, where we generated the element. Here, after setting the url, we use the asynchronous fetch function to get the data from the remote API. Next.js will cache the results of the API call and the rendered component, and the server will reuse the generated code and never re-create it.

If you use fetch without an explicit cache setting, it will use force-cache as the default to perform static rendering. To switch to incremental static regeneration instead, replace the fetch call from [Listing B-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-5) with the one in [Listing B-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-6).

```
    const response = await fetch(url, {next: {revalidate: 20}});
```

Listing B-6: The modified fetch call for ISR-like rendering

We simply add the revalidate property with a value of 30. The server will then render the component statically but invalidate the current HTML 30 seconds after the first page request and re-render it.

#### Dynamic Rendering

Dynamic rendering replaces Next.js’s traditional server-side rendering (SSR), which we used by exporting the getServerSideProps function from a page route in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). Because Next.js uses static rendering by default, we must actively opt in to using dynamic rendering in one of two ways: by disabling the cache in our fetch requests or by using a dynamic function. In [Listing B-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-7), we disable the cache.

```
export default async function ServerComponentUserList(): Promise<JSX.Element> {
    const url = "https://www.usemodernfullstack.dev/api/v1/users";
    let data: responseItemType[] | [] = [];
    let names: responseItemType[] | [];
    try {
            const response = await fetch(url, {cache: "no-cache"});
        data = (await response.json()) as responseItemType[];
    } catch (err) {
        throw new Error("Failed to fetch data");
    }
    names = data.map((item) => {
        return {id: item.id, name: item.name};
    });

    return (
        <ul>
            {names.map((item) => (
                <li key="{item.id}">{item.name}</li>
            ))}
        </ul>
    );
}
```

Listing B-7: A server component that uses dynamic rendering by disabling the cache

We explicitly set the cache property to no-cache. Now the server will re-fetch the data for the component upon each request.

Instead of disabling the cache, we could use dynamic functions, including the header function or the cookies function in server components and the useSearchParams hook in client components. These functions use dynamic data such as request headers, cookies, and search parameters that are unknown during build time and are part of the request object we pass to the function. The server needs to run these functions for each request because the required data depends on the request.

Keep in mind that dynamic rendering affects the whole route. If one server component in a route opts for dynamic rendering, Next.js will render the whole route dynamically at request time.

### Exploring the Project Structure

Let’s set up a new Next.js application to explore the features we’ve discussed. First, use the npx create-next-app@latest command with the --typescript --use-npm flags to create a sample application. When answering the setup wizard’s questions, choose to use the _app_ directory instead of the _pages_ directory.

> NOTE

_You can also use the online playground at_ [https://codesandbox.io/s/](https://codesandbox.io/s/) _to run the Next.js code examples in this appendix. Search for the official_ Next.js (App router) _template when creating a new code sandbox there._

Now enter the npm run dev command to start the application in development mode. You should see a Next.js welcome screen in your browser at _http://localhost:3000_. Unlike the welcome screen you saw in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml), which encouraged us to edit the _pages/index.tsx_ file, here the welcome screen directs us to the _app/page.tsx_ file.

Take a look at the files and folders the wizard created and compare them with the ones from [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). You should see that the _pages_ and _styles_ directories are not part of the new structure. Instead, the router replaces both with the _app_ directory. Inside it, you should see neither the __app.tsx_ file nor the __document.tsx_ file. Instead, it uses the root layout file _layout.tsx_ to define the HTML wrapper for all rendered pages and the _page.tsx_ file to render the root segment (the home page).

The _pages_ directory uses only one file to create the final content of the page route. By contrast, the _app_ directory uses multiple files to create a page route and add additional behavior.

The _page.tsx_ file generates the user interface and the content for the route, and its parent folder defines the leaf segment. Without a _page.tsx_ file, the URL path won’t be accessible. We can then add other special files to the page’s folder. Next.js will automatically apply them to this URL segment and its children. The most important of these special files are _layout.tsx_, which creates a general user interface; _loading.tsx_, which uses a React suspense boundary to automatically create a “loading” user interface while the page loads; and _error.tsx_, which uses a React error boundary to catch errors and then show the user a custom error interface.

[Figure B-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-1) compares the files and folders for the _components/weather_ page route when using the _pages_ directory and the _app_ directory.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/FigureB-1.jpg)

Figure B-1: Comparing the page route components/weather in the pages and app directory structures

When the _app_ directory is the root folder, its subfolders still correspond to URL segments, but now the folder that contains the _page.tsx_ file defines the URL’s final leaf segment. The optional special files next to it affect only the contents of the _components/weather_ page.

Let’s rebuild the _components/weather_ page route you created in [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1) with the _app_ directory. Create the _components_ folder and _weather_ subfolder inside the _app_ directory and then copy the _custom.d.ts_ file from the previous code exercises into the root folder.

#### Updating the CSS

Begin by opening the existing _app/globals.css_ file and replacing its content with the code from [Listing B-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-8). We’ll need to make some modifications to use special files in our component.

```
html,
body {
    background-color: rgb(230, 230, 230);
    font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen,
        Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
    margin: 0;
    padding: 0;
}

a {
    color: inherit;
    text-decoration: none;
}

* {
    box-sizing: border-box;
}

nav {
    align-items: center;
    background-color: #fff;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.25);
    display: flex;
    height: 3rem;
    justify-content: space-evenly;
    padding: 0 25%;
}

main {
    display: flex;
    justify-content: center;
}

main .content {
    height: 300px;
    padding-top: 1.5rem;
    width: 400px;
}

main .content li {
    height: 1.25rem;
    margin: 0.25rem;
}

main .loading {
    animation: 1s loading linear infinite;
    background: #ddd linear-gradient(110deg, #eeeeee 0%, #f5f5f5 15%, #eeeeee 30%);
    background-size: 200% 100%;
    min-height: 1.25rem;
    width: 90%;
}

@keyframes loading {
    to {
        background-position-x: -200%;
    }
}
main .error {
    background: #ff5656;
    color: #fff;
}

section {
    background: #fff;
    border: 1px dashed #888;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.25);
    margin: 2rem;
    padding: 0.5rem;
    position: relative;
}

section .flag {
    background: #888;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.25);
    color: #fff;
    font-family: monospace;
    left: 0;
    padding: 0.25rem;
    position: absolute;
    top: 0;
    white-space: nowrap;
}
```

Listing B-8: The app/globals.css file with basic styles for our code examples

We create one nav element for the navigation with a main content area below it. Then we add styles for the loading and error states we’ll create later. In addition, we use the section element to outline the boundaries of the files and flag styles to add labels to the sections.

#### Defining a Layout

Layouts are server components that define the user interface for a particular route segment. Next.js renders this layout when this segment is active. Layouts are shared across all pages, so they can be nested into each other, and all layouts for a specific route and its children will be rendered when this route segment is active. [Figure B-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-2) shows the relationship between the URL, the files, and the component hierarchy for the _components/weather_ route.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/FigureB-2.jpg)

Figure B-2: The simplified layout component hierarchy

In this example, each folder contains a _layout.tsx_ file. Next.js will render these in a nested fashion and make the page’s content the final rendered component.

Although we can fetch data in a layout, we can’t share data between a parent layout and its children. Instead, we can leverage the fetch API’s automatic deduplication to reuse data in each child segment or component. When we navigate from one page to another, only the layouts that change are re-rendered. Shared layouts won’t be re-rendered when their child segments change.

The root layout, which returns the skeleton structure with the html and body elements for the page, is required, while all other layouts we create are optional. Let’s create a root layout. First, add a new interface to the end of the _custom.d.ts_ file, which we copied from the previous exercise. We’ll use the LayoutProps interface to type the layout’s properties object:

```
interface LayoutProps {
    children: React.ReactNode;
}
```

Now open the _app/layout.tsx_ file and replace its content with the code from [Listing B-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-9).

```
import "./globals.css";

export const metadata = {
    title: "Appendix C",
    description: "The Example Code",
};

export default function RootLayout(props: LayoutProps): JSX.Element {
    return (
        <html lang="en">
            <body>
                <section>
                    <span className="flag">app/layout(.tsx)</span>
                    {props.children}
                </section>
            </body>
        </html>
    );
}
```

Listing B-9: The file app/layout.tsx defines the root layout.

We import the _global.css_ file that we created earlier and then define the default SEO metadata, the page title, and the page description through the metadata object. This replaces the next/head component we used in the _pages_ directory for all pages in the _app_ directory.

Then we define the RootLayout component, which accepts an object of the LayoutProps type and returns a JSX.Element. We also create the JSX.Element, explicitly adding the html and body elements, then use the section and a span with the CSS class flag to outline the page structure. We add the children property from the LayoutProps object to wrap them with our root HTML structure.

Now let’s add optional layouts to the _app/components_ and _app/components/weather_ folders. Create a _layout.tsx_ file in each and then place the code from [Listing B-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-10) to the _app/components/layout.tsx_ file.

```
export default function ComponentsLayout(props: LayoutProps): JSX.Element {
    return (
        <section>
            <span className="flag">app/components/layout(.tsx)</span>
            <nav>Navigation Placeholder</nav>
            <main>{props.children}</main>
        </section>
    );
}
```

Listing B-10: The file app/components/layout.tsx defines the segment layout.

This segment layout file follows the same basic structure as the root layout. We define a layout component that receives the LayoutProps object with the children property and returns a JSX.Element. Unlike in the root layout, we set only the inner structure, the nav element with the navigation placeholder, and the main content area where we render the child elements from the LayoutProps object, representing this segment’s child content (the leaf).

Lastly, create the leaf’s layout by adding the code from [Listing B-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-11) to the _app/components/weather/layout.tsx_ file.

```
export default function WeatherLayout(props: LayoutProps): JSX.Element {
    return (
        <section>
            <span className="flag">app/components/weather/layout(.tsx)</span>
            {props.children}
        </section>
    );
}
```

Listing B-11: The file app/components/weather/layout.tsx defines the leaf layout.

The leaf’s layout resembles the segment layout from [Listing B-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-10), but it returns a more straightforward HTML structure, as the children property does not contain another layout; instead, it contains the page’s content (in _page.tsx_), and the suspense boundary and error boundary from _loading.tsx_ and _error.tsx_.

#### Adding the Content and Route

To expose the page route, we need to create the _page.tsx_ file; otherwise, if we tried to visit the _components/weather_ page route at _http://localhost:3000/components/weather_, we’d see Next.js’s default _404_ error page. To re-create the page content from [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1), we’ll create two files. One is _component .tsx_, which contains the WeatherComponent, and the other is _page.tsx_, which resembles the NextPage wrapper we used in [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1). Of course, pages could contain additional components located in other folders.

Let’s start by creating the _component.tsx_ file inside the _apps/components/weather_ folder and adding the code from [Listing B-12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-12) into it.

```
"use client";

import {useState, useEffect} from "react";

export default function WeatherComponent(props: WeatherProps): JSX.Element {

    const [count, setCount] = useState(0);

    useEffect(() => {
        setCount(1);
    }, []);
    return (
        <h1 onClick={() => {setCount(count + 1)}} >
            The weather is {props.weather}, and the counter shows {count}
        </h1>
    );
}
```

Listing B-12: The file app/components/weather/component.tsx defines the WeatherComponent.

This code is similar to the code in [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1) for the WeatherComponent constant, except we add the "use client" statement to explicitly set it as a client component and export it as the default function instead of storing it in a constant. The component itself has the same functionality as before: we create a headline that shows the weather string and a counter we can increase by clicking the headline.

Now we add the _page.tsx_ file and the code from [Listing B-13](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-13) to create the page route and expose the route to the user.

```
import WeatherComponent from "./component";

export const metadata = {
    title: "Appendix C - The Weather Component (Weather & Count)",
    description: "The Example Code For The Weather Component (Weather & Count)",
};

export default async function WeatherPage() {
    return (
        <section className="content">
            <span className="flag">app/components/weather/page(.tsx)</span>
            <WeatherComponent weather="sunny" />
        </section>
    );
}
```

Listing B-13: The file app/components/weather/page.tsx defines the page route.

We import the WeatherComponent we just created and then set the SEO metadata on the page level. Then we export the page route as the default async function. When we compare it to [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1), which contains a similar page, we see that we no longer need to export a NextPage; instead, we use a basic function. The _app_ directory simplifies the structure of the code.

Now visit our _components/weather_ page route at _http://localhost:3000/components/weather_ in the browser. You should see a page that looks similar to [Figure B-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-3).

Notice two things here. First, you should recognize the component from [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml), whose counter increases when we click the headline. In addition, the combination of the styles and the span elements we added to each _.tsx_ file visualizes the relations between the files. We see that the nested layout files resemble the simplified component hierarchy from [Figure B-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-3).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/FigureB-3.jpg)

Figure B-3: The components/weather page showing the nested components

#### Catching Errors

As soon as we add an _error.tsx_ file to the folder, Next.js wraps our page’s content with a React error boundary. [Figure B-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-4) shows the simplified component hierarchy of the _components/weather_ route with an added _error.tsx_ file.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/FigureB-4.jpg)

Figure B-4: The simplified layout component hierarchy includes the error boundary.

We see that the _error.tsx_ file automatically creates an error boundary around the page’s content. By doing so, Next.js enables us to catch errors on a page level and gracefully handle those instead of freezing the whole user interface or redirecting the user to a generic error page. Think about it as a try...catch block on a component level. We can now show a tailored error message and display a button that lets the user re-render the page content in a previously working state without reloading the whole application.

The _error.tsx_ file exports a client component that the error boundary uses as the fallback interface. In other words, this component replaces the content when the code throws an error and activates the error boundary. As soon as it is active, it contains the error, ensuring that the layouts above the boundary remain active and maintain their internal state. The error component receives the error object and the reset function as parameters.

Let’s add an error boundary to the _components/weather_ route. Start by adding a new ErrorProps interface to type the component’s properties into the _customs.d.ts_ file:

```
interface ErrorProps {
    error: Error;
    reset: () => void;
}
```

Next, create the _error.tsx_ file next to _page.tsx_ in the _app/components/weather_ directory and add the code from [Listing B-14](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-14).

```
"use client";

export default function WeatherError(props: ErrorProps): JSX.Element {
    return (
        <section className="content error">
            <span className="flag">app/components/weather/error(.tsx)</span>
            <h2>Something went wrong!</h2>
            <blockquote>{props.error?.toString()}</blockquote>
            <button onClick={() => props.reset()}>Try again (re-render)</button>
        </section>
    );
}
```

Listing B-14: The file app/components/weather/error.tsx adds the error boundary and the fallback UI.

Because we know that the error component needs to be a client component, we add the "use client" directive to the top of the file and then define and export the component. We use the ErrorProps interface we just created to type the component’s properties. We then convert the error property to a string and display it to inform the user of the type of error that occurred. Finally, we render a button that calls the reset function that the component received through the properties object. The user can re-render the component into a previous working state by clicking the button.

Now, with the error boundary in place, we’ll modify _component.tsx_ to throw an error if the counter hits 4 or more. Open the file and add the code from [Listing B-15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-15) below the first useEffect hook.

```
    useEffect(() => {
        if (count && count >= 4) {
            throw new Error("Count >= 4! ");
        }
    }, [count]);
```

Listing B-15: The additional useEffect hook for app/components/weather/component.tsx

The additional useEffect hook we add to the component is straightforward; as soon as the count variable changes, we verify the error condition, and as soon as the variable’s value is 4 or more, we throw an error with the message Count >= 4!, which the error boundary catches and gracefully handles by showing the fallback user interface that the _error.tsx_ file exports.

To test this feature, open _http://localhost:3000/components/weather_ in the browser and click the headline until you trigger the error. You should see the error component instead of the weather component, as in [Figure B-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-5).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/FigureB-5.jpg)

Figure B-5: The components/weather page in the error state

The layout markers show us that _error.tsx_ has replaced _page.tsx_. We also see the string Error: Count >=4!, which we passed to the error constructor. As soon as we click the re-render button, _page.tsx_ should replace _error.tsx_, and the screen will look like [Figure B-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-4) previously.

#### Showing an Optional Loading Interface

Now we’ll create the _loading.tsx_ file. With this feature in place, Next.js automatically wraps the page content with a React suspense component, creating a component hierarchy that looks similar to [Figure B-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-6).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/FigureB-6.jpg)

Figure B-6: The simplified layout component hierarchy with the loading interface

The _loading.tsx_ file is a basic server component that returns the pre-rendered loading user interface. When we load a page or navigate between pages, Next.js will instantly display this component while loading the new segment’s content. Once rendering is complete, the runtime will swap the loading state with the new content. In this way, we can easily display meaningful loading states, such as skeletons or custom animations.

Let’s add a basic loading user interface to the weather component route by adding the code from [Listing B-16](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-16) to the _loading.tsx_ file.

```
export default function WeatherLoading(): JSX.Element {
    return (
        <section className="content">
            <span className="flag">app/components/weather/loading(.tsx)</span>
            <h1 className="loading"></h1>
        </section>
    );
}
```

Listing B-16: The file app/components/weather/loading.tsx adds a suspense boundary with the loading user interface.

We define and export the WeatherLoading component, which returns a JSX.Element. In the HTML, we add a headline element similar to the one in _page.tsx_, except this one adds the loading class we created in the _global.css_ file to the headline and shows an animated placeholder.

When we open _http://localhost:3000/components/weather_ in the browser, we should see a loading interface similar to [Figure B-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-7).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/FigureB-7.jpg)

Figure B-7: The components/weather page while loading the page’s content

If you don’t see the animated placeholder, this means Next.js has already cached your segment’s content.

#### Adding a Server Component That Fetches Remote Data

Now that you understand the folders and files in the _app_ directory, let’s add a server component that uses the fetch API to receive the list of users from the remote API _https://www.usemodernfullstack.dev/api/v1/users_ and renders it to the browser. We wrote a version of this code in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml).

Create the folder _app/components/server-component_ and add the special files _component.tsx_, _loading.tsx_, _error.tsx_, _layout.tsx_, and _page.tsx_ to it. Then set up the component’s functionality by adding the code from [Listing B-17](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-17) to the _component.tsx_ file.

```
export default async function ServerComponentUserList(): Promise<JSX.Element|Error> {
    const url = "https://www.usemodernfullstack.dev/api/v1/users";
    let data: responseItemType[] | [] = [];
    let names: responseItemType[] | [];
    try {
        const response = await fetch(url, {cache: "force-cache"});
        data = (await response.json()) as responseItemType[];
    } catch (err) {
        throw new Error("Failed to fetch data");
    }
    names = data.map((item) => {
        return {id: item.id, name: item.name};
    });

    return (
        <ul>
            {names.map((item) => (
                <li id="{item.id}" key="{item.id}">
                    {item.name}
                </li>
            ))}
        </ul>
    );
}
```

Listing B-17: The app/components/server-component/component.tsx file

Here we create a default server component that uses the fetch API to await the API response. To be able to do so, we define it as an asynchronous function that returns a promise of a JSX.Element or an Error. Then we store the API endpoint in a constant and define the variables we’ll need later on. We wrap the API call in a try...catch statement to activate the Error Boundary if the API request fails. We then transform the data in a manner similarly to the way we did in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml) and return a JSX.Element that displays a list of users.

Now we add the loading user interface that Next.js automatically displays while we await the API’s response and the component’s JSX response. Place the code from [Listing B-18](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-18) into the _loading.tsx_ file.

```
export default function ServerComponentLoading(): JSX.Element {
    return (
        <section className="content">
            <span className="flag">
                app/components/server-component/loading(.tsx)
            </span>
            <ul id="load">
                {[...new Array(10)].map((item, i) => (
                    <li className="loading"></li>
                ))}
            </ul>
        </section>
    );
}
```

Listing B-18: The app/components/server-component/loading.tsx file

As before, the loading component is a server component that returns a JSX.Element. This time, the loading skeleton is a list with 10 items resembling the component’s rendered HTML structure. You’ll see that this gives the user a good impression of the expected content and should improve the user’s experience.

Next, we create the error boundary by adding the code from [Listing B-19](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-19) to the _error.tsx_ file.

```
"use client"; // Error components must be Client components

export default function ServerComponentError(props: ErrorProps): JSX.Element {
    return (
        <section className="content">
            <span className="flag">app/components/server-component/error(.tsx)</span>
            <h2>Something went wrong!</h2>
            <code>{props.error?.toString()}</code>
            <button onClick={() => props.reset()}>Try again (re-render)</button>
        </section>
    );
}
```

Listing B-19: The app/components/server-component/error.tsx file

Except for the flag outlining the file structure, the error boundary is similar to the one we used in the weather component.

Then we add the code from [Listing B-20](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-20) to the _layout.tsx_ file.

```
export default function ServerComponentLayout(props: LayoutProps): JSX.Element {
    return (
        <section>
            <span className="flag">app/components/server-component/layout(.tsx)</span>
            {props.children}
        </section>
    );
}
```

Listing B-20: The app/components/server-component/layout.tsx file

Again, the code is similar to the code we used for the weather component. We adjust only the flag outlining the component hierarchy.

Finally, with all the parts in place, we add the code from [Listing B-21](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-21) to the _page.tsx_ file to expose the page route.

```
import ServerComponentUserList from "./component";

export const metadata = {
    title: "Appendix C - Server Side Component (User API)",
    description: "The Example Code For A Server Side Component (User API)",
};

export default async function ServerComponentUserListPage(): JSX.Element {
    return (
        <section className="content">
            <span className="flag">app/components/server-component/page(.tsx)</span>
            {/* @ts-expect-error Async Server Component */}
            <ServerComponentUserList />
        </section>
    );
}
```

Listing B-21: The app/components/server-component/page.tsx file

#### Completing the Application with the Navigation

With two pages in the application, we can now use the next/link component to replace the navigation placeholder in the nav element. This should create a fully functional application prototype that lets us navigate between the pages. Open the _app/components/layout.tsx_ file and replace the code in the file with the code from [Listing B-22](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-22).

```
import Link from "next/link";

export default function ComponentsLayout(props: LayoutProps): JSX.Element {
    return (
        <section>
            <span className="flag">app/components/layout(.tsx)</span>
            <nav>
                <Link href="/components/server-component">
                    User API <br />
                    (Server Component)
                </Link>{" "}
                |
                <Link href="/components/weather">
                    Weather Component <br />
                    (Client Component)
                </Link>
            </nav>
            <main>{props.children}</main>
        </section>
    );
}
```

Listing B-22: The updated app/components/layout.tsx file

We import the next/link component and then add two links to our navigation, one pointing to the user list server component we just created and the other pointing to the weather client component.

Let’s visit the application’s weather component page at _http://localhost:3000/components/weather_. You should see an application that looks similar to the screenshot in [Figure B-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#figB-8).

As soon as you navigate between the pages, you should see the loading user interface. With the outlines we’ve added to all the files, we easily keep track of which files Next.js uses to render the current page.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/FigureB-8.jpg)

Figure B-8: The components/weather page with the functional navigation

#### Replacing API Routes with Route Handlers

If you look at the folder structure Next.js created for you, you should see that the _app_ directory contains an _api_ subfolder. You probably already guessed that we use this folder to define APIs. But unlike the API routes discussed in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml), which were regular functions, the _app_ directory uses _route handlers_, which are functions that require a particular naming convention.

These route handlers belong in special files named _route.ts_ that usually reside in a subfolder of the _app/api_ folder. They are asynchronous functions that receive a Request object and an optional context object as parameters. We name each function after the HTTP method it should react to. For example, the code in [Listing B-23](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-23) shows how to define route handlers that handle GET and POST requests for the same API.

```
import {NextRequest, NextResponse} from 'next/server';
export async function GET(request: NextRequest): Promise<NextResponse> {
    return NextResponse.json({});
}

export async function POST(request: NextRequest): Promise<NextResponse {
    return NextResponse.json({});
}
```

Listing B-23: A skeleton structure of a route.ts file defining route handlers

To create the route handlers, we import the NextRequest and NextResponse objects from Next.js’s server package. Next.js adds additional convenience methods for cookie handling, redirects, and rewrites. You can read more about them in the official documentation at [_https://nextjs.org_](https://nextjs.org/).

We then define two asynchronous functions, both of which receive a NextRequest and return a promise of a NextResponse. The function names correspond to the HTTP method they should respond to. Next.js supports using the GET, POST, PUT, PATCH, DELETE, HEAD, and OPTIONS methods as function names.

When defining page routes and route handlers, remember that these files take over all requests for a given segment. This means that a _route.ts_ file cannot be in the same folder as a _page.tsx_ file. Also, route handlers are the only way to define APIs if you use the _app_ directory: you can’t have an _api_ folder in both the _pages_ directory and _app_ directory.

Next.js has statically and dynamically evaluated route handlers. Statically evaluated route handlers will be cached and reused for every request, whereas dynamically evaluated route handlers must request the data upon each request. By default, the runtime statically evaluates all GET route handlers that don’t use a dynamic function or the Response object. As soon as we use a different HTTP method, the dynamic cookies or headers function, or the Response object, the route handler becomes dynamically evaluated. The same applies to APIs with dynamic segments, which receive the dynamic parameters through the context object.

Let’s re-create the API route _api/v1/weather/[zipcode].ts_ from [Listing 5-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Lis5-1) as a route handler that we can use in the _app_ directory. Add the code from [Listing B-24](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-B.xhtml#LisB-24) to a _route.ts_ file in the folder structure _app/api/v1/weather/[zipcode]_.

```
import {NextResponse, NextRequest} from "next/server";

interface ReqContext {
    params: {
        zipcode: number;
    }
}

export async function GET(req: NextRequest, context: ReqContext): Promise<NextResponse> {
    return NextResponse.json(
        {
            zipcode: context.params.zipcode,
            weather: "sunny",
            temp: 35,
        },
        {status: 200}
    );
}
```

Listing B-24: The route handler in app/api/v1/weather/[zipcode]/route.ts

Notice that we’ve used the square brackets pattern on the folder structure to access the dynamic segment through the function’s second parameter context object.

Within the file, we import the Next.js Response and NextRequest objects from the server package and then define the interfaces for the route handler. On the RequestContext interface, we add the zipcode property to params, representing the API’s dynamic segment. Finally, we export the asynchronous GET function, the API route handler that reacts to all GET requests for this API endpoint. It receives the request object and the request context as parameters and uses the NextResponse’s json function to return the response data. We access the URL parameter zipcode through the context object’s params object and then add it to the response data. We set additional response options through the json function’s second parameter, explicitly setting the HTTP status code to _200_.

Now try querying the API with curl:

```
$ curl -i \
    -X GET \
    -H "Accept: application/json" \
    http://localhost:3000/api/v1/weather/12345
```

You should receive this JSON response:

```
HTTP/1.1 200 OK
content-type: application/json
--snip--
{"zipcode":"12345","weather":"sunny","temp":35}
```

This is the same response received in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml), where we accessed the API through the browser.

# Appendix C - Common Matchers

In Jest, _matchers_ let us check a specific condition, such as whether two values are equal or whether an HTML element exists in the current DOM. Jest comes with a set of built-in matchers. In addition, the _JEST-DOM_ package from the testing library provides DOM-specific matchers.

### Built-in Matchers

This section covers the most common built-in Jest matchers. You can find a complete list in the official JEST documentation at [_https://jestjs.io/docs/expect_](https://jestjs.io/docs/expect).

toBe  This matcher is the simplest and by far the most common. It’s a simple equality check to determine whether two values are identical. It behaves similarly to the strict equality (===) operator, as it considers type differences. Unlike the strict equality operator, however, it considers +0 and -0 to be different.

```
test('toBe',  () => {
    expect(1 + 1).toBe(2);
})
```

toEqual  We use toEqual to perform a deep-equality check between objects and arrays, comparing all of their properties or items. This matcher ignores undefined values and items. Furthermore, it does not check the object’s types (for example, whether they are instances or children of the same class or parent object). If you require such a check, consider using the toStrictEqual matcher instead.

```
test('toEqual', () => {
    expect([undefined, 1]).toEqual([1]);
})
```

toStrictEqual  The toStrictEqual matcher performs a structure and type comparison for objects and arrays; passing this test requires that the objects are of the same type. In addition, the matcher considers undefined values and undefined array items.

```
test('toStrictEqual', () => {
    expect([undefined, 1]).toStrictEqual([undefined, 1]);
})
```

toBeCloseTo  For floating-point numbers, we use toBeCloseTo instead of toBe. This is because JavaScript’s internal calculations of floating-point numbers are flawed, and this matcher considers those rounding errors.

```
test('toBeCloseTo', () => {
    expect(0.1 + 0.2).toBeCloseTo(0.3);
})
```

toBeGreaterThan/toBeGreaterThanOrEqual  For numeric values, we use these matchers to verify that the result is greater than or equal to a value, similar to the > and >= operators.

```
test('toBeGreaterThan', () => {
    expect(1 + 1).toBeGreaterThan(1);
})
```

toBeLessThan/toBeLessThanOrEqual  These are the opposite of the GreaterThan... matchers for numeric values, similar to the < and <= operators.

```
test('toBeLessThan', () => {
    expect(1 + 1).toBeLessThan(3);
})
```

toBeTruthy/toBeFalsy  These matchers check if a value exists, regardless of its value. They consider the six JavaScript values 0, ' ', null, undefined, NaN, and false to be falsy and everything else to be truthy.

```
test('toBeTruthy', () => {
    expect(1 + 1).toBeTruthy();
})
```

toMatch  This matcher accepts a string or a regular expression, then checks if a value contains the given string or if the regular expression returns the given result.

```
test('toMatch, () => {
    expect('apples and oranges').toMatch('apples');
})
```

toContain  The toContain matcher is similar to toMatch, but it accepts either an array or a string and checks these for a given string value. When used on an array, the matcher verifies that the array contains the given string.

```
test('toMatch, () => {
    expect(['apples', 'oranges']).toContain('apples');
})
```

toThrow  This matcher verifies that a function throws an error. The function being checked requires a wrapping function or the assertion will fail. We can pass it a string or a regular expression, similar to the toMatch function.

```
function functionThatThrows() {
    throw new Error();
}

test('toThrow', () => {
    expect(() => functionThatThrows()).toThrow();
})
```

### The JEST-DOM Matchers

The _JEST-DOM_ package provides matchers to work directly with the DOM, allowing us to easily write tests that run assertions on the DOM, such as checking for an element’s presence, HTML contents, CSS classes, or attributes.

Say we want to check that our logo element has the class name center. Instead of manually checking for the presence of an element and then checking its class name attribute with toMatch, we can use the toHaveClass matcher, as shown in [Listing C-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-C.xhtml#LisC-1).

```
<img data-testid="image" class="center full" alt="The Logo" src="logo.svg" />

test('toHaveClass', () => {
    const element = getByTestId('image');
    expect(element).toHaveClass('center');
})
```

Listing C-1: The basic syntax for testing with the DOM

First we add the data attribute testid to our image element. Then, in the test, we get the element using this ID and store the reference in a constant. Finally, we use the toHaveClass matcher on the element’s reference to see if the element’s class names contain the class center.

Let’s take a look at the most common DOM-related matchers.

getByTestId  This matcher lets us directly access a DOM element and store a reference to it, which we then use with custom matchers to assert things about this element.

```
<img data-testid="image" class="center full" alt="The Logo" src="logo.svg" />

test('toHaveClass', () => {
    const element = getByTestId('image');
--snip--
})
```

toBeInTheDocument  This matcher verifies that an element was added to the document tree. This matcher works only on elements that are currently part of the DOM and ignores detached elements.

```
<img data-testid="image" class="center full" alt="The Logo" src="logo.svg" />

test('toHaveClass', () => {
    const element = getByTestId('image');
    expect(element).toBeInTheDocument();
})
```

toContainElement  This matcher tests our assumptions about the element’s child elements, letting us verify, for example, whether an element is a descendant of the first.

```
<div data-testid="parent">
    <img data-testid="image" class="center full" alt="The Logo" src="logo.svg" />
</div>

test('toHaveClass', () => {
    const parent = getByTestId('parent');
    const element = getByTestId('image');
    expect(parent).toContainElement(element);
})
```

toHaveAttribute  This matcher lets us run assertions on the element’s attributes, such as an image’s alt attribute and the checked, disabled, or error state of form elements.

```
<img data-testid="image" class="center full" alt="The Logo" src="logo.svg" />

test('toHaveClass', () => {
    const element = getByTestId('image');
    expect(element).toHaveAttribute('alt', 'The Logo');
})
```

toHaveClass  The toHaveClass matcher is a specific variant of the toHave Attribute matcher. It lets us explicitly assert that an element has a particular class name, allowing us to write clean tests.

```
<img data-testid="image" class="center full" alt="The Logo" src="logo.svg" />

test('toHaveClass', () => {
    const element = getByTestId('image');
    expect(element).toHaveClass('center');
})
```