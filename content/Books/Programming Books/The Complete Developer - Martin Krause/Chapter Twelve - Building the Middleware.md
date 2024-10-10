---
id: 01J9QDPB3YM7PSVE2SPK7A5QPG
title: Chapter Twelve - Building the Middleware
modified: 2024-10-08T21:19:18-04:00
description: Building the middleware for FoodFinder
tags:
  - middleware
  - full-stack
  - web-development
  - books
  - programming
---
The middleware is the software glue connecting the frontend we’ll create later to the existing MongoDB instance in the backend container. In this chapter, we’ll set up Mongoose, connect it to our database, and then create a Mongoose model for the application. In the next chapter, we’ll complete the middleware by writing a GraphQL API.

This middleware is part of Next.js; hence, we’ll work with the application container. But because the Docker daemon ensures that the files in our local application directory are instantly available within the working directory inside the application container, we can use our local code editor or IDE to modify files on our local machine. There is no need to connect to the container shell, let alone interact with docker compose; you should see all changes instantly on _http://localhost:3000_.

### Configuring Next.js to Use Absolute Imports

Before we write our first line of code in Next.js, let’s make a minor adjustment to the Next.js configuration. We want the paths of any module imports to be _absolute_, meaning they start from the application’s root folder rather than the location of the file that is importing them. The imports in [Listing 12-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-1), which come from the _pages/api/graphql.ts_ file we created in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml), are examples of relative imports.

```
import {resolvers} from "../../graphql/resolvers";
import {typeDefs} from "../../graphql/schema";
```

Listing 12-1: The import statements in pages/api/graphql.ts

You should see that they start from the file’s location, then go up two levels to the root folder, and finally find the _graphql_ folder containing the _resolvers_ and _schema_ TypeScript files.

The more complex our application becomes, the more levels of nesting we’ll have, and the more inconvenient we’ll find this manual traversing of the directories up to the root folder. This is why we want to use absolute imports that start directly from the root folder, as shown in [Listing 12-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-2).

```
import {resolvers} from "graphql/resolvers";
import {typeDefs} from "graphql/schema";
```

Listing 12-2: The absolute import statements for pages/api/graphql.ts

Notice that we don’t need to traverse up to the root level before importing files. To achieve this, open the _tsconfig.json_ file that create-next-app created in the application’s code root directory, _code/foodfinder-application_, on your local machine, and add a line that sets the baseUrl to the root folder ([Listing 12-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-3)).

```
{
    "compilerOptions": {
        "baseUrl": ".",
--snip--
    }
}
```

Listing 12-3: Using absolute URLs

Restart the application’s container, as well as the Next.js application, with docker compose restart foodfinder-application in a new command line tab.

### Connecting Mongoose

Now it’s time to start working on the middleware. We’ll begin by adding Mongoose to the application. Connect to the application’s container terminal:

```
$ docker exec -it foodfinder-application npm install mongoose
```

Here we use npm install mongoose to install the package. As long as the containers are running, we don’t need to rebuild the frontend image immediately, as we’ve installed the packages directly into the running container.

#### Writing the Database Connection

To connect the Next.js application to the MongoDB instance, we’ll define the environment variable MONGO_URI and assign it a connection string that matches the backend’s exposed port and location. Create a new _.env.local_ file in the application’s root directory, next to the _tsconfig.json_ file, and add this line to it:

```
MONGO_URI=mongodb://backend:27017/foodfinder
```

Now we can connect the application to the MongoDB instance that the Docker container exposes on port 27017. Create a folder, _middleware_, in the root folder _code/foodfinder-application_. Here we’ll place all the middleware-related TypeScript files. Create a new file, _db-connect.ts_, in this folder and paste in the code from [Listing 12-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-4).

```
import mongoose, {ConnectOptions} from "mongoose";

const MONGO_URI = process.env.MONGO_URI || " ";

if (!MONGO_URI.length) {
    throw new Error(
        "Please define the MONGO_URI environment variable (.env.local)"
    );
}
let cached = global.mongoose;

if (!cached) {
    cached = global.mongoose = {conn: null, promise: null};
}

async function dbConnect(): Promise<any> {

    if (cached.conn) {
        return cached.conn;
    }

    if (!cached.promise) {

        const opts: ConnectOptions = {
            bufferCommands: false,
            maxIdleTimeMS: 10000,
            serverSelectionTimeoutMS: 10000,
            socketTimeoutMS: 20000,
        };

        cached.promise = mongoose
            .connect(MONGO_URI, opts)
            .then((mongoose) => mongoose)
            .catch((err) => {
                throw new Error(String(err));
            });
    }

    try {
        cached.conn = await cached.promise;
    } catch (err) {
        throw new Error(String(err));
    }

    return cached.conn;
}

export default dbConnect;
```

