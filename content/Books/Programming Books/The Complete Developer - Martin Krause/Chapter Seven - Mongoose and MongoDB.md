---
id: 01J9QC76QHRGNQPPP5QBYRP464
modified: 2024-10-08T21:15:52-04:00
title: Chapter Seven - Mongoose and MongoDB
description: How to use non-sql databases
tags:
  - mongodb
  - mongoose
  - non-sql
  - database
  - full-stack
  - programming
  - books
---
Most applications rely on a database management system, or _database_ for short, to organize and grant access to a collection of datasets. In this chapter, you’ll work with the MongoDB non-relational database and Mongoose, its accompanying object mapper.

Because MongoDB returns data as JSON and uses JavaScript for database queries, it provides a natural choice for full-stack JavaScript developers. In the following sections, you’ll learn how to create a Mongoose model through which you can query your database, simplify your interactions with MongoDB, and craft middleware that connects your frontend to your backend database. You’ll also write service functions to implement the four CRUD operations on the database.

In [Exercise 7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Exe7) on page 125, you’ll add a database to the GraphQL API you created in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml), replacing its current static datastore.

### How Apps Use Databases and Object-Relational Mappers

An app needs a database to store and manipulate data. So far in this book, our app’s APIs returned only predefined datasets, saved in files, that couldn’t change. We used parameters in our requests to add to the dataset but couldn’t store the data between different API calls (called _persisting_ the data). If we wanted to update the app’s weather information, for example, we’d need a database to persist the data so that the next API call could read it. In full-stack development, we commonly use databases to store user-related data. Another example of a database is the one that your email client uses to store your messages.

To work with a database, we first need to connect to it and authenticate with it. Once we have access to the data, we can execute queries to ask for certain datasets. The query returns the results containing data that our app can display or use in some other way. How each of these steps works in practice depends on the specific database in use.

Querying the data by using the database’s API tends to be clumsy because it usually requires a good amount of boilerplate code, even to simply establish and maintain the connection. Hence, we often use an _object-relational mapper_ or _object data modeling tool_, which simplifies working with the databases by abstracting some of the details. For example, the Mongoose object data modeling tool for MongoDB handles database connections for us, saving us from having to check for an open database connection during each interaction.

Mongoose also makes it easier to handle the fact that MongoDB runs on a separate database server. Working with distributed systems requires making asynchronous calls, which you learned about in [Chapter 2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml). With Mongoose, we can access the data with an object-oriented async/await interface instead of using clumsy callback functions.

In addition, MongoDB is schema-less; it doesn’t require us to predefine and strictly adhere to a schema. While convenient, this flexibility is also a common source of errors, especially in large-scale applications or projects with a rotating cast of developers. In [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml), we discussed the benefits of adding types to JavaScript by using TypeScript. Mongoose types and verifies the integrity of MongoDB’s data models similarly, as you’ll discover in “Defining a Mongoose Model” on page 118.

### Relational and Non-Relational Databases

Databases can organize data in several ways, which fall into two main categories: relational and non-relational. _Relational databases_, such as MySQL and PostgreSQL, store data in one or more tables. You can think of these databases as resembling Excel spreadsheets. As in Excel, each table has a unique name and contains columns and rows. The columns define properties, such as the data type, for all data stored in the column, and the rows contain the actual datasets, each of which is identified by a unique ID. Relational databases use some variation of Structured Query Language (SQL) for their database operations.

MongoDB is a _non-relational database_. Unlike traditional relational databases, it stores data as JSON documents instead of tables and doesn’t use SQL. Sometimes termed _NoSQL_, non-relational databases can store data in many different formats. For example, the popular NoSQL databases Redis and Memcached use key-value storage, which makes them highly performant and easily scalable. Thus, they’re often used as in-memory caches. Another NoSQL database, Neo4j, is a _graph database_ that uses graph theory to store data as nodes, a concept we mentioned in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml). These are just a few examples of non-relational databases.

