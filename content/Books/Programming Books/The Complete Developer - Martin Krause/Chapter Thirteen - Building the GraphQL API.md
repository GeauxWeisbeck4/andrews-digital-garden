---
id: 01J9QDRXXWF0N86YTK3CEKFB77
modified: 2024-10-08T21:20:06-04:00
---
In this chapter, you’ll add a GraphQL API to the middleware by defining its schema, as well as resolvers for each query and mutation. These resolvers will complement the Mongoose services created in [Chapter 12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml). The queries will be public; however, we’ll expose our mutations as protected APIs by adding an authorization layer via OAuth.

Unlike in the GraphQL API of [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml), we’ll follow a pattern of modularization to implement these schemas and resolvers. Instead of writing everything in one big file, we’ll split the elements into separate files. Like using modules in modern JavaScript, this approach has the benefit of breaking down the code into smaller logical units, each with a clear focus. These units enhance the code’s readability and maintainability.

### Setting Up

We’ll create the API’s single-entry point _/api/graphql_ with the Apollo server, which integrates into Next.js with the _@as-integrations/next_ package. Start by installing the packages necessary for the GraphQL setup from the npm registry:

```
$ docker exec -it foodfinder-application npm install @apollo/server graphql graphql-tag
@as-integrations/next \
```

After the installation is complete, create the folder _graphql/locations_ next to the _middleware_ folder in the application’s root.

### The Schemas

The first step to writing the schemas is to define the query and mutation typedefs, as well as any custom types we use for the schema. To do so, we’ll split the schema into three files, _custom.gql.ts_, _queries.gql.ts_, and _mutations.gql.ts_, in the _graphql/locations_ folder. Then we’ll use an ordinary template literal to merge them into the final schema definition.

#### The Custom Types and Directives

