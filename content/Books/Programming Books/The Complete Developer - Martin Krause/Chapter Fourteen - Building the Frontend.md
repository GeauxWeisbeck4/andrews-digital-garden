---
id: 01J9QDT136W6JYFJE69FP42AT2
modified: 2024-10-08T21:20:41-04:00
---
In this chapter, you’ll build the frontend using React components and Next.js pages, discussed in [Chapters 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml) and [5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). By the end, you’ll have an initial version of the app to which you can add OAuth authentication.

### Overview of the User Interface

Our application will consist of three Next.js pages. The _start page_ will show the list of locations retrieved from the database. Each item in the list will link to its respective _location detail page_, whose URL we’ll construct using the location’s ID, like this: _/location/:location_id_. The third page is the user’s _wish list page_. It resembles the start page and follows the same dynamic URL pattern as the location detail page, except it supplies the user’s ID instead of the location’s. This page shows only the locations already added to the wish list.

We must also consider what rendering strategy to use for each page. Because the content of the start page never changes, we’ll use static site generation (SSG) to render the HTML on build time. Because the detail page and wish list page will change based on the user’s actions, we’ll use static site rendering (SSR) to regenerate them upon every request.

Lastly, all three pages should have headers containing the logo and a link to the start page. When we add the OAuth data in the next chapter, we’ll show the user’s name, a link to the user’s wish list, and the sign-in/sign-out button in the header as well.

To achieve this, we need to create the following React components:

- The locations list component, which will use the locations list item component to render the list of locations on the start page. Later, we’ll use these same components to implement the list of locations on a user’s wish list page.
- The overall layout component, header component, and logo component, which define the global layout of each page.
- The authentication element component, which lets users log in or out in the header.
- A universal button component we’ll use for different tasks.

Let’s begin with the components necessary for the start page.

### The Start Page

We’ll begin by crafting the smallest parts of the user interface and then use these to build the more complex components and pages. On the start page, we need the layout component, the locations list component, and the locations list item component, which is the smallest building block, so we’ll start there.

Create the _components_ folder in the application’s root directory, next to the _middleware_ folder. This is where we’ll place all our React components, in their own folders.

#### The List Item

The locations list item component represents a single item in a list of locations. Create the _locations-list-item_ folder and add two files, _index.tsx_ and _index.module.css_, following the pattern we discussed in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). Then add the code in [Listing 14-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-1) to _index.module.css_. We’ll use this CSS to style the component.

```
.root {
    background-color: #fff;
    border-radius: 5px;
    color: #1d1f21;
    cursor: pointer;
    list-style: none;
    margin: 0.5rem 0;
    padding: 0.5rem;
    transition: background-color 0.25s ease-in, color 0.25s ease-in;
    will-change: background-color, color;
}

.root:hover {
    background-color: rgba(0, 118, 255, 0.9);
    color: #fff;
}

.root h2 {
    margin: 0;
    padding: 0;
}

.root small {
    font-weight: 300;
    padding: 0 1rem;
}
```

Listing 14-1: The components/locations-list-item/index.module.css file

The CSS module uses dark letters on a white background. In addition, it adds a simple hover effect, causing the background to turn blue and the font color white when a user hovers over it. We remove the list marker and set the margin and padding accordingly.

Now add the code from [Listing 14-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-2) to the _index.tsx_ file.

```
import Link from "next/link";
import styles from "./index.module.css";
import {LocationType} from "mongoose/locations/schema";

interface PropsInterface {
    location: LocationType;
}

const LocationsListItem = (props: PropsInterface): JSX.Element => {
    const location = props.location;
    return (
        <>
            {location && (
                <li className={styles.root}>
                    <Link href={`/location/${location.location_id}`}>
                        <h2>
                            {location.name}
                            <small className={styles.details}>
                                {location.cuisine} in {location.borough}
                            </small>
                        </h2>
                    </Link>
                </li>
            )}
        </>
    );
};

export default LocationsListItem;
```

Listing 14-2: The components/locations-list-item/index.tsx file

You should be familiar with this file’s structure from [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). First we import the _next/link_ component, which we need to create a link to the detail page, the styles we just added, and the LocationType from the Mongoose schema.

We then define the PropsInterface, a private interface used for the component’s properties object. The component has the usual props parameter whose structure defines the PropsInterface and returns a JSX element. These props hold the data in the location property, which we pass to the component through its location attribute. Finally, we define the LocationsListItem component and store it in a constant that we export at the end of the file.