MongoDB is the most widely used _document database_; instead of tables, rows, and columns, it organizes data in collections, documents, and fields. The _field_ is the smallest unit in the database. It defines the data type and additional properties and contains the actual data. You can consider it the rough equivalent of a column in a SQL table. _Documents_, which are made of fields, are like rows in a SQL table. We sometimes call them records, and MongoDB uses _BSON_, a binary representation of a JSON object, to store them. A _collection_ is roughly equivalent to a SQL table, but instead of rows and columns, it aggregates documents.

Because non-relational databases can store data in different formats, each database uses a specific, optimized query language for CRUD operations. These low-level APIs focus on accessing and manipulating the data, and not necessarily on the developer experience. By contrast, object-relational mappers provide a high-level abstraction with a clean and simplified interface to the query language. So, while MongoDB has the MongoDB Query Language (MQL), we’ll use Mongoose to access it.

### Setting Up MongoDB and Mongoose

Before you start using MongoDB and Mongoose, you must add them to your sample project. For the sake of simplicity, we’ll use an in-memory implementation of MongoDB rather than install and maintain a real database server on our machines. This is appropriate for testing the chapter’s examples, but not for deploying an actual application, as it does not persist the data between restarts. You’ll gain experience setting up a real MongoDB server when you build the Food Finder application in [Part II](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/part2.xhtml). [Chapter 11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter11.xhtml) will show you how to use a pre-built Docker container that contains the MongoDB server.

Run this command in the root directory of the refactored Next.js app from [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml):

```
$ npm install mongodb-memory-server mongoose
```

Then create two new folders in the root directory, next to the _package .json_ file: one for the Mongoose code, called _mongoose_, with subfolder _weather_, and one called _middleware_, which will hold the necessary middleware.

### Defining a Mongoose Model

In order to verify the integrity of our data, we must create a schema-based Mongoose _model_, which acts as a direct interface to a MongoDB collection in a database. All interactions with the database will happen through the model. Before we create the model, though, we need to create the schema itself, which defines the structure of the database’s data and maps the Mongoose instance to the documents in the collection.

Our Mongoose schema will match the schema created for the GraphQL API in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml). That’s because we’ll connect the GraphQL API to the database in [Exercise 7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Exe7) on page 125, allowing us to replace the static JSON object with datasets we queried from the database.

#### The Interface

Before writing the Mongoose model and schema in TypeScript, let’s declare a TypeScript interface. Without a matching interface, we won’t be able to type the model or schema for TSC, and the code won’t compile. Paste the code shown in [Listing 7-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-1) into the _mongoose/weather/interface.ts_ file.

```
export declare interface WeatherInterface {
    zip: string;
    weather: string;
    tempC: string;
    tempF: string;
    friends: string[];
};
```

Listing 7-1: The interface for the Mongoose weather model

The code is a regular TypeScript interface with properties matching the GraphQL and Mongoose schemas.

#### The Schema

[Listing 7-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-2) shows the Mongoose schema. Its top-level properties represent the fields in the document. Each field has a type and a flag indicating whether it is required. Fields can also have additional optional properties, such as custom or built-in validators. Here we use the built-in required validator; other common built-in validators are minlength and maxlength for strings, and min and max for numbers. Add the code to the _mongoose/weather/schema.ts_ file.

```
import {Schema} from "mongoose";
import {WeatherInterface} from "./interface";

export const WeatherSchema = new Schema<WeatherInterface>({
    zip: {
        type: "String",
        required: true,
    },
    weather: {
        type: "String",
        required: true,
    },
    tempC: {
        type: "String",
        required: true,
    },
    tempF: {
        type: "String",
        required: true,
    },
    friends: {
        type: ["String"],
        required: true,
    },
});
```

Listing 7-2: The schema for the Mongoose weather model

We use an object passed to the schema constructor to create the schema and set WeatherInterface as its SchemaType. Therefore, we import the Schema function from the _mongoose_ package and the interface we created previously.

