---
id: 01J9QC4HA12RHEJ2FAG128PHJ7
title: Chapter Six - REST and GraphQL APIs
modified: 2024-10-08T20:52:18-04:00
description: Learning about APIs
tags:
  - api
  - full-stack
  - web-development
  - programming
  - books
---
An _API_ is a generic pattern used to connect computers or computer programs. Unlike a user interface, it’s designed to be accessed not by a user but by another piece of software. One purpose of APIs is to hide the internal details of a system’s workings while exposing a standardized gateway to the system’s data or functionality.

As a full-stack developer, you’ll usually interact with, or _consume_, two kinds of APIs: internal and third-party. When querying an internal API, you’re consuming data from your own systems, typically from your own database or service. Private APIs are not available to outside parties. For example, your bank might use private APIs to check your credit score or account balance on its internal systems and display them in your online banking profile.

Third-party APIs provide access to data from an external system. For example, the OAuth login you’ll implement in [Part II](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/part2.xhtml) uses an API. You might also use an API to fetch a social media feed or weather information from an external provider to display on your website. Because external APIs are exposed to the public, you can reach them through a public URL, and they document the conventions you should use to access their data in an _API contract_. This contract defines the format of the communications, the parameters the API expects, and the possible responses you might receive for each request. We briefly discussed API and function contracts and why you should type them in [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml).

Full-stack web development primarily uses two types of APIs, REST and GraphQL, both of which transmit data over HTTP. This chapter covers these.

### REST APIs

REST is an architectural pattern used to design RESTful web APIs. These APIs are essentially a set of URLs, each of which provides access to a single resource. They rely on the use of HTTP methods and the standard HTTP status codes to transmit data and accept URL-encoded or request header parameters. Typically, they respond with the requested data in JSON or plaintext.

In fact, you’ve already built your first REST API. Recall the Next.js server you created in [Exercise 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Exe5) on page 89, which provided the _api/weather/:zipcode_ endpoint. So far, we’ve used this endpoint to play with Next.js’s routing, understand dynamic URLs, and learn how to access query parameters. You’ll soon see, however, that this API follows REST conventions: to access it, we used the HTTP GET method to consume the URL endpoint and received a JSON response with an HTTP status code of _200: OK_. Common status code ranges are _2XX_ for successful requests and _3XX_ for redirects. If the request fails, we see the _4XX_ range to indicate a client-related error, such as _401: Unauthorized_, and _5XX_ for server errors, often the generic _500: Internal Server Error_.

As full-stack developers, we might sometimes create our own APIs; more often, though, we’ll find ourselves consuming third-party APIs. Why might we consume, say, a third-party weather API? Well, imagine that we want our app to display the current weather at multiple remote locations. Instead of setting up and maintaining various weather stations on our own and then reading the data from the sensors, which would involve both providing and consuming an API for each of them, we could consume data from a third-party API offered by an existing weather service. Our code might call that API, pass in a ZIP code as a parameter, and receive the weather data for this location in a predetermined format. We’d then display this data on our website.

RESTful APIs enable us to interact with data without knowing anything about how that data was stored or what underlying technology provided it. If you follow an API’s specifications, you should receive the requested data, even if the underlying technology or architecture changes. Beyond this, there are a handful of requirements for an API to be considered RESTful.

#### The URL

A unique URL provides an interface to a RESTful API. Each of a provider’s APIs typically has the same base URL, called the _root entry point_, such as _http://localhost:3000/api_. You can think of this as the APIs’ family name. Often, you’ll see a version number added to the root entry point, because a provider might have multiple versions of an API. For example, there might be the legacy _http://localhost:3000/api/v1_ and an updated _http://localhost:3000/api/v2_. To honor this pattern, you can create a folder _v1_ inside the _api_ folder and move the REST API code there.

> NOTE

_Other common ways of versioning an API include custom headers and query strings. In the first case, the client would request the API with a custom Accept-Version header and receive a matching Content-Version header. In the second case, an API request would use_ ?version=1.0.0 _as a query parameter in the URL._

The next part of the API’s URL is the path, often called the _endpoint_. It specifies the resource we want to query (for example, the weather API). API specifications usually mention only the endpoint itself, such as _/v1/weather_, leaving the root entry point implied. The URL generally also accepts parameters. These can be path parameters that are part of the URL, like in our ZIP code API endpoint, _/v1/weather/{zipcode}_, or they can be query parameters, which are added as encoded key-value pairs after an initial question mark, as in _/v1/weather?zipcode_=<_zipcode_>. By convention, path parameters are usually used to refer to a resource or resources, and query parameters are used to perform operations on the data returned, like sorting or filtering.

