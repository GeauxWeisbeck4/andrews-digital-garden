---
id: 01J9QDTZWC1PG9Q00NYHEX7H7P
modified: 2024-10-08T21:21:12-04:00
---
In this chapter, you’ll add OAuth authentication to the Food Finder app, giving users the opportunity to log in with their GitHub accounts. You’ll also implement the wish list page to which authenticated users can add and remove locations, as well as the button component needed to accomplish this. Lastly, you’ll learn how to protect your GraphQL mutations from unauthenticated users.

### Adding OAuth with next-auth

Developers usually use third-party libraries or SDKs to implement OAuth. For the Food Finder application, we’ll use the _next-auth_ package from Auth.js, which comes with an extensive set of preconfigured templates that allow us to connect to an OAuth service easily. These templates are called _providers_, and we’ll use one of them: the GitHub provider, which adds a Log In with GitHub button to our app. For a refresher on the OAuth authentication process, return to [Chapter 9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter9.xhtml).

#### Creating a GitHub OAuth App

First we need to create an OAuth application using GitHub. This should give us the client ID and client secret that the Food Finder application needs to connect to GitHub. If you don’t already have a GitHub account, create one now at [_https://github.com_](https://github.com/), then log in. Navigate to [_https://github.com/settings/developers_](https://github.com/settings/developers) and create a new OAuth app in the OAuth Apps section. Enter the Food Finder app’s details in the resulting form, which should look similar to [Figure 15-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#fig15-1).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure15-1.jpg)

Figure 15-1: The GitHub user interface for adding a new OAuth application

Enter **Food Finder** as the name, set the home page URL to **http://localhost:3000/**, and set the authorization callback URL to **http://localhost:3000/api/auth/callback/github**. After registering the application, GitHub should show us the client ID and let us generate a client secret.

#### Adding the Client Credentials

Now copy these credentials as GITHUB_CLIENT_ID and GITHUB_CLIENT_SECRET to the _env.local_ file in the application’s code root folder. This file looks like this:

```
MONGO_URI=mongodb://backend:27017/foodfinder
GITHUB_CLIENT_ID=ADD_YOUR_CLIENT_ID_HERE
GITHUB_CLIENT_SECRET=ADD_YOUR_CLIENT_SECRET_HERE
```

Fill in the placeholders with your credentials.

#### Installing next-auth

To add Auth.js’s OAuth SDK for _next-auth_ to the Food Finder app and configure it to connect to the provider, run the following:

```
$ docker exec -it foodfinder_application npm install next-auth
```

By default, this SDK uses encrypted JWTs to store and attach session information to API requests. The library automatically handles the encryption and decryption as long as we provide it with a secret. To add such a secret, open the _env.local_ file and add the following line to the end:

```
NEXTAUTH_SECRET=78f6cc4bf633b1102f4ca4d72602c60f
```

Use any secret you’d like. The string used here was randomly generated with _OpenSSL_ at _[https://www.usemodernfullstack.dev/api/v1/generate-secret](https://www.usemodernfullstack.dev/api/v1/generate-secret)_, and you should use a fresh one for each application.

#### Creating the Authentication Callback

Now we’ll develop the _api/auth_ route for the authorization callback URL we supplied to GitHub when registering the OAuth application. Create the _auth_ folder in the _pages/api_ directory containing the file _[...nextauth].ts_. The ... in the filename tells the Next.js router that this is a catch all, meaning it handles all API calls to endpoints below _/auth_; for example, _auth/signin_ or _auth/callback/github_. Add the code from [Listing 15-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-1) to the file.

```
import GithubProvider from "next-auth/providers/github";
import {NextApiRequest, NextApiResponse} from "next";
import NextAuth from "next-auth";
import {createHash} from "crypto";

const createUserId = (base: string): string => {
    return createHash("sha256").update(base).digest("hex");
};

export default async function auth(req: NextApiRequest, res: NextApiResponse) {
    return await NextAuth(req, res, {
        providers: [
            GithubProvider({
                clientId: process.env.GITHUB_CLIENT_ID || " ",
                clientSecret: process.env.GITHUB_CLIENT_SECRET || " ",
            }),

        ],
        callbacks: {
            async jwt({token}) {
                if (token?.email && !token.fdlst_private_userId) {
                    token.fdlst_private_userId = createUserId(token.email);
                }
                return token;
            },
            async session({session}) {
                if (
                    session?.user?.email &&
                    !session?.user.fdlst_private_userId
                ) {
                    session.user.fdlst_private_userId = createUserId(
                        session?.user?.email
                    );
                }
                return session;
            },
        },
    });
}
```

Listing 15-1: The pages/api/auth/[...nextauth].ts file

We import our dependencies, including the built-in GithubProvider and the default crypto module. Then we create a simple createUserId function, which takes a string as an argument and calls the crypto module’s createHash function to return the hashed user ID from this string.

Next, we create and export the default asynchronous auth function. To do so, we initialize the NextAuth module and add the GithubProvider to the providers array. We configure it to use the clientId and the clientSecret we stored in the environment variables.

Since we want to keep our application as simple as possible, we’ll keep it stateless; hence, we use the jwt and session callbacks, which _next-auth_ uses every time it creates a new session or JWT internally. In the callback, we calculate the hashed user ID from the user’s email with our createId function (if it’s not already available in the current token or session object). Finally, we store it in a private claim.

We’ve just created a new property, fdlst_private_userId, on the user object in the _next-auth_ session. As expected, TSC warns us that this property doesn’t exist on the Session type. We need to augment the type’s interface by adjusting the _customs.d.ts_ file in our application’s root directory to match [Listing 15-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-2).

```
import mongoose from "mongoose";
import {DefaultSession} from "next-auth";

declare global {
    var mongoose: mongoose;
}

declare module "next-auth" {
    interface Session {
        user: {
            fdlst_private_userId: string;
        } & DefaultSession["user"];
    }
}
```

Listing 15-2: The updated customs.d.ts file with the augmented Session interface

In the updated code, we import the _next-auth_ package’s DefaultSession, which defines the default session object, and then create and redeclare the Session interface’s user object with the new fdlst_private_userId property. Because TypeScript overwrites the existing user object, we explicitly add it from the DefaultSession object. In other words, we add our new fdlst_private_userId property to the Session interface.

#### Sharing the Session Across Pages and Components

With the callback URL set up, we need to ensure that a user’s session is shared among all Next.js pages and React components. We can use the useContext hook discussed in [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml), which _next-auth_ provides for us. In the _pages/_app .tsx_ file, wrap the application in a SessionProvider, as shown in [Listing 15-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-3).

```
import "../styles/globals.css";
import "../styles/layout.css";
import type {AppProps} from "next/app";
import Layout from "components/layout";
import {SessionProvider} from "next-auth/react";

export default function App({
    Component, pageProps: {session, ...pageProps}}: AppProps) {
    return (
        <SessionProvider session={session}>
            <Layout>
                <Component {...pageProps} />
            </Layout>
        </SessionProvider>
    );
}
```

Listing 15-3: The modified pages/_app.tsx file

We import the SessionProvider from the _next-auth_ package and enhance the pageProps with the session object. We store the current session in the provider’s session attribute, making it available throughout the Next.js application.

Before we can access the session in the frontend and middleware, we need to add the auth-element with the Sign In button, which will allow users to log in.

### The Generic Button Component

It’s time to implement the generic button component we mentioned earlier. Technically, this component will be a generic div element that we’ll style to look like a button, with a few variations. It will serve as the Sign In/Sign Out button in the auth-element and the Add To/Remove From button in the location detail component. Create a new folder, _button_, in the _components_ folder, adding an _index.module.css_ file with the code in [Listing 15-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-4), as well as an _index.tsx_ file.

```
.root {
    align-items: center;
    border-radius: 5px;
    color: #1d1f21;
    cursor: pointer;
    display: inline-flex;
    font-weight: 500;
    height: 35px;
    letter-spacing: 0;
    margin: 0;
    overflow: hidden;
    place-content: flex-start;
    position: relative;
    white-space: nowrap;
}

.root > a,
.root > span {
    padding: 0 1rem;
    white-space: nowrap;
}

.root {
    transition: border-color 0.25s ease-in, background-color 0.25s ease-in,
        color 0.25s ease-in;
    will-change: border-color, background-color, color;
}

.root.default,
.root.default:link,
.root.default:visited {
    background-color: transparent;
    border: 1px solid transparent;
    color: #1d1f21;
}

.root.default:hover,
.root.default:active {
    background-color: transparent;
    border: 1px solid #dbd8e3;
    color: #1d1f21;
}

.root.blue,
.root.blue:link,
.root.blue:visited {
    background-color: rgba(0, 118, 255, 0.9);
    border: 1px solid rgba(0, 118, 255, 0.9);
    color: #fff;
    text-decoration: none;
}

.root.blue:hover,
.root.blue:active {
    background-color: transparent;
    border: 1px solid #1d1f21;
    color: #1d1f21;
    text-decoration: none;
}

.root.outline,
.root.outline:link,
.root.outline:visited {
    background-color: transparent;
    border: 1px solid #dbd8e3;
    color: #1d1f21;
    text-decoration: none;
}

.root.outline:hover,
.root.outline:active {
    background-color: transparent;
    border: 1px solid rgba(0, 118, 255, 0.9);
    color: rgba(0, 118, 255, 0.9);
    text-decoration: none;
}

.root.disabled,
.root.disabled:link,
.root.disabled:visited {
    background-color: transparent;
    border: 1px solid #dbd8e3;
    color: #dbd8e3;
    text-decoration: none;
}

.root.disabled:hover,
.root.disabled:active {
    background-color: transparent;
    border: 1px solid #dbd8e3;
    color: #dbd8e3;
    text-decoration: none;
}
```

Listing 15-4: The components/button/index.module.css file

We add styles for each of the button variations we’d like to create. All are 35 pixels tall and have rounded corners. We define a default style, a variation with a blue background and white color, and an outlined version whose background is white. In addition, we define styles to use for deactivated buttons.

With the styles in place, we can write code for the component. Copy the contents of [Listing 15-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-5) into the component’s _index.tsx_ file.

```
import React from "react";
import styles from "./index.module.css";

interface PropsInterface {
    disabled?: boolean;
    children?: React.ReactNode;
    variant?: "blue" | "outline";
    clickHandler?: () => any;
}

const Button = (props: PropsInterface): JSX.Element => {
    const {children, variant, disabled, clickHandler} = props;

    const renderContent = (children: React.ReactNode) => {
        if (disabled) {
            return (
                <span className={styles.span}>
                    {children}
                </span>
            );
        } else {
            return (
                <span className={styles.span} onClick={clickHandler}>
                    {children}
                </span>
            );
        }
    };

    return (
        <div
            className={[
                styles.root,
                disabled ? styles.disabled : " ",
                styles[variant || "default"],
            ].join(" ")}
        >
            {renderContent(children)}
        </div>
    );
};

export default Button;
```

Listing 15-5: The components/button/index.tsx file

After importing the dependencies, we define the interface for the component’s prop argument. We also define the Button component as a function that returns a JSX element and then use object-destructuring syntax to split the props object into constants representing the object’s key-value pairs. We define the internal renderContent function with one argument, children, typed as a ReactNode and rendered wrapped in a span element. Depending on the state of the disabled property, we also add the click handler from the props object. The component itself returns a div that we styled to look like a button.

### The AuthElement Component

Although we’ve added the _next-auth_ package to the project, created the OAuth API route, and configured our OAuth provider, we still can’t access session information, as there is no Sign In button. Let’s create this AuthElement component and then add it to the header. This component uses our default button component, and as soon as the user is logged in, it displays their full name, as well as a link to their wish list.

Create the folder _auth-element_ in the _components/header_ directory and then add the _index.module.css_ file with the code in [Listing 15-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-6).

```
.root {
    align-items: center;
    display: flex;
    justify-content: space-between;
    margin: 0;
    padding: 1rem 0;
    width: auto;
}

.root > * {
    margin: 0 0 0 2rem;
}

.name {
    margin: 1rem 0 0 0;
}

@media (min-width: 600px) {
    .name {
        margin: 0 0 0 1rem;
    }
}
```

Listing 15-6: The components/header/auth-element/index.module.css file

We define a set of basic styles for the component, using a flexbox and margins to align them vertically, and change their layout for smaller screens.

To write the component itself, add an _index.tsx_ file to the _auth-element_ folder and enter the code from [Listing 15-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-7) into it.

```
import Link from "next/link";
import {signIn, signOut, useSession} from "next-auth/react";
import Button from "components/button";
import styles from "./index.module.css";

const AuthElement = (): JSX.Element => {
    const {data: session, status} = useSession();

    return (
        <>
            {status === "authenticated" (
                <span className={styles.name}>
                    Hi <b>{session?.user?.name}</b>
                </span>
            )}

            <nav className={styles.root}>
                {status === "authenticated" && (
                    <>
                        <Button variant="outline">
                            <Link
href={`/list/${session?.user.fdlst_private_userId}`}
                            >
                                Your wish list
                            </Link>
                        </Button>

                        <Button variant="blue" clickHandler={() => signOut()}>
                            Sign out
                        </Button>
                    </>
                )}
                {status == "unauthenticated" && (
                    <>
                        <Button variant="blue" clickHandler={() => signIn()}>
                            Sign in
                        </Button>
                    </>
                )}
            </nav>
        </>
    );
};
export default AuthElement;
```

Listing 15-7: The components/header/auth-element/index.tsx file

The most notable imports are the signIn and signOut functions and the useSession hook from _next-auth_. The latter enables us to access session information easily, whereas the two functions trigger the sign-in flow or terminate the session.

We then define the AuthElement component and retrieve the session data and the session status from the useSession hook. We need both of these to construct the JSX element we return from the component. On the client side, we can access the session information directly via the useSession hook. On the server side, though, we’ll need to access it through the JWT, because the session information is part of the API request’s HTTP cookies.

When the session’s status is authenticated, we render the user’s name from the session data and add the Your Wish List and Sign Out buttons to the navigation’s nav element. Otherwise, we add the Sign In button to start the OAuth flow. For all of those, we use the generic button component and the signIn and signOut functions we imported from the _next-auth_ module, both of which handle the OAuth flow automatically.

We use the next/link element to link to the user’s wish list. (This is another Next.js page we’ll implement in a moment.) The wish list is available at the dynamic route _/list/:userId_, which uses the user ID we created by hashing the user’s email address and storing it in fdlst_private_userId.

### Adding the AuthElement Component to the Header

Now we have to add the new component to the header. Open the _index.tsx_ file in the _components/header_ directory and adjust it so that it matches [Listing 15-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-8).

```
import styles from "./index.module.css";
import Logo from "components/header/logo";
import AuthElement from "components/header/auth-element";
const Header = (): JSX.Element => {
    return (
        <header className={styles.root}>
            <div className="layout-grid">
                <Logo />
                <AuthElement />
            </div>
        </header>
    );
};

export default Header;
```

Listing 15-8: The modified components/header/index.tsx file

The update is simple; we import the AuthElement component and add it next to the Logo inside the header.

Test the OAuth workflow to see our session management in practice. When you open _http://localhost:3000_, the Sign In button should be in the header, as in [Figure 15-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#fig15-2).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure15-2.jpg)

Figure 15-2: The header in a logged-out state with the Sign In button

Let’s log in using OAuth. Click the **Sign In** button, and _next-auth_ should redirect you to the login screen, where you can select to sign in with the configured OAuth providers to use ([Figure 15-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#fig15-3)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure15-3.jpg)

Figure 15-3: OAuth requires us to choose a provider.

Click the button to log in. OAuth should redirect you to the application, where the AuthElement renders your name and new buttons based on the session information. The screen should look similar to [Figure 15-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#fig15-4).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure15-4.jpg)

Figure 15-4: The header in the logged-in state with the session information

The header element has changed according to the session’s state. We display the current user’s name received from the OAuth provider, the link to their public wish list, and the Sign Out button.

### The Wish List Next.js Page

The wish list button in the header should link to a wish list page at the dynamic URL _list/:userId_. This regular Next.js page should display all locations whose on_wishlist property contains the user ID specified in the dynamic URL. It will look quite similar to the start page, and we can build it out of existing components.

To create the page’s route, create the _list_ folder with the _[userId].tsx_ file in the _pages_ directory. Then add the code from [Listing 15-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-9) to this _.tsx_ file.

```
import type {
    GetServerSideProps,
    GetServerSidePropsContext,
    NextPage,
    PreviewData,
    InferGetServerSidePropsType,
} from "next";
import Head from "next/head";
import {ParsedUrlQuery} from "querystring";

import dbConnect from "middleware/db-connect";
import {onUserWishlist} from "mongoose/locations/services";
import {LocationType} from "mongoose/locations/schema";
import LocationsList from "components/locations-list";

import {useSession} from "next-auth/react";

const List: NextPage = (
    props: InferGetServerSidePropsType<typeof getServerSideProps>
) => {
    const locations: LocationType[] = JSON.parse(props.data?.locations);
    const userId: string | undefined = props.data?.userId;
    const {data: session} = useSession();
    let title = `The Food Finder- A personal wish list`;
    let isCurrentUsers =
        userId && session?.user.fdlst_private_userId === userId;
    return (
        <div>
            <Head>
                <title>{title}</title>
                content={`The Food Finder. A personal wish list.`}
            </Head>
            <h1>
                {isCurrentUsers ? " Your " : " A "}
                wish list!
            </h1>
            {isCurrentUsers && locations?.length === 0 && (
                <>
                    <h2>Your list is currently empty! :(</h2>
                    <p>Start adding locations to your wish list!</p>
                </>
            )}
            <LocationsList locations={locations} />
        </div>
    );
};

export const getServerSideProps: GetServerSideProps = async (
    context: GetServerSidePropsContext<ParsedUrlQuery, PreviewData>
) => {
    let {userId} = context.query;
    let locations: LocationType[] | [] = [];
    try {
        await dbConnect();
        locations = await onUserWishlist(userId as string);
    } catch (err: any) {}
    return {
        // the props will be received by the page component
        props: {
            data: {locations: JSON.stringify(locations), userId: userId},
        },
    };
};
export default List;
```

Listing 15-9: The pages/list/[userId].tsx file

Although we want the wish list page to look similar to the start page, we use SSR, with getServerSideProps, as we did for the location detail page. The wish list page is highly dynamic; hence, we need to regenerate the HTML on each request.

Another approach would be to use client-side rendering, then request the user’s locations through the GraphQL API in a useEffect hook. However, this would cause the user to see a loading screen each time they opened the wish list page. We can avoid this inferior user experience altogether with SSR.

In the server-side part of the page’s code, we first extract the URL parameter, userId, from the context’s query object. We use the user’s ID and the onUsersWishlist service to get all locations for the user’s wish list. If there is an error, we simply continue instead of redirecting to the _404_ error page, rendering an empty list.

We then pass the locations array and the user’s ID to the Next.js page, where we extract the locations as usual, as well as the userId. We compare the user ID from the URL with the user ID in the current session. If they match, we know that the currently logged-in user has visited their own wish list and adjust the user interface accordingly.

### Adding the Button to the Location Detail Component

We can now visit the wish list page, but it will always be empty. We haven’t yet provided users with a way to add items to it. To change this, we’ll place a button in the location details component that lets users add or remove a particular location. We’ll use the generic button component and session information. Open the _index.ts_ file in the _components/location-details.tsx_ directory and modify the code to match [Listing 15-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-10).

```
import {LocationType} from "mongoose/locations/schema";
import styles from "./index.module.css";

import {useSession} from "next-auth/react";
import {useEffect, useState} from "react";
import Button from "components/button";

interface PropsInterface {
    location: LocationType;
}

interface WishlistInterface {
    locationId: string;
    userId: string;
}

const LocationDetail = (props: PropsInterface): JSX.Element => {
    let location: LocationType = props.location;

    const {data: session} = useSession();
    const [onWishlist, setOnWishlist] = useState<Boolean>(false);
    const [loading, setLoading] = useState<Boolean>(false);

    useEffect(() => {
        let userId = session?.user.fdlst_private_userId;
        setOnWishlist(
            userId && location.on_wishlist.includes(userId) ? true : false
        );
    }, [session]);

    const wishlistAction = (props: WishlistInterface) => {

        const {locationId, userId} = props;

        if (loading) {return false;}
        setLoading(true);

        let action = !onWishlist ? "addWishlist" : "removeWishlist";

        fetch("/api/graphql", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                query: `mutation wishlist {
                    ${action}(
                        location_id: "${locationId}",
                        user_id: "${userId}"
                    ) {
                        on_wishlist
                    }
                }`,
            }),
        })
        .then((result) => {
            if (result.status === 200) {
                setOnWishlist(action === "addWishlist" ? true : false);
            }
        })
        .finally(() => {
            setLoading(false);
        });
    };

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

            {session?.user.fdlst_private_userId && (
                <Button
                    variant={!onWishlist ? "outline" : "blue"}
                    disabled={loading ? true : false}
                    clickHandler={() =>
                        wishlistAction({
                            locationId: session?.user.fdlst_private_userId,
                            userId: session?.user?.userId,
                        })
                    }
                >
                    {onWishlist && <>Remove from your Wishlist</>}
                    {!onWishlist && <>Add to your Wishlist</>}
                </Button>
            )}

        </div>
    );
};
export default LocationDetail;
```

Listing 15-10: The modified components/location-details/index.tsx file

First we import useSession from next-auth, useEffect and useState from React, and the generic Button component. Then we define WishlistInterface, the interface for the wishlistAction function we’ll implement in a bit.

Inside the component, we get the session from the useSession hook, then create the onWishlist and loading state variables with useState as Boolean values. We use the first state variable to specify whether a location is currently on the user’s wish list, then update the user interface accordingly. We calculate the initial state in the useEffect hook based on the location’s on_wishlist property. As soon as we’ve successfully added or removed the location to or from the wish list, we update the state variable and the button’s text.

We implement the wishlistAction function to update the on_wishlist property. First we deconstruct the argument object and then check the loading state to see if there is currently a running request. If so, we exit the function. Otherwise, we set the loading state to true to block the user interface, calculate the action for the GraphQL mutations, and use it to call the wishlist mutation. After successfully modifying the document in the database, we update the onWishlist state and unblock the user interface.

We check the current session to see if the user is logged in. If so, we render the Button component and set the disabled and class name attributes based on the loading state, as well as an on-click event. With each click of the button, we call the wishlistAction function with the current location ID and user ID as arguments. Finally, we set the button’s text based on the onWishlist state, either adding the current location to the wish list or removing it.

Try adding and removing a few locations to the wish list before moving on. Check that the button’s text changes accordingly and that a list of locations similar to the one on the start page appears on the wish list page.

### Securing the GraphQL Mutations

There is one more thing we have to do to wrap up the application: secure the GraphQL API. While the queries should be publicly available, the mutations should be accessible only to logged-in users, who should be able to add or remove only their own user ID for the on_wishlist property.

But if you test the API with the curl command, you’ll see that, currently, everyone can access the API. Note that you must enter the values supplied to the -d flag on a single line, or the server might return an error:

```
$ curl -v \
    -X POST \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -d '{"query":"mutation wishlist {removeWishlist(location_id: \"12340\",
 user_id: \"exampleid\") {on_wishlist}}"}' \
    http://localhost:3000/api/graphql

< HTTP/1.1 200 OK
<
{"data":{"removeWishlist":{"on_wishlist":[]}}}
```

As a test, we send a simple mutation to remove the location with the ID 12340 from a nonexistent user’s wish list. (The mutation won’t work, which is fine; we just want to verify whether the API is accessible to the public.) The command receives a _200_ response and the expected JSON, proving that the mutations are public.

Let’s implement an authGuard to protect our mutations. A _guard_ is a pattern that checks a condition and then throws an error if it isn’t met, and an _auth guard_ protects a route or an API from unauthorized access.

We begin by creating the file _auth-guards.ts_ in the _middleware_ folder and adding the code in [Listing 15-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-11).

```
import {GraphQLError} from "graphql/error";
import {JWT} from "next-auth/jwt";

interface paramInterface {
    user_id: string;
    location_id: string;
}
interface contextInterface {
    token: JWT;
}

export const authGuard = (
    param: paramInterface,
    context: contextInterface
): boolean | Error => {

 ❶ if (!context || !context.token || !context.token.fdlst_private_userId) {
        return new GraphQLError("User is not authenticated", {
            extensions: {
                http: {status: 500},
                code: "UNAUTHENTICATED",
            },
        });
    }

 ❷ if (context?.token?.fdlst_private_userId !== param.user_id) {
        return new GraphQLError("User is not authorized", {
            extensions: {
                http: {status: 500},
                code: "UNAUTHORIZED",
            },
        });
    }
    return true;
};
```

Listing 15-11: The middleware/auth-guards.ts file

We also import JWT from next-auth and the GraphQLError constructor from graphql. We’ll use the latter to create the error objects returned to the user if authentication fails. Next, we define our interfaces for the authGuard function’s arguments and export the function itself.

We’ll call the auth guard from the mutation resolver with two parameters: an object with the user ID and the location ID, for which we defined the paramInterface, and the context object with the token, the contextInterface. The auth guard returns either a Boolean indicating that authentication succeeded or an error. In the authGuard function, we verify that every access to our mutation has a token with a private claim ❶ and that the user ID in the private claim matches the user ID we pass to the mutation ❷. In other words, we verify that a logged-in user has made the API request and that they’re modifying their own wish list.

If the checks fail, we create an error with a message and code. In addition, we set the HTTP status code to _500_. Remember that unlike REST APIs, which rely on an extensive list of precise HTTP status codes to communicate with the caller, a GraphQL API usually uses either _200_ or _500_ as the status code for errors. Broadly speaking, we send a _500_ status code when GraphQL can’t execute the query at all and _200_ when the query can be executed. In both cases, the GraphQL API should include precise information about what error occurred.

Now we must pass the user’s OAuth token to the resolvers, which will then pass it to the auth guard. To do so, we’ll use the context function we implemented in the startServerAndCreateNextHandler function, found in the _pages/api/graphql.ts_ file. Open the file and adjust it to match the code in [Listing 15-12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-12).

```
import {ApolloServer, BaseContext} from "@apollo/server";
import {startServerAndCreateNextHandler} from "@as-integrations/next";

import {resolvers} from "graphql/resolvers";
import {typeDefs} from "graphql/schema";
import dbConnect from "middleware/db-connect";

import {NextApiHandler, NextApiRequest, NextApiResponse} from "next";

import {getToken} from "next-auth/jwt";

const server = new ApolloServer<BaseContext>({
    resolvers,
    typeDefs,
});

const handler = startServerAndCreateNextHandler(server, {
    context: async (req: NextApiRequest) => {
        const token = await getToken({req});
        return {token};
    },
});

const allowCors =
    (fn: NextApiHandler) =>
    async (req: NextApiRequest, res: NextApiResponse) => {
        res.setHeader("Allow", "POST");
        res.setHeader("Access-Control-Allow-Origin", "*");
        res.setHeader("Access-Control-Allow-Methods", "POST");
        res.setHeader("Access-Control-Allow-Headers", "*");
        res.setHeader("Access-Control-Allow-Credentials", "true");

        if (req.method === "OPTIONS") {
            res.status(200).end();
        }
        return await fn(req, res);
    };

const connectDB =
    (fn: NextApiHandler) =>
    async (req: NextApiRequest, res: NextApiResponse) => {
        await dbConnect();
        return await fn(req, res);
    };

export default connectDB(allowCors(handler));
```

Listing 15-12: The modified pages/api/graphql.ts file with the JWT token

Unlike on the client side, where we can access the session information directly via the useSession hook, here we need to access it through the JWT on the server side. This is because the session information is part of the API request’s HTTP cookies on the server instead of the SessionProvider’s shared session state, and we need to extract it from the request. To do so, we import the getToken function from the _next-auth_ jwt module. Then we pass the request object we receive from the context function to call getToken and await the decoded JWT. Next, we return the token from the context function so that we can access the token in the resolver functions.

Finally, let’s use the token to add the authGuard to our resolvers to protect them from unauthenticated and unauthorized access. Open the _graphql/locations/mutations.ts_ file and update it with the code from [Listing 15-13](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-13).

```
import {updateWishlist} from "mongoose/locations/services";
import {authGuard} from "middleware/auth-guard";
import {JWT} from "next-auth/jwt";

interface UpdateWishlistInterface {
    user_id: string;
    location_id: string;
}

interface contextInterface {
    token: JWT;
}

export const locationMutations = {
    removeWishlist: async (
        _: any,
        param: UpdateWishlistInterface,
        context: contextInterface
    ) => {

        const guard = authGuard(param, context);
        if (guard !== true) {return guard;}

        return await updateWishlist(param.location_id, param.user_id, "remove");
    },

    addWishlist: async (
        _: any,
        param: UpdateWishlistInterface,
        context: contextInterface
    ) => {

        const guard = authGuard(param, context);
        if (guard !== true) {return guard;}

        return await updateWishlist(param.location_id, param.user_id, "add");
    },
};
```

Listing 15-13: The graphql/locations/mutations.ts file with the added authGuard

We define a new interface for the context and update the context parameter to contain the JWT. Next, we add the authGuard function to our mutations and follow the guard pattern by returning the error immediately instead of proceeding with the code.

To test the authGuard functionality, run curl again. The command line output should look similar to [Listing 15-14](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml#Lis15-14).

```
$ curl -v \
    -X POST \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -d '{"query":"mutation wishlist {removeWishlist(location_id: \"12340\",
  user_id: \"exampleid\") {on_wishlist}}"}' \
  http://localhost:3000/api/graphql

< HTTP/1.1 500 Internal Server Error
<
{
    "errors":[
        {
            "message":"User is not authenticated",
            "locations": [{"line":1,"column":20}],
            "path": ["removeWishlist"],
            "extensions": {"code":"UNAUTHENTICATED","data":null}
        }
    ]
}
```

Listing 15-14: The curl command to test our API

Unlike the previous curl call, the GraphQL API now responds with HTTP/1.1 500 Internal Server Error and an extensive error message, which we defined when we created the GraphQLError in the _auth-guards.ts_ file.

### Summary

We’ve successfully added an OAuth authentication flow to the Food Finder application. Now the user can log in with their GitHub account. Once logged in, they can maintain their personal public wish list. In addition, we’ve protected the GraphQL mutations, meaning they are no longer available to anyone; instead, only logged-in users can access them. In the final chapter, we’ll add automated tests to evaluate the application using Jest.