Listing 12-4: The TypeScript code to connect the application to the database in db-connect.ts

We import the _mongoose_ package and the ConnectOptions type, both of which we need to connect to the database. We then load the connection string from the environment variables and verify that the string is not empty.

Next, we set up our connection cache. We use a global variable to maintain the connection across hot-reloads and ensure that multiple calls to our dbConnect function always return the same connection. Otherwise, there is the risk that our application will create new connections during each hot-reload or on each call of the function, both of which would fill up our memory quickly. If there’s no cached connection, we initialize it with a dummy object.

We create the asynchronous function dbConnect, which actually opens and handles the connection. The database is remote and not instantly available, so we use an async function that we export as the module’s default function. Inside the function’s body, we first check for an already existing cached connection and directly return any existing ones. Otherwise, we create a new one. Therefore, we define the connection options, and then we create a new connection; here, we use the promise pattern to remind us of the two possible ways to handle asynchronous calls. Finally, we await the connection to be available, and then return the Mongoose instance.

To open a cached connection to MongoDB through Mongoose, we can now import the dbConnect function from the _middleware/db-connect_ module and await the Mongoose connection.

#### Fixing the TypeScript Warning

In your IDE, you should immediately see that TSC warns us about using global.mongoose. A closer look at the message, Element implicitly has an 'any' type because type 'typeof globalThis' has no index signature.ts (7017), tells us that we need to add the mongoose property to the globalThis object.

As we discussed in [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml), we use the _custom.d.ts_ file to define custom global types. Create a new file, _custom.d.ts_, next to the _middleware_ folder in the root directory. As soon as you paste the code from [Listing 12-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-5) into it, the global namespace should contain the mongoose property typed as mongoose, and TSC can find it.

```
import mongoose from "mongoose";

declare global {
    var mongoose: mongoose;
}
```

Listing 12-5: The code in custom.d.ts used to define the custom global type mongoose

With the custom global type definition in place, the TSC should no longer complain about the missing type definition for global.mongoose. We can move on to create the Mongoose model for our full-stack application.

### The Mongoose Model

Our application has one database containing a collection of documents representing location data, as you saw in the seed script from [Chapter 11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter11.xhtml). We’ll create a Mongoose model for this location collection. In [Chapter 7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml), you learned that this requires having an interface to type the documents for TypeScript, a schema to describe the documents for the model, a type definition, and a set of custom types to define the Mongoose model. In addition, we’ll create a set of custom types to perform the CRUD operations on the locations model for the application.

Create a _mongoose_ folder with the subfolder _locations_ next to the _middleware_ folder in Next.js’s root directory. The _mongoose_ folder will host all files relevant to Mongoose in general, and the _locations_ folder will contain all files specific to the location model.

#### Creating the Schema

In [Chapter 7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml), you learned that the schema describes the structure of a database’s documents and that you need to create a TypeScript interface before creating a schema so that you can type the schema and model accordingly. Technically, in versions of Mongoose later than 6.3.1, we don’t need to define this interface by ourselves. Instead, we can automatically infer the interface as a type from the schema. Create the file _schema.ts_ inside the _mongoose/locations_ folder and paste the code from [Listing 12-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-6) into it.

```
import {Schema, InferSchemaType} from "mongoose";

export const LocationSchema: Schema = new Schema<LocationType>({
    address: {
        type: "String",
        required: true,
    },
    street: {
        type: "String",
        required: true,
    },
    zipcode: {
        type: "String",
        required: true,
    },
    borough: {
        type: "String",
        required: true,
    },
    cuisine: {
        type: "String",
        required: true,
    },
    grade: {
        type: "String",
        required: true,
    },
    name: {
        type: "String",
        required: true,
    },
    on_wishlist: {
        type: ["String"],
        required: true,
    },
    location_id: {
        type: "String",
        required: true,
    },
});

export declare type LocationType = InferSchemaType<typeof LocationSchema>;
```

Listing 12-6: The mongoose/locations/schema.ts file

We import the Schema constructor and InferSchemaType, the function for inferring the schema type, both of which are part of the Mongoose module. Then we define and directly export the schema. The schema itself is straightforward. A document in the location collection has a few self-explanatory properties that are all typed as strings except for the on_wishlist property, which is an array of strings. To keep the application simple, we will store the IDs of users who added a particular location to their wish list directly in a location’s document instead of creating a new Mongoose model and MongoDB document for each user’s wish list. This isn’t a great design for a real application, but it’s fine for our purposes. Lastly, we infer and export the LocationType directly from the schema instead of creating the interface manually.