Add the code from [Listing 13-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter13.xhtml#Lis13-1) to the _custom.gql.ts_ file to define the schema for the GraphQL queries.

```
export default `
    directive @cacheControl(maxAge: Int) on FIELD_DEFINITION | OBJECT
    type Location @cacheControl(maxAge: 86400) {
        address: String
        street: String
        zipcode: String
        borough: String
        cuisine: String
        grade: String
        name: String
        on_wishlist: [String] @cacheControl(maxAge: 60)
        location_id: String
    }
`;
```

Listing 13-1: The graphql/locations/custom.gql.ts file

The GraphQL API will return location objects from the Mongoose schema. Therefore, we must define a custom type representing these location objects. Create a custom Location type. To instruct the server to cache the retrieved values, set an @cacheControl directive for the whole custom type and a shorter one for the on_wishlist property because we expect this particular property to change frequently.

#### The Query Schema

Now add the code from [Listing 13-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter13.xhtml#Lis13-2) to the _queries.gql.ts_ file to define the schema for the queries.

```
export default `
    allLocations: [Location]!
    locationsById(location_ids: [String]!): [Location]!
    onUserWishlist(user_id: String!): [Location]!
`;
```

Listing 13-2: The graphql/locations/queries.gql.ts file

We define a template literal with three GraphQL queries, all of which are entry points to the services we implemented for the Mongoose locations model in [Chapter 12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml). The names and parameters are similar to those in the services, and the queries follow the GraphQL syntax you learned about in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml).

#### The Mutation Schema

To define the mutation schema, paste the code from [Listing 13-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter13.xhtml#Lis13-3) into the _mutations.gql.ts_ file.

```
export default `
    addWishlist(location_id: String!, user_id: String!): Location!
    removeWishlist(location_id: String!, user_id: String!): Location!
`;
```

Listing 13-3: The graphql/locations/mutations.gql.ts file

We create two mutations as template literals using GraphQL syntax: one for adding an item to the user’s wish list and one for removing it. Both will use the updateWishlist function we implemented on the location services, so they require the location_id and the user_id as parameters.

### Merging the Typedefs into the Final Schema

We’ve split the location schema into two files, one for the queries and one for the mutations, and placed their custom types in a third file; however, to initiate the Apollo server, we’ll need a unified schema. Luckily, the typedefs are nothing more than template literals, and if we use template literal placeholders, the parser can interpolate these into a complete string. To accomplish this, create a new file, _schema.ts_, in the _graphql_ folder and add the code from [Listing 13-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter13.xhtml#Lis13-4).

```
import gql from "graphql-tag";

import locationTypeDefsCustom from "graphql/locations/custom.gql";
import locationTypeDefsQueries from "graphql/locations/queries.gql";
import locationTypeDefsMutations from "graphql/locations/mutations.gql";

export const typeDefs = gql`

    ${locationTypeDefsCustom}

    type Query {
        ${locationTypeDefsQueries}
    }

    type Mutation {
        ${locationTypeDefsMutations}
    }

`;
```

Listing 13-4: The graphql/schema.ts file

We import the gql tag from the _graphql-tag_ package. Even though doing so is optional when working with the Apollo server, we keep the gql tag in front of our tagged template to ensure compatibility with all other GraphQL implementations. This also produces proper syntax highlighting in the IDE, which statically analyzes type definitions as GraphQL tags.

Next, we import the dependencies and schema fragments we’ll use to implement the unified schema. Finally, we create a tagged template literal with the gql function, using template literal placeholders to merge the schema fragments into the schema skeleton. We add the custom Location type and then merge the queries’ typedefs into the Query object and the Mutations into the mutation object and export the schema const as typedefs.

### The GraphQL Resolvers

Now that we have the schema, we’ll turn to the resolvers. We’ll use a similar development pattern, writing the queries and mutations in separate files, then merging them into the single file we need for the Apollo server. Start by creating the _queries.ts_ and _mutations.ts_ files in the _graphql/locations_ folder and then add the code from [Listing 13-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter13.xhtml#Lis13-5) to _queries.ts_.

```
import {
    findAllLocations,
    findLocationsById,
    onUserWishlist,
} from "mongoose/locations/services";

export const locationQueries = {
    allLocations: async (_: any) => {
        return await findAllLocations();
    },
    locationsById: async (_: any, param: {location_ids: string[]}) => {
        return await findLocationsById(param.location_ids);
    },
    onUserWishlist: async (_: any, param: {user_id: string}) => {
        return await onUserWishlist(param.user_id);
    },
};
```

Listing 13-5: The graphql/locations/queries.ts file

We import our services from the Mongoose folder and then create and export the location query object. The structure of each query follows the structure discussed in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml). We make one query for each service, and their parameters match those in the services.

For our mutations, add the code from [Listing 13-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter13.xhtml#Lis13-6) to the _mutations.ts_ file.

```
import {updateWishlist} from "mongoose/locations/services";

interface UpdateWishlistInterface {
    user_id: string;
    location_id: string;
}

export const locationMutations = {
    removeWishlist: async (
        _: any,
        param: UpdateWishlistInterface,
        context: {}
    ) => {
        return await updateWishlist(param.location_id, param.user_id,
            "remove"
        );
    },
    addWishlist: async (_: any, param: UpdateWishlistInterface, context: {}) => {
        return await updateWishlist(param.location_id, param.user_id, "add");
    },
};
```

Listing 13-6: The graphql/locations/mutations.ts file

Here we import only the updateWishlist function from our services. This is because we defined it as the single entry point for updating our documents, and we opted to use the third parameter, with the value add or remove, to distinguish between the two actions the mutation should perform. We also create the UpdateWishlistInterface, which we don’t export. Instead, we’ll use it inside this file to avoid repeating code when we define the interface for the functions’ param argument.

As mutations, we create two functions at the locationMutations object, one for adding an item from a user’s wish list and one for removing it. Both use the updateWishlist service and supply the value parameter corresponding to the action the user would like to take. The two mutations, removeWishlist and addWishlist, also take a third object called context. For now, it’s an empty object, but in [Chapter 15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml), we’ll replace it with the session information necessary to verify the identity of the user performing the action.

Create the final resolvers file, _resolvers.ts_, in the _graphql_ folder and add the code from [Listing 13-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter13.xhtml#Lis13-7) to it. This code will merge the mutation and query definitions.

```
import {locationQueries} from "graphql/locations/queries";
import {locationMutations} from "graphql/locations/mutations";

export const resolvers = {
    Query: {
        ...locationQueries,
    },
    Mutation: {
        ...locationMutations,
    },
};
```

Listing 13-7: The graphql/resolvers.ts file

In addition to the schema, we must pass the Apollo server an object containing all resolvers, as discussed in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml). To be able to do so, we must import the queries and mutations. Then we use the spread operator to merge the imported objects into the resolvers object, which we export. Now, with the schema and resolvers object available, we can create the API endpoint and instantiate the Apollo server.

### Adding the API Endpoint to Next.js

When we discussed the differences between REST and GraphQL APIs, we pointed out that unlike REST, where every API has its own endpoint, GraphQL provides only one endpoint, typically exposed as _/graphql_. To create this endpoint, we’ll use the Apollo server’s Next.js integration, as we did in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml).

Create the _graphql.ts_ file in the _pages/api_ folder and copy the code in [Listing 13-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter13.xhtml#Lis13-8), which defines the API handler and its single entry point.

```
import {ApolloServer, BaseContext} from "@apollo/server";
import {startServerAndCreateNextHandler} from "@as-integrations/next";

import {resolvers} from "graphql/resolvers";
import {typeDefs} from "graphql/schema";
import dbConnect from "middleware/db-connect";

import {NextApiHandler, NextApiRequest, NextApiResponse} from "next";

❶ const server = new ApolloServer<BaseContext>({
    resolvers,
    typeDefs,
});

❷ const handler = startServerAndCreateNextHandler(server, {
    context: async () => {
        const token = {};
        return {token};
    },
});

❸ const allowCors =
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

❹ const connectDB =
    (fn: NextApiHandler) =>
    async (req: NextApiRequest, res: NextApiResponse) => {
        await dbConnect();
        return await fn(req, res);
    };

export default connectDB(allowCors(handler));
```

Listing 13-8: The pages/api/graphql.ts file

We import all the elements we need to create the API handler: the Apollo server, a helper for the Apollo–Next.js integration, our resolvers, the GraphQL schema files, the function used to connect to the database, and the Next.js API helpers.

We create a new Apollo server with the resolvers and schema ❶. Then we use the Next.js integration helper ❷ to start the Apollo server and return a Next.js handler. The integration helper uses a serverless Apollo setup to smoothly integrate into the Next.js custom server instead of creating its own. In addition, we pass the context with an empty token to the handler. This is how we’ll access the JWT we receive in the OAuth flow and pass it to the resolvers later.

Next, we create the wrapper functions discussed in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml) to add the CORS headers ❸ and ensure that we have a database connection on each API call ❹. We can safely do so because we set up our database connection in a way that returns the existing cached connection. Finally, we export the returned asynchronous wrapped handler.

Visit the Apollo sandbox at _http:/localhost:3000/api/graphql_ and run a few queries to test the GraphQL API before moving on to the next chapter. If you see the weather queries and mutations instead of the Food Finder’s, clear your browser’s cache and do a hard reload.

### Summary

We’ve successfully added the GraphQL API to the middleware. With the code in this chapter, we can now use the Apollo sandbox to read and update values in the database. We’ve also already prepared the Apollo handler for authentication by providing it with an empty token. Now we’re ready to use the JWT token we’ll receive from the OAuth flow in [Chapter 15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter15.xhtml) to protect the API’s mutations. But before we add this authentication, let’s build the frontend.