Like TypeScript, which adds custom types to JavaScript, Mongoose casts each property to its associated _SchemaType_, which provides the configuration of the model. The available types are a mixture of built-in JavaScript types, like Array, Boolean, Date, Number, and String, and custom types, like Buffer and ObjectId, the latter of which refers to the default unique _id property that Mongoose adds to each document upon creation. This is similar to the primary key you might know from relational databases.

The weather API we created in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml) returned an object with four properties: zip, weather, tempC, and tempF, each of which is a string. In addition, we have one array of strings in the friends property. In this schema, we define the same properties, then export the schema.

#### The Model

Now that we have a schema, we can create the Mongoose model. This wrapper on the schema will provide access to the MongoDB documents in the collection for all CRUD operations. We write the model in the _mongoose/weather/model.ts_ file, whose code is in [Listing 7-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-3). Keep in mind that we haven’t yet connected it to the MongoDB database on the server.

```
import mongoose, {model} from "mongoose";
import {WeatherInterface} from "./interface";
import {WeatherSchema} from "./schema";

export default mongoose.models.Weather ||
    model<WeatherInterface>("Weather", WeatherSchema);
```

Listing 7-3: The Mongoose weather model

First we import the Mongoose module and the model constructor from the _mongoose_ package, as well as the interface and the schema we created earlier. Then we set up the Weather model, using WeatherInterface to type it. We pass it two parameters: the model’s name, Weather, and the schema, which defines the model’s internal data structure. Mongoose binds the newly created model to our MongoDB instance’s collection. The Weathers collection resides in the _Weather_ database, both of which Mongoose creates. Note that we need to check for an existing Weather model on mongoose.models before creating a new one; otherwise, Mongoose will throw an error. We export the model so that we can use it in our following modules.

#### The Database-Connection Middleware

Several times in this book so far, we’ve mentioned that full-stack development covers an application’s frontend, backend, and middleware, which is often also referred to as “application glue.” Now it’s time to create our first dedicated middleware.

This middleware will open a connection to the database, then use Mongoose’s asynchronous helper function to maintain that connection. Next, it will map Mongoose’s models to the MongoDB collections so that we can access them through Mongoose. Conveniently, the connection helper will buffer the operations and reconnect to the database if necessary, so we don’t need to handle connectivity issues by ourselves. Paste the code from [Listing 7-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-4) into the _middleware/db-connect.ts_ file.

```
import mongoose from "mongoose";
import {MongoMemoryServer} from "mongodb-memory-server";

async function dbConnect(): Promise<any | String> {
    const mongoServer = await MongoMemoryServer.create();
    const MONGOIO_URI = mongoServer.getUri();
    await mongoose.disconnect();
    await mongoose.connect(MONGOIO_URI, {
        dbName: "Weather"
    });
}

export default dbConnect;
```

Listing 7-4: The Mongoose middleware

We import the _mongoose_ package and the _mongodb-memory-server_ database. The async function dbConnect, which we define and then export, manages the connection to the database server through the mongoose.connect function. We create an instance of the MongoMemoryServer to persist our data in memory rather than use a real database server, as discussed. Then we store the connection string in the constant MONGOIO_URI. Because we are using the in-memory server, this string is dynamic, but for a remote database, it would be a static string representing the database’s server address. Then we close all existing connections and use Mongoose to open a new connection. The Mongoose models are already mapped and available, so we’re ready to perform our first queries.

### Querying the Database

Now it’s time to write database queries. Instead of sprinkling these queries around your application code or writing them directly in the GraphQL resolvers, you should extract them as services.

A _service_ is a function that performs the actual CRUD operations on the Mongoose model and returns the result. Each GraphQL resolver can then call a service function, and all internal database access should happen through these functions. Moreover, each service should be responsible for only one specific CRUD operation. Mongoose automatically queues the commands and executes them, maintains the connection, and then processes the queue as soon as there is a connection to the database.