In the component itself, we have a list item that contains a Next.js Link element linking to the location’s detail page. These links use a dynamic URL pattern that incorporates the respective location’s ID, so we create the link target to match _/location/:location_id_. In addition, we render the location’s name, cuisine, and borough values to the component. Keep in mind that until we create the page for the route _/location/:location_id_, clicking those links will result in a _404_ error page.

#### The Locations List

Using the list item component, we’ll build the locations list. This component will loop through an array of locations and display them on the start page and wish list page. Create the _components/locations-list_ folder and then add the files _index.tsx_ and _index.module.css_ to them. Copy the code in [Listing 14-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-3) to the _index.module.css_ file.

```
.root {
    margin: 0;
    padding: 0;
}
```

Listing 14-3: The components/locations-list/index.module.css file

The styles for the locations list component are simple; we remove the margin and padding from the component’s root element. We create the component itself in [Listing 14-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-4), which you should copy to _index.tsx_.

```
import LocationsListItem from "components/locations-list-item";
import styles from "./index.module.css";
import {LocationType} from "mongoose/locations/schema";

interface PropsInterface {
    locations: LocationType[];
}

const LocationsList = (props: PropsInterface): JSX.Element => {
    return (
        <ul className={styles.root}>
            {props.locations.map((location) => {
                return (
                    <LocationsListItem
                        location={location}
                        key={location.location_id}
                    />
                );
            })}
        </ul>
    );
};

export default LocationsList;
```

Listing 14-4: The components/locations-list/index.tsx file

We import the LocationsListItem we just implemented, along with the module’s styles and the LocationType from Mongoose’s schema. We then define the component’s PropsInterface to describe the component’s props object. In the LocationsList component, we use the array map function to iterate over the location objects, rendering a LocationsListItem component for each array item and using the location attribute to pass the location details to the components. React requires that each item rendered in a loop have a unique ID. We use the location IDs for this purpose.

We can now create the start page and pass all available locations to this component. Later, we’ll use the same component for the wish list page to return the locations on the user’s wish list.

#### The Page

At this point, we have the components we need for the start page, which is a basic Next.js page. Save this page’s global styles in _styles/globals.css_ and its code in _pages/index.tsx_. [Listing 14-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-5) contains the styles. Delete all other files from the _styles_ directory. Those are default styles we don’t need for the application.

```
html,
body {
    font-family: -apple-system, Segoe UI, Roboto, sans-serif;
    margin: 0;
    padding: 0;
}

* {
    box-sizing: border-box;
}

h1 {
    font-size: 3rem;
}

a {
    color: inherit;
    text-decoration: none;
}
```

Listing 14-5: The styles/globals.css file

We set a few global styles, such as the default font family, and change the box model to the more intuitive border-box for all elements. By using a border-box instead of a content-box, an element adopts whatever width we assign to it with the width property. Otherwise, the width property would define only the width of the content, and we’d need to add the border and padding to calculate the actual dimensions of the element on the page. We set the font families to the defaults for each operating system to ensure readability.

Now replace the existing content of the _pages/index.tsx_ file with the code in [Listing 14-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-6).

```
import Head from "next/head";
import type {GetStaticProps, InferGetStaticPropsType, NextPage} from "next";

import LocationsList from "components/locations-list";
import dbConnect from "middleware/db-connect";
import {findAllLocations} from "mongoose/locations/services";
import {LocationType} from "mongoose/locations/schema";

❶ const Home: NextPage = (
    props: InferGetStaticPropsType<typeof getStaticProps>
) => {

  ❷ const locations: LocationType[] = JSON.parse(props.data?.locations);
    let title = `The Food Finder - Home`;

    return (
        <div>
            <Head>
                <title>{title}</title>
                <meta name="description" content="The Food Finder - Home" />
            </Head>

            <h1>Welcome to the Food Finder!</h1>
            <LocationsList locations={locations} />
        </div>
    );
};

❸ export const getStaticProps: GetStaticProps = async () => {
    let locations: LocationType[] | [];
    try {
        await dbConnect();
      ❹ locations = await findAllLocations();
    } catch (err: any) {
        return {notFound: true};
    }
  ❺ return {
        props: {
            data: {locations: JSON.stringify(locations)},
        },
    };
};

export default Home;
```

Listing 14-6: The pages/index.tsx file

We implemented the Next.js page, similar to the structure discussed in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). First we import all dependencies; then we create the NextPage and store it in a constant that we export at the end of the file ❶.

