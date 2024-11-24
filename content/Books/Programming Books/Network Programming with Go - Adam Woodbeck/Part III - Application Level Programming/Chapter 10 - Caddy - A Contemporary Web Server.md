---
id: 01JCH9DPMZ23AXGNEKQDJJ1S4F
modified: 2024-11-12T17:57:23-05:00
title: Chapter 10 - Caddy - A Contemporary Web Server
---
# 10  
CADDY: A CONTEMPORARY WEB SERVER

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

Chapter 9 focused on the web service building blocks available to you in Go’s standard library. You learned how to create a simple web server with relatively little code by using handlers, middleware, and multiplexers. Although you can build a capable web server with those tools alone, writing your own server from scratch may not always be the quickest approach. Adding support for logging, metrics, authentication, access control, and encryption, to name a few features, can be daunting and hard to get right. Instead, you may find it more convenient to use an existing, comprehensive web server to host your web services.

This chapter will introduce you to the Caddy web server and show you how to focus your time on writing web services while relying on Caddy to serve your application. You’ll get Caddy up and running and then take a dive into its real-time configuration API. Next, you’ll learn how to extend Caddy’s functionality by using custom modules and configuration adapters. You’ll then use Caddy to serve your application’s static files and proxy requests to your web services. Finally, you’ll learn about Caddy’s automatic TLS support by using free certificates from Let’s Encrypt and automated key management.

After reading this chapter, you should feel comfortable choosing the best solution for your web applications: either a simple `net/http`-based web server or a comprehensive solution like Caddy.

## What Is Caddy?

_Caddy_ is a contemporary web server that focuses on security, performance, and ease of use. Among its hallmark features, it offers automatic TLS certificate management, allowing you to easily implement HTTPS. Caddy also takes advantage of Go’s concurrency primitives to serve a considerable amount of all web traffic. It’s one of the few open source projects with enterprise-grade support.

### Let’s Encrypt Integration

_Let’s Encrypt_ is a nonprofit certificate authority that supplies digital certificates free of charge for the public to facilitate HTTPS communication. Let’s Encrypt certificates run on more than half of all websites on the internet, and they’re trusted by all popular web browsers. You can retrieve certificates for your website by using Let’s Encrypt’s automated issuance and renewal protocol, known as _Automated Certificate Management Environment (ACME)_.

Typically, getting a certificate requires three steps: a certificate request, domain validation, and certificate issuance. First, you request a certificate for your domain from Let’s Encrypt. Let’s Encrypt then confirms your domain to make sure you administer it. Once Let’s Encrypt has ensured that you’re the domain’s rightful owner, it issues you a certificate, which your web server can use for HTTPS support. Each certificate is good for 90 days, though you should renew it every 60 days to prevent service interruption.

Caddy has inherent support for the ACME protocol and will automatically request, validate, and install Let’s Encrypt certificates if Caddy can properly derive the domain names it hosts. We’ll discuss how best to do this in “Adding Automatic HTTPS” on page 237. Caddy also handles automatic renewals, eliminating the need for you to keep track of certificate expiration dates.

### How Does Caddy Fit into the Equation?