This section introduces service functions and basic Mongoose commands. However, it isn’t a complete reference. When you start working with Mongoose on your own projects, look up all the functions you’ll need in the Mongoose documentation.

#### Creating a Document

The first and most basic operation is the “create” operation. It is conveniently called mongoose.create and, fortunately, we can use it to both create and update a dataset. That’s because Mongoose automatically creates a new database entry, or document, if the entry doesn’t already exist. Hence, we don’t need to check whether a dataset exists and then conditionally create it before updating it.

[Listing 7-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-5) shows a basic implementation of a service function that stores a dataset in the database. Place the code in the _mongoose/weather/ services.ts_ file.

```
import WeatherModel from "./model";
import {WeatherInterface} from "./interface";

export async function storeDocument(doc: WeatherInterface): Promise<boolean> {
    try {
        await WeatherModel.create(doc);
    } catch (error) {
        return false;
    }
    return true;
}
```

Listing 7-5: Creating a document through Mongoose

To store a document, we create and export the async function storeDocument, which takes the dataset as the argument. Here we type it as WeatherInterface. Then we call the create function on the model and pass the dataset to it. The function will create and insert the document in WeatherModel, which is the weather collection in the MongoDB instance. Finally, it returns a Boolean to indicate the status of the operation.

#### Reading a Document

To implement the “read” operation, we query MongoDB through Mongoose’s findOne function. It takes one argument, an object with the properties to look for, and returns the first match. Extend the _mongoose/weather/services.ts_ file with the code in [Listing 7-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-6). It defines a findByZip function to find and return the first document from the Weathers collection whose zip property matches the ZIP code passed to the function as a parameter.

```
export async function findByZip(
    paramZip: string
): Promise<Array<WeatherInterface> | null> {
    try {
        return await WeatherModel.findOne({zip: paramZip});
    } catch (err) {
        console.log(err);
    }
    return [];
}
```

Listing 7-6: Reading data through Mongoose

We add and export the async function readByZip to the services in the _services.ts_ file. The function takes one string parameter, the ZIP code, and returns either an array with documents or an empty array. Inside the new service function, we call Mongoose’s findOne function on the model and pass a filter object, looking for the document whose zip field matches the parameter’s value. Finally, the function returns the result or null.

#### Updating a Document

We mentioned that we can use the create function to update documents. However, there is also a specific API for this task: updateOne. It takes two arguments. The first is the filter object, similar to the filter we used with findOne, and the second is an object with the new values. You can think of updateOne as a combination of the “find” and “create” functions. Extend the _mongoose/weather/services.ts_ file with the code from [Listing 7-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-7).

```
export async function updateByZip(
    paramZip: string,
    newData: WeatherInterface
): Promise<boolean> {
    try {
        await WeatherModel.updateOne({zip: paramZip}, newData);
        return true;
    } catch (err) {
        console.log(err);
    }
    return false;
}
```

Listing 7-7: Updating data through Mongoose

The updateByZip function that we add to the services takes two parameters. The first one is a string, paramZip, which is the ZIP code we use to query for the document we want to update. The second parameter is the new dataset, which we type as WeatherInterface. We call Mongoose’s updateOne function on the model, passing it a filter object and the latest data. The function should return a Boolean to indicate the status.

#### Deleting a Document

The last CRUD operation we need to implement is a service to delete a document. For this, we use Mongoose’s deleteOne function and add the code from [Listing 7-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-8) to the _mongoose/weather/services.ts_ file. It is similar to the findOne function, except that it directly deletes the query’s result. Mongoose queues the operations and deletes the document from the database automatically once there is a connection.

```
export async function deleteByZip(
        paramZip: string
    ): Promise<boolean> {
    try {
        await WeatherModel.deleteOne({zip: paramZip});
        return true;
    } catch (err) {
        console.log(err);
    }
    return false;
}
```

Listing 7-8: Deleting data through Mongoose

The async function deleteByZip takes one string parameter, zip. We use it to query the model and find the document to delete, passing the filter to Mongoose’s deleteOne function. The function should return a Boolean.