#### The Specification

The resources themselves are separate from the representations returned to the client. In other words, the server might send data in formats like HTML, XML, JSON, or others, regardless of the way in which the data is stored in the application’s database. You can learn about an API’s response format in its _specification_, which serves as the manual for an API. One excellent way to document your APIs is with the OpenAPI format, which is widely used in the industry and is part of the Linux Foundation. You can use the Swagger graphical editor at [_https://editor.swagger.io_](https://editor.swagger.io/) to experiment with it.

For example, [Listing 6-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-1) shows a specification for the _/v1/weather/ {zipcode}_ endpoint, written as JSON. Paste the code into the Swagger editor to explore the generated documentation in a more user-friendly manner.

```
{
    "openapi": "3.0.0",
    "info": {
        "title": "Sample Next.js - OpenAPI 3.x",
        "description": "The example APIs from our Next.js application",
        "version": "1.0.0"
    },
    "servers": [
        {"url": "https://www.usemodernfullstack.dev/api/"},
        {"url": "http://localhost:3000/api/"}
    ],
    "paths": {
        "/v1/weather/{zipcode}": {
            "get": {
                "summary": "Get weather by zip code",
                "parameters": [
                    {
                        "name": "zipcode",
                        "in": "path",
                        "description": "The zip code for the location as string.",
                        "required": true,
                        "schema": {
                          "type": "string",
                          "example": 96815
                        }
                    }
                ],
                "responses": {
                    "200": {
                        "description": "Successful operation",
                        "content": {
                            "application/json": {
                                "schema": {
                                    "$ref": "#/components/schemas/weatherDetailType"
                                }
                              }
                        }
                    }
                }
            }
          }
    },
    "components": {
        "schemas": {
            "weatherDetailType": {
                "type": "object",
                "properties": {
                    "zipcode": {
                        "type": "string",
                        "example": 96815
                    },
                    "weather": {
                        "type": "string",
                        "example": "sunny"
                    },
                    "temp": {
                        "type": "integer",
                        "format": "int64",
                        "example": 35
                    }
                }
            }
        }
    }
}
```

Listing 6-1: The OpenAPI specification for the /v1/weather/{zipcode} endpoint

First we define general information, such as the API’s title and description. The most important value here is the API version. In [Exercise 6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Exe6) on page 108, we’ll adjust our server to reflect this version. The next property we set is the server, or the root entry point of the API. We use localhost here, because our Next.js application runs locally for now.

Then we specify the unique API endpoints under paths, setting the path, parameters, and responses for each of them. In this example, we specify the minimum required data for one endpoint, the _/v1/weather/{zipcode}_, and clarify that it uses the GET method. The curly brackets ({}) indicate the URL parameter, but we also set the parameter with the name zipcode explicitly in the path. In addition, we define the _schema_, or format, for the parameter, which should be a string.

Next, in the responses section, we set the response that the API should return if the HTTP status code is _200: OK_. This content, in the application/json format, is weatherDetailType, which you should already be familiar with from previous chapters. It’s similar to the custom type definition in our _custom.d.ts_ file, except here we use JSON instead of TypeScript.

Note that the Swagger editor also generates an interactive playground based on the specification that lets us test the API’s endpoints against a running server. In addition, we can generate a server and client directly in the editor’s interface. The generated server will provide the REST API described in the specification, whereas the client will generate a library we can use in any application that consumes the API from the spec. This interactive playground and generated code make working with third-party APIs very easy.

#### State and Authentication

RESTful APIs are _stateless_, meaning they don’t store session information on the server. _Session information_ is any data about previous user interactions. For example, imagine an online store’s shopping cart. In a stateful design, the application would store the content of your cart on the server and update it whenever you add new items. In a RESTful design, the client instead sends all relevant session data in each request. User–server interactions are understood in isolation, without context from previous requests.

Even so, public RESTful APIs often require some form of authentication. In order to distinguish the requests of authenticated users from the requests of unauthenticated users, those APIs typically provide a token that users should include in subsequent requests. Consumers send this token as part of the request’s data or in the HTTP Authorization header. We’ll provide more details about authorization tokens and how they work in [Chapter 9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter9.xhtml).

This stateless design means that the authentication works regardless of whether the client requests the data from the end server directly, a proxy, or a load balancer. Therefore, a RESTful API is capable of handling a layered system. Stateless architectures are also ideal in high-volume situations, because they remove the server load caused by the retrieval of session information from a database.

#### HTTP Methods

In REST, there are four standard ways to interact with a dataset: create, read, update, and delete. These interactions are commonly called _CRUD_ operations. REST APIs use the following HTTP methods to perform these operations on the request’s resource:

**GET**  Used to retrieve data from a resource. It’s the most common request; each time you visit a website in your browser, you make a GET request to the website’s address.

**POST**  Used to add a new element to a collection resource. Sending the same POST request multiple times creates a new element for each request, resulting in multiple elements with the same content. When you send an email or submit a web form, your client is usually sending a POST request behind the scenes, because you’re creating a new resource in a database.

**PUT**  Used to overwrite or update an existing resource. Sending the same PUT request multiple times creates or overwrites a single element with updated content. For example, when you re-upload a picture on Instagram or Facebook, you might send a PUT request.

**PATCH**  Used to partially update an existing resource. Unlike with PUT, you’re sending only the data that differs from the current dataset. Hence, it’s a smaller and more performant operation. For example, an update to your profile on a social media page might be done with a PATCH request.

**DELETE**  Used to delete a resource (for example, to remove a picture on Instagram).

REST API requests suffer from the same performance implications as do all HTTP requests. Developers must consider critical factors such as network bandwidth, latency, and server load. While the application usually can’t influence the network latency or user bandwidth, it can increase its performance by caching the requests and responding with the previously cached results.

In general, the recommended approach is to cache requests aggressively. By avoiding additional server requests, we can speed up our application significantly. Unfortunately, not all HTTP requests are cacheable. The responses for GET requests are cacheable by default, but PUT and DELETE answers aren’t cacheable at all, because they don’t guarantee a predictable response. Between two similar PUT requests, a DELETE request might have deleted the resource, or vice versa. POST and PATCH request responses can, in theory, be cached if the response provides an Expire header or a Cache-Control header and your subsequent calls are GET requests to the same resource. Still, servers frequently won’t cache those two types.

### Working with REST

Let’s practice working with REST by taking a look at our fictional weather service. Say we read the API contract and see that an authorized user can receive and update datasets from the service by using its public REST API. The API returns JSON data, the server’s URL is _https://www.usemodernfullstack.dev_, and the endpoint at _/api/v2/weather/{zipcode}_ accepts GET and PUT requests. In this section, we walk through the requests and responses for getting the current weather data for a specific ZIP code with a GET request to the API, as well as for updating the stored weather data with a PUT request.

#### Reading Data

To receive the weather for your location, you might make a GET request containing the ZIP code 96815 and an authorization token. We can make such a GET request with a command line tool like cURL, which should be part of your system. If necessary, you can install it from [_https://curl.se_](https://curl.se/). A typical cURL request looks like this:

```
$ curl -i url
```

The -i flag displays the header details we are interested in. We can set the HTTP method with the -X flag and send an additional header with the -H flag. Use the escape character to send a multiline command (\ on macOS and ^ on Windows). Avoid adding a space character behind the escape character. If you’re curious, try using cURL to query one of the API endpoints in the app you created in [Exercise 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml#Exe5) on page 89. A cURL call for a GET request to the weather API _v2/weather/{zipcode}_ at _https://www.usemodernfullstack.dev/api_ would look like this:

```
$ curl -i \
    -X GET \
    -H "Accept: application/json" \
    -H "Authorization: Bearer 83dedad0728baaef3ad3f50bd05ed030" \
    https://www.usemodernfullstack.dev/api/v2/weather/96815
```

We make this request to the API endpoint _v2/weather/{zipcode}_ on the server at _https://www.usemodernfullstack.dev/api._ The ZIP code is included in the URL. We set the return format to JSON in the Accept header and pass an access token in the Authorization header. Because this is an example API, it accepts any token; if one is not supplied, the API returns a status code of _401_.

Here is what the API’s response to our GET request looks like:

```
HTTP/2 200
content-type: application/json ; charset=utf-8
access-control-allow-origin: *

{"weather":"sunny","tempC":"25","tempF":"77","friends":["96814","96826"]}
```

The API responds with an HTTP status code of _200_, indicating that the request was successful. We asked for a JSON response, and the content-type header confirms that the response data is indeed of that type.

The Access-Control-Allow-Origin header, which we discussed in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml), here allows access to any domain. With this setting, a browser whose client-side JavaScript wants to access the API will allow these requests regardless of the website’s domain. Without the CORS header, the browser would block the request and the script’s access to the response and instead throw a CORS error.

Finally, we see that the response’s body contains a JSON string with the API response.

#### Updating Data

Now imagine that you want to add display data from your neighborhood (96814) and the adjacent one (96826) to your website. Unfortunately, these ZIP codes aren’t yet available on the API. Luckily, because it’s open source, we can hook up our own weather station and extend the system. Say we’ve set up our weather sensors and connected them to the API. As soon as the weather changes, we add the dataset to it.

Here is the PUT request we make to update the weather for the ZIP code 96814. PUT requests store data in the request body; therefore, we use the -d flag in the cURL command to send the encoded JSON:

```
$ curl -i \
    -X PUT \
    -H "Accept: application/json" \
    -H "Authorization: Bearer 83dedad0728baaef3ad3f50bd05ed030" \
    -H "Content-Type: application/json" \
    -d "{\"weather\":\"sunny\",\"tempC\":\"20\",\"tempF\":\"68\",
        \"friends\":\"['96815','96826']\"}" \
    https://www.usemodernfullstack.dev/api/v2/weather/96815
```

We request the same API endpoint, _/api/v2/weather/_, but replace the GET method with PUT, because we don’t want to get the data from the database; instead, we want to add data. We use the Content-Type header to tell the API provider that the payload in the request body is a JSON string. The API updates the dataset and responds with a status code of _200_ and a JSON object with additional status details:

```
HTTP/2 200
content-type: application/json ; charset=utf-8
access-control-allow-origin: *
{"status":"ok"}
```

You can learn more about RESTful APIs at [_https://restfulapi.net_](https://restfulapi.net/), which covers more specific topics, such as compression and security models, and guides you through designing your own RESTful APIs. Now let’s turn our attention to GraphQL, a different, more advanced type of API.

### GraphQL APIs

Unlike REST, GraphQL isn’t merely an architectural pattern. It’s a complete, open source data query and manipulation language for APIs. It’s also the most popular REST alternative in full-stack web development, used by Airbnb, GitHub, PayPal, and many other companies. In fact, 10 percent of the top 10,000 sites reportedly use GraphQL. This section covers only certain of its features but should give you a solid understanding of GraphQL principles.

> NOTE

_Despite its name, GraphQL doesn’t require the use of a graph database like Neo4j. We can use it to query any data source connected to the GraphQL server, including common databases such as MySQL and MongoDB._

Like REST, GraphQL APIs operate over HTTP. However, a GraphQL implementation exposes only a single API endpoint, typically called _/graphql_, for accessing all resources and performing all CRUD operations. By contrast, REST has one dedicated endpoint per resource.

Another difference is that we connect to the GraphQL server by using POST requests exclusively. Rather than using HTTP methods to define a desired CRUD operation, we use queries and mutations in the POST request body. _Queries_ are read operations, while _mutations_ are operations for creating, updating, and deleting data.

And unlike REST, which relies on standard HTTP status codes, GraphQL returns _500_, that is, an _Internal Server Error_, when an operation cannot be executed at all. Otherwise, the response uses _200_ even if there are problems with the queries or mutations. The reason for this is that the resolver might have partially executed before encountering an issue. Keep this in mind when deploying a GraphQL API in production. Many standard operational practices and tools may need to change to account for this behavior.

#### The Schema

A GraphQL API defines the available queries and mutations in its schema, which is equivalent to the REST API’s specification. Also called a _typedef_, the schema is written in the Schema Definition Language (SDL). SDL’s core elements are _types_, which are objects that contain typed _fields_ defining their properties, and optional _directives_ that add additional information, for example, to specify caching rules for queries or mark fields as deprecated.

[Listing 6-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-2) shows a GraphQL schema for our fictional weather API, which returns the weather data for a location.

```
export const typeDefs = gql`

    type LocationWeatherType {
        zip: String!
        weather: String!
        tempC: String!
        tempF: String!
        friends: [String]!
    }

    input LocationWeatherInput {
        zip: String!
        weather: String
        tempC: String
        tempF: String
        friends: [String]
    }

    type Query {
        weather(zip: String): [LocationWeatherType]!
    }

    type Mutation {
        weather(data: LocationWeatherInput): [LocationWeatherType]!
    }
`;
```

Listing 6-2: The GraphQL schema for the weather API

You should notice that the schema is a tagged template literal, which you learned about in [Chapter 2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml). We begin by describing custom GraphQL object types. These object types represent the data that the API returns. They are similar to the custom types we defined in TypeScript. A type has a name and can implement an interface. Each of these custom object types contains fields, which have a name and a type. GraphQL has the built-in scalar types Int, Float, String, Boolean, and ID. Exclamation marks (!) denote non-nullable fields, and lists within square brackets ([]) indicate arrays.

The first custom object type is LocationWeatherType, which describes the location information for a weather query. Here we use the String! expression to mark the ZIP field as non-nullable. Hence, the GraphQL service always returns a value for this field. We define a friends field as an array of strings to represent related weather stations by ZIP code. It is also non-nullable, so it will always contain an array (with zero or more items) when added to the return values. The String inside the friends array ensures that every item will be a string.

Then we define the input type object for our first mutation. These types are necessary for mutations, and they represent the input received from the API’s consumer. Because consumers should be able to pass in only the fields they’d like to update, we omit the exclamation marks to make the fields optional. In GraphQL, we need to define input objects and types for the return value separately, with the built-in types. Unlike in TypeScript, we can’t use generic custom types.

The schema also defines the query and mutation functions. These are the operations that consumers can send to the API. The weather query takes a ZIP code as a parameter and always returns an array of WeatherLocationType objects. The weather mutation takes a WeatherLocationInput parameter and always returns the modified WeatherLocationType object.

Our schema doesn’t contain any directives, and we won’t go deep into their syntax in this chapter. However, one reason to use directives is for caching. Because GraphQL queries use POST, which isn’t cacheable using the default HTTP cache, we must implement caching manually, on the server side. We can configure caching statically on the type definitions with the directive @cacheControl or dynamically, in the resolver functions, with cacheControl.setCacheHint.

#### The Resolvers

In GraphQL, the _resolvers_ are the functions that implement the schema. Each resolver function maps to a field. Query resolvers implement the reading of data, whereas mutation resolvers implement the creation, updating, and deletion of data. Together they provide complete CRUD functionality.

To understand how resolvers work, you can think of each GraphQL operation as a tree of nested function calls. In such an _abstract syntax tree (AST)_, each part of the operation represents a node. For example, consider a complex, nested GraphQL query, which asks for the location’s current weather, as well as the weather of all its neighbors. Our GraphQL schema for this example looks like [Listing 6-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-3).

```
export const typeDefs = gql`

    type FriendsType {
        zip: String!
        weather: String!
    }

    type LocationWeatherType {
        zip: String!
        weather: String!
        tempC: String!
        tempF: String!
        friends: [FriendsType]!
    }

    type Query {
        weather(zip: String): [LocationWeatherType]!
    }
`;
```

Listing 6-3: The GraphQL schema for the nested GraphQL query example

In the schema for the example, we replace the content of the friends array. Instead of a simple string, we want it to contain an object with a ZIP code and weather information. Therefore, we define a new FriendsType and use this type for the array items.

[Listing 6-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-4) defines the complex example query.

```
query GetWeatherWithFriends {
    weather(zip: "96815") {
        weather
        friends {
            weather
        }
    }
}
```

Listing 6-4: The nested GraphQL query

This query takes the zip parameter 96815 and then returns its weather property, as well as all of its friends’ weather properties, as strings. But how does the query work under the hood?

[Figure 6-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#fig6-1) shows the resolver chain and corresponding AST. The GraphQL server would first parse the query into this structure and then validate the AST against the type-definition schema to ensure that the query can be executed without running into logical problems. Finally, the server would execute the query.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure6-1.jpg)

Figure 6-1: Querying the GraphQL AST

Let’s examine the resolver chain for the query. The Query.weather function takes one argument, the ZIP code, and returns the location object for this ZIP code. Then the server continues along each branch separately. For the weather in the query, it returns the weather property of the location object, Location.weather, at which point the branch ends. The second part of the query, which asks for all friends from the location object and their weather properties, runs the Location.Friends query and then returns the Friends.weather property for each result. The resolver object of each step contains the result returned by the parent field’s resolver.

Let’s return to our weather schema and define the resolvers. We’ll keep these simple. In [Listing 6-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-5), you can see that their names match those defined in the schema.

```
export const resolvers = {
    Query: {
        weather: async (_: any, param: WeatherInterface) => {
            return [
                {
                    zip: param.zip,
                    weather: "sunny",
                    tempC: "25C",
                    tempF: "70F",
                    friends: []
                }
            ];
        },
    },
    Mutation: {
        weather: async (_: any, param: {data: WeatherInterface}) => {
            return [
                {
                    zip: param.data.zip,
                    weather: "sunny",
                    tempC: "25C",
                    tempF: "70F",
                    friends: []
                }
            ];
        }
    },
};
```

Listing 6-5: The GraphQL resolvers for the weather API

We first define async functions for the query and mutation properties and assign the object to the const resolvers. Each takes two parameters. The first one represents the previous resolver object in the resolver chain. We aren’t using a nested or complex query; hence, here it is always undefined. For this reason, we use the any type to avoid a TypeScript error and use the underscore (_) convention you learned in [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml) to mark it as unused. The second parameter is an object containing the data passed to the function on invocation. For the weather query and the weather mutation, it is an object that implements the WeatherInterface.

For now, both functions ignore this parameter for the most part, using only the zip property to reflect the input. Also, they return a static JSON object similar to the REST API we created in the previous listings. The static data is just a placeholder, which we’ll replace with the result from our database queries later. The response honors the API contract we defined in the GraphQL schema, as this data consists of arrays with weather location datasets.

### Comparing GraphQL to REST

We’ve already implemented RESTful APIs in our Next.js application, and as you saw in this chapter, REST is fairly simple to work with. You might be wondering why you’d even consider using GraphQL. Well, GraphQL solves two problems common in REST APIs: over-fetching and under-fetching.

#### Over-Fetching

When a client queries a REST endpoint, the API always returns the complete dataset for that endpoint. This means that, often, the API delivers more data than necessary, a common performance problem called _over-fetching_. For example, our example RESTful weather API at _/api/v2/weather/zip/96815_ delivers all weather data for a ZIP code even if all you need is the temperature in Celsius. You’d then need to manually filter the results. In GraphQL, the API requests explicitly define the data they want returned.

Let’s look at an example to see how GraphQL lets us keep the API response data to a minimum. The following GraphQL query returns only the temperature in Celsius for the location with the ZIP code 96815:

```
query Weather {
    weather(zip: "96815") {
        tempC
    }
}
```

In GraphQL, we send the query as a JSON string with the POST request’s data:

```
$ curl -i \
    -X POST \
    -H "Accept: application/json" \
    -H "Authorization: Bearer 83dedad0728baaef3ad3f50bd05ed030" \
    -H "Content-Type: application/json" \
    -d '{"query":"\nquery Weather  {\n  weather(zip: \"96815\") {\n    tempC  \n  }\n}"}' \
    https://www.usemodernfullstack.dev/api/graphql
```

We consume the API with a POST request to the _/api/graphql_ endpoint, then set the Content-Type header and Accept header to JSON to explicitly tell the API that we’re sending a JSON object in the request body and expect a JSON response. We set the access token in the Authorization header, as in a RESTful request. The POST body contains the query for the weather data, and the \n control characters indicate the newlines in the GraphQL query. As defined in the contract, the query expects a parameter, zip, for which we pass in the ZIP code 96815. In addition, we request that the API return only the tempC field of the weather node.

Here is the response from the GraphQL API:

```
HTTP/2 200
content-type: application/json ; charset=utf-8
access-control-allow-origin: *

{"data":{"weather":[{"tempC":"25C"}]}}
```

The API responds with a status code of _200_. We specified in the request’s query that we are interested in only the requested field tempC of the weather object, so this is what we received. The API doesn’t return the ZIP code, temperature in Fahrenheit, weather string, or friends array.

#### Under-Fetching

On the other hand, a REST dataset might not contain all the data you need, requiring you to send follow-up requests. This problem is called _under-fetching_. Imagine that your friends also have weather stations and that you want to get the current weather at their ZIP codes. The RESTful weather API returns an array with related ZIP codes (friends). However, you’d need to make additional requests for each ZIP code to receive their weather information, potentially causing performance issues.

GraphQL treats datasets as nodes in a graph, with relationships between them. Therefore, extending a single query to receive related data is pretty simple. Our example GraphQL server’s resolvers are set up to fetch additional data about friends if the request’s query contains the friends field. We define the GraphQL query as follows:

```
query Weather {
    weather(zip: "96815") {
        tempC
        friends {
            tempC
        }
    }
}
```

The following shows an example request that fetches all related nodes through the friends array. Again, we define the return data and query the friends only for the field tempC:

```
$ curl -i \
    -X POST \
    -H "Accept: application/json" \
    -H "Authorization: Bearer 83dedad0728baaef3ad3f50bd05ed030" \
    -H "Content-Type: application/json" \
    -d '{"query":"query Weather  {\n  weather(zip: \"96815\")
        {\n    tempC\n    friends {\n      tempC\n    }\n  }\n}"}' \
    https://www.usemodernfullstack.dev/api/graphql
```

The POST body contains the query for weather data pertaining to the 96815 ZIP code in one line and asks for the tempC field, as in the previous request. To extend the query, we add a sub-selection on the friends field. Now GraphQL traverses the related nodes and their fields and returns the tempC field of the nodes whose ZIP codes match the ones in the 96815 node’s friends array.

Here is the response from the GraphQL server. We see that it contains data from the related nodes:

```
HTTP/2 200
content-type: application/json ; charset=utf-8
access-control-allow-origin: *

{"data":{"weather":[{"tempC":"25C","friends":
[{"tempC":"20C"},{"tempC":"30C"}]}]}}
```

As you’ve discovered, GraphQL lets us easily extend queries by adjusting the data in the request.

Exercise 6: Add a GraphQL API to Next.js

Let’s rework our weather application’s API to use GraphQL. To do so, we must first add GraphQL to the project. GraphQL isn’t a pattern but an environment that consists of a server and a query language, both of which we must add to Next.js.

We’ll install the stand-alone Apollo server, one of the most popular GraphQL servers, which also provides a Next.js integration. Open your terminal and navigate to the refactored application you built in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml). In the directory’s top level, next to the _package.json_ file, execute this command:

```
$ npm install @apollo/server @as-integrations/next graphql graphql-tag
```

This command also installs the GraphQL language and the GraphQL tag modules we’ll need.

#### Creating the Schema

As we discussed, every GraphQL API starts with a schema definition. Create a folder called _graphql_ next to the _pages_ folder in the Next.js directory. This is where we’ll add all GraphQL-related files.

Now create a file called _schema.ts_ and paste the code you wrote back in [Listing 6-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-2). We’ve already defined and discussed the type definition used here. Simply add one line to the top of the file:

```
import gql from "graphql-tag";
```

This line imports the qql tagged template literal we use to define the schema.

#### Adding Data

We want our API to return different data depending on the parameters and properties of the queries sent to it. Therefore, we need to add datasets to our project. GraphQL can query any database, even static JSON data. So let’s implement a JSON dataset. Create the file _data.ts_ inside the _graphql_ directory and add the code from [Listing 6-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-6).

```
export const db = [
    {
        zip: "96815",
        weather: "sunny",
        tempC: "25C",
        tempF: "70F",
        friends: ["96814", "96826"]
    },
    {
        zip: "96826",
        weather: "sunny",
        tempC: "30C",
        tempF: "86F",
        friends: ["96814", "96814"]
    },
    {
        zip: "96814",
        weather: "sunny",
        tempC: "20C",
        tempF: "68F",
        friends: ["96815", "96826"]
    }
];
```

Listing 6-6: The graphql/data.ts file for the GraphQL API

This JSON defines three weather locations and their properties. A consumer will be able to query our API for these datasets.

#### Implementing Resolvers

Now we can define our resolvers. Add the file _resolvers.ts_ to the _graphql_ directory and paste in the code from [Listing 6-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-7). This is similar to the code we previously discussed when we introduced resolvers, but instead of returning the same static JSON object to the consumer, we query our new dataset.

```
import {db} from "./data";

declare interface WeatherInterface {
    zip: string;
    weather: string;
    tempC: string;
    tempF: string;
    friends: string[];
}

export const resolvers = {
    Query: {
        weather: async (_: any, param: WeatherInterface) => {
            return [db.find((item) => item.zip === param.zip)];
        }
    },
    Mutation: {
        weather: async (_: any, param: {data: WeatherInterface}) => {
            return [db.find((item) => item.zip === param.data.zip)];
        }
    }
};
```

Listing 6-7: The graphql/resolvers.ts file for the GraphQL API

We import the array of JSON objects we created earlier and define an interface for the resolvers. The query resolver finds an object by using the ZIP code passed to it and returns it to the Apollo server. The mutation does the same, except that the parameter structure is slightly different: it is accessible through the data property. Alas, we can’t actually change the data by using the mutation, as the data is a static JSON file. We’ve implemented the mutation here for illustration purposes only.

#### Creating the API Route

The Apollo GraphQL server exposes one endpoint, _graphql/_, which we’ll implement now. Create a new file, _graphql.ts_, in the _api_ folder and add the code from [Listing 6-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#Lis6-8). This code initializes the GraphQL server and adds a CORS header so that we can access the API from different domains and use the built-in GraphQL sandbox explorer to play with GraphQL later. You saw this header in the previous cURL responses.

```
import {ApolloServer} from "@apollo/server";
import {startServerAndCreateNextHandler} from "@as-integrations/next";
import {resolvers} from "../../graphql/resolvers";
import {typeDefs} from "../../graphql/schema";
import {NextApiHandler, NextApiRequest, NextApiResponse} from "next";

//@ts-ignore
const server = new ApolloServer({
    resolvers,
    typeDefs
});

const handler = startServerAndCreateNextHandler(server);

const allowCors =
    (fn: NextApiHandler) => async (req: NextApiRequest, res: NextApiResponse) => {
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

export default allowCors(handler);
```

Listing 6-8: The api/graphql.ts file, which creates the API entry point for GraphQL

This code is all we need to create the GraphQL entry point. First we import the necessary modules, including our GraphQL schema and the resolvers, both of which we created previously. Then we initialize a new GraphQL server with typedefs and resolvers.

We start the server and continue by creating the API handler. To do this, we use the Next.js integration helper to start the server and return the Next.js handler. The integration helper connects the serverless Apollo instance to the Next.js custom server. Before we define the default export as an async function that takes the API’s request and response objects as parameters, we create a wrapper to add the CORS headers to the request. The first block inside the function sets up the CORS headers, and we limit the allowed request to POST requests. We need the CORS headers here to make our GraphQL API publicly available. Otherwise, we wouldn’t be able to connect to the API from a website running on a different domain or even use the server’s built-in GraphQL sandbox.

Part of the CORS setup here is that we immediately return _200_ for any OPTIONS requests. The CORS patterns use OPTIONS requests as preflight checks. Here the browser requests only headers, and then checks the response’s CORS headers to verify that the domain from which it calls the API is allowed to access the resource before making the actual request.

However, our Apollo server allows only POST and GET requests and would return _405: Method Not Allowed_ for the preflight OPTIONS request. So, instead of passing this request to the Apollo server, we end the request and return _200_ with the previous CORS headers. The browser should then proceed with the CORS pattern. Finally, we start the server and create the API handler on the desired path, _api/graphql_.

#### Using the Apollo Sandbox

Start your Next.js server with npm run dev. You should see the Next.js application running on _http://localhost:3000_. If you navigate to the GraphQL API at _http://localhost:3000/api/graphql_, you’ll find the Apollo sandbox interface for querying the API, as in [Figure 6-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#fig6-2).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure6-2.jpg)

Figure 6-2: The Apollo sandbox’s API querying interface

In the Documentation pane on the left side, we see the available queries as fields of the query object we defined earlier. As expected, we see the Weather query here, and when we click it, a new query appears in the Operation pane in the middle. At the same time, the interface changes, and we see the available arguments and fields. Clicking each provides more information. Using the plus (+) button, we can add fields to the Query pane and run them against the data.

Try creating a Weather query that returns the zip and weather properties. This query requires a ZIP code as an argument; add it through the user interface on the left-hand side, and then add the ZIP code 96826 as a string to the JSON object in the variables section of the lower pane. Now run the query by clicking the **Weather** button at the top of the Operation pane. You should receive the result for this ZIP code in the Response pane on the right as JSON. Compare your screen with [Figure 6-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter6.xhtml#fig6-3).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure6-3.jpg)

Figure 6-3: The GraphQL query and response from the server

Play around with crafting queries, accessing properties, and creating errors with invalid arguments to get a feel for GraphQL before moving on to the next chapter.

### Summary

This chapter explored RESTful and GraphQL web APIs and their role in full-stack development. Although we used a REST design in previous chapters, you should now be familiar with the concept of stateless servers, as well as the five HTTP methods for performing CRUD operations in REST. You also practiced working with a public REST API to read and update data, then evaluated its requests and responses.

GraphQL APIs require a bit more work to implement, but they reduce the over-fetching and under-fetching issues often experienced in REST. You learned to define the API contract with a schema and implement its functionality with resolvers. Then you queried an API and defined the dataset to return in the request.

Finally, you added a GraphQL API to your existing Next.js application by adding the Apollo server to it. You should now be able to create your own GraphQL API and consume third-party resources. To learn more about GraphQL, I recommend the tutorials at [_https://www.howtographql.com_](https://www.howtographql.com/) and the official GraphQL introduction at [_https://graphql.org/learn/_](https://graphql.org/learn/).

In the next chapter, you’ll explore the MongoDB database and Mongoose, an object data modeling library, for storing data.

Copycopy

Copy

Highlighthighlight

Add NotenoteGet Linklink

Get Link