---
id: 01JCH9DPMZ23AXGNEKQDJJ1S4F
modified: 2024-11-12T17:56:36-05:00
title: Chapter 9 - Building HTTP Services
---
# 9  
BUILDING HTTP SERVICES

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

Now that you’ve written client code to send HTTP requests, let’s build a server that can process these requests and send resources to the client. The `net/http` package handles most of the implementation details for you, so you can focus on instantiating and configuring a server, creating resources, and handling each client request.

In Go, an HTTP server relies on several interacting components: handlers, middleware, and a multiplexer. When it includes all these parts, we call this server a _web service_. We’ll begin by looking at a simple HTTP web service and then explore each of its components over the course of the chapter. The big picture should help you understand topics that beginners often find abstract.

You’ll also learn more advanced uses of the `net/http` package, such as adding TLS support and pushing data to HTTP/2 clients. By the end, you should feel comfortable configuring a Go-based HTTP server, writing middleware, and responding to requests with handlers.

## The Anatomy of a Go HTTP Server

[Figure 9-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#figure9-1) illustrates the path a request takes in a typical `net/http`-based server.

![f09001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c09/f09001.png)

Figure 9-1: Client request culminating in a server response in the handler

First, the server’s _multiplexer_ (_router_, in computer-networking parlance) receives the client’s request. The multiplexer determines the destination for the request, then passes it along to the object capable of handling it. We call this object a _handler_. (The multiplexer itself is a handler that routes requests to the most appropriate handler.) Before the handler receives the request, the request may pass through one or more functions called _middleware_. Middleware changes the handlers’ behavior or performs auxiliary tasks, such as logging, authentication, or access control.

[Listing 9-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-1) creates an HTTP server that follows this basic structure. If you have trouble following along, don’t worry; you’ll spend the rest of the chapter learning how these parts work.

```
package main

import (
    "bytes"
    "fmt"
    "io"
    "io/ioutil"
    "net"
    "net/http"
    "testing"
    "time"

    "github.com/awoodbeck/gnp/ch09/handlers"
)

func TestSimpleHTTPServer(t *testing.T) {
    srv := &http.Server{
        Addr: "127.0.0.1:8081",
        Handler: 1http.TimeoutHandler(
            handlers.DefaultHandler(), 2*time.Minute, ""),
        IdleTimeout:       5 * time.Minute,
        ReadHeaderTimeout: time.Minute,
    }

    l, err := 2net.Listen("tcp", srv.Addr)
    if err != nil {
        t.Fatal(err)
    }

    go func() {
        err := 3srv.Serve(l)
        if err != http.ErrServerClosed {
            t.Error(err)
        }
    }()
```

Listing 9-1: Instantiating a multiplexer and an HTTP server (_server_test.go_)

Requests sent to the server’s handler first pass through middleware named `http.TimeoutHandler`1, then to the handler returned by the `handlers.DefaultHandler` function. In this very simple example, you specify only a single handler for all requests instead of relying on a multiplexer.

The server has a few fields. The `Handler` field accepts a multiplexer or other object capable of handling client requests. The `Address` field should look familiar to you by now. In this example, you want the server to listen to port 8081 on IP address 127.0.0.1. I’ll explain the `IdleTimeout` and `ReadHeaderTimeout` fields in the next section. Suffice it to say now, you should always define these two fields.

Finally, you create a new `net.Listener` bound to the server’s address 2 and instruct the server to `Serve`3 requests from this listener. The `Serve` method returns `http.ErrServerClosed` when it closes normally.

Now let’s test this server. [Listing 9-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-2) picks up where [Listing 9-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-1) leaves off. It details a few test requests and their expected results.

```
--snip--

    testCases := []struct {
        method   string
        body     io.Reader
        code     int
        response string
    }{
      1{http.MethodGet, nil, http.StatusOK, "Hello, friend!"},
      2{http.MethodPost, bytes.NewBufferString("<world>"), http.StatusOK,
            "Hello, &lt;world&gt;!"},
      3{http.MethodHead, nil, http.StatusMethodNotAllowed, ""},
    }

    client := new(http.Client)
    path := fmt.Sprintf("http://%s/", srv.Addr)
```

Listing 9-2: Request test cases for the HTTP server (_server_test.go_)

First, you send a `GET` request 1, which results in a 200 OK status code. The response body has the `Hello, friend!` string.

In the second test case 2, you send a `POST` request with the string `<world>` in its body. The angle brackets are intentional, and they show an often-overlooked aspect of handling client input in the handler: always escape client input. You’ll learn about escaping client input in “Handlers” on page 193. This test case results in the string `Hello, &lt;world&gt;!` in the response body. The response looks a bit silly, but your web browser renders it as `Hello, <world>!`.

The third test case 3 a sends a `HEAD` request to the HTTP server. The handler returned by the `handlers.DefaultHandler` function, which you’ll explore shortly, does not handle the `HEAD` method. Therefore, it returns a 405 Method Not Allowed status code and an empty response body.

[Listing 9-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-3) continues the code in [Listing 9-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-2) and runs through each test case.

```
--snip--

    for i, c := range testCases {
        r, err := 1http.NewRequest(c.method, path, c.body)
        if err != nil {
            t.Errorf("%d: %v", i, err)
            continue
        }

        resp, err := 2client.Do(r)
        if err != nil {
            t.Errorf("%d: %v", i, err)
            continue
        }

        if resp.StatusCode != c.code {
            t.Errorf("%d: unexpected status code: %q", i, resp.Status)
        }

        b, err := 3ioutil.ReadAll(resp.Body)
        if err != nil {
            t.Errorf("%d: %v", i, err)
            continue
        }
        _ = 4resp.Body.Close()

        if c.response != string(b) {
            t.Errorf("%d: expected %q; actual %q", i, c.response, b)
        }
    }

    if err := 5srv.Close(); err != nil {
        t.Fatal(err)
    }
}
```

Listing 9-3: Sending test requests to the HTTP server (_server_test.go_)

First, you create a new request, passing the parameters from the test case 1. Next, you pass the request to the client’s `Do` method 2, which returns the server’s response. You then check the status code and read in the entire response body 3. You should be in the habit of consistently closing the response body if the client did not return an error 4, even if the response body is empty or you ignore it entirely. Failure to do so may prevent the client from reusing the underlying TCP connection.

Once all tests complete, you call the server’s `Close` method 5. This causes its `Serve` method in [Listing 9-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-1) to return, stopping the server. The `Close` method abruptly closes client connections. You’ll see an example of the HTTP server’s graceful shutdown support when we discuss HTTP/2 pushes later in this chapter.

Go’s HTTP server supports a few other features, which we’ll explore in the following sections. It can proactively serve, or push, resources to clients. It also offers graceful shutdown support. Abruptly shutting down your web server may leave some clients in an awkward state if they were waiting for a response when you stopped the server, because those clients will never receive a response. Graceful shutdowns allow for all pending responses to reach each client before the server is stopped.

### Clients Don’t Respect Your Time

Just as I recommended setting the client’s time-out values, I recommend that you manage the various server time-out values, for the simple reason that clients won’t otherwise respect your server’s time. A client can take its sweet time sending a request to your server. Meanwhile, your server uses resources waiting to receive the request in its entirety. Likewise, your server is at the client’s mercy when it sends the response because it can send data only as fast as the client reads it (or can send only as much as there is TCP buffer space available). Avoid letting the client dictate the duration of a request-response life cycle.

[Listing 9-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-1) includes a server instance with two of its time-out values specified: the length of time clients can remain idle between requests and how long the server should wait to read a request header:

```
srv := &http.Server{
    Addr:              "127.0.0.1:8081",
    Handler:           mux,
    IdleTimeout:       5 * time.Minute,
    ReadHeaderTimeout: time.Minute,
}
```

Although several time-out fields on the `http.Server` are available to you, I recommend setting only the `IdleTimeout` and `ReadHeaderTimeout` fields. The `IdleTimeout` field dictates how long the server will keep its side of the TCP socket open while waiting for the next client request when the communication uses keepalives. The `ReadHeaderTimeout` value determines how long the server will wait to finish reading the request headers. Keep in mind that this duration has no bearing on the time it takes to read the request body.

If you want to enforce a time limit for reading an incoming request across all handlers, you could manage the request deadline by using the `ReadTimeout` field. If the client hasn’t sent the complete request (the headers and body) by the time the `ReadTimeout` duration elapses, the server ends the TCP connection. Likewise, you could give the client a finite duration in which to send the request and read the response by using the `WriteTimeout` field. The `ReadTimeout` and `WriteTimeout` values apply to all requests and responses because they dictate the `ReadDeadline` and `WriteDeadline` values of the TCP socket, as discussed in Chapter 4.

These blanket time-out values may be inappropriate for handlers that expect clients to send large files in a request body or handlers that indefinitely stream data to the client. In these two examples, the request or response may abruptly time out even if everything went ahead as expected. Instead, a good practice is to rely on the `ReadHeaderTimeout` value. You can separately manage the time it takes to read the request body and send the response using middleware or handlers. This gives you the greatest control over request and response durations per resource. You’ll learn how to manage the request-response duration by using middleware in “Middleware” on page 202.

### Adding TLS Support

HTTP traffic is plaintext by default, but web clients and servers can use HTTP over an encrypted TLS connection, a combination known as _HTTPS_. Go’s HTTP server enables HTTP/2 support over TLS connections only, but enabling TLS is a simple matter. You need to modify only two lines from [Listing 9-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-1)’s server implementation: the port number and the `Serve` method:

```
    srv := &http.Server{
        Addr:            1"127.0.0.1:8443",
        Handler:           mux,
        IdleTimeout:       5 * time.Minute,
        ReadHeaderTimeout: time.Minute,
    }

    l, err := net.Listen("tcp", srv.Addr)
    if err != nil {
        t.Fatal(err)
    }

    go func() {
     2 err := srv.ServeTLS(l, "cert.pem", "key.pem")
        if err != http.ErrServerClosed {
            t.Error(err)
        }
    }()
```

Technically, you don’t need to change the port number 1, but the convention is to serve HTTPS over port 443, or an augmentation of port 443, like 8443. Using the server’s `ServeTLS` method, you instruct the server to use TLS over HTTP 2. The `ServeTLS` method requires the path to both a certificate and a corresponding private key. I recommend you check out the mkcert project at [https://github.com/FiloSottile/mkcert/](https://github.com/FiloSottile/mkcert/) to get a key pair. You can use mkcert to create locally trusted key pairs for development purposes only. For production use, you should consider using and supporting Let’s Encrypt at [https://letsencrypt.org/](https://letsencrypt.org/).

## Handlers

When a client sends a request to an HTTP server, the server needs to figure out what to do with it. The server may need to retrieve various resources or perform an action, depending on what the client requests. A common design pattern is to specify bits of code to handle these requests, known as _handlers_. You may have a handler that knows how to retrieve an image and another handler that knows how to retrieve information from a database. We’ll discuss how the server figures out which handler is most apt for each request in “Multiplexers” on page 207.

In Go, handlers are objects that implement the `http.Handler` interface. They read client requests and write responses. The `http.Handler` interface consists of a single method to receive both the response and the request:

```
type Handler interface {
    ServeHTTP(http.ResponseWriter, *http.Request)
}
```

Any object that implements the `http.Handler` interface may handle client requests, as far as the Go HTTP server is concerned. We often define handlers as functions, as in this common pattern:

```
handler := http.HandlerFunc(
    func(w http.ResponseWriter, r *http.Request) {
        _, _ = w.Write([]byte("Hello, world!"))
    },
)
```

Here, you wrap a function that accepts an `http.ResponseWriter` and an `http.Request` pointer in the `http.HandlerFunc` type, which implements the `Handler` interface. This results in an `http.HandlerFunc` object that calls the wrapped `func(w http.ResponseWriter, r *http.Request)` function when the server calls its `ServeHTTP` method. This handler responds to the client with the string `Hello, world!` in the response body.

Notice that you ignore the number of written bytes and any potential write error. In the wild, writes to a client can fail for any number of reasons. It isn’t worth logging these errors. Instead, one option is to keep track of the write error frequency and have your server send you an alert should the number of errors exceed an appropriate threshold. You’ll learn about instrumenting your code in Chapter 13.

Now that you’re familiar with the structure of a handler, let’s have a look at the handler returned by the `handlers.DefaultHandler` function in [Listing 9-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-4).

```
package handlers

import (
    "html/template"
    "io"
    "io/ioutil"
    "net/http"
)

var t = 1template.Must(template.New("hello").Parse("Hello, {{.}}!"))

func DefaultHandler() http.Handler {
    return http.HandlerFunc(
        func(w http.ResponseWriter, r *http.Request) {
         2 defer func(r io.ReadCloser) {
                _, _ = io.Copy(ioutil.Discard, r)
                _ = r.Close()
            }(r.Body)

            var b []byte

         3 switch r.Method {
            case http.MethodGet:
                b = []byte("friend")
            case http.MethodPost:
                var err error
                b, err = ioutil.ReadAll(r.Body)
                if err != nil {
             4 http.Error(w, "Internal server error",
                    http.StatusInternalServerError)
                return
                }
            default:
                // not RFC-compliant due to lack of "Allow" header
             5 http.Error(w, "Method not allowed", 
                    http.StatusMethodNotAllowed)
                return
            }

            _ = 6t.Execute(w, string(b))
        },
    )
}
```

Listing 9-4: The default handler implementation (_handlers/default.go_)

The `handlers.DefaultHandler` function returns a function converted to the `http.HandlerFunc` type. The `http.HandlerFunc` type implements the `http.Handler` interface. Go programmers commonly convert a function with the signature `func(w http.ResponseWriter, r *http.Request)` to the `http.HandlerFunc` type so the function implements the `http.Handler` interface.

The first bit of code you see is a deferred function that drains and closes the request body 2. Just as it’s important for the client to drain and close the response body to reuse the TCP session, it’s important for the server to do the same with the request body. But unlike the Go HTTP client, closing the request body does not implicitly drain it. Granted, the `http.Server` will close the request body for you, but it won’t drain it. To make sure you can reuse the TCP session, I recommend you drain the request body at a minimum. Closing it is optional.

The handler responds differently depending on the request method 3. If the client sent a `GET` request, the handler writes `Hello, friend!` to the response writer. If the request method is a `POST`, the handler first reads the entire request body. If an error occurs while reading the request body, the handler uses the `http.Error` function 4 to succinctly write the message `Internal server error` to the response body and set the response status code to 500. Otherwise, the handler returns a greeting using the request body contents. If the handler receives any other request method, it responds with a 405 Method Not Allowed status 5. The 405 response is technically not RFC-compliant without an Allow header showing which methods the handler accepts. We’ll shore up this deficiency in “Any Type Can Be a Handler” on page 198. Finally, the handler writes the response body.

This code could have a security vulnerability since part of the response body might come from the request body. A malicious client can send a request payload that includes JavaScript, which could run on a client’s computer. This behavior can lead to an XSS attack. To prevent these attacks, you must properly escape all client-supplied content before sending it in a response. Here, you use the `html/template` package to create a simple template 1 that reads `Hello, {{.}}!`, where `{{.}}` is a placeholder for part of your response. Templates derived from the `html/template` package automatically escape HTML characters when you populate them and write the results to the response writer 6. HTML-escaping explains the funky characters in [Listing 9-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-2)’s second test case. The client’s browser will properly display the characters instead of interpreting them as part of the HTML and JavaScript in the response body. The bottom line is to always use the `html/template` package when writing untrusted data to a response writer.

### Test Your Handlers with httptest

Saying “make sure you test your code” is the developer’s equivalent of my mother telling me to clean my bedroom. It’s good advice, but I’d much rather continue hacking away than write test code. But my mother was correct, and writing test code now will serve me well in the future. The Go standard library architects—motivated by clean bedrooms, no doubt—made sure to give us the `net/http/httptest` package. This package makes unit-testing handlers painless.

The `net/http/httptest` package exports a `NewRequest` function that accepts an HTTP method, a target resource, and a request body `io.Reader`. It returns a pointer to an `http.Request` ready for use in an `http.Handler`:

```
func NewRequest(method, target string, body io.Reader) *http.Request
```

Unlike its `http.NewRequest` equivalent, `httptest.NewRequest` will panic instead of returning an error. This is preferable in tests but not in production code.

The `httptest.NewRecorder` function returns a pointer to an `httptest.ResponseRecorder`, which implements the `http.ResponseWriter` interface. Although the `httptest.ResponseRecorder` exports fields that look tempting to use (I don’t want to tempt you by mentioning them), I recommend you call its `Result` method instead. The `Result` method returns a pointer to an `http.Response` object, just like the one we used in the last chapter. As the method’s name implies, it waits until the handler returns before retrieving the `httptest.ResponseRecorder`‘s results.

If you’re interested in performing integration tests, the `net/http/httptest` package includes a test server implementation. For the purposes of this chapter, we’ll use `httptest.NewRequest` and `httptest.NewRecorder`.

### How You Write the Response Matters

Here’s one potential pitfall: the order in which you write to the response body and set the response status code matters. The client receives the response status code first, followed by the response body from the server. If you write the response body first, Go infers that the response status code is 200 OK and sends it along to the client before sending the response body. To see this in action, look at [Listing 9-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-5).

```
package handlers

import (
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestHandlerWriteHeader(t *testing.T) {
    handler := func(w http.ResponseWriter, r *http.Request) {
        _, _ = 1w.Write([]byte("Bad request"))
      2w.WriteHeader(http.StatusBadRequest)
    }
    r := httptest.NewRequest(http.MethodGet, "http://test", nil)
    w := httptest.NewRecorder()
    handler(w, r)
    t.Logf("Response status: %q", 3w.Result().Status)

    handler = func(w http.ResponseWriter, r *http.Request) {
      4w.WriteHeader(http.StatusB)
        _, _ = 5w.Write([]byte("Bad request"))
    }
    r = httptest.NewRequest(http.MethodGet, "http://test", nil)
    w = httptest.NewRecorder()
    handler(w, r)
    t.Logf("Response status: %q", 6w.Result().Status)
}
```

Listing 9-5: Writing the status first and the response body second for expected results (_handlers/pitfall_test.go_)

At first glance, it may seem like the first handler function generates a response status code of 400 Bad Request and the string `Bad request` in the response body. But this isn’t what happens. Calling the `ResponseWriter`’s `Write` method causes Go to make an implicit call to the response’s `WriteHeader` method with `http.StatusOK` for you. Once the response’s status code is set with an explicit or implicit call to `WriteHeader`, you cannot change it.

The Go authors made this design choice because they reasoned you’d need to call `WriteHeader` only for adverse conditions, and in that case, you should do so before you write anything to the response body. Remember, the server sends the response status code before the response body. Once the response’s status code is set with an explicit or implicit call to `WriteHeader`, you cannot change it because it’s likely on its way to the client.

In this example, however, you make a call to the `Write` method 1, which implicitly calls `WriteHeader(http.StatusOK)`. Since the status code is not yet set, the response code is now 200 OK. The next call to `WriteHeader`2 is effectively a no-op because the status code is already set. The response code 200 OK persists 3.

Now, if you switch the order of the calls so you set the status code 4 before you write to the response body 5, the response has the proper status code 6.

Let’s have a look at the test output to confirm that this is the case:

```
=== RUN   TestHandlerWriteHeader
    TestHandlerWriteHeader: pitfall_test.go:17: Response status: "200 OK"
    TestHandlerWriteHeader: pitfall_test.go:26: Response status: "400 Bad Request"
--- PASS: TestHandlerWriteHeader (0.00s)
PASS
```

As you can see from the test output, any writes to the response body before you call the `WriteHeader` method result in a 200 OK status code. The only way to dictate the response status code is to call the `WriteHeader` method before any writes to the response body.

You can improve this code even further by using the `http.Error` function, which simplifies the process of writing a response status code and response body. For example, you could replace your handlers with this single line of code:

```
http.Error(w, "Bad request", http.StatusBadRequest)
```

This function sets the content type to _text/plain_, sets the status code to 400 Bad Request, and writes the error message to the response body.

### Any Type Can Be a Handler

Because `http.Handler` is an interface, you can use it to write powerful constructs for handling client requests. Let’s improve upon the default handler from [Listing 9-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-4) by defining a new type that implements the `http.Handler` interface in [Listing 9-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-6). This type will allow you to appropriately respond to specific HTTP methods and will automatically implement the `OPTIONS` method for you.

```
package handlers

import (
    "fmt"
    "html"
    "io"
    "io/ioutil"
    "net/http"
    "sort"
    "strings"
)

1 type Methods map[string]http.Handler

func (h Methods) 2ServeHTTP(w http.ResponseWriter, r *http.Request) {
 3 defer func(r io.ReadCloser) {
        _, _ = io.Copy(ioutil.Discard, r)
        _ = r.Close()
    }(r.Body)

    if handler, ok := h[r.Method]; ok {
        if handler == nil {
     4 http.Error(w, "Internal server error",
                http.StatusInternalServerError)
        } else {
     5 handler.ServeHTTP(w, r)
        }

        return
    }

 6 w.Header().Add("Allow", h.allowedMethods())
    if r.Method != 7http.MethodOptions {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

func (h Methods) allowedMethods() string {
    a := make([]string, 0, len(h))

    for k := range h {
        a = append(a, k)
    }
    sort.Strings(a)

    return strings.Join(a, ", ")
}
```

Listing 9-6: Methods map that dynamically routes requests to the right handler (_handlers/methods.go_)

The new type, named `Methods`, is a map 1 whose key is an HTTP method and whose value is an `http.Handler`. It has a `ServeHTTP` method 2 to implement the `http.Handler` interface, so you can use `Methods` as a handler itself. The `ServeHTTP` method first defers a function to drain and close the request body 3, saving the map’s handlers from having to do so.

The `ServeHTTP` method looks up the request method in the map and retrieves the handler. To protect us from panics, the `ServeHTTP` method makes sure the corresponding handler is not `nil`, responding with 500 Internal Server Error 4 if it is. Otherwise, you call the corresponding handler’s `ServeHTTP` method 5. The `Methods` type is a multiplexer (router) since it routes requests to the appropriate handler.

If the request method isn’t in the map, `ServeHTTP` responds with the Allow header 6 and a list of supported methods in the map. All that’s left do now is determine whether the client explicitly requested the `OPTIONS`7 method. If so, the `ServeHTTP` method returns, resulting in a 200 OK response to the client. If not, the client receives a 405 Method Not Allowed response.

[Listing 9-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-7) uses the `Methods` handler to implement a better default handler than the one found in [Listing 9-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-4). The old default handler did not automatically add the Allow header and support the `OPTIONS` method. This one will, which makes your job a bit easier. All you need to determine is which methods your `Methods` handler should support, then implement them.

```
--snip--

func DefaultMethodsHandler() http.Handler {
    return Methods{
     1 http.MethodGet: http.HandlerFunc(
            func(w http.ResponseWriter, r *http.Request) {
                _, _ = w.Write([]byte("Hello, friend!"))
            },
        ),
     2 http.MethodPost: http.HandlerFunc(
            func(w http.ResponseWriter, r *http.Request) {
                b, err := ioutil.ReadAll(r.Body)
                if err != nil {
                    http.Error(w, "Internal server error",
                        http.StatusInternalServerError)
                    return
                }

                _, _ = fmt.Fprintf(w, "Hello, %s!",
                    html.EscapeString(string(b)))
            },
        ),
    }
}
```

Listing 9-7: Default implementation of the Methods handler (_methods.go_)

Now, the handler returned by the `handlers.DefaultMethodsHandler` function supports the `GET`, `POST`, and `OPTIONS` methods. The `GET` method simply writes the `Hello, friend!` message to the response body 1. The `POST` method greets the client with the HTML-escaped request body contents 2. The remaining functionality to support the `OPTIONS` method and properly set the Allow header are inherent to the `Methods` type’s `ServeHTTP` method.

The handler returned by the `handlers.DefaultMethodsHandler` function is a drop-in replacement for the handler returned by the `handlers.DefaultHandler` function. You can exchange the following snippet of code from [Listing 9-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-1):

```
Handler: http.TimeoutHandler(handlers.DefaultHandler(), 2*time.Minute, ""),
```

for this code:

```
Handler: http.TimeoutHandler(handlers.DefaultMethodsHandler(), 2*time.Minute, ""),
```

to take advantage of the added functionality provided by the `Methods` handler.

### Injecting Dependencies into Handlers

The `http.Handler` interface gives you access to the request and response objects. But it’s likely you’ll require access to additional functionality like a logger, metrics, cache, or database to handle a request. For example, you may want to inject a logger to record request errors or inject a database object to retrieve data used to create the response. The easiest way to inject an object into a handler is by using a closure.

[Listing 9-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-8) demonstrates how to inject a SQL database object into an `http.Handler`.

```
dbHandler := func(1db *sql.DB) http.Handler {
    return http.HandlerFunc(
        func(w http.ResponseWriter, r *http.Request) {
            err := 2db.Ping()
            // do something with the database here…
        },
    )
}

http.Handle("/three", 3dbHandler(db))
```

Listing 9-8: Injecting a dependency into a handler using a closure

You create a function that accepts a pointer to a SQL database object 1 and returns a handler, then assign it to a variable named `dbHandler`. Since this function closes over the returned handler, you have access to the `db` variable in the handler’s scope 2. Instantiating the handler is as simple as calling `dbHandler` and passing in a pointer to a SQL database object 3.

This approach can get a bit cumbersome if you have multiple handlers that require access to the same database object or your design is evolving and you’re likely to require access to additional objects in the future. A more extensible approach is to use a struct whose fields represent objects and data you want to access in your handler and to define your handlers as struct methods (see [Listing 9-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-9)). Injecting dependencies involves adding struct fields instead of modifying a bunch of closure definitions.

```
type Handlers struct {
    db *sql.DB
  1log *log.Logger
}

func (h *Handlers) Handler1() http.Handler {
    return http.HandlerFunc(
        func(w http.ResponseWriter, r *http.Request) {
            err := h.db.Ping()
            if err != nil {
              2h.log.Printf("db ping: %v", err)
            }
            // do something with the database here
        },
    )
}

func (h *Handlers) Handler2() http.Handler {
    return http.HandlerFunc(
        func(w http.ResponseWriter, r *http.Request) {
            // ...
        },
    )
}
```

Listing 9-9: Injecting dependencies into multiple handlers defined as struct methods

You define a struct that contains pointers to a database object and a logger 1. Any method you define on the handler now has access to these objects 2. If your handlers require access to additional resources, you simply add fields to the struct.

[Listing 9-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-10) illustrates how to use the `Handlers` struct.

```
h := &Handlers{
    db: 1db,
    log: log.New(os.Stderr, "handlers: ", log.Lshortfile),
}
http.Handle("/one", h.Handler1())
http.Handle("/two", h.Handler2())
```

Listing 9-10: Initializing the Handlers struct and using its handlers

Assuming `db`1 is a pointer to a `sql.DB` object, you initialize a `Handlers` object and use its methods with `http.Handle`, for example.

## Middleware

Middleware comprises reusable functions that accept an `http.Handler` and return an `http.Handler`:

```
func(http.Handler) http.Handler
```

You can use middleware to inspect the request and make decisions based on its content before passing it along to the next handler. Or you might use the request content to set headers in the response. For example, the middleware could respond to the client with an error if the handler requires authentication and an unauthenticated client sent the request. Middleware can also collect metrics, log requests, or control access to resources, to name a few uses. Best of all, you can reuse them on multiple handlers. If you find yourself writing the same handler code over and over, ask yourself if you can put the functionality into middleware and reuse it across your handlers.

[Listing 9-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-11) shows just a few uses of middleware, such as enforcing which methods the handler allows, adding headers to the response, or performing ancillary functions like logging.

```
func Middleware(next http.Handler) http.Handler {
    return 1http.HandlerFunc(
     2 func(w http.ResponseWriter, r *http.Request) {
            if r.Method == http.MethodTrace {
              3http.Error(w, "Method not allowed",
                    http.StatusMethodNotAllowed)
            }

          4w.Header().Set("X-Content-Type-Options", "nosniff")

            start := time.Now()
          5next.ServeHTTP(w, r)
          6log.Printf("Next handler duration %v", time.Now().Sub(start))
        },
    )
}
```

Listing 9-11: Example middleware function

The `Middleware` function uses a common pattern you first saw in [Listing 9-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-4): it defines a function that accepts an `http.ResponseWriter` and a pointer to an `http.Request`2 and wraps it with an `http.HandlerFunc`1.

In most cases, middleware calls the given handler 5. But in some cases that may not be proper, and the middleware should block the next handler and respond to the client itself 3. Likewise, you may want to use middleware to collect metrics, ensure specific headers are set on the response 4, or write to a log file 6.

[Listing 9-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-11) is a contrived example. I don’t recommend performing so many tasks in a single middleware function. Instead, it’s best to follow the Unix philosophy and write minimalist middleware, with each function doing one thing very well. Ideally, you would split the middleware in [Listing 9-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-11) into three middleware functions to check the request method and respond to the client 3, enforce response headers 4, and collect metrics 6.

The `net/http` package includes useful middleware to serve static files, redirect requests, and manage request time-outs. Let’s dig into their source code to see how you might use them. In addition to these standard library functions, check out the middleware at [https://go.dev/](https://go.dev/).

### Timing Out Slow Clients

As I mentioned earlier, it’s important not to let clients dictate the duration of a request-response life cycle. Malicious clients could use this leniency to their ends and exhaust your server’s resources, effectively denying service to legit clients. Yet at the same time, setting read and write time-outs server-wide makes it hard for the server to stream data or use different time-out durations for each handler.

Instead, you should manage time-outs in middleware or individual handlers. The `net/http` package includes a middleware function that allows you to control the duration of a request and response on a per-handler basis. The `http.TimeoutHandler` accepts an `http.Handler`, a duration, and a string to write to the response body. It sets an internal timer to the given duration. If the `http.Handler` does not return before the timer expires, the `http.TimeoutHandler` blocks the `http.Handler` and responds to the client with a 503 Service Unavailable status.

[Listing 9-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-12) uses the `http.TimeoutHandler` to wrap an `http.Handler` that mimics a slow client.

```
package middleware

import (
    "io/ioutil"
    "net/http"
    "net/http/httptest"
    "testing"
    "time"
)

func TestTimeoutMiddleware(t *testing.T) {
    handler := 1http.TimeoutHandler(
        http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            w.WriteHeader(http.StatusNoContent)
          2time.Sleep(time.Minute)
        }),
        time.Second,
        "Timed out while reading response",
    )

    r := httptest.NewRequest(http.MethodGet, "http://test/", nil)
    w := httptest.NewRecorder()
    handler.ServeHTTP(w, r)

    resp := w.Result()
    if resp.StatusCode != 3http.StatusServiceUnavailable {
        t.Fatalf("unexpected status code: %q", resp.Status)
    }

    b, err := 4ioutil.ReadAll(resp.Body)
    if err != nil {
        t.Fatal(err)
    }
    _ = resp.Body.Close()

 5 if actual := string(b); actual != "Timed out while reading response" {
        t.Logf("unexpected body: %q", actual)
    }
}
```

Listing 9-12: Giving clients a finite time to read the response (_middleware/timeout_test.go_)

Despite its name, `http.TimeoutHandler` is middleware that accepts an `http.Handler` and returns an `http.Handler`1. The wrapped `http.Handler` purposefully sleeps for a minute 2 to simulate a client’s taking its time to read the response, preventing the `http.Handler` from returning. When the handler doesn’t return within one second, `http.TimeoutHandler` sets the response status code to 503 Service Unavailable 3. The test reads the entire response body 4, properly closes it, and makes sure the response body has the string written by the middleware 5.

### Protecting Sensitive Files

Middleware can also keep clients from accessing information you’d like to keep private. For example, the `http.FileServer` function simplifies the process of serving static files to clients, accepting an `http.FileSystem` interface, and returning an `http.Handler`. The problem is, it won’t protect against serving up potentially sensitive files. Any file in the target directory is fair game. By convention, many operating systems store configuration files or other sensitive information in files and directories prefixed with a period and then hide these dot-prefixed files and directories by default. (This is particularly true in Unix-compatible systems.) But the `http.FileServer` will gladly serve dot-prefixed files or traverse dot-prefixed directories.

The `net/http` package documentation includes an example of an `http.FileSystem` that prevents the `http.FileServer` from serving dot-prefixed files and directories. [Listing 9-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-13) takes a different approach by using middleware to offer the same protection.

```
package middleware

import (
    "net/http"
    "path"
    "strings"
)

func RestrictPrefix(prefix string, next http.Handler) http.Handler {
    return 1http.HandlerFunc(
        func(w http.ResponseWriter, r *http.Request) {
         2 for _, p := range strings.Split(path.Clean(r.URL.Path), "/") {
                if strings.HasPrefix(p, prefix) {
                 3 http.Error(w, "Not Found", http.StatusNotFound)
                    return
                }
            }
          next.ServeHTTP(w, r)
        },
    )
}
```

Listing 9-13: Protecting any file or directory with a given prefix (_middleware/restrict_prefix.go_).

The `RestrictPrefix` middleware 1 examines the URL path 2 to look for any elements that start with a given prefix. If the middleware finds an element in the URL path with the given prefix, it preempts the `http.Handler` and responds with a 404 Not Found status 3.

[Listing 9-14](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-14) uses the `RestrictPrefix` middleware with a series of test cases.

```
package middleware

import (
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestRestrictPrefix(t *testing.T) {
    handler := 1http.StripPrefix("/static/",
      2RestrictPrefix(".", 3http.FileServer(http.Dir("../files/"))),
    )

    testCases := []struct {
        path string
        code int
    }{
      4{"http://test/static/sage.svg", http.StatusOK},
        {"http://test/static/.secret", http.StatusNotFound},
        {"http://test/static/.dir/secret", http.StatusNotFound},
    }

    for i, c := range testCases {
        r := httptest.NewRequest(http.MethodGet, c.path, nil)
        w := httptest.NewRecorder()
        handler.ServeHTTP(w, r)

        actual := w.Result().StatusCode
        if c.code != actual {
            t.Errorf("%d: expected %d; actual %d", i, c.code, actual)
        }
    }
}
```

Listing 9-14: Using the RestrictPrefix middleware (_middleware/restrict_prefix_test.go_)

It’s important to realize the server first passes the request to the `http.StripPrefix` middleware 1, then the `RestrictPrefix` middleware 2, and if the `RestrictPrefix` middleware approves the resource path, the `http.FileServer`3. The `RestrictPrefix` middleware evaluates the request’s resource path to determine whether the client is requesting a restricted path, no matter whether the path exists or not. If so, the `RestrictPrefix` middleware responds to the client with an error without ever passing the request onto the `http.FileServer`.

The static files served by this test’s `http.FileServer` exist in a directory named _files_ in the _restrict_prefix_test.go_ file’s parent directory. Files in the _../files_ directory are in the root of the filesystem passed to the `http.FileServer`. For example, the _../files/sage.svg_ file on the operating system’s filesystem is at _/sage.svg_ in the `http.FileSystem` passed to the `http.FileServer`. If a client wanted to retrieve the _sage.svg_ file from the `http.FileServer`, the request path should be _/sage.svg_.

But the URL path for each of our test cases 4 includes the _/static/_ prefix followed by the static filename. This means that the test requests _static/sage.svg_ from the `http.FileServer`, which doesn’t exist. The test uses another bit of middleware from the `net/http` package to solve this path discrepancy. The `http.StripPrefix` middleware strips the given prefix from the URL path before passing along the request to the `http.Handler`, the `http.FileServer` in this test.

Next, you block access to sensitive files by wrapping the `http.FileServer` with the `RestrictPrefix` middleware to prevent the handler from serving any file or directory prefixed with a period. The first test case results in a 200 OK status, because no element in the URL path has a period prefix. The `http.StripPrefix` middleware removes the _/static/_ prefix from the test case’s URL, changing it from _/static/sage.svg_ to _sage.svg_. It then passes this path to the `http.FileServer`, which finds the corresponding file in its `http.FileSystem`. The `http.FileServer` writes the file contents to the response body.

The second test case results in a 404 Not Found status because the _.secret_ filename has a period as its first character. The third case also results in a 404 Not Found status due to the _.dir_ element in the URL path, because your `RestrictPrefix` middleware considers the prefix of each segment in the path, not just the file.

A better approach to restricting access to resources would be to block all resources by default and explicitly allow select resources. As an exercise, try implementing the inverse of the `RestrictPrefix` middleware by creating middleware that permits requests for only an allowed list of resources.

## Multiplexers

One afternoon, I walked into the University of Michigan’s library, the fourth largest library in the United States. I was looking for a well-worn copy of Kurt Vonnegut’s _Cat’s Cradle_ and had no idea where to start my search. I found the nearest librarian and asked for help finding the book. When we arrived at the correct location, the book was 404 Not Found.

A _multiplexer_, like the friendly librarian routing me to the proper bookshelf, is a general handler that routes a request to a specific handler. The `http.ServeMux` multiplexer is an `http.Handler` that routes an incoming request to the proper handler for the requested resource. By default, `http.ServeMux` responds with a 404 Not Found status for all incoming requests, but you can use it to register your own patterns and corresponding handlers. It will then compare the request’s URL path with its registered patterns, passing the request and response writer to the handler that corresponds to the longest matching pattern.

[Listing 9-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-1) used a multiplexer to send all requests to a single endpoint. [Listing 9-15](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-15) introduces a slightly more complex multiplexer that has three endpoints. This one evaluates the requested resource and routes the request to the right endpoint.

```
package main

import (
    "fmt"
    "io"
    "io/ioutil"
    "net/http"
    "net/http/httptest"
    "testing"
)

1 func drainAndClose(next http.Handler) http.Handler {
    return http.HandlerFunc(
        func(w http.ResponseWriter, r *http.Request) {
          2next.ServeHTTP(w, r)
            _, _ = io.Copy(ioutil.Discard, r.Body)
            _ = r.Body.Close()
        },
    )
}

func TestSimpleMux(t *testing.T) {
    serveMux := http.NewServeMux()
 3 serveMux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusNoContent)
    })
    serveMux.HandleFunc(4"/hello", func(w http.ResponseWriter,
        r *http.Request) {
        _, _ = fmt.Fprint(w, "Hello friend.")
    })
    serveMux.HandleFunc(5"/hello/there/", func(w http.ResponseWriter,
        r *http.Request) {
        _, _ = fmt.Fprint(w, "Why, hello there.")
    })
    mux := drainAndClose(serveMux)
```

Listing 9-15: Registering patterns to a multiplexer and wrapping the entire multiplexer with middleware (_mux_test.go_).

The test creates a new multiplexer and registers three routes using the multiplexer’s `HandleFunc` method 3. The first route is simply a forward slash, showing the default or empty URL path, and a function that sets the 204 No Content status in the response. This route will match all URL paths if no other route matches. The second is _/hello_4, which writes the string `Hello friend.` to the response. The final path is _/hello/there/_5, which writes the string `Why, hello there.` to the response.

Notice that the third route ends in a forward slash, making it a subtree, while the earlier route 4 did not end in a forward slash, making it an absolute path. This distinction tends to be a bit confusing for unaccustomed users. Go’s multiplexer treats absolute paths as exact matches: either the request’s URL path matches, or it doesn’t. By contrast, it treats subtrees as prefix matches. In other words, the multiplexer will look for the longest registered pattern that comes at the beginning of the request’s URL path. For example, _/hello/there/_ is a prefix of _/hello/there/you_ but not of _/hello/you_.

Go’s multiplexer can also redirect a URL path that doesn’t end in a forward slash, such as _/hello/there_. In those cases, the `http.ServeMux` first attempts to find a matching absolute path. If that fails, the multiplexer appends a forward slash, making the path _/hello/there/_, for example, and responds to the client with it. This new path becomes a permanent redirect. You’ll see an example of this in [Listing 9-16](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-16).

Now that you’ve defined routes for the multiplexer, it’s ready to use. But there’s one issue with the handlers: none of them drain and close the request body. This isn’t a big concern in a test like this, but you should stick to best practices, nonetheless. If you don’t do so in a real scenario, you may cause increased overhead and potential memory leaks. Here, you use middleware 1 to drain and close the request body. In the `drainAndClose` middleware, you call the `next` handler first 2 and then drain and close the request body. There is no harm in draining and closing a previously drained and closed request body.

[Listing 9-16](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-16) tests a series of requests against [Listing 9-15](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-15)’s multiplexer.

```
--snip--

    testCases := []struct {
        path     string
        response string
        code     int
    }{
     1 {"http://test/", "", http.StatusNoContent},
        {"http://test/hello", "Hello friend.", http.StatusOK},
        {"http://test/hello/there/", "Why, hello there.", http.StatusOK},
     2 {"http://test/hello/there",
            "<a href=\"/hello/there/\">Moved Permanently</a>.\n\n",
            http.StatusMovedPermanently},
     3 {"http://test/hello/there/you", "Why, hello there.", http.StatusOK},
     4 {"http://test/hello/and/goodbye", "", http.StatusNoContent},
        {"http://test/something/else/entirely", "", http.StatusNoContent},
        {"http://test/hello/you", "", http.StatusNoContent},
    }

    for i, c := range testCases {
        r := httptest.NewRequest(http.MethodGet, c.path, nil)
        w := httptest.NewRecorder()
        mux.ServeHTTP(w, r)
        resp := w.Result()

        if actual := resp.StatusCode; c.code != actual {
            t.Errorf("%d: expected code %d; actual %d", i, c.code, actual)
        }

        b, err := 5ioutil.ReadAll(resp.Body)
        if err != nil {
            t.Fatal(err)
        }
        _ = 6resp.Body.Close()

        if actual := string(b); c.response != actual {
            t.Errorf("%d: expected response %q; actual %q", i,
                c.response, actual)
        }
    }
}
```

Listing 9-16: Running through a series of test cases and verifying the response status code and body (_mux_test.go_).

The first three test cases 1, including the request for the _/hello/there/_ path, match exact patterns registered with the multiplexer. But the fourth test case 2 is different. It doesn’t have an exact match. When the multiplexer appends a forward slash to it, however, it discovers that it exactly matches a registered pattern. Therefore, the multiplexer responds with a 301 Moved Permanently status and a link to the new path in the response body. The fifth test case 3 matches the _/hello/there/_ subtree and receives the `Why, hello there.` response. The last three test cases 4 match the default path of _/_ and receive the 204 No Content status.

Just as the test relies on middleware to drain and close the request body, it drains 5 and closes 6 the response body.

## HTTP/2 Server Pushes

The Go HTTP server can push resources to clients over HTTP/2, a feature that has the potential to improve efficiency. For example, a client may request the home page from a web server, but the client won’t know it needs the associated style sheet and images to properly render the home page until it receives the HTML in the response. An HTTP/2 server can proactively send the style sheet and images along with the HTML in the response, saving the client from having to make subsequent calls for those resources. But server pushes have the potential for abuse. This section shows you how to use server pushes and then discusses cases when you should avoid doing so.

### Pushing Resources to the Client

Let’s retrieve the HTML page in [Listing 9-17](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-17) over HTTP/1.1, then retrieve the same page over HTTP/2 and compare the differences.

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>H2 Server Push</title>
 1 <link href="/static/style.css" rel="stylesheet">
</head>
<body>
 2 <img src="/static/hiking.svg" alt="hiking gopher">
</body>
</html>
```

Listing 9-17: Simple index file having links to two resources (_files/index.html_)

This HTML file requires the browser to retrieve two more resources, a style sheet 1 and an SVG image 2, to properly show the entire page. [Figure 9-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#figure9-2) shows Google Chrome’s request accounting for the HTML when served using HTTP/1.1.

![f09002](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c09/f09002.png)

Figure 9-2: Downloaded index page and associated resources over HTTP/1.1

Aside from the _favicon.ico_ file, which Chrome retrieves on its own, the browser made three requests to retrieve all required resources—one for the HTML file, one for the style sheet, and one for the SVG image. Any web browser requesting the _index.html_ file (_localhost_ in [Figure 9-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#figure9-2)) will also request the _style.css_ and _hiking.svg_ files to properly render the _index.html_ file. The web server could improve efficiency and proactively push these two files to the web browser, since it knows the web browser will inevitably request them. This proactive approach by the web server would save the web browser from the overhead of having to make two more requests.

[Figure 9-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#figure9-3) shows the same retrieval using HTTP/2. In this case, the server pushes the _style.css_ and _hiking.svg_ files.

![f09003](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c09/f09003.png)

Figure 9-3: Downloaded index page with resources pushed by the server side

The client receives all three resources after a single request to the server for the _index.html_ file. The Initiator column in [Figure 9-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#figure9-3) shows that Chrome retrieved the resources from its dedicated push cache.

Let’s write a command line executable that can push resources to clients. [Listing 9-18](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-18) shows the first part of the program.

```
package main

import (
    "context"
    "flag"
    "log"
    "net/http"
    "os"
    "os/signal"
    "path/filepath"
    "time"

    "github.com/awoodbeck/gnp/ch09/handlers"
    "github.com/awoodbeck/gnp/ch09/middleware"
)

var (
    addr  = flag.String("listen", "127.0.0.1:8080", "listen address")
 1 cert  = flag.String("cert", "", "certificate")
 2 pkey  = flag.String("key", "", "private key")
    files = flag.String("files", "./files", "static file directory")
)

func main() {
    flag.Parse()

    err := 3run(*addr, *files, *cert, *pkey)
    if err != nil {
        log.Fatal(err)
    }

    log.Println("Server gracefully shutdown")
}
```

Listing 9-18: Command line arguments for the HTTP/2 server (_server.go_)

The server needs the path to a certificate 1 and a corresponding private key 2 to enable TLS support and allow clients to negotiate HTTP/2 with the server. If either value is empty, the server will listen for plain HTTP connections. Next, pass the command line flag values to a `run` function 3.

The `run` function, defined in [Listing 9-19](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-19), has the bulk of your server’s logic and ultimately runs the web server. Breaking this functionality into a separate function eases unit testing later.

```
--snip--

func run(addr, files, cert, pkey string) error {
    mux := http.NewServeMux()
 1 mux.Handle("/static/",
        http.StripPrefix("/static/",
            middleware.RestrictPrefix(
                ".", http.FileServer(http.Dir(files)),
            ),
        ),
    )
 2 mux.Handle("/",
        handlers.Methods{
            http.MethodGet: http.HandlerFunc(
                func(w http.ResponseWriter, r *http.Request) {
                 3 if pusher, ok := w.(http.Pusher); ok {
                        targets := []string{
                          4"/static/style.css",
                            "/static/hiking.svg",
                        }
                        for _, target := range targets {
                            if err := 5pusher.Push(target, nil); err != nil {
                                log.Printf("%s push failed: %v", target, err)
                            }
                        }
                    }

                 6 http.ServeFile(w, r, filepath.Join(files, "index.html"))
                },
            ),
        },
    )
 7 mux.Handle("/2",
        handlers.Methods{
            http.MethodGet: http.HandlerFunc(
                func(w http.ResponseWriter, r *http.Request) {
                    http.ServeFile(w, r, filepath.Join(files, "index2.html"))
                },
            ),
        },
    )
```

Listing 9-19: Multiplexer, middleware, and handlers for the HTTP/2 server (_server.go_)

The server’s multiplexer has three routes: one for static files 1, one for the default route 2, and one for the _/2_ absolute path 7. If the `http.ResponseWriter` is an `http.Pusher`3, it can push resources to the client 5 without a corresponding request. You specify the path to the resource from the client’s perspective 4, not the file path on the server’s filesystem because the server treats the request as if the client originated it to facilitate the server push. After you’ve pushed the resources, you serve the response for the handler 6. If, instead, you sent the _index.html_ file before pushing the associated resources, the client’s browser may send requests for the associated resources before it handles the pushes.

Web browsers cache HTTP/2-pushed resources for the life of the connection and make it available across routes. Therefore, if the _index2.html_ file served by the _/2_ route 7 references the same resources pushed by the default route, and the client first visits the default route, the client’s web browser may use the pushed resources when rendering the _/2_ route.

You have one more task to complete: instantiate an HTTP server to serve your resources. [Listing 9-20](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-20) does this by making use of the multiplexer.

```
--snip--

    srv := &http.Server{
        Addr:              addr,
        Handler:           mux,
        IdleTimeout:       time.Minute,
        ReadHeaderTimeout: 30 * time.Second,
    }

    done := make(chan struct{})
    go func() {
        c := make(chan os.Signal, 1)
        signal.Notify(c, os.Interrupt)

        for {
         1 if <-c == os.Interrupt {
             2 if err := srv.Shutdown(context.Background()); err != nil {
                    log.Printf("shutdown: %v", err)
                }
                close(done)
                return
            }
        }
    }()

    log.Printf("Serving files in %q over %s\n", files, srv.Addr)

    var err error
    if cert != "" && pkey != "" {
        log.Println("TLS enabled")
     3 err = srv.ListenAndServeTLS(cert, pkey)
    } else {
     4 err = srv.ListenAndServe()
    }

    if err == http.ErrServerClosed {
        err = nil
    }

    <-done

    return err
}
```

Listing 9-20: HTTP/2-capable server implementation (_server.go_)

When the server receives an `os.Interrupt` signal 1, it triggers a call to the server’s `Shutdown` method 2. Unlike the server’s `Close` method, which abruptly closes the server’s listener and all active connections, `Shutdown` gracefully shuts down the server. It instructs the server to stop listening for incoming connections and blocks until all client connections end. This gives the server the opportunity to finish sending responses before stopping the server.

If the server receives a path to both a certificate and a corresponding private key, the server will enable TLS support by calling its `ListenAndServeTLS` method 3. If it cannot find or parse either the certificate or private key, this method returns an error. In the absence of these paths, the server uses its `ListenAndServe` method 4.

Go ahead and test this server. As mentioned in Chapter 8, Go doesn’t include the support needed to test the server’s push functionality with code, but you can interact with this program by using your web browser.

### Don’t Be Too Pushy

Although HTTP/2 server pushing can improve the efficiency of your communications, it can do just the opposite if you aren’t careful. Remember that web browsers store pushed resources in a separate cache for the lifetime of the connection. If you’re serving resources that don’t change often, the web browser will likely already have them in its regular cache, so you shouldn’t push them. Once it caches them, the browser can use them for future requests spanning many connections. You probably shouldn’t push the resources in [Listing 9-19](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c09.xhtml#listing9-19), for instance, because they’re unlikely to change often.

My advice is to be conservative with server pushes. Use your handlers and rely on metrics to figure out when you should push a resource. If you do push resources, do so before writing the response.

## What You’ve Learned

Go’s `net/http` package includes a capable server implementation. In this chapter, you used its handlers, middleware, multiplexer, and HTTP/2 support to process client requests intelligently and efficiently.

Go’s `http.Handler` is an interface that describes an object capable of accepting a request and responding with a status code and payload. A special handler, known as a _multiplexer_, can parse a request and pass it along to the most proper handler, effectively functioning as a request router. _Middleware_ is code that augments the behavior of handlers or performs auxiliary tasks. It might change the request, add headers to the response, collect metrics, or preempt the handler, to name a few use cases. Finally, Go’s server supports HTTP/2 over TLS. When it uses HTTP/2, the server can push resources to clients, potentially making the communication more efficient.

By putting these features together, you can build comprehensive, useful HTTP-based applications with surprisingly little code.