### Creating an End-to-End Query

In full-stack development, _end-to-end_ typically refers to the ability of data to travel all the way from the app’s frontend (or from one of its APIs) through the middleware to the backend, and then all the way back to its original source. For practice, let’s create a simple end-to-end example using the _/zipcode_ endpoint of our REST API.

We’ll modify the API to take the query parameter from the URL, find the weather object for the requested ZIP code in the database, and then return it, effectively replacing the static JSON response with a dynamic query result. Modify the file _pages/api/v1/weather/[zipcode].ts_ to match [Listing 7-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-9).

```
import type {NextApiRequest, NextApiResponse} from "next";
import {findByZip} from "./../../../../mongoose/weather/services";
import dbConnect from "./../../../..//middleware/db-connect";
dbConnect();

export default async function handler(
    req: NextApiRequest,
    res: NextApiResponse
): Promise<NextApiResponse<WeatherDetailType> | void> {
    let data = await findByZip(req.query.zipcode as string);
    return res.status(200).json(data);
}
```

Listing 7-9: The end-to-end REST API

Notice the modified API handler. We made two major changes to it. First we called dbConnect to connect to the database. Then we used the imported findByZip service and passed it the query parameter cast to a string type. Instead of the static JSON object as before, we now return the dynamic data that we receive from the service function.

We need to perform one more step before we can receive data in response to the API call: _seeding_ the database, or adding initial datasets to it. For simplicity, we use the storeDocuments service and seed directly in the dbConnect function. Modify the _middleware/db-connect.ts_ file to match the code in [Listing 7-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-10), which imports the storeDocument service and adds the datasets after establishing the database connection.

```
import mongoose from "mongoose";
import {MongoMemoryServer} from "mongodb-memory-server";
import {storeDocument} from "../mongoose/weather/services";

async function dbConnect(): Promise<any | String> {
    const mongoServer = await MongoMemoryServer.create();
    const MONGOIO_URI = mongoServer.getUri();
    await mongoose.disconnect();

    let db = await mongoose.connect(MONGOIO_URI, {
        dbName: "Weather"
    });

    await storeDocument({
        zip: "96815",
        weather: "sunny",
        tempC: "25C",
        tempF: "70F",
        friends: ["96814", "96826"]
    });
    await storeDocument({
        zip: "96814",
        weather: "rainy",
        tempC: "20C",
        tempF: "68F",
        friends: ["96815", "96826"]
    });
    await storeDocument({
        zip: "96826",
        weather: "rainy",
        tempC: "30C",
        tempF: "86F",
        friends: ["96815", "96814"]
    });

}
export default dbConnect;
```

Listing 7-10: The naive data seeding in the dbConnect function

Now we can perform the end-to-end request. Visit the REST API endpoint in the browser at _http://localhost:3000/api/v1/weather/96815_. You should see the dataset from the MongoDB database as the API response. Try adjusting the query parameter in the URL to another valid ZIP code. You should get another dataset in the response.

Exercise 7: Connect the GraphQL API to the Database

Let’s rework our weather application’s GraphQL API so that it reads the response data from the database instead of from a static JSON file. The code will look familiar, as we’ll use the same patterns as for the REST API example in the preceding section.

First, verify that you’ve added the MongoDB memory implementation and Mongoose to your project. If not, add them now by following the instructions in “Setting Up MongoDB and Mongoose” on page 117. Next, check that you’ve created the files in the _middleware_ and _mongoose_ folders described throughout this chapter and that they contain the code from [Listings 7-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-1) through [7-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-10).

Now, to connect the GraphQL API to the database, we need to do two things: implement the database connection and refactor the GraphQL resolvers to use its datasets.

#### Connecting to the Database

To query the database through the GraphQL API, we need to have a connection to the database. As you learned in [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml), all API calls have the same endpoint, _/graphql_. This fact will now prove incredibly convenient for us; because all requests have the same entry point, we need to handle the database connection only once. Hence, we open the file _api/graphql.ts_ and modify it to match the code in [Listing 7-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-11).