The Next.js page’s props object, the page properties, contains the data we return from the getStaticProps function ❺, discussed in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). In this asynchronous function, we connect to the database ❸. As soon as the connection is ready, we call the service method to retrieve all locations ❹ and then pass them as a JSON string to the NextPage in the data.locations property of the props object. Next.js calls the getStaticProps function on build time and generates the HTML for this page only once. We can use this rendering method because the list of available locations never changes; it is static.

Then we retrieve the locations from the page properties ❷, parse the JSON string back to an array, and store the page title in a variable. We type the locations constant explicitly because TSC cannot easily infer the type. Then we construct the JSX. In the first step, we use the next/head component to set the page-specific metadata. Then we call the LocationList component we previously implemented with the locations array in the locations attribute. By doing so, the LocationList component renders all locations as an overview list.

As soon as you save the file, you should see, in the Docker command line, that Next.js recompiles the application. Open the web application at _http://localhost:3000_ in your browser to see a list of locations similar to [Figure 14-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#fig14-1).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure14-1.jpg)

Figure 14-1: The start page showing all available locations

Now we’ll move on to styling the frontend and adding basic global components, such as the application’s header with the Food Finder logo.

### The Global Layout Components

Now it’s time to create the three global components. These include the overall layout component, which we’ll use to format the start and wish list page content, a _sticky_ header (which is always visible, “sticking” to the browser’s upper edge), and the Food Finder logo to go in the header. Again, we’ll start with the smallest units and then use those as building blocks for the overall components.

#### The Logo

The smallest component, the logo, is nothing more than a next/image component wrapped in a next/link element; when users click the logo image, they’ll be redirected to the start page. Add a _header_ folder to the _components_ folder, then add a _logo_ folder to the _header_ folder and create two files there, _index.tsx_ and _index.module.css_, into which you should paste the code in [Listing 14-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-7).

```
.root {
    display: inline-block;
    height: 35px;
    position: relative;
    width: 119px;
}

@media (min-width: 600px) {
    .root {
        height: 50px;
        width: 169px;
    }
}
```

Listing 14-7: The components/header/logo/index.module.css file

These basic styles for the component’s root element set the image’s dimensions. We use a _mobile-first design pattern_ by initially defining the styles to use on smaller screens and then, using a standard CSS media query, modifying them for screens bigger than 600px. We’ll use a bigger image on bigger screens.

Now let’s create the logo component. Create an _assets_ subfolder in the Next.js _public_ folder and place the _logo.svg_ file extracted from _assets.zip_ into it. Then add the code in [Listing 14-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-8) to the logo’s _index.tsx_ file.

```
import Image from "next/image";
import Link from "next/link";
import logo from "/public/assets/logo.svg";
import styles from "./index.module.css";

const Logo = (): JSX.Element => {
    return (
        <Link href="/" passHref className={styles.root}>
            <Image
                src={logo}
                alt="Logo: Food Finder"
                sizes="100vw"
                fill
                priority
            />
        </Link>
    );
};

export default Logo;
```

Listing 14-8: The components/header/logo/index.tsx file

As usual, we import the dependencies and then create an exported constant that contains the JSX code. We don’t pass any data to it through attributes or child elements; hence, we don’t need to define the component’s props object here.

We use a basic next/image inside a next/link element to link back to the start page and set the next/image’s attributes to fill the available space defined in the CSS file.

#### The Header

The header component will wrap the logo component we just created. Create the _index.tsx_ file and _index.module.css_ file in the _header_ folder, then add the code in [Listing 14-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-9) to the CSS file.

```
.root {
    background: white;
    border-bottom: 1px solid #eaeaea;
    padding: 1rem 0;
    position: sticky;
    top: 0;
    width: 100%;
    z-index: 1;
}
```

Listing 14-9: The components/header/index.module.css file

We use the CSS definitions position: sticky and top: 0 to stick the header to the upper edge of the browser. Now the header will automatically stay there even when users scroll down the page; the page’s content should scroll below the header because we set the header’s z-index, placing the header in front of the other elements. You can think of the z-index as determining which floor of a building an element is on.

[Listing 14-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-10) shows the code for the header component. Copy it into the component’s _index.tsx_ file.

```
import styles from "./index.module.css";
import Logo from "components/header/logo";

const Header = (): JSX.Element => {
    return (
        <header className={styles.root}>
            <div className="layout-grid">
                <Logo />
            </div>
        </header>
    );
};

export default Header;
```

Listing 14-10: The components/header/index.tsx file

We define a basic component that displays the logo. Then we wrap the imported Logo component in an element with a global layout-grid class, which we’ll define in the next section.