#### Creating the Location Model

With the schema and required interface in place, it’s time to create the model. Create the file _model.ts_ in the _mongoose/location_ folder and paste the code from [Listing 12-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-7) into it.

```
import mongoose, {model} from "mongoose";
import {LocationSchema, LocationType} from "mongoose/locations/schema";

export default mongoose.models.locations ||
    model<LocationType>("locations", LocationSchema);
```

Listing 12-7: The mongoose/locations/model.ts file

After importing the required dependencies from the Mongoose package, we import the LocationSchema and LocationType from the _schema.ts_ file we created previously. Then we use these to create and export our locations model, unless there is already a model called _locations_ initialized and present. In this case, we return the existing one.

At this point, we’ve successfully created the Mongoose model and connected it to the database. We can now access the MongoDB instance and create, read, update, and delete documents in the locations collection through Mongoose’s API.

To test that everything is working, try creating a temporary REST API that initializes a connection to the database and then queries all documents through the model. You can make this new file, _test-middleware.ts_, in the application’s _pages/api_ folder and paste the code from [Listing 12-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-8) into it.

```
import type {NextApiRequest, NextApiResponse} from "next";

import dbConnect from "middleware/db-connect";
import Locations from "mongoose/locations/model";

export default async function handler(
    req: NextApiRequest, res: NextApiResponse<any>
) {
    await dbConnect();
    const locations = await Locations.find({});
    res.status(200).json(locations);
}
```

Listing 12-8: A temporary REST API to test the database connection

This API imports required dependencies from Next.js, the dbConnect function, and the Locations model we created earlier. In the asynchronous API handler, it calls the dbConnect function and waits until Mongoose connects to the database. Then it calls Mongoose’s find API on the Locations model with an empty filter object. Once it receives the locations, the API handler will send them to the client.

If you open _http://localhost:3000/api/test-middleware_, you should see a JSON object with all available locations, similar to [Figure 12-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#fig12-1).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure12-1.jpg)

Figure 12-1: The API to test the middleware returns a JSON object with all locations stored in the database.

You’ve successfully created the Mongoose model and run your first database query.

### The Model’s Services

[Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml) discussed how we usually abstract database CRUD operations into service calls to simplify the implementation of GraphQL APIs down the line. This is what we’ll do now, and as a first step, let’s outline the required functionality.

We need one public service that queries all available locations so that they can be displayed in the app’s overview page. To display a location’s details, we need another public service that can find a specific location. We’ll opt to use the location’s ID as a parameter for the service and then look up the location by ID. To handle the wish list functionality, we need a service that can update a user’s wish list, as well as another service that we can use to decide whether a given location is currently on the user’s wish list; depending on the result, we’ll display either an Add To or Remove From button.

To design the service calls that find and return locations, we’ll create one public function for each public API and a unified internal function, findLocations, that calls Mongoose’s find function. The public APIs construct the filter object that Mongoose uses to filter the documents in the collection. In other words, it creates the database query. Also, it sets up additional options we’ll pass to the Mongoose API. This design should reduce the amount of code we need to write and prevent repetition.

#### Creating the Location Service’s Custom Types

You may have noticed that we’ll need two custom types for the parameters to the unified findLocations function. One parameter defines the properties for a find operation related to the wish list, and one is a location’s ID. Create the file _custom.d.ts_ in the _mongoose/location_ folder to define these types, as shown in [Listing 12-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-9).

```
export declare type FilterLocationType = {
    location_id: string | string[];
};

export declare type FilterWishlistType = {
    on_wishlist: {
        $in: string[];
    };
};
```

Listing 12-9: The mongoose/locations/custom.d.ts file

We define and directly export these two custom types. FilterLocationType is straightforward. It defines an object with one property, the location’s ID, which is either a string or an array of strings. We use it to find a location by its ID. The second type is FilterWishlistType, which we’ll use to find all locations that contain the user’s ID in their on_wishlist property. We set the value for Mongoose’s $in operator as an array of strings.

#### Creating the Location Services

Now that we’ve created custom types for the services, we can implement them. As usual, we create a file _services.ts_ in the _mongoose/location_ folder and add the code from [Listing 12-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-10) to it.

