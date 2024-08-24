---
id: 01J5HQST6T9ZB96RGJJSJH7NXR
title: GraphQL Servers and Clients
modified: 2024-08-17T23:18:23-04:00
description: How to interact with servers and how to use clients to interact with requests
tags:
  - graphql
  - api
  - web-development
  - servers
  - clients
  - tags
---
# GraphQL Servers

GraphQL is known for its benefits on the client-side, but what about the server-side? The GraphQL servers do the heavy lifting to ensure that the right amount of data is fetched with a minimal number of database lookups and API calls.

## How Does GraphQL Work With the Server?

GraphQL servers are composed of two main parts:

1. Schema
2. Resolvers

The GraphQL Schema defines what the API looks like. That means it specifies the available types, fields and operations. The schema acts as a contract between the client and server, guaranteeing that the data requirements are always met.

A GraphQL Resolver is a function that specifies how to process a specific GraphQL operation and turn it into data.

Each time the server receives a request, it goes through the following steps:

- Parsing the document
- Identify the operation to execute (if more than one)
- Validate the request and return errors if it fails
- Execute the operation (query / mutation / subscription)

There are many approaches to writing a GraphQL server. Let's look at the most common ones used by the GraphQL community.

## Resolver Approach

The most common way of writing a GraphQL server is by defining the schema and writing resolvers for the different operations and fields.

Imagine resolvers as functions that contain instructions on how to process a particular field based on the context.

The basic signature of a resolver looks like the following:

Copy

resolverFunc(data, args, context, info)

- `data` - previously fetched data from the parent
- `args` - key-value pairs of arguments, optional
- `context` - state information per request, typically used for auth logic
- `info` - metadata about the selection context for traversal

This resolver function is now executed for every field in a GraphQL query.

### N+1 Performance Problem

Let's consider we have to fetch a list of authors and their articles. In a simple REST API, the naive version would look something like this:

Copy

fetchData: async () => ORM.getAuthors().getArticles();

There are two (SQL) queries to the database - one to fetch the list of authors and another to fetch the list of articles of each author.

Now let's do this using GraphQL.

The GraphQL query for this would look something like this:

Copy

query {

  author {

    id

    name

    articles {

      id

      title

      content

    }

  }

}

The resolver would look something like this:

Copy

resolvers = {

  Query: {

    author: async () => {

      return ORM.getAllAuthors()

    }

  },

  Author: {

    articles:  async (authorObj, args) => {

      return ORM.getArticlesBy(authorObj.id)

    }

  },

}

Now let's see what the execution for this would look like. Consider there are 3 authors, each having 2 articles.

The first resolver would be called for `author`, which returns all the authors (3 in this case). Now for the relational query `articles`, the resolver `articles` would be called once for each author. This leads to 4 hits to the database (one for author and 3 for articles) in this naive approach.

You can see the apparent performance implications with this approach.

### Dataloader

Dataloader is a utility used as part of your application’s data fetching layer. In trying to solve the N+1 problem, it waits for all the resolvers to load in their individual values, coalesce all individual loads and call the batch function with the requested keys.

## Compiler Approach

Batching of resolvers solves the performance problem to a large extent. It reduces multiple hits to the database. But even with batching, there would still be multiple hits to the database depending on query depth.

The compiler approach lets you map a GraphQL query of any depth to one database query. This is more performant if your GraphQL query deals with data from the database.

Learn how Hasura does [performant GraphQL query execution](https://hasura.io/blog/fast-graphql-execution-with-query-caching-prepared-statements/) using the compiler approach.

## Hybrid Approach

If data is coming from different sources, we need to use a combination of the above approaches. The compiler approach works well for the query’s database parts, while batching queries with DataLoader works best for batching external data sources / HTTP requests.

The hybrid approach architecture uses a server with a connected database for primary CRUD operations. It uses the resolver approach to execute other fields fetching or mutating data from different data sources.

If you are writing your GraphQL server from scratch, you will be using the resolver approach, writing functions to resolve each query field. If you are looking to map your database to GraphQL for instant CRUD, the compiler approach fits in.

Usually, a Hybrid approach is recommended. You would use a server like Hasura, which gives instant CRUD for databases and allows you to write your resolvers if you have some other custom business logic.

## How to Deploy a GraphQL Server?

Deploying a GraphQL server depends on the language you use for its implementation and the platform you choose for deployment.

In most cases, you deploy a GraphQL server the same way you deploy a REST server. You can use a platform like Heroku or an alternative to deploy your GraphQL server.

One of the main benefits of using Hasura is that it gives you a scalable, highly available, globally distributed, fully managed, secure GraphQL API as a service! That means you do not need to worry about deployment.

# GraphQL Clients

In this section, we will look at how specialized GraphQL clients can help with better querying, caching, and building reusable modules.

A GraphQL request can be made using native JavaScript Fetch API. For example, to fetch a list of authors, we can make the query using the following code:

Copy

const limit = 5;

const query = `query author($limit: Int!) {

    author(limit: $limit) {

        id

        name

    }

}`;

fetch('/graphql', {

  method: 'POST',

  headers: {

    'Content-Type': 'application/json',

    'Accept': 'application/json',

  },

  body: JSON.stringify({

    query,

    variables: { limit },

  })

})

  .then(r => r.json())

  .then(data => console.log('data returned:', data));

This, of course, assumes that your server accepts GraphQL requests over HTTP. (Remember GraphQL is protocol agnostic?).

## Why do I need a GraphQL Client?

Now that you have learned that requests can be made using the old fetch API method, what is the point of a GraphQL client?

### Constructing query, processing response

A GraphQL client can help construct the entire query with just the GraphQL document as input and add the relevant headers and context information. So instead of you writing the fetch API call every time, the client will handle it for you giving the response data and error after parsing.

### Managing UI State

A GraphQL client is also helpful in managing the UI state and syncing data across multiple UI components.

### Updating cache

A GraphQL client can also manage cached entries of data fetched from queries or mutations. Reactive updates to UI, as mentioned above, are achieved using a cache.

Popular GraphQL Clients in the community are [Apollo Client](https://github.com/apollographql/apollo-client) and [Relay](https://github.com/facebook/relay).

## Fluent GraphQL Clients

When you are writing GraphQL queries or mutations using a client, you will notice that it is just a raw string with its own syntax. This string is usually parsed into a valid GraphQL query using external libraries.

With fluent GraphQL clients, you can write these queries as objects. Fluent APIs aim to make code more readable via method chaining by returning this or self from each method. Fluent GraphQL clients allow you to write your query as an object, which they then convert to a string query behind the scenes.

In addition to freeing you from strings, they offer

- Strong typing
- Single source of truth for type definitions
- Autocompletion of queries

Here is a [list of Fluent GraphQL Clients](https://github.com/hasura/awesome-fluent-graphql) that you can try out.

# GraphQL Tag

## Parsing queries with graphql-tag gql

When you use a GraphQL client, you will most likely use `graphql-tag`. `graphql-tag` is a library for parsing GraphQL operations. It includes the `gql` tag, which parses a GraphQL operation (query/mutation/subscription) string into an AST (Abstract Syntax Tree) document your client understands.

That means you can define the GraphQL operations in your code using template literals and use `gql` to compile them to an AST document.