#### The Layout

Currently, we have one Next.js page (the start page) and a header component. The easiest way to add the header to the page would be to import it into the Next.js page and place it directly into the JSX. However, we’ll add two more pages to the app, the wish list page and the location detail page, so we want to avoid importing the header three times.

To streamline the overall app design, Next.js provides the concept of a _layout_, which is really just another component, and we can use it to add the header component as a sibling element to a page’s content. Let’s create a new layout component. First, to create this component’s CSS file, add _layout.css_ to the _styles_ folder and paste the code in [Listing 14-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-11) into it.

```
.layout-grid {
    align-items: center;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    margin: 0 auto;
    max-width: 800px;
    padding: 0 1rem;
    width: 100%;
}

@media (min-width: 600px) {
    .layout-grid {
        flex-direction: row;
        padding: 0 2rem;
    }
}
```

Listing 14-11: The styles/layout.css file

We use the mobile-first pattern once again to define a basic grid wrapper, setting the global padding and maximum width for the content area. We set the wrapper’s left and right margins to auto, which centers the container, because the margins take up all available space between the fixed-width wrapper and the window’s edges.

We use flexbox to set the direction of the wrapper’s direct child elements to column, displaying them one on top of the next. Because the logo and all other upcoming header elements are direct children of an element with the layout-grid class, they are affected by the flexbox layout. In contrast, the location items aren’t direct siblings. Hence, they won’t change their direction when switching between screen sizes.

Then we use a media query to adjust the styles for screens whose width is greater than 600px. Here we increase the padding and change the layout order of the direct child elements. Instead of using column, we set it to row, and immediately we display the elements next to one another.

Because this is a global styles file and not a CSS module, Next.js won’t automatically scope the class names. Hence, we prefix them with layout- and don’t import the styles into the component before using them.

Now create a _layout_ folder inside the _components_ folder and add the _index.tsx_ file to it with the component code in [Listing 14-12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-12).

```
import Header from "components/header";

interface PropsInterface {
    children: React.ReactNode;
}

const Layout = (props: PropsInterface): JSX.Element => {
    return (
        <>
            <Header />
            <main className="layout-grid">
                {props.children}
            </main>
        </>
    );
};
export default Layout;
```

Listing 14-12: The components/layout/index.tsx file

In the layout component, we define a private interface and the component with the usual structure. Inside the component, we add the Header and the main element that uses the global layout styles and acts as a wrapper for the children elements we’ll pass to this component in the __app.tsx_ file.

Open the __app.tsx_ file and modify it as shown in [Listing 14-13](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-13).

```
import "../styles/globals.css";
import "../styles/layout.css";
import type {AppProps} from "next/app";
import Layout from "components/layout";

export default function App({Component, pageProps}: AppProps) {
    return (
        <Layout>
            <Component {...pageProps} />
        </Layout>
    );
}
```

Listing 14-13: The pages/_app.tsx file

First we add _layout.css_ as a global style. As for the layout, we have only one layout component we’ll use for all pages, and we import it here. Then we wrap our application, the pages, with the layout and pass the current page in the component’s children property.

Now all our Next.js pages will follow the same structure: they’ll have the Header component next to the main element containing the page’s content. One advantage of following this pattern is that the component’s state will be preserved across page changes and React component re-rendering.

Once Next.js has recompiled the application, try reloading the application at _http://localhost:3000_ in your browser. It should look like [Figure 14-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#fig14-2).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure14-2.jpg)

Figure 14-2: The start page with the header and layout component

You should now see the header, and the new layout component centers the content.

### The Location Details Page

Our application now has a start page with a header and a list of all available locations. The list items link to their particular location’s detail page because we added a next/link component to them, but those pages don’t exist yet. If you click one of the links, you’ll get a _404_ error. To display the location details pages, we first need to implement the component that lists a particular location’s details and then create a new Next.js page.

#### The Component

Let’s start with the details component. Create the _location-details_ folder in the _components_ directory and add the _index.module.css_ and _index.tsx_ files to it. Then add the code from [Listing 14-14](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-14) to the CSS module.

```
.root {
    margin: 0 0 2rem 0;
    padding: 0;
}
.root li {
    list-style: none;
    margin: 0 0 0.5rem 0;
}
```

Listing 14-14: The components/locations-details/index.module.css file

The styles for the component are basic. We remove the default margin and padding, as well as the list styles, and then add a custom margin at the end of each list item and root element.

To implement the location details component, add the code from [Listing 14-15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-15) to the _index.tsx_ file in the _components/locations-details_ folder.

