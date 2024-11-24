---
id: 01JCH9DPMZ23AXGNEKQDJJ1S4F
modified: 2024-11-12T17:55:43-05:00
title: Chapter 8 - Writing HTTP Clients
---
# 8  
WRITING HTTP CLIENTS

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

The _HyperText Transfer Protocol__(HTTP)_ is an application layer protocol used by the World Wide Web. In an HTTP communication, a web client sends a _uniform resource locator__(URL)_ to a web server, and the web server responds with the corresponding media resources. In this context, a _resource_ could be an image, a style sheet, an HTML document, a JavaScript file, and so on. For example, if your web browser sent the URL _www.google.com_ to Google’s web servers, the servers would return Google’s main page. Most of us make such web transactions daily, whether they originate from our phones, computers, or Internet of Things (IoT) devices, such as doorbells, thermostats, or toasters (yes, really).

This chapter will introduce you to Go’s HTTP client. First, you’ll learn the basics of HTTP, including request methods and response codes. Next, you’ll explore Go’s HTTP client to request resources from web servers, paying attention to potential pitfalls along the way. Then, you’ll move into the standard library code and learn the implementations that facilitate the request-response communication between an HTTP client and server. Finally, you’ll see common pitfalls to look for when interacting with web servers using Go’s HTTP client.

This chapter will give you the basics for interacting with services over HTTP. You’ll need this foundation to understand how to handle requests from the server’s point of view in the next chapter.

## Understanding the Basics of HTTP

HTTP is a sessionless client-server protocol in which the client initiates a request to the server and the server responds to the client. HTTP is an application layer protocol and serves as the foundation for communication over the web. It uses TCP as its underlying transport layer protocol.

This chapter assumes that you’re using HTTP version 1.1 (HTTP/1.1). We’ll also cover functionality introduced in HTTP version 2.0 (HTTP/2). Thankfully, Go abstracts many of the differences between these protocols, so we can easily use either protocol with the same bit of code.

### Uniform Resource Locators

