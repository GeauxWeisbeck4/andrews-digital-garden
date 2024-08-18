---
id: 01J5HPZGRWJXGX7WXR5X25ZHDR
title: Intro to GraphQL
description: A course introducing GraphQL to new users on Hashura
modified: 2024-08-17T23:07:43-04:00
tags:
  - graphql
  - api
  - web-development
---
# Intro to GraphQL
- [[Intro to GraphQL]]
- [[GraphQL vs. Rest]]
- 


# Course Introduction
GraphQL is becoming the new way to use APIs in modern web and mobile apps.
However, learning new things always takes time and, without getting your hands dirty, it’s hard to understand the nuances of new technology.
So, we put together a practical and concise course that will introduce you to GraphQL and its core concepts.
We will explore the fundamentals of GraphQL and what makes it especially suitable for modern applications, like its realtime capabilities!
## Key topics and takeaways:

- [What is GraphQL?](https://hasura.io/learn/graphql/intro-graphql/what-is-graphql/)
- [GraphQL vs REST](https://hasura.io/learn/graphql/intro-graphql/graphql-vs-rest/)
- [Core Concepts](https://hasura.io/learn/graphql/intro-graphql/core-concepts/)
- [Introspection](https://hasura.io/learn/graphql/intro-graphql/introspection/)
- [Queries](https://hasura.io/learn/graphql/intro-graphql/graphql-queries/)
- [Mutations](https://hasura.io/learn/graphql/intro-graphql/graphql-mutations/)
- [Subscriptions](https://hasura.io/learn/graphql/intro-graphql/graphql-subscriptions/)
- [GraphQL Servers](https://hasura.io/learn/graphql/intro-graphql/graphql-server/)
- [GraphQL Clients](https://hasura.io/learn/graphql/intro-graphql/graphql-client/)
- [What next?](https://hasura.io/learn/graphql/intro-graphql/what-next/)

## What is GraphQL?
GraphQL is a specification for how to talk to an API. It's typically used over HTTP where the key idea is to `POST` a "query" to an HTTP endpoint, instead of hitting different HTTP endpoints for different resources.
GraphQL is designed for developers of web/mobile apps (HTTP clients) to be able to make API calls to fetch exactly the data they need from their backend APIs.
Before going further in understanding GraphQL, it's useful to get a sense of how GraphQL is actually used in an HTTP client.
## GraphQL over HTTP
Check out the diagram below, to get a sense of how GraphQL is typically used in your stack:

![GraphQL over HTTP](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-react/graphql-on-http.png)

### GraphQL client-server flow:
1. Note that the GraphQL query is not really JSON; it looks like the shape of the JSON you _want_. So when we make a 'POST' request to send our GraphQL query to the server, it is sent as a "string" by the client.
2. The server gets the JSON object and extracts the query string. As per the GraphQL syntax and the graph data model (GraphQL schema), the server processes and validates the GraphQL query.
3. Just like a typical API server, the GraphQL API server then makes calls to a database or other services to fetch the data that the client requested.
4. The server then takes the data and returns it to the client in a JSON object.
### Example GraphQL client setup:
In your day to day work, you don't actually need to worry about the underlying HTTP requests & responses.
Just like when you work with a REST API and use a HTTP client to reduce the boilerplate in making API calls and handling responses, you can choose a GraphQL client to make writing GraphQL queries, sending them and handling responses much easier.
In fact, the mechanism of how you send the GraphQL query and accept the GraphQL response has become standard. This makes working with GraphQL very easy on the client.
Here's what a typical GraphQL client setup and making a query would look like:

Copy

// Setup a GraphQL client to use the endpoint
```javascript
const client = new Client("https://myapi.com/graphql");
// Now, send your query as a string (Note that ` is used to create a multi-line string in javascript).
client.query(`
  query {
    user {
      id
      name
    }
  }`)
```
const client = new Client("https://myapi.com/graphql");

// Now, send your query as a string (Note that ` is used to create a multi-line

// string in javascript).

client.query(`

  query {

    user {

      id

      name

    }

  }`);

Do note that, you can make a GraphQL API call using simple JavaScript `fetch` API and you actually do not need a GraphQL client for simple use cases. We will look into this more in the GraphQL Client section later.