```
import Locations from "mongoose/locations/model";
import {
    FilterWishlistType,
    FilterLocationType,
} from "mongoose/locations/custom";
import {LocationType} from "mongoose/locations/schema";
import {QueryOptions} from "mongoose";

async function findLocations(
    filter: FilterLocationType | FilterWishlistType | {}
): Promise<LocationType[] | []> {
    try {
    let result: Array<LocationType | undefined> = await Locations.find(
            filter
        );
        return result as LocationType[];
    } catch (err) {
        console.log(err);
    }
    return [];
}

export async function findAllLocations(): Promise<LocationType[] | []> {
    let filter = {};
    return await findLocations(filter);
}

export async function findLocationsById(
    location_ids: string[]
): Promise<LocationType[] | []> {
    let filter = {location_id: location_ids};
    return await findLocations(filter);
}

export async function onUserWishlist(
    user_id: string
): Promise<LocationType[] | []> {
    let filter: FilterWishlistType = {
        on_wishlist: {
            $in: [user_id],
        },
    };
    return await findLocations(filter);
}

export async function updateWishlist(
    location_id: string,
    user_id: string,
    action: string
) : Promise<LocationType | null | {}>
 {
    let filter = {location_id: location_id};
    let options: QueryOptions = {upsert: true, returnDocument: "after"};
    let update = {};

    switch (action) {
        case "add":
            update = {$push: {on_wishlist: user_id}};
            break;
        case "remove":
            update = {$pull: {on_wishlist: user_id}};
            break;
    }

    try {
        let result: LocationType | null = await Locations.findOneAndUpdate(
            filter,
            update,
            options
        );
        return result;
    } catch (err) {
        console.log(err);
    }
    return {};
}
```

Listing 12-10: The mongoose/locations/services.ts file

After importing dependencies, we create the function that will actually call Mongoose’s find API on the model and await the data from the database. This function will query the database for all public services that use find, so it’s the foundation of all our services. Its one parameter, the filter object, can be passed to the model’s find function to retrieve the documents that match the filter. The filter is either an empty object that returns all locations or one of our custom types, FilterLocationType or FilterWishlistType. As soon as we have the data from the database, we cast it to the LocationType and then return it. If there is an error, we log it and then return an empty array to match the defined return types: either an array of LocationTypes or an empty array.

The following three functions are the public services, which will provide database access to other TypeScript modules and the user interface. All follow the same structure. First, within the findLocationsById function, we set the filter object to a particular parameter. Then we call the findLocations function with this service-specific filter object. Because every service calls the same function, services also have the same return signature, and each returns an array of locations or an empty array. The first uses an empty object. Hence, it filters for nothing and instead returns all documents from the collection. The function findLocationsById uses the FilterLocationType and returns the documents that match the given location IDs.

The next function, onUserWishlist, uses a slightly more complex filter object. It has the type FilterWishlistType, and we pass it to the findLocations function to get all locations whose on_wishlist array contains the given user ID. Note that we type the filter objects explicitly upon declaration. This deviates from the advice given in [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml), but we do it here to ensure that TSC verifies the object properties, as it cannot infer the types from their usage in this case.

Finally, we implement the updateWishlist function. It is slightly different from the previous ones, but the overall structure should look familiar. Again, we build the filter object from the first parameter, and we use the second one, the user ID, to update the on_wishlist array. Unlike in previous functions, however, we use another parameter to specify whether we want to add or remove the user ID to or from the array. Using a switch/case statement here is a convenient way to reduce the number of exposed services. Depending on the action parameter, we fill the update object with either the $push operator, which adds the user ID to the on_wishlist array, or the $pull operator, which removes the user ID. We pass the object to Mongoose’s findOneAndUpdate API to look for the first document that matches the filter, and we directly update the record and then return the updated document or an empty object.

#### Testing the Services

Let’s use our temporary REST API to evaluate the services. Open the _test-middleware.ts_ file we created earlier and update it with the code from [Listing 12-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter12.xhtml#Lis12-11).

```
import type {NextApiRequest, NextApiResponse} from "next";
import dbConnect from "middleware/db-connect";

import {findAllLocations} from "mongoose/locations/services";

export default async function handler(
    req: NextApiRequest,
    res: NextApiResponse<any>
) {
    await dbConnect();
    const locations = await findAllLocations();
    res.status(200).json(locations);
}
```

Listing 12-11: The pages/api/test-middleware.ts file using the services

Instead of directly importing the model and using Mongoose’s find method on it, we import the location services and query all locations with the findAllLocations service. If you open the API at _http://localhost:3000/api/test-middleware_ in your browser, you should once again see a JSON object with all available locations.

### Summary

We’ve successfully created the first part of the middleware. With the code in this chapter, we can use a Mongoose model to create, read, update, and delete documents in the MongoDB collection. To perform these actions, we set up the services we’ll connect to our upcoming GraphQL API. In the next chapter, we’ll delete the temporary testing API middleware and replace it with a proper GraphQL API.