A _URL_ is an address of sorts used by the client to locate a web server and identify the requested resource. It’s composed of five parts: a required _scheme_ indicating the protocol to use for the connection, an optional _authority_ for the resource, the _path_ to the resource, an optional _query_, and an optional _fragment_. A colon (:) followed by two forward slashes (//) separates the scheme from the authority. The authority includes an optional colon-delimited username and password suffixed with an at symbol (@), a hostname, and an optional port number preceded by a colon. The path is a series of segments preceded by a forward slash. A question mark (?) indicates the start of the query, which is conventionally composed of key-value pairs separated by an ampersand (&). A hash mark (#) precedes the fragment, which is an identifier to a subsection of the resource. Taken together, a URL follows this pattern:

```
scheme://user:password@host:port/path?key1=value1&key2=value2#table_of_contents
```

The typical URL you use over the internet includes a scheme and a hostname at minimum. For example, if you felt compelled to look up images of gophers, you could visit Google’s image search by entering the following URL in your web browser’s address bar, then searching for _gophers_ in the image search tab:

```
1https://2images.google.com3/
```

The scheme 1 informs your browser that you want to use HTTPS to connect to the address _images.google.com_2 and that you want the default resource 3. If you specify the web server address without any specific resource, the web server will respond with a default resource. Just as it’s helpful for large corporations to send your call to a receptionist when you omit an extension number, it’s helpful for web servers to serve up a default resource if you don’t specify the resource you want. Google receives your request and responds with the image search page. When you type _gophers_ in the search box and submit the form, your browser sends a request using a URL like this, truncated for brevity:

```
https://www.google.com/1search2?3q=gophers&tbm=isch . . .
```

This URL asks Google for a resource named _search_1 and includes a _query string_. The query string, indicated by the question mark 2, contains ampersand-separated parameters defined by, and meaningful to, the web server. In this example, the value of the `q` parameter 3 is your search query, `gophers`. The `tbm` parameter’s value of `isch` tells Google you’re performing an image search. Google defines the parameters and their values. You pass them along to Google’s web servers as part of the request. The actual URL in your browser’s address bar is quite a bit longer and includes other details Google needs in order to satisfy your request.

If my wife were to send me shopping using HTTP, the URL she would give me might look like this:

```
automobile://the.grocery.store/purchase?butter=irish&eggs=12&coffee=dark_roast
```

This tells me I’m to drive my car to the grocery store and pick up Irish butter, a dozen eggs, and dark roast coffee. It’s important to mention that the scheme is relevant only to the context in which it’s used. My web browser wouldn’t know what to do with the _automobile_ scheme, but for the sake of my marriage, I sure do.

### Client Resource Requests

An _HTTP request_ is a message sent from a client to a web server that asks the server to respond with a specific resource. The request consists of a method, a target resource, headers, and a body. The _method_ tells the server what you want it to do with the target resource. For example, the `GET` method followed by _robots.txt_ tells the server you want it to send you the _robots.txt_ file, whereas the `DELETE` method indicates to the server that you want it to delete that resource.

_Request headers_ contain metadata about the content of the request you are sending. The `Content-Length` header, for example, specifies the size of the request body in bytes. The _request body_ is the payload of the request. If you upload a new profile picture to a web server, the request body will contain the image encoded in a format suitable for transport over the network, and the `Content-Length` header’s value will be set to the size in bytes of the image in the request body. Not all request methods require a request body.

[Listing 8-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-1) details a simple `GET` request for Google’s _robots.txt_ file over Netcat to Google’s web server. “Testing the Service with Netcat” on page 159 walks you through installing Netcat.

```
$ nc www.google.com 80
1 GET /robots.txt HTTP/1.1

2 HTTP/1.1 200 OK
3 Accept-Ranges: none
Vary: Accept-Encoding
Content-Type: text/plain
Date: Mon, 02 Jan 2006 15:04:05 MST
Expires: Mon, 02 Jan 2006 15:04:05 MST
Cache-Control: private, max-age=0
Last-Modified: Mon, 02 Jan 2006 15:04:05 MST
X-Content-Type-Options: nosniff
Server: sffe
X-XSS-Protection: 0
Transfer-Encoding: chunked

4 User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
--snip-- 
```

Listing 8-1: Sending a request for Google’s _robots.txt_ file and receiving a response with its contents

The `GET` request 1 tells Google’s web server you want the _/robots.txt_ file using HTTP/1.1. Following the request, you press the ENTER key twice to send the request followed by an empty line. The web server promptly responds with a status line 2, a series of headers 3, an empty line delimiting the headers from the response body, and the contents of the _robots.txt_ file in the response body 4. You’ll learn about server responses a bit later in this chapter.

Using Go’s `net/http` package, you can create a request with nothing but an HTTP method and a URL. The `net/http` package includes constants for the most common RFC 7231 and RFC 5789 request methods. The RFCs contain quite a bit of jargon with respect to request methods. The following descriptions describe how to use these methods in practice:

1. `GET` As in the earlier example, the `GET` method instructs the server to send you the target resource. The server will deliver the target resource in the response’s body. It’s important to note that the target resource does not need to be a file; the response could deliver you dynamically generated content, like the _gophers_ image search result discussed earlier. The server should never change or remove the resource as the result of a `GET` request.
2. `HEAD` The `HEAD` method is like `GET` except it tells the server to exclude the target resource in its response. The server will send only the response code and other various bits of metadata stored in the response headers. You can use this method to retrieve meaningful details about a resource, such as its size, to determine whether you want to retrieve the resource in the first place. (The resource may be larger than you expect.)
3. `POST` A `POST` request is a way for you to upload data included in the request body to a web server. The `POST` method tells the server that you are sending data to associate with the target resource. For example, you may post a new comment to a news story, in which case the news story would be the target resource. In simple terms, think of the `POST` method as the method for creating new resources on the server.
4. `PUT` Like `POST`, you can use a `PUT` request to upload data to a web server. In practice, the `PUT` method usually updates or completely replaces an existing resource. You could use `PUT` to edit the comment you `POST`ed to the news story.
5. `PATCH` The `PATCH` method specifies partial changes to an existing resource, leaving the rest of the resource unmodified. In this way, it’s like a _diff_. Let’s assume you are buying a Gopher Plush for that someone special in your life, and you’ve proceeded past the shipping address step of the checkout process when you realize you made a typo in your street address. You jump back to the shipping address form and correct the typo. Now, you could `POST` the form again and send all its contents to the server. But a `PATCH` request would be more efficient since you made only a single correction. You’ll likely encounter the `PATCH` method in APIs, rather than HTML forms.
6. `DELETE` The `DELETE` method instructs the server to remove the target resource. Let’s say your comment on the news story was too controversial, and now your neighbors avoid making eye contact with you. You can make a `DELETE` request to the server to remove your comment and restore your social status.
7. `OPTIONS` You can ask the server what methods a target resource supports by using the `OPTIONS` method. For example, you could send an `OPTIONS` request with your news story comment as the target resource and learn that the `DELETE` method is not one of the methods the server will support for your comment, meaning your best option now is to find another place to live and meet new neighbors.
8. `CONNECT` The client uses `CONNECT` to request that the web server perform _HTTP tunneling,_ or establish a TCP session with a target destination and proxy data between the client and the destination.
9. `TRACE` The `TRACE` method instructs the web server to echo the request back to you instead of processing it. This method allows you to see whether any intermediate nodes modify your request before it reaches the web server.
    
    ## NOTE
    
    Before adding server-side support for the `TRACE` method, I strongly recommend you read up on its role in cross-site tracing (XST) attacks, whereby an attacker uses a cross-site scripting (XSS) attack to steal authenticated user credentials. The risk of adding a potential attack vector to your web server likely does not outweigh the diagnostic benefits of `TRACE` support.
    

It’s important to mention that web servers are under no obligation to implement these request methods. In addition, you may find that some web servers don’t correctly implement them. Trust, but verify.

### Server Responses

Whereas the client request always specifies a method and a target resource, the web server’s response always includes a status code to inform the client of the status of its request. A successful request results in a response containing a 200-class status code.

If the client makes a request that requires further action on the client’s part, the server will return a 300-class status code. For example, if the client requests a resource that has not changed since the client’s last request for the resource, the server may return a 304 status code to inform the client that it should instead render the resource from its cache.

If an error occurs because of the client’s request, the server will return a 400-class status code in its response. The most common example of this scenario occurs when a client requests a nonexistent target resource, in which case the server responds with a 404 status code to inform the client that it could not find the resource.

The 500-class status codes inform the client that an error has occurred on the server side that prevents the server from fulfilling the request. Let’s assume that your request requires the web server to retrieve assets from an upstream server to satisfy your request, but that the upstream server fails to respond. The web server will respond to you with a 504 status code indicating that a time-out occurred during the communication with its upstream server.

A handful of 100-class status codes exist in HTTP/1.1 to give direction to the client. For example, the client can ask for guidance from the server while sending a `POST` request. To do so, the client would send the `POST` method, target resource, and request headers to the server, one of which tells the server that the client wants permission to proceed sending the request body. The server can respond with a 100 status code indicating that the client can continue the request and send the body.

The IANA maintains the official list of HTTP status codes, which you can find at [https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml). If you encounter a relatively obscure status code, you can typically find a description of it in RFC 7231 at [https://tools.ietf.org/html/rfc7231#section-6](https://tools.ietf.org/html/rfc7231#section-6).

Go defines many of these status codes as constants in its `net/http` package, and I suggest you use the constants in your code. It’s much easier to read `http.StatusOK` than it is to remember what 200 means. The most common HTTP status codes you’ll encounter include the following:

1. `200 OK` Indicates a successful request. If the request method was `GET`, the response body contains the target resource.
2. `201 Created` Returned when the server has successfully processed a request and added a new resource, as may be the case with a `POST` request.
3. `202 Accepted` Often returned if the request was successful but the server hasn’t yet created a new resource. The creation of the resource may still fail despite the successful request.
4. `204 No Content` Often returned if the request was successful but the response body is empty.
5. `304 Not Modified` Returned when a client requests an unchanged resource. The client should instead use its cached copy of the resource. One method of caching is using an entity tag (ETag) header. When a client requests a resource from the server, the response may include an optional server-derived ETag header, which has meaning to the server. If the client requests the same resource in the future, the client can pass along the cached ETag header and value in its request. The server will check the ETag value in the client’s request to determine whether the requested resource has changed. If it is unchanged, the server will likely respond with a 304 status code and an empty response body.
6. `400 Bad Request` Returned if the server outright rejects the client’s request for some reason. This may be due to a malformed request, like one that includes a request method but no target resource.
7. `403 Forbidden` Often returned if the server accepts your request but determines you do not have permission to access the resource, or if the server itself does not have permission to access the requested resource.
8. `404 Not Found` Returned if you request a nonexistent resource. You may also find this status code used as a _Glomar response_ when a server does not want to confirm or deny your permission to access a resource. In other words, a web server may respond with a 404 status code for a resource you do not have permission to access instead of explicitly responding with a 403 status code confirming your lack of permissions to the resource. Attackers attempting to access sensitive resources on your web server would want to focus their efforts only on existing resources, even if they currently lack permissions to access those resources. Returning a 404 status code for both nonexistent and forbidden resources prevents attackers from differentiating between the two, providing a measure of security. The downside to this approach is you’ll have a harder time debugging your permissions on your server, because you won’t know whether the resource you’re requesting exists or you simply lack permissions. I suggest you articulate the difference in your server logs.
9. `405 Method Not Allowed` Returned if you specify a request method for a target resource that the server does not support. Remember the controversial comment you attempted to delete in our discussion of the `OPTIONS` request method? You would receive a 405 status code in response to that `DELETE` request.
10. `426 Upgrade Required` Returned to instruct the client to first upgrade to TLS before requesting the target resource.
11. `500 Internal Server Error` A catchall code of sorts returned when an error occurs on the server that prevents it from satisfying the client’s request but that doesn’t match the criteria of any other status code. Servers have returned many a 500 error because of some sort of configuration error or syntax error in server-side code. If your server returns this code, check your logs.
12. `502 Bad Gateway` Returned when the server proxies data between the client and an upstream service, but the upstream service is unavailable and not accepting requests.
13. `503 Service Unavailable` Returned if a web server is unavailable to accept a request. For example, a web server may return a 503 status code for all incoming connections when it’s put into maintenance mode.
14. `504 Gateway Timeout` Returned by a proxy web server to indicate that the upstream service accepted the request but did not provide a timely reply.

### From Request to Rendered Page

A web page is often composed of various resources, such as images, videos, layout instructions for your web browser, third-party ads, and so on. Accessing each resource requires a separate request to the server. In HTTP version 1.0 (HTTP/1.0), clients must initiate a separate TCP connection for each request. HTTP/1.1 eliminates this requirement, reducing the latency and request-connection overhead associated with multiple HTTP requests to the same web server. Instead, it allows multiple requests and responses over the same TCP connection. (All contemporary web server software and web browsers support HTTP/1.1 at a minimum, so you’re unlikely to use HTTP/1.0 at all.)

[Table 8-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#table8-1) demonstrates the retrieval of an HTML document and the subsequent `GET` calls of all resources specified in that document.

Table 8-1: Retrieving Additional Resources After Requesting the Index HTML Document

|Status|Method|Domain|Resource|Type|Bytes transferred|Duration of transfer|
|---|---|---|---|---|---|---|
|200|GET|_woodbeck.net_|_/_|HTML|1.83KB|49 ms|
|200|GET|_woodbeck.net_|_main.min.css_|CSS|1.30KB|20 ms|
|200|GET|_woodbeck.net_|_code.css_|CSS|0.99KB|20 ms|
|304|GET|_woodbeck.net_|_avatar.jpeg_|JPEG|0 bytes|0 ms|
|404|GET|_woodbeck.net_|_favicon.ico_|IMG|0 bytes|0 ms|

The initial `GET` request for [https://woodbeck.net/](https://woodbeck.net/) successfully retrieved the HTML document specified by the default resource. This HTML document included links to additional resources necessary to properly render the page, so the web browser requested those too. Since this transfer used HTTP/1.1, the web browser used the same TCP connection to retrieve the remaining resources. The web server instructed the web browser to use its cached copy of _avatar.jpeg_, since that resource hadn’t changed since the last time the web browser received it. The web server was unable to find the _favicon.ico_ file, so it returned a 404 status code to the web browser.

The latest version of HTTP, HTTP/2, aims to further reduce latency. In addition to reusing the same TCP connection for subsequent requests, the HTTP/2 server can proactively push resources to the client. If the conversation in [Table 8-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#table8-1) occurred over HTTP/2, it may have transpired like this. The client requested the default resource. The server responded with the default resource. But since the server knew that the default resource had dependent resources, it _pushed_ those resources to the client without the client’s needing to make separate requests for each resource.

The Go HTTP client and server transparently support HTTP/1.0, HTTP/1.1, and HTTP/2, meaning that you can write your code to retrieve and serve resources while letting code in Go’s `net/http` package negotiate the optimal HTTP version. However, while the Go HTTP/2 server implementation can push resources to clients, the Go HTTP/2 client implementation cannot yet consume those server pushes.

## Retrieving Web Resources in Go

Just like your web browser, Go can communicate with web servers by using the `net/http` package’s HTTP client. Unlike your web browser, Go won’t directly render an HTML page to your screen. Instead, you could use Go to scrape data from websites (such as financial stock details), submit form data, or interact with APIs that use HTTP for their application protocol, to name a few examples.

Despite the simplicity of making HTTP requests in Go, you’ll have to handle some client-side pitfalls. You’ll learn about these pitfalls shortly. First, let’s look at a simple example request.

### Using Go’s Default HTTP Client

The `net/http` package includes a default client that allows you to make one-off HTTP requests. For example, you can use the `http.Get` function to send a `GET` request to a given URL.

[Listing 8-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-2) demonstrates one way you can retrieve the current time from a trusted authority—_time.gov_’s web server—and compare it with the local time on your computer. This will give you a rough idea of how far ahead or behind the local time is on your computer. You certainly wouldn’t want to rely on this method for any sort of forensics, but the example serves to demonstrate the Go HTTP client workflow by using a `HEAD` request and response.

```
package main

import (
    "net/http"
    "testing"
    "time"
)

func TestHeadTime(t *testing.T) {
    resp, err := 1http.Head("https://www.time.gov/")
    if err != nil {
        t.Fatal(err)
    }
    _ = 2resp.Body.Close() // Always close this without exception.

    now := time.Now().Round(time.Second)
    date := 3resp.Header.Get("Date")
    if date == "" {
        t.Fatal("no Date header received from time.gov")
    }

    dt, err := time.Parse(time.RFC1123, date)
    if err != nil {
        t.Fatal(err)
    }

    t.Logf("time.gov: %s (skew %s)", dt, now.Sub(dt))
}
```

Listing 8-2: Retrieving a timestamp from _time.gov_ (_time_test.go_)

The `net/http` package includes a few helper functions to make `GET`, `HEAD`, or `POST` requests. Here, we use the `http.Get` function 1 to [https://www.time.gov/](https://www.time.gov/) to retrieve the default resource. Go’s HTTP client automatically upgrades to HTTPS for you because that’s the protocol indicated by the URL’s scheme. Although you don’t read the contents of the response body, you must close it 2. The next section covers why you need to close the response body in every case.

Now that you have a response, you retrieve the `Date` header 3, which indicates the time at which the server created the response. You can then use this value to calculate the clock skew of your computer. Granted, you lose accuracy because of latency between the server’s generating the header and your code’s processing it, as well as the lack of nanosecond resolution of the `Date` header itself.

### Closing the Response Body

As mentioned earlier, HTTP/1.1 allows the client to maintain a TCP connection with a server for multiple HTTP requests (we call this _keepalive support_). Even so, the client cannot reuse a TCP session when unread bytes from the previous response remain on the wire. Go’s HTTP client automatically drains the response body when you close it. This allows your code to reuse the underlying TCP session if you are diligent about closing every response body.

Let’s revisit the response from [Listing 8-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-1) to see how Go parses the response ([Listing 8-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-3)).

```
1 HTTP/1.1 200 OK
Accept-Ranges: none
Vary: Accept-Encoding
Content-Type: text/plain
Date: Mon, 02 Jan 2006 15:04:05 MST
Expires: Mon, 02 Jan 2006 15:04:05 MST
Cache-Control: private, max-age=0
Last-Modified: Mon, 02 Jan 2006 15:04:05 MST
X-Content-Type-Options: nosniff
Server: sffe
X-XSS-Protection: 0
Transfer-Encoding: chunked

2 User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
--snip-- 
```

Listing 8-3: Parsing the HTTP response

Go’s HTTP client reads the response status and headers 1 from the network socket, and this data immediately becomes available to your code as part of the response object. The client doesn’t automatically read the response body, however 2. The body remains unconsumed until your code explicitly reads it or until you close it and Go implicitly drains any unread bytes.

The Go HTTP client’s implicit draining of the response body on closing could potentially bite you. For example, let’s assume you send a `GET` request for a file and receive a response from the server. You read the response’s `Content-Length` header and realize the file is much larger than you anticipated. If you close the response body without reading any of its bytes, Go will download the entire file from the server as it drains the body regardless.

A better alternative would be to send a `HEAD` request to retrieve the `Content-Length` header. This way, no unread bytes exist in the response body, so closing the response body will not incur any additional overhead while draining it. You properly closed the response body in [Listing 8-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-2), so the Go HTTP client could reuse the TCP session if you made additional calls in the future.

On the rare occasion that you make an HTTP request and want to explicitly drain the response body, the most efficient way is to use the `io.Copy` function:

```
_, _ = io.Copy(ioutil.Discard, response.Body) 
_ = response.Close()
```

The `io.Copy` function drains the `response.Body` by reading all bytes from it and writing those bytes to `ioutil.Discard`. As its name indicates, `ioutil.Discard` is a special `io.Writer` that discards all bytes written to it.

You do not have to ignore the return values of `io.Copy` and `response.Close`, but doing so lets other developers know you intentionally ignored these values. Some developers may regard this as unnecessary verbosity, and it’s true that `io.Copy` or `response.Close` will rarely return errors in this context, but it’s still a good practice. I’ve encountered code that implicitly ignores errors, presumably out of habit, when the developer should have otherwise handled the errors.

The bottom line is that you must close the response body no matter whether you read it or not, to avoid resource leaks.

### Implementing Time-outs and Cancellations

Go’s default HTTP client and the requests created with the `http.Get`, `http.Head`, and `http.Post` helper functions do not time out. The consequences of this may not be obvious until you get bit by the following fact (after which you’ll never forget it): the lack of a time-out or deadline means that a misbehaving or malicious service could cause your code to block indefinitely without producing an error to indicate that anything’s wrong. You might not find out that your service is malfunctioning until users start calling you to complain.

For example, [Listing 8-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-4) demonstrates a simple test that causes the HTTP client to block indefinitely.

```
package main

import (
    "context"
    "errors"
    "net/http"
    "net/http/httptest"
    "testing"
    "time"
)

func blockIndefinitely(w http.ResponseWriter, r *http.Request) {
    select {}
}

func TestBlockIndefinitely(t *testing.T) {
    ts := 1httptest.NewServer(2http.HandlerFunc(3blockIndefinitely))
    _, _ = http.Get(4ts.URL)
    t.Fatal("client did not indefinitely block")
}
```

Listing 8-4: The test server causes the default HTTP client to block indefinitely (_block_test.go_).

The `net/http/httptest` package includes a useful HTTP test server. The `httptest.NewServer`1 function accepts an `http.HandlerFunc`2, which in turn wraps the `blockIndefinitely` function 3. The test server passes any request it receives at its URL 4 to the `http.HandlerFunc`’s `ServeHTTP` method. This method sends the request and response objects to the `blockIndefinitely` function, where control blocks indefinitely.

Because the helper function `http.Get` uses the default HTTP client, this `GET` request won’t time out. Instead, the `go test` runner will eventually time out and halt the test, printing the stack trace.

## NOTE

If you run this test, I recommend you pass the argument `-timeout 5s` to `go test` to keep from waiting too long.

To solve this issue, production code should use the technique you learned for timing out network sockets in “Using a Context with a Deadline to Time Out a Connection” on page 57. Create a context and use it to initialize a new request. You can then manually cancel the request by either using the context’s `cancel` function or creating a context with a deadline or time-out.

Let’s fix the test in [Listing 8-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-4) by replacing it with the test in [Listing 8-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-5). The new test will time out the request after five seconds without an answer from the server.

```
--snip--

func TestBlockIndefinitelyWithTimeout(t *testing.T) {
    ts := httptest.NewServer(http.HandlerFunc(blockIndefinitely))

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    req, err := 1http.NewRequestWithContext(ctx, http.MethodGet, ts.URL, nil)
    if err != nil {
        t.Fatal(err)
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        if !errors.Is(err, context.DeadlineExceeded) {
            t.Fatal(err)
        }
        return
    }
    _ = resp.Body.Close()
}
```

Listing 8-5: Adding a time-out to the GET request (_block_test.go_)

First, you create a new request 1 by passing in the context, the request method, the URL, and a `nil` request body, since your request does not have a payload. Keep in mind that the context’s timer starts running as soon as you initialize the context. The context controls the entire life cycle of the request. In other words, the client has five seconds to connect to the web server, send the request, read the response headers, and pass the response to your code. You then have the remainder of the five seconds to read the response body. If you are in the middle of reading the response body when the context times out, your next read will immediately return an error. So, use generous time-out values for your specific application.

Alternatively, create a context without a time-out or deadline and control the cancellation of the context exclusively by using a timer and the context’s `cancel` function, like this:

```
    ctx, cancel := context.WithCancel(context.Background())
    timer := time.AfterFunc(5*time.Second, 1cancel)
    // Make the HTTP request, read the response headers, etc.
    // ...
    // Add 5 more seconds before reading the response body.
    timer.Reset(5*time.Second)
```

This snippet demonstrates how to use a timer that will call the context’s `cancel` function 1 after it expires. You can reset the timer as needed to push the call to `cancel` further into the future.

### Disabling Persistent TCP Connections

By default, Go’s HTTP client maintains the underlying TCP connection to a web server after reading its response unless explicitly told to disconnect by the server. Although this is desirable behavior for most use cases because it allows you to use the same TCP connection for multiple requests, you may inadvertently deny your computer the ability to open new TCP connections with other web servers.

This is because the number of active TCP connections a computer can maintain is finite. If you write a program that makes one-off requests to numerous web servers, you could find that your program stops working after exhausting all your computer’s available TCP connections, leaving it unable to open new ones. In this scenario, TCP session reuse can work against you. Instead of disabling TCP session reuse in the client, a more flexible option is to inform the client what to do with the TCP socket on a per-request basis.

```
--snip--
    req, err := http.NewRequestWithContext(ctx, http.MethodGet, ts.URL, nil)
    if err != nil {
        t.Fatal(err)
    }
  1req.Close = true 
--snip--
```

Setting the request’s `Close` field 1 to `true` tells Go’s HTTP client that it should close the underlying TCP connection after reading the web server’s response. If you know you’re going to send four requests to a web server and no more, you could set the `Close` field to `true` on the fourth request. All four requests will use the same TCP session, and the client will terminate the TCP connection after receiving the fourth response.

## Posting Data over HTTP

Sending a `POST` request and its payload to a web server is like the calls you’ve made thus far. The difference, of course, is that the request body contains a payload. This payload can be any object that implements the `io.Reader` interface, including a file handle, standard input, an HTTP response body, or a Unix domain socket, to name a few. But as you’ll see, sending data to the web server involves a little more code than a `GET` request because you must prepare that request body.

### Posting JSON to a Web Server

Before you can send data to a test server, you need to create a handler that can accept it. [Listing 8-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-6) creates a new type named `User` that you will encode to JavaScript Object Notation (JSON) and post to the handler.

```
package main

import (
    "bytes"
    "context"
    "encoding/json"
    "fmt"
    "io"
    "io/ioutil"
    "mime/multipart"
    "net/http"
    "net/http/httptest"
    "os"
    "path/filepath"
    "testing"
    "time"
)

type User struct {
    First string
    Last  string
}

1 func handlePostUser(t *testing.T) func(http.ResponseWriter, *http.Request) {
    return func(w http.ResponseWriter, r *http.Request) {
        defer func(r io.ReadCloser) {
            _, _ = 2io.Copy(ioutil.Discard, r)
            _ = r.Close()
        }(r.Body)

        if r.Method != 3http.MethodPost {
          4http.Error(w, "", http.StatusMethodNotAllowed)
            return
        }

        var u User
        err := json.NewDecoder(r.Body).Decode(&u)
        if err != nil {
            t.Error(err)
            http.Error(w, "Decode Failed", http.StatusBadRequest)
            return
        }

      5w.WriteHeader(http.StatusAccepted) 
    }
}
```

Listing 8-6: A handler that can decode JSON into a User object (_post_test.go_)

The `handlePostUser` function 1 returns a function that will handle `POST` requests. If the request method is anything other than `POST`3, it returns a status code indicating that the server disallows the method 4. The function then attempts to decode the JSON in the request body to a `User` object. If successful, the response’s status is set to _Accepted_5.

Unlike the Go HTTP client, the Go HTTP server must explicitly drain the request body 2 before closing it. We’ll discuss this in more detail in Chapter 9.

The test in [Listing 8-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-7) encodes a `User` object into JSON and sends it in a `POST` request to the test server.

```
--snip--

func TestPostUser(t *testing.T) {
    ts := httptest.NewServer(http.HandlerFunc(handlePostUser(t)))
    defer ts.Close()

    resp, err := http.Get(ts.URL)
    if err != nil {
        t.Fatal(err)
    }
    if 1resp.StatusCode != http.StatusMethodNotAllowed {
        t.Fatalf("expected status %d; actual status %d",
            http.StatusMethodNotAllowed, resp.StatusCode)
    }

    buf := new(bytes.Buffer)
    u := User{First: "Adam", Last: "Woodbeck"}
 2 err = json.NewEncoder(buf).Encode(&u)
    if err != nil {
        t.Fatal(err)
    }

    resp, err = 3http.Post(ts.URL, "application/json", buf)
    if err != nil {
        t.Fatal(err)
    }
    if resp.StatusCode != 4http.StatusAccepted {
        t.Fatalf("expected status %d; actual status %d",
            http.StatusAccepted, resp.StatusCode)
    }
    _ = resp.Body.Close()
}
```

Listing 8-7: Encoding a User object to JSON and POST to the test server (_post_test.go_)

The test first makes sure that the test server’s handler properly responds with an error if the client sends the wrong type of request 1. If the test server receives anything other than a `POST` request, it will respond with a Method Not Allowed error. Then, the test encodes a `User` object into JSON and writes the data to a bytes buffer 2. It makes a `POST` request to the test server’s URL with the content type _application/json_ because the bytes buffer, representing the request body, contains JSON 3. The content type informs the server’s handler about the type of data to expect in the request body. If the server’s handler properly decoded the request body, the response status code is 202 Accepted 4.

### Posting a Multipart Form with Attached Files

Posting JSON to a web server is easy. Simply set the appropriate content type and send along the JSON payload in the request body. But how do you handle sending various bits of data to a web server in a single `POST` request? Answer: use the `mime/multipart` package.

The `mime/multipart` package allows you to craft multipart _Multipurpose Internet Mail Extensions (MIME) messages_, which separate each bit of data you want to send from the other bits of data by a string known as a _boundary_. You’ll see an example of a boundary a bit later in this section, though it isn’t something you typically need to worry about.

Each MIME part includes optional headers that describe the content, as well as a body that contains the content itself. For example, if a web server parsed a MIME part with a `Content-Type` header set to _text/plain_, it would treat the part’s body as plaintext.

[Listing 8-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-8) introduces a new test that walks you through the process of building up a multipart request body using the `mime/multipart` package.

```
--snip--

func TestMultipartPost(t *testing.T) {
    reqBody := 1new(bytes.Buffer)
    w := 2multipart.NewWriter(reqBody)

    for k, v := range map[string]string{
        "date":        time.Now().Format(time.RFC3339),
        "description": "Form values with attached files",
    } {
        err := 3w.WriteField(k, v)
        if err != nil {
            t.Fatal(err)
        }
    }
```

Listing 8-8: Creating a new request body, multipart writer, and write form data (_post_test.go_)

First, you create a new buffer 1 to act as the request body. You then create a new multipart writer 2 that wraps the buffer. The multipart writer generates a random boundary upon initialization. Finally, you write form fields to the multipart writer 3. The multipart writer separates each form field into its own part, writing the boundary, appropriate headers, and the form field value to each part’s body.

At this point, your request body has two parts, one for the _date_ form field and one for the _description_ form field. Let’s attach a couple of files in [Listing 8-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-9).

```
--snip--

    for i, file := range []string{
        "./files/hello.txt",
        "./files/goodbye.txt",
    } {
        filePart, err := 1w.CreateFormFile(fmt.Sprintf("file%d", i+1),
            filepath.Base(file))
        if err != nil {
            t.Fatal(err)
        }

        f, err := os.Open(file)
        if err != nil {
            t.Fatal(err)
        }

        _, err = 2io.Copy(filePart, f)
        _ = f.Close()
        if err != nil {
            t.Fatal(err)
        }
    }

    err := 3w.Close()
    if err != nil {
        t.Fatal(err)
    }
```

Listing 8-9: Writing two files to the request body, each in its own MIME part (_post_test.go_)

Attaching a field to a request body isn’t as straightforward as adding form field data. You have an extra step. First, you need to create a multipart section writer from [Listing 8-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-8)’s multipart writer 1. The `CreateFormField` method accepts a field name and a filename. The server uses this filename when parsing the MIME part. It does not need to match the filename you attach. Now, you just open the file and copy its contents to the MIME part writer 2.

When you’re done adding parts to the request body, you must close the multipart writer 3, which finalizes the request body by appending the boundary.

[Listing 8-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-10) posts the request to a well-regarded test server, _httpbin.org_.

```
--snip--

    ctx, cancel := context.WithTimeout(context.Background(),
        60*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, http.MethodPost,
      1"https://httpbin.org/post", 2reqBody)
    if err != nil {
        t.Fatal(err)
    }
    req.Header.Set("Content-Type", 3w.FormDataContentType())

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = resp.Body.Close() }()

    b, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        t.Fatal(err)
    }
    if resp.StatusCode != http.StatusOK {
        t.Fatalf("expected status %d; actual status %d",
            http.StatusOK, resp.StatusCode)
    }

    t.Logf("\n%s", b)
}
```

Listing 8-10: Sending a POST request to _httpbin.org_ with Go’s default HTTP client (_post_test.go_)

First, you create a new request and pass it a context that will time out in 60 seconds. Since you’re making this call over the internet, you don’t have as much certainty that your request will reach its destination as you do when testing over localhost. The `POST` request is destined for [https://www.httpbin.org/](https://www.httpbin.org/)1 and will send the multipart request body 2 in its payload.

Before you send the request, you need to set the `Content-Type` header to inform the web server you’re sending multiple parts in this request. The multipart writer’s `FormDataContentType` method 3 returns the appropriate `Content-Type` value that includes its boundary. The web server uses the boundary from this header to determine where one part stops and another starts as it reads the request body.

Once you run the test with the `-v` flag, you should see output like the JSON in [Listing 8-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c08.xhtml#listing8-11).

```
{
  "args": {},
  "data": "",
1"files": {
    "file1": "Hello, world!\n",
    "file2": "Goodbye, world!\n"
  },
2"form": {
    "date": "2006-01-02T15:04:05-07:00",
    "description": "Form fields with attached files"
  },
  "headers": {
    "Accept-Encoding": "gzip",
    "Content-Length": "739",
 3 "Content-Type": "multipart/form-data; boundary=e9ad4b62e0dfc8d7dc57ccfa8ba62244342f1884608e6d88018f9de8abcb",
    "Host": "httpbin.org",
    "User-Agent": "Go-http-client/1.1"
  },
  "json": null,
  "origin": "192.168.0.1",
  "url": "https://httpbin.org/post"
}
```

Listing 8-11: Response body from the multipart POST request

This is _httpbin.org_’s standard `POST` response and includes some fields irrelevant to the request you sent. But if you have a look, you’ll see the contents of each text file you attached 1 and the form fields you submitted 2. You can also see the `Content-Type` header 3 added by the multipart writer. Notice the boundary is a random string. With your code as is, the boundary will randomly change with each request. But you can set a boundary by using the multipart writer’s `SetBoundary` method if you so choose.

## What You’ve Learned

HTTP allows clients to send requests to, and receive resources from, servers over the World Wide Web. This chapter showed you how to use Go to craft HTTP requests. Target resources can take the form of web pages, images, videos, documents, files, games, and so on. To retrieve a resource, the HTTP client sends a `GET` request with a URL to the web server. The web server uses the URL to locate the correct resource and send it along to the client in the server’s response. The client always initiates this HTTP request-response workflow.

The client can send diverse types of resource requests to the server. The most used request methods are `GET`, `HEAD`, `POST`, `PUT`, and `DELETE`. A `GET` request asks the server to retrieve the specified resource. The client may send a `HEAD` request to retrieve the response headers without the requested payload. This can be useful for determining whether a resource exists and inspecting response headers before retrieving the resource. A `POST` request allows the client to send a resource to the server, whereas you typically use a `PUT` request to update an existing resource on the server. The client can request a resource’s deletion from the server by sending a `DELETE` request.

The `net/http` package provides all necessary types and functions to interact with servers over HTTP. It includes a default HTTP client that allows you to make a quick, one-off HTTP request and receive the response. However, you must diligently close the response body to prevent resource leaks, no matter whether you read the body’s contents or not. It’s also important to note that the default HTTP client and the requests sent using helper functions such as `http.Get`, `http.Head`, and `http.Post` do not time out. This means that a misbehaving or malicious service could cause your code to block indefinitely. Therefore, it’s important to manage request cancellation yourself by using a context.

The `mime/multipart` package allows you to easily add multiple MIME parts to a request body. You can efficiently craft requests that upload files and form content to a web server.