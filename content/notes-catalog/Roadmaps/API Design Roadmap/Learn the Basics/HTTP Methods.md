---
id: 01J8J8D3GTQ36K4DX74QNWBGTR
title: HTTP Methods
modified: 2024-09-24T10:58:02-04:00
description: The different HTTP methods
tags:
  - http
  - api
  - roadmaps
  - programming
---
- HTTP (Hypertext Transfer Protocol) Methods play a significant role in API design. They define the type of request a client can make to a server, providing the framework for interaction between client and server. Understanding HTTP methods is paramount to creating a robust and effective API. Some of the common HTTP methods used in API design include GET, POST, PUT, DELETE, and PATCH. Each of these methods signifies a different type of request, allowing for various interactions with your API endpoints. This in turn creates a more dynamic, functional, and user-friendly API.

# HTTP request methods

HTTP defines a set of **request methods** to indicate the purpose of the request and what is expected if the request is successful. Although they can also be nouns, these request methods are sometimes referred to as _HTTP verbs_. Each request method has it's own semantics, but some characteristics are shared across multiple methods, specifically request methods can be [safe](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP), [idempotent](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent), or [cacheable](https://developer.mozilla.org/en-US/docs/Glossary/Cacheable).

[`GET`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)

The `GET` method requests a representation of the specified resource. Requests using `GET` should only retrieve data and should not contain a request [content](https://developer.mozilla.org/en-US/docs/Glossary/HTTP_Content).

[`HEAD`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/HEAD)

The `HEAD` method asks for a response identical to a `GET` request, but without a response body.

[`POST`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)

The `POST` method submits an entity to the specified resource, often causing a change in state or side effects on the server.

[`PUT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT)

The `PUT` method replaces all current representations of the target resource with the request [content](https://developer.mozilla.org/en-US/docs/Glossary/HTTP_Content).

[`DELETE`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/DELETE)

The `DELETE` method deletes the specified resource.

[`CONNECT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT)

The `CONNECT` method establishes a tunnel to the server identified by the target resource.

[`OPTIONS`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS)

The `OPTIONS` method describes the communication options for the target resource.

[`TRACE`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/TRACE)

The `TRACE` method performs a message loop-back test along the path to the target resource.

[`PATCH`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH)

The `PATCH` method applies partial modifications to a resource.

## [Safe, idempotent, and cacheable request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods#safe_idempotent_and_cacheable_request_methods)

The following table lists HTTP request methods and their categorization in terms of safety, cacheability, and idempotency.

|Method|Safe|Idempotent|Cacheable|
|---|---|---|---|
|[`GET`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)|Yes|Yes|Yes|
|[`HEAD`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/HEAD)|Yes|Yes|Yes|
|[`OPTIONS`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS)|Yes|Yes|No|
|[`TRACE`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/TRACE)|Yes|Yes|No|
|[`PUT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT)|No|Yes|No|
|[`DELETE`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/DELETE)|No|Yes|No|
|[`POST`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)|No|No|Conditional*|
|[`PATCH`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH)|No|No|Conditional*|
|[`CONNECT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT)|No|No|No|

* `POST` and `PATCH` are cacheable when responses explicitly include [freshness](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) information and a matching [`Content-Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Location) header.

## [Specifications](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods#specifications)

| Specification                                                                  |
| ------------------------------------------------------------------------------ |
| [HTTP Semantics  <br># CONNECT](https://httpwg.org/specs/rfc9110.html#CONNECT) |
| [HTTP Semantics  <br># DELETE](https://httpwg.org/specs/rfc9110.html#DELETE)   |
| [HTTP Semantics  <br># GET](https://httpwg.org/specs/rfc9110.html#GET)         |
| [HTTP Semantics  <br># HEAD](https://httpwg.org/specs/rfc9110.html#HEAD)       |
| [HTTP Semantics  <br># OPTIONS](https://httpwg.org/specs/rfc9110.html#OPTIONS) |
| [HTTP Semantics  <br># POST](https://httpwg.org/specs/rfc9110.html#POST)       |
| [HTTP Semantics  <br># PUT](https://httpwg.org/specs/rfc9110.html#PUT)         |
# W3 Schools HTTP Request Methods
## What is HTTP?

The Hypertext Transfer Protocol (HTTP) is designed to enable communications between clients and servers.

HTTP works as a request-response protocol between a client and server.

Example: A client (browser) sends an HTTP request to the server; then the server returns a response to the client. The response contains status information about the request and may also contain the requested content.

---

## HTTP Methods

- **GET**
- **POST**
- **PUT**
- **HEAD**
- **DELETE**
- **PATCH**
- **OPTIONS**
- **CONNECT**
- **TRACE**

The two most common HTTP methods are: GET and POST.

---

## The GET Method

GET is used to request data from a specified resource.

Note that the query string (name/value pairs) is sent in the URL of a GET request:

/test/demo_form.php?name1=value1&name2=value2

**Some notes on GET requests:**

- GET requests can be cached
- GET requests remain in the browser history
- GET requests can be bookmarked
- GET requests should never be used when dealing with sensitive data
- GET requests have length restrictions
- GET requests are only used to request data (not modify)

---

## The POST Method

POST is used to send data to a server to create/update a resource.

The data sent to the server with POST is stored in the request body of the HTTP request:

POST /test/demo_form.php HTTP/1.1  
Host: w3schools.com  
  
name1=value1&name2=value2

**Some notes on POST requests:**

- POST requests are never cached
- POST requests do not remain in the browser history
- POST requests cannot be bookmarked
- POST requests have no restrictions on data length

---

## Compare GET vs. POST

The following table compares the two HTTP methods: GET and POST.

||GET|POST|
|---|---|---|
|BACK button/Reload|Harmless|Data will be re-submitted (the browser should alert the user that the data are about to be re-submitted)|
|Bookmarked|Can be bookmarked|Cannot be bookmarked|
|Cached|Can be cached|Not cached|
|Encoding type|application/x-www-form-urlencoded|application/x-www-form-urlencoded or multipart/form-data. Use multipart encoding for binary data|
|History|Parameters remain in browser history|Parameters are not saved in browser history|
|Restrictions on data length|Yes, when sending data, the GET method adds the data to the URL; and the length of a URL is limited (maximum URL length is 2048 characters)|No restrictions|
|Restrictions on data type|Only ASCII characters allowed|No restrictions. Binary data is also allowed|
|Security|GET is less secure compared to POST because data sent is part of the URL  <br>  <br>Never use GET when sending passwords or other sensitive information!|POST is a little safer than GET because the parameters are not stored in browser history or in web server logs|
|Visibility|Data is visible to everyone in the URL|Data is not displayed in the URL|
# Resources
- [HTTP request methods - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [HTTP Request Methods W3 Schools](https://www.w3schools.com/tags/ref_httpmethods.asp)
- [What are HTTP Methods? | Postman Blog](https://blog.postman.com/what-are-http-methods/)