```
import {ApolloServer} from "@apollo/server";
import {startServerAndCreateNextHandler} from "@as-integrations/next";
import {resolvers} from "../../graphql/resolvers";
import {typeDefs} from "../../graphql/schema";
import {NextApiHandler, NextApiRequest, NextApiResponse} from "next";
import dbConnect from "../../middleware/db-connect";
//@ts-ignore
const server = new ApolloServer({
    resolvers,
    typeDefs
});

const handler = startServerAndCreateNextHandler(server);

const allowCors = (fn: NextApiHandler) =>
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

const connectDB = (fn: NextApiHandler) =>
    async (req: NextApiRequest, res: NextApiResponse) => {
        await dbConnect();
        return await fn(req, res);
    };

export default connectDB(allowCors(handler));
```

Listing 7-11: The api/graphql.ts file including a connection to the database

We made three changes to the file. First we imported the dbConnect function from the middleware; then we created a new wrapper similar to the allowCors function and used it to ensure that each API call connects to the API. We could safely do so because we implemented dbConnect to enforce only one database connection at the same time. Finally, we wrapped the handler with the new wrapper and exported it as the default.

#### Adding Services to GraphQL Resolvers

Now it’s time to add the services to the resolvers. In [Chapter 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml), you learned that query resolvers implement the reading of data, whereas mutation resolvers implement the creation, updating, and deletion of data.

There, we also defined two resolvers: one to return a weather object for a given ZIP code and one to update a location’s weather data. Now we’ll add the services findByZip and updateByZip, which we created in this chapter, to the resolvers. Instead of the naïve implementations with the static data object, we modify the resolvers to query and update the MongoDB documents through the services.

[Listing 7-12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter7.xhtml#Lis7-12) shows the modified code for the _graphql/resolvers.ts_ file in which we refactor these two resolvers.

```
import {WeatherInterface} from "../mongoose/weather/interface";
import {findByZip, updateByZip} from "../mongoose/weather/services";

export const resolvers = {
    Query: {
        weather: async (_: any, param: WeatherInterface) => {
            let data = await findByZip(param.zip);
            return [data];
        },
    },
    Mutation: {
        weather: async (_: any, param: {data: WeatherInterface}) => {
            await updateByZip(param.data.zip, param.data);
            let data = await findByZip(param.data.zip);
            return [data];
        },
    },
};
```

Listing 7-12: The graphql/resolvers.ts file using services

We replace the naive array.filter functionality with the appropriate services. To query the data, we use the findByZip service and pass it the zip variable from the request payload and then return the result data wrapped in an array. For the mutation, we use the updateByZip service. Per type definition, the weather mutation returns the updated dataset. To do so, we query for the modified document with the findByZip service once again and return the result as an array item.

Visit the GraphQL sandbox at _http://localhost:3000/api/graphql_ and play with the API endpoints to read and update documents from the MongoDB database.

### Summary

In this chapter, you explored using the non-relational database MongoDB and its Mongoose object data modeling tool, which lets you add and enforce schemas as well as perform CRUD operations on MongoDB instances. We covered the differences between relational and non-relational databases and how they store data. Then you created a Mongoose schema and a model, connected Mongoose to the MongoDB instance, and wrote the services to perform operations on the MongoDB collection.

Finally, you connected the REST and GraphQL APIs to the MongoDB database. Now, instead of static datasets, all of your APIs return dynamic documents, and you can both read and update documents through them.

MongoDB and Mongoose are extensive technologies with a huge array of functionalities. To learn more about them, consult the official documentation at [_https://mongoosejs.com_](https://mongoosejs.com/) and read the articles at [_https://www.geeksforgeeks.org/mongoose-module-introduction_](https://www.geeksforgeeks.org/mongoose-module-introduction)/.

The next chapter covers Jest, a modern testing framework for conducting unit, snapshot, and integration tests.