```
import {LocationType} from "mongoose/locations/schema";
import styles from "./index.module.css";

interface PropsInterface {
    location: LocationType;
}

const LocationDetail = (props: PropsInterface): JSX.Element => {
    let location = props.location;
    return (
        <div>
            {location && (
                <ul className={styles.root}>
                    <li>
                        <b>Address: </b>
                        {location.address}
                    </li>
                    <li>
                        <b>Zipcode: </b>
                        {location.zipcode}
                    </li>
                    <li>
                        <b>Borough: </b>
                        {location.borough}
                    </li>
                    <li>
                        <b>Cuisine: </b>
                        {location.cuisine}
                    </li>
                    <li>
                        <b>Grade: </b>
                        {location.grade}
                    </li>
                </ul>
            )}
        </div>
    );
};
export default LocationDetail;
```

Listing 14-15: The components/locations-details/index.tsx file

The locations detail component is structurally similar to the locations list item. Both take an object containing the location’s data and add a specific set of properties to the returned JSX element. The main difference is in the JSX structure we create. Otherwise, we follow the known pattern, importing the required styles and type, defining the component’s props interface using the LocationType, and then returning a JSX element with the location details.

#### The Page

We mentioned in “Overview of the User Interface” on page 215 that a location’s detail page should be available at the dynamic URL _location/:location"ePub-I">location_ folder in the _pages_ directory and add the _[locationId].tsx_ file containing the code in [Listing 14-16](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter14.xhtml#Lis14-16).

```
import Head from "next/head";
import type {
    GetServerSideProps,
    GetServerSidePropsContext,
    InferGetServerSidePropsType,
    PreviewData,
    NextPage,
} from "next";
import LocationDetail from "components/location-details";
import dbConnect from "middleware/db-connect";
import {findLocationsById} from "mongoose/locations/services";
import {LocationType} from "mongoose/locations/schema";
import {ParsedUrlQuery} from "querystring";

const Location: NextPage = (
    props: InferGetServerSidePropsType<typeof getServerSideProps>
) => {
    let location: LocationType = JSON.parse(props.data?.location);
  ❶ let title = `The Food Finder - Details for ${location?.name}`;
    return (
        <div>
            <Head>
                <title>{title}</title>
                <meta
                    name="description"
                    content={`The Food Finder. 
                        Details for ${location?.name}`}
                />
            </Head>
            <h1>{location?.name}</h1>
          ❷ <LocationDetail location={location} />
        </div>
    );
};

❸ export const getServerSideProps: GetServerSideProps = async (
    context: GetServerSidePropsContext<ParsedUrlQuery, PreviewData>
) => {
     let locations: LocationType[] | [];
  ❹ let {locationId} = context.query;
     try {
        await dbConnect();
        locations = await findLocationsById([locationId as string]);
      ❺ if (!locations.length) {
            throw new Error(`Locations ${locationId} not found`);
        }
    } catch (err: any) {
        return {
            notFound: true,
        };
    }
    return {
      ❻ props: {data: {location: JSON.stringify(locations.pop())}},
    };
};
```

Listing 14-16: The pages/location/[locationId].tsx file

The start page and location detail page look fairly similar. The only visual difference is the page’s title, which we construct with the location’s name ❶, and instead of the LocationsList component, we use the LocationDetail component with a single location object ❷.

From a functional perspective, however, the pages are not similar. Unlike the start page, which uses SSG, the location detail page uses SSR with getServerSideProp ❸. This is because as soon as we add the wish list functionality and implement the Add To/Remove button, the page’s content should change along with a user’s action. Hence, we need to regenerate the HTML on each request. We discussed the differences between SSR and SSG in depth in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml).

We use the page’s context and its query property to get the location ID from the dynamic URL ❹. Then we use the ID to get the matching location from the database. As before, we use the service directly instead of calling the publicly exposed API, as Next.js runs both get...Prop functions on the server side and can directly access the services in our application’s middleware.

We also implement two exit scenarios. First, if there is no result, we throw an error to step into the catch block ❺, and by doing so, redirect the user to the _404 Not Found_ error page. Otherwise, we store the first location from the results in the location property ❻ and pass it to the Next.js page function we export in the last line.

### Summary

We’ve successfully built the frontend for the Food Finder application. At this point, you’ve implemented a full-stack web application that reads data from a MongoDB database and renders the results as React user interface components in Next.js. Next, we’ll add an OAuth authentication flow with GitHub so that users can log in with their GitHub account and store a personalized wish list.