Caddy works just like other popular web servers, such as NGINX and Apache. It’s best positioned on the edge of your network, between web clients and your web services, as shown in [Figure 10-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#figure10-1).

![f10001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c10/f10001.png)

Figure 10-1: Caddy reverse-proxying client requests to web services

Caddy can serve static files and forward requests between clients and backend services, a process known as _reverse proxying_. In this example, you can see Caddy serving a WordPress blog through PHP’s FastCGI Process Manager (PHP-FPM), static files, and a Go-based web service. We’ll replicate a similar setup later in the chapter, sans WordPress blog.

Caddy helps abstract web services from clients in much the same way we use abstraction in our code. If you use Caddy’s automatic TLS, static file server, data compression, access control, and logging features, you won’t have to add that functionality to each web service. In addition, using Caddy has the benefit of allowing you to abstract your network topography from clients. As the services increase in popularity and the capacity on web services starts to negatively affect clients, you can add web services to Caddy and instruct Caddy to balance the load among them all, without interruption to your clients.

## Retrieving Caddy

In this chapter, we’ll use version 2 of Caddy. You have a few options for installation, described in this section.

### Downloading Caddy

You can install Caddy by using a static binary, built by the Caddy team. This binary is available through the download link at [https://caddyserver.com/](https://caddyserver.com/)_._

Caddy is also available as a Docker image; a DigitalOcean droplet; an Advanced Package Tool (APT) source for Debian derivatives; and in the Fedora Copr build system for use in Fedora, CentOS, or Red Hat Enterprise Linux. You can find details in the Install documentation at [https://caddyserver.com/docs/download](https://caddyserver.com/docs/download).

### Building Caddy from Source Code

If you do not find a suitable static binary for your operating system and architecture, or if you wish to customize Caddy, you can also compile Caddy from source code.

Caddy relies heavily on Go’s support for modules. Therefore, you need to use at least Go 1.14 before running the following commands:

```
$ git clone "https://github.com/caddyserver/caddy.git"
Cloning into 'caddy'...
$ cd caddy/cmd/caddy
$ go build
```

Clone the Caddy Git repository and change to the _caddy/cmd/caddy_ subdirectory, where you’ll find the `main` package. Run `go build` to create a binary named _caddy_ in the current directory for your operating system and architecture. To simplify commands, the rest of this chapter assumes that the _caddy_ binary is in your `PATH`.

While you’re in this subdirectory, make note of the _main.go_ file. You’ll revisit it later in this chapter when you learn how to customize Caddy by adding modules.

## Running and Configuring Caddy

For configuration purposes, Caddy exposes an administration endpoint on TCP port 2019, over which you can interact with Caddy’s configuration in real time. You can configure Caddy by posting JSON to this endpoint, and you can read the configuration with a `GET` request. Caddy’s full JSON API documentation is available at [https://caddyserver.com/docs/json/](https://caddyserver.com/docs/json/).

Before you can configure Caddy, you need to start it. Running this command starts Caddy as a background process:

```
$ caddy start
2006/01/02 15:04:05.000 INFO    admin   endpoint started
{"address": "tcp/localhost:2019", "enforce_origin": false,
"origins": ["localhost:2019", "[::1]:2019", "127.0.0.1:2019"]}
2006/01/02 15:04:05.000 INFO    serving initial configuration
Successfully started Caddy (pid=24587) - Caddy is running in the background
```

You’ll see log entries showing that the admin endpoint started and Caddy is using the initial configuration. You’ll also see log entries printed to standard output as you interact with the admin endpoint.

Caddy’s configuration is empty by default. Let’s send meaningful configuration data to Caddy. [Listing 10-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-1) uses the `curl` command to post JSON to the `load` resource on Caddy’s admin endpoint.

```
$ curl localhost:2019/load \
1 -X POST -H "Content-Type: application/json" \ 
-d '
{
  "apps": {
    "http": {
      "servers": {
        "hello": {
          "listen": ["localhost:2020"],
       2 "routes": [{
              "handle": [{
           3 "handler": "static_response",
                  "body": "Hello, world!"
              }]
          }]
        }
      }
    }
  }
}'
```

Listing 10-1: Posting configuration to Caddy’s admin endpoint

You send a `POST` request containing JSON in the request body 1 to the `load` resource of the Caddy instance listening on port 2019. The top-level `apps` namespace lists the applications Caddy will load at runtime. In this case, you’re telling Caddy to load the `http` application. The `http` application configuration consists of one or more servers. This example sets up a single server named `hello` listening on localhost port 2020. Feel free to name your server whatever you’d like.

Since the `listen` value is an array of addresses, you can configure this server to listen to more than one socket address. Caddy passes these address values to `net.Listen`, just as you did in Chapter 3. You also have the option of specifying a port range, such as _localhost:2020-2025_. Caddy will recognize that you used a range and properly extrapolate the range into separate socket addresses. Caddy allows you to restrict listeners to specific network types by prefixing the socket address. For example, _udp/localhost:2020_ tells the server to bind to UDP port 2020 on localhost. The forward slash is not part of the address but rather a separator. If you want the server to bind to a Unix socket _/tmp/caddy.sock_, specify the address _unix//tmp/caddy.sock_.

The `hello` server’s `routes` value 2 is an array of routes, like the multiplexer from the preceding chapter, which dictates how the server will handle incoming requests. If a route matches the request, Caddy passes the request onto each handler in the `handle` array. Since `handle` is an array, you can specify more than one handler per route. Caddy will pass the request to each successive handler in the same way you chained middleware together in the preceding chapter. In this example, you specify a single route to match all requests and add a single handler to this route. You’re using the built-in `static_response` handler 3, which will write the value of the `body` (`Hello, world!` in this example) in the response body.

Provided there are no errors in the configuration, Caddy will at once start using the new configuration. Let’s confirm Caddy is now listening on both the administrative port 2019 and your `hello` server port 2020:

```
$ lsof -Pi :2019-2025
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
caddy   24587 user    3u  IPv4 811511      0t0  TCP localhost:2019 (LISTEN)
caddy   24587 user    9u  IPv4 915742      0t0  TCP localhost:2020 (LISTEN)
```

Looks good. This command won’t work on Windows. Instead, you can see similar output by running the `netstat -b` command in an Administrator command prompt. Now, you can ask Caddy for its configuration by sending a `GET` request:

```
$ curl localhost:2019/config/
{"apps":{"http":{"servers":{"hello":{"listen":["localhost:2020"],
"routes":[{"handle":[{"body":"Hello, world!","handler":"static_response"}]}]}}}}}
```

Caddy returns its JSON-formatted configuration in the response body. Note that you need to write the trailing slash on the `/config/` resource, because `/config/` is the resource prefix under which Caddy exposes its configuration. You are asking Caddy for all resources found under the /`config/` prefix. If you accidentally omit the trailing slash, Caddy thinks you’re asking for an absolute resource named `/config`, which doesn’t exist in Caddy’s admin API on port 2019.

Caddy supports _configuration traversal_. Configuration traversal lets you request a subset of the configuration by treating each JSON key in the configuration data as a resource address. For example, you can request the `listen` value for the `hello` server from our example configuration by issuing a `GET` request, like this:

```
$ curl localhost:2019/config/apps/http/servers/hello/listen
["localhost:2020"]
```

Caddy returns a JSON array containing _localhost:2020_, just as you’d expect. Let’s send a `GET` request to this socket address:

```
$ curl localhost:2020
Hello, world!
```

You see the `Hello, world!` string returned from the `static_response` handler.

### Modifying Caddy’s Configuration in Real Time

You can use the other HTTP verbs you learned in Chapter 8 to modify your server’s configuration. Any changes you make will take immediate effect, so long as Caddy can parse the JSON you send. If Caddy fails to parse the JSON, or if a fundamental error exists in the new configuration, Caddy will log an error with an explanation of what went wrong and continue to use its existing configuration.

Let’s say you want to make your `hello` server listen on port 2021 as well. You can append another `listen` value by using a `POST` request and immediately check that the change took effect:

```
$ curl localhost:2019/config/apps/http/servers/hello/listen \
-X POST -H "Content-Type: application/json" -d '"localhost:2021"'
$ lsof -Pi :2019-2025
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
caddy   24587 user    3u  IPv4  811511      0t0  TCP localhost:2019 (LISTEN)
caddy   24587 user    9u  IPv4  915742      0t0  TCP localhost:2020 (LISTEN)
1 caddy   24587 user   11u  IPv4 1148212      0t0  TCP localhost:2021 (LISTEN)
```

You can see that Caddy is now listening on port 2021 1 in addition to ports 2019 and 2020.

Suppose you want to replace the listening addresses and use a range instead. For that, you can send a `PATCH` request with the new `listen` array value you want Caddy to use:

```
$ curl localhost:2019/config/apps/http/servers/hello/listen \
-X PATCH -H "Content-Type: application/json" -d '["localhost:2020-2025"]'
$ lsof -Pi :2019-2025
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
caddy   24587 user    3u  IPv4  811511      0t0  TCP localhost:2019 (LISTEN)
1 caddy   24587 user    9u  IPv4  915742      0t0  TCP localhost:2020 (LISTEN)
caddy   24587 user   10u  IPv4 1149557      0t0  TCP localhost:2021 (LISTEN)
caddy   24587 user   11u  IPv4 1166333      0t0  TCP localhost:2022 (LISTEN)
caddy   24587 user   12u  IPv4 1169409      0t0  TCP localhost:2023 (LISTEN)
caddy   24587 user   13u  IPv4 1169413      0t0  TCP localhost:2024 (LISTEN)
2 caddy   24587 user   14u  IPv4 1169417      0t0  TCP localhost:2025 (LISTEN)
```

In addition to the admin port 2019, Caddy is now listening on ports 2020 1 through 2025 2.

Although you may not find yourself changing Caddy’s configuration on the fly very often, it’s a handy feature for development, because it lets you quickly spin up a new server to add functionality. Let’s add a new server to Caddy while it’s running. You’ll name this new server `test` and configure it to listen on port 2030. [Listing 10-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-2) adds the new `test` server to Caddy in real time.

```
$ curl localhost:2019/config/apps/http/servers/test \
-X POST -H "Content-Type: application/json" \
-d '{
  "listen": ["localhost:2030"],
  "routes": [{
    "handle": [{
      "handler": "static_response",
      "body": "Welcome to my temporary test server."
    }]
  }]
}'
```

Listing 10-2: Adding a new server to Caddy in real time

The name of the new server, `test`, is part of the resource you `POST` to. You can think of `test` as the key and the JSON in the request body as the value, if you defined this server in the original configuration from [Listing 10-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-1). At this point, Caddy has two servers: `hello` listening on ports 2020 to 2025 and `test` listening on port 2030. To confirm Caddy is serving `test`, you can check the new endpoint on port 2030:

```
$ curl localhost:2030
Welcome to my temporary test server.
```

The `static_response` handler properly responds with the expected message. If you want to remove the `test` server, it’s as simple as issuing a `DELETE` request:

```
$ curl localhost:2019/config/apps/http/servers/test -X DELETE
```

Here again, you specify the `test` server in the resource. Caddy is no longer listening on localhost port 2030, and the test server no longer exists. You were able to stand up a new server to handle entirely different requests without interrupting the functionality of your `hello` server. Changing the configuration in real time opens possibilities. Do you want a server or route to be accessible only certain times of the day? No problem. Do you want to temporarily redirect traffic without having to bounce your entire web server, interrupting existing web traffic? Sure, go ahead.

### Storing the Configuration in a File

We typically provide Caddy with its configuration as part of the startup process. Write the JSON configuration from [Listing 10-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-1) to a file named _caddy.json_. Then start Caddy by using the following command:

```
$ caddy start --config caddy.json
Successfully started Caddy (pid=46112) - Caddy is running in the background
$ curl localhost:2019/config/
{"apps":{"http":{"servers":{"hello":{"listen":["localhost:2020"],
"routes":[{"handle":[{"body":"Hello, world!","handler":"static_response"}]}]}}}}}
```

Caddy starts in the background, as in [Listing 10-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-1)—but this time, it populates its configuration from the _caddy.json_ file during initialization.

## Extending Caddy with Modules and Adapters

Caddy uses a modular architecture to organize its functionality. This modular approach allows you to extend Caddy’s capabilities by writing your own modules and configuration adapters. In this section, we’ll walk through the process of writing a configuration adapter that will allow you to store your Caddy configuration in a Tom’s Obvious, Minimal Language (TOML) file. We’ll also replicate the `restrict_prefix` middleware from the preceding chapter in a proper Caddy module.

### Writing a Configuration Adapter

Although JSON is a perfectly good format for configuration files, it isn’t as well suited for human consumption as other formats. JSON lacks support for comments and multiline strings, two characteristics that make configuration files easier for people to read. Caddy supports the use of _configuration adapters_ that adapt one format, such as TOML, to Caddy’s native JSON format. TOML is a configuration file format that is easy for humans to read. It supports both comments and multiline strings. You can find more details at [https://github.com/toml-lang/toml/tree/v0.5.0/](https://github.com/toml-lang/toml/tree/v0.5.0/).

Caddy version 1 supported a custom configuration file format named _Caddyfile_, which was also the name of the configuration file by convention. If you want to use Caddyfile with Caddy v2, you must rely on a configuration adapter so Caddy can ingest it. Caddy is smart enough to know it needs to use the `caddyfile` adapter when you specify a filename that starts with _Caddyfile_. But to specify an adapter from the command line, you explicitly tell Caddy which adapter to use:

```
$ caddy start --config Caddyfile --adapter caddyfile
```

The `adapter` flag tells Caddy which adapter it should use. Caddy will invoke the adapter to adapt the configuration file to JSON and then parse the JSON returned by the adapter as if you had presented the configuration in JSON format in the first place.

But Caddy doesn’t ship with an official configuration adapter for TOML, so let’s take a crack at writing one. You need to first create a Go module for your TOML configuration adapter:

```
$ mkdir caddy-toml-adapter
$ cd caddy-toml-adapter
1 $ go mod init github.com/awoodbeck/caddy-toml-adapter
go: creating new go.mod: module github.com/awoodbeck/caddy-toml-adapter
```

You should use a fully qualified module name 1 different from the one used here. I created this module on GitHub under my _awoodbeck_ account. The fully qualified name for your module will differ depending on where, and under what account, it’s hosted.

Now that you’ve created a module, you can write the code. Create a file in the current directory named _toml.go_ and add the code in [Listing 10-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-3).

```
package tomladapter

import (
    "encoding/json"

    "github.com/caddyserver/caddy/v2/caddyconfig"
    "github.com/pelletier/go-toml"
)

func init() {
    caddyconfig.RegisterAdapter(1"toml", 2Adapter{})
}

// Adapter converts a TOML Caddy configuration to JSON.
type Adapter struct{}

// Adapt the TOML body to JSON.
func (a Adapter) Adapt(body []byte, _ map[string]interface{}) (
    []byte, []caddyconfig.Warning, error) {
    tree, err := 3toml.LoadBytes(body)
    if err != nil {
        return nil, nil, err
    }

    b, err := json.Marshal(4tree.ToMap())

    return b, nil, err
}
```

Listing 10-3: Creating a TOML configuration adapter and registering it with Caddy

You use Thomas Pelletier’s _go-toml_ library to parse the configuration file contents 3. This saves a considerable amount of code. You then convert the parsed TOML into a map 4 and marshal the map to JSON.

The last bit of accounting is to register your configuration adapter with Caddy. For this, you include a call to `caddyconfig.RegisterAdapter` in the `init` function and pass it the adapter’s type 1 and an `Adapter` object 2 implementing the `caddyconfig.Adapter` interface. When you import this module from Caddy’s _main.go_ file, the configuration adapter registers itself with Caddy, adding support for parsing the TOML configuration file. You’ll look at a concrete example of importing this module from Caddy in “Injecting Your Module into Caddy” on page 231.

Now that you’ve created the _toml.go_ file, tidy up the module:

```
$ go mod tidy
go: finding module for package github.com/caddyserver/caddy/v2/caddyconfig
go: found github.com/caddyserver/caddy/v2/caddyconfig in
github.com/caddyserver/caddy/v2 v2.0.0
```

This command adds the Caddy dependency to the _go.mod_ file. All that’s left to do is to publish your module to GitHub, as in this example, or another suitable version-control system supported by `go get`.

### Writing a Restrict Prefix Middleware Module

Chapter 9 introduced the concept of middleware, a design pattern that allows your code to manipulate a request and a response and to perform ancillary tasks when the server receives a request, such as logging request details. Let’s explore how to use middleware with Caddy.

In Go, middleware is a function that accepts an `http.Handler` and returns an `http.Handler`:

```
func(http.Handler) http.Handler
```

An `http.Handler` describes an object with a `ServeHTTP` method that accepts an `http.RequestWriter` and an `http.Request`:

```
type Handler interface {
    ServeHTTP(http.ResponseWriter, *http.Request)
}
```

The handler reads from the request and writes to the response. Assuming `myHandler` is an object that implements the `http.Handler` interface, and `middleware1`, `middleware2`, and `middleware3` all accept an `http.Handler` and return an `http.Handler`, you can apply the middleware functions to `myHandler` in [Listing 10-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-4).

```
h := middleware1(middleware2(middleware3(myHandler)))
```

Listing 10-4: Multiple middleware functions wrapping a handler

You can replace any of the middleware functions with the `RestrictPrefix` middleware you wrote in the preceding chapter, since it’s a function that accepts an `http.Handler` and returns an `http.Handler`.

Unfortunately for us, Caddy’s middleware does not use this design pattern, so it cannot use `RestrictPrefix`. Caddy includes interfaces for both handlers and middleware, unlike `net/http`, which describes only handlers. Caddy’s equivalent of the `http.Handler` interface is `caddyhttp.Handler`:

```
type Handler interface {
    ServeHTTP(http.ResponseWriter, *http.Request) error
}
```

The only difference between `caddyhttp.Handler` and `http.Handler` is that the former’s `ServeHTTP` method returns an `error` interface.

Caddy middleware is a special type of handler that implements the `caddyhttp.MiddlewareHandler` interface:

```
type MiddlewareHandler interface {
    ServeHTTP(http.ResponseWriter, *http.Request, Handler) error
}
```

Like `caddyhttp.Handler`, Caddy’s middleware accepts both an `http.ResponseWriter` and an `http.Request`, and it returns an `error` interface. But it accepts an additional argument: the `caddyhttp.Handler`, downstream from the middleware in the same way that `myHandler` is downstream from `middleware3` in [Listing 10-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-4). Instead of accepting an `http.Handler` and returning an `http.Handler`, Caddy expects its middleware to act as handlers, with access to the `caddyhttp.Handler` that should receive the request and response after the middleware is done with them.

Let’s create a new Caddy module that replicates the functionality of your `RestrictPrefix` middleware:

```
$ mkdir caddy-restrict-prefix
$ cd caddy-restrict-prefix
$ go mod init github.com/awoodbeck/caddy-restrict-prefix
go: creating new go.mod: module github.com/awoodbeck/caddy-restrict-prefix
```

As before, your fully qualified module name will differ from mine. Create a new file named _restrict_prefix.go_ and add the code from [Listing 10-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-5) to the file.

```
package restrictprefix

import (
    "fmt"
    "net/http"
    "strings"

    "github.com/caddyserver/caddy/v2"
    "github.com/caddyserver/caddy/v2/modules/caddyhttp"
    "go.uber.org/zap"
)

func init() {
  1caddy.RegisterModule(RestrictPrefix{})
}

// RestrictPrefix is middleware that restricts requests where any portion
// of the URI matches a given prefix.
type RestrictPrefix struct {
  2Prefix string `json:"prefix,omitempty"`
  3logger *zap.Logger
}

// CaddyModule returns the Caddy module information.
func (RestrictPrefix) 4CaddyModule() caddy.ModuleInfo {
    return caddy.ModuleInfo{
      5ID:  "http.handlers.restrict_prefix",
      6New: func() caddy.Module { return new(RestrictPrefix) },
    }
}
```

Listing 10-5: Defining and registering a new Caddy module

The `RestrictPrefix` middleware implementation from the preceding chapter expected the prefix of a URL path as a string. Here, you’re storing the prefix in the `RestrictPrefix` struct 2 and assigning it a struct tag to use the `json.Unmarshal` behavior of matching incoming keys to struct tags. The struct tag tells `json.Unmarshal` which JSON key corresponds to this field. In this example, you’re telling `json.Unmarshal` that it should take the value associated with the `prefix` key in the JSON configuration and assign it to the struct’s `Prefix` field. The `RestrictPrefix` struct also has a `logger` field 3 so you can log events, as necessary.

Your module needs to register itself with Caddy upon initialization 1. The `caddy.RegisterModule` function accepts any object that implements the `caddy.Module` interface. For that, you add the `CaddyModule` method 4 to return information to Caddy about your module. Caddy requires an ID 5 for each module. Since you’re creating an HTTP middleware handler, you’ll use the ID `http.handler.restrict_prefix`, where `restrict_prefix` is the unique name of your module. Caddy also expects a function 6 that can create a new instance of your module.

Now that you can register your module with Caddy, let’s add more functionality so you can retrieve the logger from Caddy and validate your module’s settings. [Listing 10-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-6) picks up where we left off.

```
--snip--

// Provision a Zap logger to RestrictPrefix.
func (p *RestrictPrefix) 1Provision(ctx caddy.Context) error {
    p.logger = 2ctx.Logger(p)
    return nil
}

// Validate the prefix from the module's configuration, setting the
// default prefix "." if necessary.
func (p *RestrictPrefix) 3Validate() error {
    if p.Prefix == "" {
        p.Prefix = "."
    }
    return nil
}
```

Listing 10-6: Implementing various Caddy interfaces

You add the `Provision` method 1 to your struct. Caddy will recognize that your module implements the `caddy.Provisioner` interface and call this method. You can then retrieve the logger from the given `caddy.Context`2. Likewise, Caddy will call your module’s `Validate` method 3 since it implements the `caddy.Validator` interface. You can use this method to make sure all required settings have been unmarshaled from the configuration into your module. If anything goes wrong, you can return an error and Caddy will complain on your behalf. In this example, you’re using this method to set the default prefix if one was not provided in the configuration.

You’re almost done. The last piece of the puzzle is the middleware implementation itself. [Listing 10-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-7) rounds out your module’s implementation by adding support for the `caddyhttp.MiddlewareHandler` interface.

```
--snip--

// ServeHTTP implements the caddyhttp.MiddlewareHandler interface.
func (p RestrictPrefix) ServeHTTP(w http.ResponseWriter, r *http.Request,
    next caddyhttp.Handler) error {
 1 for _, part := range strings.Split(r.URL.Path, "/") {
        if strings.HasPrefix(part, p.Prefix) {
          2http.Error(w, "Not Found", http.StatusNotFound)
            if p.logger != nil {
              3p.logger.Debug(fmt.Sprintf(
                    "restricted prefix: %q in %s", part, r.URL.Path))
            }
            return nil
        }
    }
    return 4next.ServeHTTP(w, r)
}

var (
 5 _ caddy.Provisioner           = (*RestrictPrefix)(nil)
    _ caddy.Validator             = (*RestrictPrefix)(nil)
    _ caddyhttp.MiddlewareHandler = (*RestrictPrefix)(nil)
)
```

Listing 10-7: Implementing the MiddlewareHandler interface

The logic is almost identical to the middleware from the preceding chapter. You loop through the URL path components, checking each one for the prefix 1. If you find a match, you respond with a 404 Not Found status 2 and log the occurrence for debugging purposes 3. If everything checks out, you pass control onto the next handler in the chain 4.

It’s a good practice to guard against interface changes by explicitly making sure your module implements the expected interfaces 5. If one of these interfaces happens to change in the future (for example, if you add a new method), these interface guards will cause compilation to fail, giving you an early warning that you need to adapt your code.

The final steps are to tidy up your module’s dependencies and publish it:

```
$ go mod tidy
go: finding module for package github.com/caddyserver/caddy/v2
go: finding module for package github.com/caddyserver/caddy/v2/modules/caddyhttp
go: finding module for package go.uber.org/zap
go: found github.com/caddyserver/caddy/v2 in github.com/caddyserver/caddy/v2 v2.0.0
go: found go.uber.org/zap in go.uber.org/zap v1.15.0
go: downloading github.com/golang/mock v1.4.1
go: downloading github.com/onsi/gomega v1.8.1
go: downloading github.com/smallstep/assert v0.0.0-20200103212524-b99dc1097b15
go: downloading github.com/onsi/ginkgo v1.11.0
go: downloading github.com/imdario/mergo v0.3.7
go: downloading github.com/chzyer/test v0.0.0-20180213035817-a1ea475d72b1
go: downloading github.com/golang/glog v0.0.0-20160126235308-23def4e6c14b
go: downloading github.com/alangpierce/go-forceexport v0.0.0-20160317203124-
8f1d6941cd75
go: downloading github.com/chzyer/logex v1.1.10
go: downloading github.com/hpcloud/tail v1.0.0
go: downloading gopkg.in/tomb.v1 v1.0.0-20141024135613-dd632973f1e7
go: downloading gopkg.in/fsnotify.v1 v1.4.7
```

Publish your module to GitHub or a similar version-control system supported by `go get`.

### Injecting Your Module into Caddy

The module and adapter you wrote are both self-registering. All you need to do to include their functionality in Caddy is to import them at build time. To do that, you need to compile Caddy from source. Start by making a directory for your build:

```
$ mkdir caddy
$ cd caddy
```

Building Caddy from source code requires a small amount of boilerplate code, to which you’ll include your modules. Your modules register themselves with Caddy as a side effect of the import. Create a new file named _main.go_ and add the code from [Listing 10-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-8) into it.

```
package main

import (
 1 cmd "github.com/caddyserver/caddy/v2/cmd"
 2 _ "github.com/caddyserver/caddy/v2/modules/standard"

    // Injecting custom modules into Caddy
 3 _ "github.com/awoodbeck/caddy-restrict-prefix"
 4 _ "github.com/awoodbeck/caddy-toml-adapter"
)

func main() {
    cmd.Main()
}
```

Listing 10-8: Injecting custom modules into Caddy

First, you import the Caddy command module 1 into your build. This has the `Main` function that starts the caddy server. Then, you import the standard modules 2 that you’ll find in Caddy’s binary distribution. Finally, you include your restrict prefix module 3 and your TOML configuration adapter 4.

All that’s left to do now is initialize the `caddy` module and build it:

```
$ go mod init caddy
$ go build
```

At this point, you should have a binary named `caddy` in the current directory. You can verify that it has your custom imports by looking for them in the `caddy` binary’s list of modules. The following command is specific to Linux and macOS:

```
$ ./caddy list-modules | grep "toml\|restrict_prefix"
caddy.adapters.toml
http.handlers.restrict_prefix
```

For my Windows friends, run this command instead:

```
> caddy list-modules | findstr "toml restrict_prefix"
caddy.adapters.toml
http.handlers.restrict_prefix
```

The `caddy` binary you built can read its configuration from TOML files and deny clients access to resources whose path includes a given prefix.

## Reverse-Proxying Requests to a Backend Web Service

You now have all the building blocks to create something meaningful in Caddy. Let’s put everything you’ve learned together by configuring Caddy to reverse-proxy requests to a backend web service and serve up static files on behalf of the backend web service. You’ll create two endpoints in Caddy. The first endpoint will serve up only static content from Caddy’s file server, showing Caddy’s static file-serving abilities. The second endpoint will reverse-proxy requests to a backend web service. This backend service will send the client HTML that will prompt the client to retrieve static files from Caddy, which will show how your web services can lean on Caddy to serve static content on their behalf.

Before you start building, you need to set up the proper directory structure. If you’re following along, you’re currently in the _caddy_ directory, which has a `caddy` binary built from the code in [Listing 10-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-8). Create two subdirectories, _files_ and _backend_:

```
$ mkdir files backend
```

You can retrieve the contents of the _files_ subdirectory from [https://github.com/awoodbeck/gnp/tree/master/ch10/files/](https://github.com/awoodbeck/gnp/tree/master/ch10/files/). The _backend_ subdirectory will store a simple backend service created in the next section.

### Creating a Simple Backend Web Service

You need a backend web service for Caddy to reverse-proxy requests to, as illustrated in [Figure 10-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#figure10-1). This service will respond to all requests with an HTML document that includes the static files Caddy serves on the service’s behalf.

[Listing 10-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-9) is the initial code for the backend web service.

```
package main

import (
    "flag"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "time"
)

var addr = flag.String("listen", 1"localhost:8080", "listen address")

func main() {
    flag.Parse()

    c := make(chan os.Signal, 1)
    signal.Notify(c, os.Interrupt)

    err := 2run(*addr, c)
    if err != nil {
        log.Fatal(err)
    }

    log.Println("Server stopped")
}
```

Listing 10-9: Creating a backend service (_backend/main.go_)

This bit of code should be familiar since it’s a simplified version of what you wrote in the preceding chapter. You’re setting up a web service that listens on port 8080 of localhost 1. Caddy will direct requests to this socket address. [Listing 10-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-10) implements the `run` function 2.

```
--snip--

func run(addr string, c chan os.Signal) error {
    mux := http.NewServeMux()
    mux.Handle("/",
        http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            clientAddr := r.Header.Get(1"X-Forwarded-For")
            log.Printf("%s -> %s -> %s", clientAddr, r.RemoteAddr, r.URL)
            _, _ = w.Write(2index)
        }),
    )

    srv := &http.Server{
        Addr:              addr,
        Handler:           mux,
        IdleTimeout:       time.Minute,
        ReadHeaderTimeout: 30 * time.Second,
    }

    go func() {
        for {
            if <-c == os.Interrupt {
                _ = srv.Close()
                return
            }
        }
    }()

    fmt.Printf("Listening on %s ...\n", srv.Addr)
    err := srv.ListenAndServe()

    if err == http.ErrServerClosed {
        err = nil
    }

    return err
}
```

Listing 10-10: The main logic of the backend service (_backend/main.go_)

The web service receives all requests from Caddy, no matter which client originated the request. Likewise, it sends all responses back to Caddy, which then routes the response to the right client. Conveniently, Caddy adds an `X-Forwarded-For` header 1 to each request with the originating client’s IP address. Although you don’t do anything other than log this information, your backend service could use this IP address to differentiate between client requests. The service could deny requests based on client IP address, for example.

The handler writes a slice of bytes 2 to the response that has HTML defined in [Listing 10-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-11).

```
--snip--

var index = []byte(`<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Caddy Backend Test</title>
    <link href=1"/style.css" rel="stylesheet">
</head>
<body>
    <p><img src=2"/hiking.svg" alt="hiking gopher"></p>
</body>
</html>`)
```

Listing 10-11: The index HTML served by the backend service (_backend/main.go_)

The `/style.css`1 and `/hiking.svg`2 resources do not include a full URL (such as [http://localhost:2020/style.css](http://localhost:2020/style.css)) because the backend web service does not know anything about Caddy or how clients access Caddy. When you exclude the scheme, hostname, and port number in the resource address, the client’s web browser should encounter `/style.css` in the HTML and prepend the scheme, hostname, and port number it used for the initial request before sending the request to Caddy. For that all to work, you need to configure Caddy in the next section to send some requests to the backend web service and serve static files for the rest of the requests.

### Setting Up Caddy’s Configuration

As mentioned earlier in the chapter, Caddy uses JSON as its native configuration format. You could certainly write your configuration in JSON, but you’ve already written a perfectly good configuration adapter that allows you to use TOML, so you’ll implement that instead.

You want to configure Caddy to reverse-proxy requests to your backend web service and serve static files from the _files_ subdirectory. You’ll need two routes: one to the backend web service and one for static files. Let’s start by defining your server configuration in a file named _caddy.toml_ ([Listing 10-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-12)).

```
1 [apps.http.servers.test_server]
listen = [
    'localhost:2020',
]
```

Listing 10-12: Caddy test server configuration (_caddy.toml_)

Your TOML adapter directly converts TOML to JSON. Therefore, you need to make sure you’re using the same namespaces Caddy expects. The namespace for your server is `apps.http.servers.test_server`1. (For simplicity, you’ll refer to this namespace simply as `test_server` from here on out.) It listens for incoming connections on port 2020 of localhost.

### Adding a Reverse-Proxy to Your Service

Caddy includes a powerful reverse-proxy handler that makes quick work of sending incoming requests to your backend web service. Just as in the server implementation in the preceding chapter, Caddy matches an incoming request to a route and then passes the request onto the associated handler.

[Listing 10-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-13) adds a route and a reverse-proxy handler to the _caddy.toml_ file.

```
--snip--

1 [[apps.http.servers.test_server.routes]]
2 [[apps.http.servers.test_server.routes.match]]
path = [
    '/backend',
 3 '/backend/*',
]
4 [[apps.http.servers.test_server.routes.handle]]
handler = 'reverse_proxy'
5 [[apps.http.servers.test_server.routes.handle.upstreams]]
dial = 6'localhost:8080'
```

Listing 10-13: Adding a reverse proxy to the backend service (_caddy.toml_)

The `test_server` configuration includes a `routes` array 1, and each route in the array has zero or more matchers 2. A _matcher_ is a special module that allows you to specify matching criteria for incoming requests, like the `http.ServeMux.Handle` method’s pattern matching discussed in the preceding chapter. Caddy includes matcher modules that allow you to consider each part of a request.

For this route, you add a single matcher that matches any request for the absolute path _/backend_ or any path starting with _/backend/_3. The _*_ character is a wildcard that tells Caddy you want to match on the _/backend/_ prefix. For example, a request for the resource _/backend/this/is/a/test_ will also match.

The route may have one or more handlers 4. Here, you tell Caddy you want to send all matching requests to the reverse-proxy handler. The reverse-proxy handler needs to know where to send the requests. You specify an upstream entry 5 with its dial property set to the backend server’s socket address 6.

### Serving Static Files

You relied on the `http.FileServer` to serve static files for you in the preceding chapter. Caddy exposes similar functionality with its `file_server` handler. [Listing 10-14](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-14) adds a second route to your _caddy.toml_ file for serving static files.

```
--snip--

1 [[apps.http.servers.test_server.routes]]
2 [[apps.http.servers.test_server.routes.handle]]
handler = 'restrict_prefix'
prefix = '.'
3 [[apps.http.servers.test_server.routes.handle]]
handler = 'file_server'
root = 4'./files'
index_names = [
  5'index.html',
]
```

Listing 10-14: Adding a default route to serve static files (_caddy.toml_)

Unlike the route you added in [Listing 10-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c10.xhtml#listing10-13), this route 1 does not include any matchers. As such, Caddy would send every request to this route’s handler if the request didn’t match previous routes. In other words, this route is your default route, and so its position in the file matters. If you moved this route before the reverse-proxy route, all requests would match it, and no requests would ever make their way to the reverse proxy. Whenever you specify a route with no matches, make sure you put it at the end of your routes array, as you do here.

As with the file server in the preceding chapter, you want to protect against accidentally serving sensitive files prefixed with a period. Therefore, you include your `restrict_prefix` middleware 2 in the array of handlers before the `file_server` handler 3. You add more configuration options to serve files found in the _files_ subdirectory 4 and return the _index.html_ file 5 if the request didn’t specify a file.

### Checking Your Work

Everything is in place. Start Caddy and verify that your configuration works as expected. Since some of the static files are images, I recommend you use a web browser to interact with Caddy while it runs on your computer.

Start Caddy by using the _caddy.toml_ file and the _toml_ adapter:

```
$ ./caddy start --config caddy.toml --adapter toml
```

On Windows, the command looks like this:

```
> caddy start --config caddy.toml --adapter toml
```

Now, run the backend web service:

```
$ cd backend
$ go run backend.go
Listening on localhost:8080 ...
```

Open your web browser and visit [http://localhost:2020/](http://localhost:2020/). Caddy will send your request to the file server handler, which in turn will respond with the _index.html_ file, since you didn’t indicate a specific file in the request. Your browser then asks Caddy for the _style.css_ and _sage.svg_ files to finish rendering the page. If everything succeeds, you should now be looking at a sage gopher.

Now, let’s test the reverse proxy to the backend web service. Visit [http://localhost:2020/backend](http://localhost:2020/backend). This request matches the reverse-proxy route’s matcher, so the reverse-proxy handler should handle it, sending the request onto the backend service. The backend web service responds with HTML that instructs your browser to retrieve the _style.css_ and _hiking.svg_ files from Caddy, where the file server handler happily serves them up. You should now be looking at a hiking gopher rendered using HTML from the backend web service and static files from Caddy.

If you copied the _files_ subdirectory from this book’s source code repository, it should contain _./files/.secret_ and _./files/.dir/secret_ files. Your middleware should block access to both files. In other words, both [http://localhost:2020/files/.secret](http://localhost:2020/files/.secret) and [http://localhost:2020/files/.dir/secret](http://localhost:2020/files/.dir/secret) will return a 404 Not Found status if you try to request them.

### Adding Automatic HTTPS

Now let’s add Caddy’s key feature to your web server: automatic HTTPS.

I once used Caddy to stand up a website with full HTTPS support, using certificates trusted by all contemporary web browsers, in a matter of minutes. The server has been rock-solid ever since, happily rotating Let’s Encrypt keys every few months with no intervention on my part. This isn’t to say I couldn’t replicate this functionality in my own Go-based web server; my time was simply best spent building services and leaving the web serving to Caddy. If Caddy lacked any functionality, I could add it as a module.

Caddy automatically enables TLS when it can determine what domain names you’ve configured it to serve. The _caddy.toml_ configuration created in this chapter didn’t give Caddy enough information to determine which domain it was serving. Therefore, Caddy didn’t enable HTTPS for you. You told Caddy to bind to localhost, but that tells Caddy only what it’s listening to, not what domains it’s serving.

The most common way to enable automatic HTTPS is by adding a host matcher to one of Caddy’s routes. Here’s an example matcher:

```
[[apps.http.servers.server.routes.match]]
host = [
    'example.com',
]
```

This host matcher supplies enough information for Caddy to determine that it is serving the _example.com_ domain. If Caddy doesn’t already have a valid certificate for _example.com_ to enable HTTPS, it will go through the process of validating the domain with Let’s Encrypt and retrieving a certificate. Caddy will manage your certificate, automatically renewing it, as necessary.

Caddy’s `file-server` subcommand tells Caddy you want it to exclusively serve files over HTTP. The `file-server`’s `--domain` flag is enough information for Caddy to invoke its automatic HTTPS and allow you to serve files over HTTPS as well.

Caddy’s `reverse-proxy` subcommand allows you to put Caddy into a reverse-proxy-only mode, where it will send all incoming requests onto the socket address specified by the `--to` flag. Caddy will retrieve a TLS certificate and enable automatic HTTPS if you specify a hostname with the `--from` flag.

I encourage you to read more about Caddy’s automatic HTTPS in production environments at [https://caddyserver.com/docs/automatic-https](https://caddyserver.com/docs/automatic-https).

## NOTE

Caddy defaults to using Let’s Encrypt as its certificate authority for non-localhost names and IP addresses. But Caddy supports its own internal certificate authority for enabling HTTPS over localhost, which you may want to do for testing purposes. If you specify _localhost_ as your hostname in the earlier settings to enable automatic HTTPS, Caddy will use its internal certificate authority for its TLS certificate. It will also try to install its root certificate in your operating system’s root certificate trust store, which is where your operating system keeps track of all root certificates it inherently trusts.

## What You’ve Learned

Caddy is a contemporary web server written in Go that offers security, performance, and extensibility through modules and configuration adapters. Caddy can automatically use HTTPS through its integration with Let’s Encrypt, a nonprofit certificate authority that supplies free digital certificates. Together, Caddy and Let’s Encrypt allow you to stand up a web server with seamless HTTPS support.

Caddy uses JSON as its native configuration format, and it exposes an API on localhost port 2019 that allows you to post JSON to change its configuration. The configuration changes take immediate effect. But since JSON isn’t an ideal configuration format, Caddy makes use of configuration adapters. Configuration adapters translate configuration files from more configuration-friendly formats, like TOML, to JSON. If you don’t want to use JSON for your Caddy configuration or if you don’t find a configuration adapter that meets your needs, you can also write your own, as you did in this chapter.

You can also extend Caddy’s functionality with the use of modules. This chapter shows how to write a middleware module, compile it into Caddy, configure the module, and put it to effective use.

Finally, this chapter showed you how to integrate Caddy into your network architecture. You’ll often make Caddy the first server in your network, using it to receive client requests before sending the requests onto their final destinations. In this chapter, you configured an instance of Caddy to reverse-proxy client requests to your backend web service and serve static files on behalf of the backend web service. As a result, you kept your backend web service as simple as possible and saved it from having to manage its static content. Your backend web service can use Caddy for HTTPS support, caching, and file serving.

Now that you have some experience with Caddy, you should be able to determine whether your web services would do better when served by a comprehensive web server solution or a comparatively minimal `net/http` web server implementation. If you expect to make your web service available to the public, using a proven web server like Caddy at the edge of your application will free up time you can better spend on your backend web service.