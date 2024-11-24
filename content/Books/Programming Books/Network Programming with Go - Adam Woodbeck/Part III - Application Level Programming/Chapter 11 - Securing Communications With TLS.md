---
id: 01JCH9DPMZ23AXGNEKQDJJ1S4F
modified: 2024-11-12T17:58:06-05:00
title: Chapter 11 - Securing Communications With TLS
---
# 11  
SECURING COMMUNICATIONS WITH TLS

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

Five years before whistleblower Edward Snowden showed us how much we took our electronic privacy for granted, author and activist Cory Doctorow wrote, “We should treat personal electronic data with the same care and respect as weapons-grade plutonium—it is dangerous, long-lasting, and once it has leaked, there’s no getting it back.”

Prior to 2013, most people communicated on the internet by using plaintext. Social Security numbers, credit card details, passwords, sensitive emails, and other potentially embarrassing information traveled over the internet, ripe for interception by malicious actors. Most popular websites defaulted to HTTP; Google was one of the only major tech companies supporting HTTPS.

Today, it’s unusual to find a website that doesn’t support HTTPS, particularly now that Let’s Encrypt offers free TLS certificates for your domain. We’re treating information in transit more like weapons-grade plutonium, helping ensure the privacy and integrity of the information we share. Our network applications should be no different. We should strive to authenticate our communication and use encryption where appropriate, particularly when that information has the potential to leak over insecure networks.

Up to this point, we’ve used TLS only as an afterthought in our code. This is partly because Go’s `net/http` library makes its use relatively effortless, but it’s also because we haven’t adequately explored the TLS protocol and the infrastructure that makes it possible. To write secure software, you should carefully plan for security before development starts and then use good security practices as you write code. TLS is a terrific way to improve the security posture of your software by protecting data in transit.

This chapter will introduce you to the basics of TLS from a programmer’s perspective. You’ll learn about the client-server handshake process and the inherent trust that makes that process work. Then we’ll discuss how things can (and do) go wrong even when you use TLS. Finally, we’ll look at practical examples of how to incorporate TLS into your applications, including mutual client-server authentication.

## A Closer Look at Transport Layer Security

The TLS protocol supplies secure communication between a client and a server. It allows the client to authenticate the server and optionally permits the server to authenticate clients. The client uses TLS to encrypt its communication with the server, preventing third-party interception and manipulation.

TLS uses a handshake process to establish certain criteria for the stateful TLS session. If the client initiated a TLS 1.3 handshake with the server, it would go something like this:

1. Client Hello _google.com_. I’d like to communicate with you using TLS version 1.3. Here is a list of ciphers I’d like to use to encrypt our messages, in order of my preference. I generated a public- and private-key pair specifically for this conversation. Here’s my public key.
2. Server Greetings, client. TLS version 1.3 suits me fine. Based on your cipher list, I’ve decided we’ll use the Advanced Encryption Standard with Galois/Counter Mode (AES-GCM) cipher. I, too, created a new key pair for this conversation. Here is my public key and my certificate so you can prove that I truly am _google.com_. I’m also sending along a 32-byte value that corresponds to the TLS version you asked me to use. Finally, I’m including both a signature and a _message authentication code_ _(MAC)_ derived using your public key of everything we’ve discussed so far so you can verify the integrity of my reply when you receive it.
3. Client (to self) An authority I trust signed the server’s certificate, so I’m confident I’m speaking to _google.com_. I’ve derived this conversation’s symmetric key from the server’s signature by using my private key. Using this symmetric key, I’ve verified the MAC and made sure no one has tampered with the server’s reply. The 32 bytes in the reply corresponds to TLS version 1.3, so no one is attempting to trick the server into using an older, weaker version of TLS. I now have everything I need to securely communicate with the server.
4. Client (to server) _Here is some encrypted data._

The 32-byte value in the server’s hello message prevents _downgrade attacks_, in which an attacker intercepts the client’s hello message and modifies it to request an older, weaker version of TLS. If the client asked for TLS v1.3, but an attacker changed the client’s hello message to ask for TLS v1.1, the 32-byte value in the server’s hello message would correspond to TLS v1.1. When the client received the server’s hello message, it would notice that the value indicated the wrong TLS version and abort the handshake.

From this point forward, the client and server communicate using AES-GCM symmetric-key cryptography (in this hypothetical example). Both the client and the server encapsulate application layer payloads in TLS records before passing the payloads onto the transport layer.

Despite its name, TLS is not a transport layer protocol. Instead, it’s situated between the transport and application layers of the TCP/IP stack. TLS encrypts an application layer protocol’s payload before passing the payload onto the transport layer. Once the payload reaches its destination, TLS receives the payload from the transport layer, decrypts it, and passes the payload along to the application layer protocol.

### Forward Secrecy

The handshake method in our hypothetical conversation is an example of the Diffie-Hellman (DH) key exchange used in TLS v1.3. The _DH key exchange_ calls for the creation of new client and server key pairs, and a new symmetric key, all of which should exist for only the duration of the session. Once a session ends, the client and server shall discard the session keys.

The use of per-session keys means that TLS v1.3 gives you _forward secrecy_; an attacker who compromises your session keys can compromise only the data exchanged during that session. An attacker cannot use those keys to decrypt data exchanged during any other session.

### In Certificate Authorities We Trust

My father and I took a trip to Ireland shortly before I started authoring this book. In preparation for our adventure, I needed to obtain a new passport, since my old one had long since expired. The process was easy. I filled out an application, collected my vital records, took a picture of the bad side of my head, and presented everything, along with an application fee, to my local US Post Office branch. I also attested I was myself to the notary. A few weeks later, I received a newly minted US passport in the mail.

When we arrived in Ireland, a lovely customs agent greeted us and requested our passports. She asked questions about our holiday as her computer authenticated our identities. After no more than three minutes, she returned our passports and welcomed us to Ireland.

My passport represents the US government’s attestation that I am Adam Woodbeck. But it’s only as good as Ireland’s trust in the US government’s ability to verify my identity. If Ireland doesn’t trust the United States, it will not take the United States’ word that I am me and will most likely refuse to let me enter the country. (If I’m being honest, I’m not charming enough to convince the customs agent to let me in on my word alone.)

TLS’s certificates work in much the same way as my passport. If I wanted a new TLS certificate for _woodbeck.net_, I would send a request to a certificate authority, such as Let’s Encrypt. The certificate authority would then verify I am the proper owner of _woodbeck.net_. Once satisfied, the certificate authority would issue a new certificate for _woodbeck.net_ and cryptographically sign it with its certificate. My server can present this certificate to clients so they can authenticate my server by confirming the certificate authority’s signature, giving them the confidence that they’re communicating with the real _woodbeck.net_, not an impostor.

A certificate authority issuing a signed certificate for _woodbeck.net_ is analogous to the US government issuing my passport. They are both issued by trusted institutions that attest to their subject’s authenticity. Like Ireland’s trust of the United States, clients are inclined to trust the _woodbeck.net_ certificate only if they trust the certificate authority that signed it. I could create my own certificate authority and self-sign certificates as easy as I could create a document claiming to be my passport. But Ireland would sooner admit that Jack Daniel’s Tennessee Whiskey is superior to Jameson Irish Whiskey than trust my self-issued passport, and no operating system or web browser in the world would trust my self-signed certificate.

### How to Compromise TLS

On December 24, 2013, Google learned that the Turktrust certificate authority in Turkey had mistakenly issued a certificate that allowed a malicious actor to masquerade as _google.com_. This meant that attackers could fool your web browser into thinking it was talking to Google over a TLS connection and trick you into divulging your credentials. Google quickly noticed the mistake and took steps to remedy the situation.

Turktrust’s mess-up undermined its authority and compromised our trust. But even if the certificate authorities operate correctly, attackers can narrow their focus and target individuals instead. If an attacker were able to install his own CA certificate in your operating system’s trusted certificate storage, your computer would trust any certificate he signs. This means an attacker could compromise all your TLS traffic.

Most people don’t get this kind of special attention. Instead, an attacker is more likely to compromise a server. Once compromised, the attacker could capture all TLS traffic and the corresponding session keys from memory.

You’re unlikely to encounter any of these scenarios, but it’s important to be aware that they are possible. Overall, TLS 1.3 offers excellent security and is tough to compromise because of its full handshake signature, downgrade protection, forward secrecy, and strong encryption.

## Protecting Data in Transit

Ensuring the integrity of the data you transmit over a network should be your primary focus, no matter whether it’s your own data or the data of others. Go makes using TLS so easy that you would have a tough time justifying not using it. In this section, you’ll learn how to add TLS support to both the client and the server. You’ll also see how TLS works over TCP and how to mitigate the threat of malicious certificates with certificate pinning.

### Client-side TLS

The client’s primary concern during the handshake process is to authenticate the server by using its certificate. If the client cannot trust the server, it cannot consider its communication with the server secure. The `net/http/httptest` package provides constructs that easily demonstrate Go’s HTTP-over-TLS support (see [Listing 11-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-1)).

```
package ch11

import (
    "crypto/tls"
    "net"
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"
    "time"

    "golang.org/x/net/http2"
)

func TestClientTLS(t *testing.T) {
    ts := 1httptest.NewTLSServer(
        http.HandlerFunc(
            func(w http.ResponseWriter, r *http.Request) {
                if 2r.TLS == nil {
                    u := "https://" + r.Host + r.RequestURI
                    http.Redirect(w, r, u, http.StatusMovedPermanently)
                    return
                }

                w.WriteHeader(http.StatusOK)
            },
        ),
    )
    defer ts.Close()

    resp, err := 3ts.Client().Get(ts.URL)
    if err != nil {
        t.Fatal(err)
    }

    if resp.StatusCode != http.StatusOK {
        t.Errorf("expected status %d; actual status %d",
            http.StatusOK, resp.StatusCode)
    }
```

Listing 11-1: Testing HTTPS client and server support (_tls_client_test.go_)

The `httptest.NewTLSServer` function returns an HTTPS server 1. Aside from the function name, this bit of code looks identical to our use of `httptest` in Chapter 8. Here, the `httptest.NewTLSServer` function handles the HTTPS server’s TLS configuration details, including the creation of a new certificate. No trusted authority signed this certificate, so no discerning HTTPS client would trust it. You’ll see how to work around this detail in just a moment by using a preconfigured client.

If the server receives the client’s request over HTTP, the request’s `TLS` field will be `nil`. You can check for this case 2 and redirect the client to the HTTPS endpoint accordingly.

For testing purposes, the server’s `Client` method 3 returns a new `*http.Client` that inherently trusts the server’s certificate. You can use this client to test TLS-specific code within your handlers.

Let’s see what happens in [Listing 11-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-2) when you attempt to communicate with the same server by using a new client without inherent trust for the server’s certificate.

```
--snip--

    tp := &http.Transport{
        TLSClientConfig: &tls.Config{
            CurvePreferences: []tls.CurveID{1tls.CurveP256},
            MinVersion:       tls.VersionTLS12,
        },
    }

    err = 2http2.ConfigureTransport(tp)
    if err != nil {
        t.Fatal(err)
    }

    client2 := &http.Client{Transport: tp}

    _, err = client2.Get(ts.URL)
    if err == nil || !strings.Contains(err.Error(),
        "certificate signed by unknown authority") {
        t.Fatalf("expected unknown authority error; actual: %q", err)
    }

 3 tp.TLSClientConfig.InsecureSkipVerify = true

    resp, err = client2.Get(ts.URL)
    if err != nil {
        t.Fatal(err)
    }

    if resp.StatusCode != http.StatusOK {
        t.Errorf("expected status %d; actual status %d",
            http.StatusOK, resp.StatusCode)
    }
}
```

Listing 11-2: Testing the HTTPS server with a discerning client (_tls_client_test.go_)

You override the default TLS configuration in your client’s transport by creating a new transport, defining its TLS configuration, and configuring `http2` to use this transport. It’s good practice to restrict your client’s curve preference to the P-256 curve 1 and avoid the use of P-384 and P-521. P-256 is immune to timing attacks, whereas P-384 and P-521 are not. Also, your client will negotiate a minimum of TLS 1.2.

An _elliptic curve_ is a plane curve in which all points along the curve satisfy the same polynomial equation. Whereas first-generation cryptography like RSA uses large prime numbers to derive keys, elliptic curve cryptography uses points along an elliptic curve for key generation. P-256, P-384, and P-521 are specific elliptic curves defined in the National Institute of Standards and Technology’s Digital Signature Standard. You can find more details in the Federal Information Processing Standards (FIPS) publication 186-4 ([https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)).

Since your transport no longer relies on the default TLS configuration, the client no longer has inherent HTTP/2 support. You need to explicitly bless your transport with HTTP/2 support 2 if you want to use it. Of course, this test doesn’t rely on HTTP/2, but this implementation detail can trip you up if you’re unaware that overriding the transport’s TLS configuration removes HTTP/2 support.

Your client uses the operating system’s trusted certificate store because you don’t explicitly tell it which certificates to trust. The first call to the test server results in an error because your client doesn’t trust the server certificate’s signatory. You could work around this and configure your client’s transport to skip verification of the server’s certificate by setting its `InsecureSkipVerify` field to `true`3. I don’t recommend you entertain enabling `InsecureSkipVerify` for anything other than debugging. Shipping code with this enabled is a code smell in my opinion. You’ll learn a better alternative later in this chapter when we discuss a concept known as _certificate pinning_. As the field name implies, enabling it makes your client inherently insecure and susceptible to man-in-the-middle attacks, since it now blindly trusts any certificate a server offers up. If you make the same call with your newly naive client, you’ll see that it happily negotiates TLS with the server.

### TLS over TCP

TLS is stateful; a client and a server negotiate session parameters during the initial handshake only, and once they’ve agreed, they exchange encrypted TLS records for the duration of the session. Since TCP is also stateful, it’s the ideal transport layer protocol with which to implement TLS, because you can leverage TCP’s reliability guarantees to maintain your TLS sessions.

Let’s take the application protocol out of the picture for a moment and learn how to establish a TLS connection over TCP. [Listing 11-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-3) demonstrates how to use the `crypto/tls` package to initiate a TLS connection with a few lines of code.

```
--snip--

func TestClientTLSGoogle(t *testing.T) {
    conn, err := 1tls.DialWithDialer(
        &net.Dialer{Timeout: 30 * time.Second},
        "tcp",
        "www.google.com:443",
        &tls.Config{
            CurvePreferences: []tls.CurveID{tls.CurveP256},
            MinVersion:       tls.VersionTLS12,
        },
    )
    if err != nil {
        t.Fatal(err)
    }

    state := 2conn.ConnectionState()
    t.Logf("TLS 1.%d", state.Version-tls.VersionTLS10)
    t.Log(tls.CipherSuiteName(state.CipherSuite))
    t.Log(state.VerifiedChains[0][0].Issuer.Organization[0])

    _ = conn.Close()
}
```

Listing 11-3: Starting a TLS connection with _www.google.com_ (_tls_client_test.go_)

The `tls.DialWithDialer` function 1 accepts a `*net.Dialer`, a network, an address, and a `*tls.Config`. Here, you give your dialer a time-out of 30 seconds and specify recommended TLS settings. If successful, you can inspect the connection’s state 2 to glean details about your TLS connection.

[Listing 11-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-4) shows the output of [Listing 11-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-3)’s test.

```
$ go test -race -run TestClientTLSGoogle -v ./...
=== RUN   TestClientTLSGoogle
    TestClientTLSGoogle: tls_client_test.go:89: TLS 1.3
    TestClientTLSGoogle: tls_client_test.go:90: TLS_AES_128_GCM_SHA256
    TestClientTLSGoogle: tls_client_test.go:91: Google Trust Services
--- PASS: TestClientTLSGoogle (0.31s)
PASS
```

Listing 11-4: Running the `TestClientTLSGoogle` test

Your TLS client is using the TLS_AES_128_GCM_SHA256 cipher suite over TLS version 1.3. Notice that `tls.DialWithDialer` did not object to the server’s certificate. The underlying TLS client used the operating system’s trusted certificate storage and confirmed that _www.google.com_’s certificate is signed by a trusted CA—Google Trust Services, in this example.

### Server-side TLS

The server-side code isn’t much different from what you’ve learned thus far. The main difference is that the server needs to present a certificate to the client as part of the handshake process. You can create one with the _generate_cert.go_ file found in Go’s _src/crypto/tls_ subdirectory. For production use, you’re better off using certificates from Let’s Encrypt or another certificate authority. You can use the LEGO library ([https://github.com/go-acme/lego/](https://github.com/go-acme/lego/)) to add certificate management to your services. Generate a new cert and private key, like so:

```
$ go run $GOROOT/src/crypto/tls/generate_cert.go -host localhost -ecdsa-curve P256
```

This command creates a certificate named _cert.pem_ with the hostname _localhost_ and a private key named _key.pem_. The rest of the code in this section assumes that both files exist in the current directory.

Keeping with the tradition of earlier chapters, [Listing 11-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-5) includes the first bit of code for a TLS-only echo server.

```
package ch11

import (
    "context"
    "crypto/tls"
    "fmt"
    "net"
    "time"
)

func NewTLSServer(ctx context.Context, address string,
    maxIdle time.Duration, tlsConfig *tls.Config) *Server {
    return &Server{
        ctx:       ctx,
        ready:     make(chan struct{}),
        addr:      address,
        maxIdle:   maxIdle,
        tlsConfig: tlsConfig,
    }
}

type Server struct {
    ctx   context.Context
    ready chan struct{}

    addr      string
    maxIdle   time.Duration
    tlsConfig *tls.Config
}

func (s *Server) 1Ready() {
    if s.ready != nil {
        <-s.ready
    }
}
```

Listing 11-5: Server struct type and constructor function (_tls_echo.go_)

The `Server` struct has a few fields used to record its settings, its TLS configuration, and a channel to signal when the server is ready for incoming connections. You’ll write a test case and use the `Ready` method 1 a little later in this section to block until the server is ready to accept connections.

The `NewTLSServer` function accepts a context for stopping the server, an address, the maximum duration the server should allow connections to idle, and a TLS configuration. Although controlling for idling clients isn’t related to TLS, you’ll use the maximum idle duration to push the socket deadline forward, as in Chapter 3.

Servers you used in earlier chapters rely on the separate concepts of listening and serving. Often, you’ll invoke a helper function that will do both for you, such as the `net/http` server’s `ListenAndServe` method. [Listing 11-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-6) adds a similar method to the echo server.

```
--snip--

func (s *Server) ListenAndServeTLS(certFn, keyFn string) error {
    if s.addr == "" {
        s.addr = "localhost:443"
    }

    l, err := net.Listen("tcp", s.addr)
    if err != nil {
        return fmt.Errorf("binding to tcp %s: %w", s.addr, err)
    }

    if s.ctx != nil {
        go func() {
            <-s.ctx.Done()
            _ = l.Close()
        }()
    }

    return s.ServeTLS(l, certFn, keyFn)
}
```

Listing 11-6: Adding methods to listen and serve and signal the server’s readiness for connections (_tls_echo.go_)

The `ListenAndServe` method accepts full paths to a certificate and a private key and returns an error. It creates a new `net.Listener` bound to the server’s address and then spins off a goroutine to close the listener when you cancel the context. Finally, the method passes the listener, the certificate path, and the key path onto the server’s `ServeTLS` method.

[Listing 11-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-7) rounds out the echo server’s implementation with its `ServeTLS` method.

```
--snip--

func (s Server) ServeTLS(l net.Listener, certFn, keyFn string) error {
    if s.tlsConfig == nil {
        s.tlsConfig = &tls.Config{
            CurvePreferences:         []tls.CurveID{tls.CurveP256},
            MinVersion:               tls.VersionTLS12,
         1 PreferServerCipherSuites: true,
        }
    }

    if len(s.tlsConfig.Certificates) == 0 &&
        s.tlsConfig.GetCertificate == nil {
        cert, err := 2tls.LoadX509KeyPair(certFn, keyFn)
        if err != nil {
            return fmt.Errorf("loading key pair: %v", err)
        }

        s.tlsConfig.Certificates = []tls.Certificate{cert}
    }

    tlsListener := 3tls.NewListener(l, s.tlsConfig)
    if s.ready != nil {
        close(s.ready)
    }
```

Listing 11-7: Adding TLS support to a net.Listener (_tls_echo.go_)

The `ServeTLS` method first checks the server’s TLS configuration. If it’s `nil`, it adds a default configuration with `PreferServerCipherSuites` set to `true`1. `PreferServerCipherSuites` is meaningful to the server only, and it makes the server use its preferred cipher suite instead of deferring to the client’s preference.

If the server’s TLS configuration does not have at least one certificate, or if its `GetCertificate` method is `nil`, you create a new `tls.Certificate` by reading in the certificate and private-key files from the filesystem 2.

At this point in the code, the server has a TLS configuration with at least one certificate ready to present to clients. All that’s left is to add TLS support to the `net.Listener` by passing it and the server’s TLS configuration to the `tls.NewListener` function 3. The `tls.NewListener` function acts like middleware, in that it augments the listener to return TLS-aware connection objects from its `Accept` method.

[Listing 11-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-8) finishes up the `ServeTLS` method by accepting connections from the listener and handling them in separate goroutines.

```
--snip--

    for {
        conn, err := 1tlsListener.Accept()
        if err != nil {
            return fmt.Errorf("accept: %v", err)
        }

        go func() {
            defer func() { _ = 2conn.Close() }()

            for {
                if s.maxIdle > 0 {
                    err := 3conn.SetDeadline(time.Now().Add(s.maxIdle))
                    if err != nil {
                        return
                    }
                }

                buf := make([]byte, 1024)
                n, err := 4conn.Read(buf)
                if err != nil {
                    return
                }

                _, err = conn.Write(buf[:n])
                if err != nil {
                    return
                }
            }
        }()
    }
}
```

Listing 11-8: Accepting TLS-aware connections from the listener (_tls_echo.go_)

This pattern is like the one you’ve seen in earlier chapters. You use an endless `for` loop to continually block on the listener’s `Accept` method 1, which returns a new `net.Conn` object when a client successfully connects. Since you’re using a TLS-aware listener, it returns connection objects with underlying TLS support. You interact with these connection objects the same as you always do. Go abstracts the TLS details away from you at this point. You then spin off this connection into its own goroutine to handle the connection from that point forward.

The server handles each connection the same way. It first conditionally sets the socket deadline to the server’s maximum idle duration 3, then waits for the client to send data. If the server doesn’t read anything from the socket before it reaches the deadline, the connection’s `Read` method 4 returns an _I/O time-out_ error, ultimately causing the connection to close 2.

If, instead, the server reads data from the connection, it writes that same payload back to the client. Control loops back around to reset the deadline and then wait for the next payload from the client.

### Certificate Pinning

Earlier in the chapter, we discussed ways to compromise the trust that TLS relies on, whether by a certificate authority issuing fraudulent certificates or an attacker injecting a malicious certificate into your computer’s trusted certificate storage. You can mitigate both attacks by using certificate pinning.

_Certificate pinning_ is the process of scrapping the use of the operating system’s trusted certificate storage and explicitly defining one or more trusted certificates in your application. Your application will trust connections only from hosts presenting a pinned certificate or a certificate signed by a pinned certificate. If you plan on deploying clients in zero-trust environments that must securely communicate with your server, consider pinning your server’s certificate to each client.

Assuming the server introduced in the preceding section uses the _cert.pem_ and the _key.pem_ you generated for the hostname _localhost_, all clients will abort the TLS connection as soon as the server presents its certificate. Clients won’t trust the server’s certificate because no trusted certificate authority signed it.

You could set the `tls.Config`’s `InsecureSkipVerify` field to `true`, but as this method is insecure, I don’t recommend you consider it a practical choice. Instead, let’s explicitly tell our client it can trust the server’s certificate by pinning the server’s certificate to the client. [Listing 11-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-9) has the beginnings of a test to show that process.

```
package ch11

import (
    "bytes"
    "context"
    "crypto/tls"
    "crypto/x509"
    "io"
    "io/ioutil"
    "strings"
    "testing"
    "time"
)

func TestEchoServerTLS(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    serverAddress := "localhost:34443"
    maxIdle := time.Second
    server := NewTLSServer(ctx, serverAddress, maxIdle, nil)
    done := make(chan struct{})

    go func() {
        err := 1server.ListenAndServeTLS("cert.pem", "key.pem")
        if err != nil && !strings.Contains(err.Error(),
            "use of closed network connection") {
            t.Error(err)
            return
        }
        done <- struct{}{}
    }()
2     server.Ready()
```

Listing 11-9: Creating a new TLS echo server and starting it in the background (_tls_echo_test.go_)

Since the hostname in _cert.pem_ is _localhost_, you create a new TLS echo server listening on _localhost_ port 34443. The port isn’t important here, but clients expect the server to be reachable by the same hostname as the one in the certificate it presents. You spin up the server in the background by using the _cert.pem_ and _key.pem_ files 1 and block until it’s ready for incoming connections 2.

[Listing 11-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-10) picks up where we left off by creating a client TLS configuration with explicit trust for the server’s certificate.

```
--snip--

    cert, err := ioutil.ReadFile("cert.pem")
    if err != nil {
        t.Fatal(err)
    }

    certPool := 1x509.NewCertPool()
    if ok := certPool.AppendCertsFromPEM(cert); !ok {
        t.Fatal("failed to append certificate to pool")
    }

    tlsConfig := &tls.Config{
        CurvePreferences: []tls.CurveID{tls.CurveP256},
        MinVersion:       tls.VersionTLS12,
     2 RootCAs:          certPool,
    }
```

Listing 11-10: Pinning the server certificate to the client (_tls_echo_test.go_)

Pinning a server certificate to the client is straightforward. First, you read in the _cert.pem_ file. Then, you create a new certificate pool 1 and append the certificate to it. Finally, you add the certificate pool to the `tls.Config`’s `RootCAs` field 2. As the name suggests, you can add more than one trusted certificate to the certificate pool. This can be useful when you are migrating to a new certificate but have yet to completely phase out the old certificate.

The client, using this configuration, will authenticate only servers that present the _cert.pem_ certificate or any certificate signed by it. Let’s confirm this behavior in the rest of the test (see [Listing 11-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-11)).

```
--snip--

    conn, err := 1tls.Dial("tcp", serverAddress, tlsConfig)
    if err != nil {
        t.Fatal(err)
    }

    hello := []byte("hello")
    _, err = conn.Write(hello)
    if err != nil {
        t.Fatal(err)
    }

    b := make([]byte, 1024)
    n, err := conn.Read(b)
    if err != nil {
        t.Fatal(err)
    }

    if actual := b[:n]; !bytes.Equal(hello, actual) {
        t.Fatalf("expected %q; actual %q", hello, actual)
    }

 2 time.Sleep(2 * maxIdle)
    _, err = conn.Read(b)
    if err != 3io.EOF {
        t.Fatal(err)
    }

    err = conn.Close()
    if err != nil {
        t.Fatal(err)
    }

    cancel()
    <-done
}
```

Listing 11-11: Authenticating the server by using a pinned certificate (_tls_echo_test.go_)

You pass `tls.Dial` the `tls.Config` with the pinned server certificate 1. Your TLS client authenticates the server’s certificate without having to resort to using `InsecureSkipVerify` and all the insecurity that option introduces.

Now that you’ve set up a trusted connection with a server, even though the server presented an unsigned certificate, let’s make sure the server works as expected. It should echo back any message you send it. If you idle long enough 2, you find that your next interaction with the socket results in an error 3, showing the server closed the socket.

## Mutual TLS Authentication

In the preceding section, you learned how clients authenticate servers by using the server’s certificate and a trusted third-party certificate or by configuring the client to explicitly trust the server’s certificate. Servers can authenticate clients in the same manner. This is particularly useful in zero-trust network infrastructures, where clients and servers must each prove their identities. For example, you may have a client outside your network that must present a certificate to a proxy before the proxy will allow the client to access your trusted network resources. Likewise, the client authenticates the certificate presented by your proxy to make sure it’s talking to your proxy and not one controlled by a malicious actor.

You can instruct your server to set up TLS sessions with only authenticated clients. Those clients would have to present a certificate signed by a trusted certificate authority or pinned to the server. Before you can look at example code, the client needs a certificate it can present to the server for authentication. However, clients cannot use the certificates generated with _$GOROOT/src/crypto/tls/generate_cert.go_ for client authentication. Instead, you need to create your own certificate and private key.

### Generating Certificates for Authentication

Go’s standard library contains everything you need to generate your own certificates using the elliptic curve digital signature algorithm (ECDSA) and the P-256 elliptic curve. [Listing 11-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-12) shows the beginnings of a command line utility for doing exactly that. As you go through it, keep in mind that it may not entirely fit your use case. For example, it creates 10-year certificates and uses my name as the certificate’s subject, which you likely don’t want to use in your code (though if you do, I’m flattered). Tweak, as necessary.

```
package main

import (
    "crypto/ecdsa"
    "crypto/elliptic"
    "crypto/rand"
    "crypto/x509"
    "crypto/x509/pkix"
    "encoding/pem"
    "flag"
    "log"
    "math/big"
    "net"
    "os"
    "strings"
    "time"
)

var (
    host = flag.String("host", "localhost",
        "Certificate's comma-separated host names and IPs")
    certFn = flag.String("cert", "cert.pem", "certificate file name")
    keyFn  = flag.String("key", "key.pem", "private key file name")
)

func main() {
    flag.Parse()

    serial, err := 1rand.Int(rand.Reader, new(big.Int).Lsh(big.NewInt(1), 
        128))
    if err != nil {
        log.Fatal(err)
    }

    notBefore := time.Now()
    template := x509.Certificate{
        SerialNumber: serial,
        Subject: pkix.Name{
            Organization: []string{"Adam Woodbeck"},
        },
        NotBefore: notBefore,
        NotAfter:  notBefore.Add(10 * 356 * 24 * time.Hour),
        KeyUsage: x509.KeyUsageKeyEncipherment |
            x509.KeyUsageDigitalSignature |
            x509.KeyUsageCertSign,
        ExtKeyUsage: []x509.ExtKeyUsage{
            x509.ExtKeyUsageServerAuth,
          2x509.ExtKeyUsageClientAuth,
        },
        BasicConstraintsValid: true,
        IsCA:                  true,
    }
```

Listing 11-12: Creating an X.509 certificate template (_cert/generate.go_)

The command line utility accepts a comma-separated list of hostnames and IP addresses that will use the certificate. It also allows you to specify the certificate and private-key filenames, but it defaults to our familiar _cert.pem_ and _key.pem_ filenames.

The process of generating a certificate and a private key involves building a template in your code that you then encode to the X.509 format. Each certificate needs a serial number, which a certificate authority typically assigns. Since you’re generating your own self-signed certificate, you generate your own serial number using a cryptographically random, unsigned 128-bit integer 1. You then create an `x509.Certificate` object that represents an X.509-formatted certificate and set various values, such as the serial number, the certificate’s subject, the validity lifetime, and various usages for this certificate. Since you want to use this certificate for client authentication, you must include the `x509.ExtKeyUsageClientAuth` value 2. If you omit this value, the server won’t be able to verify the certificate when presented by the client.

The template is almost ready. You just need to add the hostnames and IP addresses before generating the certificate (see [Listing 11-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-13)).

```
--snip--

    for _, h := range 1strings.Split(*host, ",") {
        if ip := net.ParseIP(h); ip != nil {
         2 template.IPAddresses = append(template.IPAddresses, ip)
        } else {
         3 template.DNSNames = append(template.DNSNames, h)
        }
    }

    priv, err := 4ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
    if err != nil {
        log.Fatal(err)
    }

    der, err := 5x509.CreateCertificate(rand.Reader, &template,
        &template, &priv.PublicKey, priv)
    if err != nil {
        log.Fatal(err)
    }

    cert, err := os.Create(*certFn)
    if err != nil {
        log.Fatal(err)
    }

    err = 6pem.Encode(cert, &pem.Block{Type: "CERTIFICATE", Bytes: der})
    if err != nil {
        log.Fatal(err)
    }

    if err := cert.Close(); err != nil {
        log.Fatal(err)
    }
    log.Println("wrote", *certFn)
```

Listing 11-13: Writing the Privacy-Enhanced Mail (PEM)–encoded certificate (_cert/generate.go_)

You loop through the comma-separated list of hostnames and IP addresses 1, assigning each to its appropriate slice in the template. If the hostname is an IP address, you assign it to the `IPAddresses` slice 2. Otherwise, you assign the hostname to the `DNSNames` slice 3. Go’s TLS client uses these values to authenticate a server. For example, if the client connects to _https://www.google.com_ but the common name or alternative names in the server’s certificate do not match _www.google.com_’s hostname or resolved IP address, the client fails to authenticate the server.

Next you generate a new ECDSA private key 4 using the P-256 elliptic curve. At this point, you have everything you need to generate the certificate. The `x509.CreateCertificate` function 5 accepts a source of entropy (`crypto/rand`’s `Reader` is ideal), the template for the new certificate, a parent certificate, a public key, and a corresponding private key. It then returns a slice of bytes containing the Distinguished Encoding Rules (DER)–encoded certificate. You use your template for the parent certificate since the resulting certificate signs itself. All that’s left to do is create a new file, generate a new `pem.Block` with the DER-encoded byte slice, and PEM-encode everything to the new file 6. You don’t have to concern yourself with the various encodings. Go is quite happy with using PEM-encoded certificates on disk.

Now that you have a new certificate on disk, let’s write the corresponding private key in [Listing 11-14](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-14).

```
--snip--

    key, err := os.OpenFile(*keyFn, os.O_WRONLY|os.O_CREATE|os.O_TRUNC,
      10600)
    if err != nil {
        log.Fatal(err)
    }

    privKey, err := 2x509.MarshalPKCS8PrivateKey(priv)
    if err != nil {
        log.Fatal(err)
    }

    err = 3pem.Encode(key, &pem.Block{Type: "EC PRIVATE KEY",
    Bytes: privKey})
    if err != nil {
        log.Fatal(err)
    }

    if err := key.Close(); err != nil {
        log.Fatal(err)
    }
    log.Println("wrote", *keyFn)
}
```

Listing 11-14: Writing the PEM-encoded private key (_cert/generate.go_)

Whereas the certificate is meant to be publicly shared, the private key is just that: private. You should take care to assign it minimal permissions. Here, you’re giving only the user read-write access to the private-key file 1 and removing access for everyone else. We marshal the private key into a byte slice 2 and, similarly, assign it to a new `pem.Block` before writing the PEM-encoded output to the private-key file 3.

[Listing 11-15](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-15) uses the preceding code to generate certificate and key pairs for the server and the client.

```
$ go run cert/generate.go -cert serverCert.pem -key serverKey.pem -host localhost
2006/01/02 15:04:05 wrote serverCert.pem
2006/01/02 15:04:05 wrote serverKey.pem
$ go run cert/generate.go -cert clientCert.pem -key clientKey.pem -host localhost
2006/01/02 15:04:05 wrote clientCert.pem
2006/01/02 15:04:05 wrote clientKey.pem
```

Listing 11-15: Generating a certificate and private-key pair for the server and the client

Since the server binds to _localhost_ and the client connects to the server from _localhost_, this value is appropriate for both the client and server certificates. If you want to move the client to a different hostname or bind the server to an IP address, for example, you’ll need to change the _host_ flag accordingly.

### Implementing Mutual TLS

Now that you’ve generated certificate and private-key pairs for both the server and the client, you can start writing their code. Let’s write a test that implements mutual TLS authentication between our echo server and a client, starting in [Listing 11-16](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-16).

```
package ch11

import (
    "bytes"
    "context"
    "crypto/tls"
    "crypto/x509"
    "errors"
    "io/ioutil"
    "strings"
    "testing"
)

func caCertPool(caCertFn string) (*x509.CertPool, error) {
    caCert, err := 1ioutil.ReadFile(caCertFn)
    if err != nil {
        return nil, err
    }

    certPool := x509.NewCertPool()
    if ok := 2certPool.AppendCertsFromPEM(caCert); !ok {
        return nil, errors.New("failed to add certificate to pool")
    }

    return certPool, nil
}
```

Listing 11-16: Creating a certificate pool to serve CA certificates (_tls_mutual_test.go_)

Both the client and server use the `caCertPool` function to create a new X.509 certificate pool. The function accepts the file path to a PEM-encoded certificate, which you read in 1 and append to the new certificate pool 2. The certificate pool serves as a source of trusted certificates. The client puts the server’s certificate in its certificate pool, and vice versa.

[Listing 11-17](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-17) details the initial test code to demonstrate mutual TLS authentication between a client and a server.

```
--snip--

func TestMutualTLSAuthentication(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    serverPool, err := caCertPool(1"clientCert.pem")
    if err != nil {
        t.Fatal(err)
    }

    cert, err := 2tls.LoadX509KeyPair("serverCert.pem", "serverKey.pem")
    if err != nil {
        t.Fatalf("loading key pair: %v", err)
    }
```

Listing 11-17: Instantiating a CA cert pool and a server certificate (_tls_mutual_test.go_)

Before creating the server, you need to first populate a new CA certificate pool with the client’s certificate 1. You also need to load the server’s certificate at this point 2 instead of relying on the server’s `ServeTLS` method to do it for you, as you have in previous listings. Why you need the server’s certificate now will be clear when you see the TLS configuration changes in [Listing 11-18](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-18).

```
--snip--

    serverConfig := &tls.Config{
        Certificates: []tls.Certificate{cert},
     1 GetConfigForClient: func(hello *tls.ClientHelloInfo) (*tls.Config,
            error) {
            return &tls.Config{
                Certificates:             []tls.Certificate{2cert},
             3 ClientAuth:               tls.RequireAndVerifyClientCert,
             4 ClientCAs:                serverPool,
                CurvePreferences:         []tls.CurveID{tls.CurveP256},
                MinVersion:             5tls.VersionTLS13,
                PreferServerCipherSuites: true,
```

Listing 11-18: Accessing the client’s hello information using GetConfigForClient (_tls_mutual_test.go_)

Remember that in [Listing 11-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-13), you defined the `IPAddresses` and `DNSNames` slices of the template used to generate your client’s certificate. These values populate the common name and alternative names portions of the client’s certificate. You learned that Go’s TLS client uses these values to authenticate the server. But the server does not use these values from the client’s certificate to authenticate the client.

Since you’re implementing mutual TLS authentication, you need to make some changes to the server’s certificate verification process so that it authenticates the client’s IP address or hostnames against the client certificate’s common name and alternative names. To do that, the server at the very least needs to know the client’s IP address. The only way you can get client connection information before certificate verification is by defining the `tls.Config`’s `GetConfigForClient` method 1. This method allows you to define a function that receives the `*tls.ClientHelloInfo` object created as part of the TLS handshake process with the client. From this, you can retrieve the client’s IP address. But first, you need to return a proper TLS configuration.

You add the server’s certificate to the TLS configuration 2 and the server pool to the TLS configuration’s `ClientCAs` field 4. This field is the server’s equivalent to the TLS configuration’s `RootCAs` field on the client. You also need to tell the server that every client must present a valid certificate before completing the TLS handshake process 3. Since you control both the client and the server, specify a minimum TLS protocol version of 1.3 5.

This function returns the same TLS configuration for every client connection. As mentioned, the only reason you’re using the `GetConfigForClient` method is so you can retrieve the client’s IP from its hello information. [Listing 11-19](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-19) implements the verification process that authenticates the client by using its IP address and its certificate’s common name and alternative names.

```
--snip--

             1 VerifyPeerCertificate: func(rawCerts [][]byte,
                    verifiedChains [][]*x509.Certificate) error {

                    opts := x509.VerifyOptions{
                        KeyUsages: []x509.ExtKeyUsage{
                          2x509.ExtKeyUsageClientAuth,
                        },
                        Roots: 3serverPool,
                    }

                    ip := strings.Split(hello.Conn.RemoteAddr().String(),
                        ":")[0]
                    hostnames, err := 4net.LookupAddr(ip)
                    if err != nil {
                        t.Errorf("PTR lookup: %v", err)
                    }
                    hostnames = append(hostnames, ip)

                    for _, chain := range verifiedChains {
                        opts.Intermediates = x509.NewCertPool()
                        for _, cert := range 5chain[1:] {
                            opts.Intermediates.AddCert(cert)
                        }

                        for _, hostname := range hostnames {
                            opts.DNSName = 6hostname
                            _, err = chain[0].Verify(opts)
                            if err == nil {
                                return nil
                            }
                        }
                    }

                    return errors.New("client authentication failed")
                },
            }, nil
        },
    }
```

Listing 11-19: Making the server authenticate the client’s IP and hostnames (_tls_mutual_test.go_)

Since you want to augment the usual certificate verification process on the server, you define an appropriate function and assign it to the TLS configuration’s `VerifyPeerCertificate` method 1. The server calls this method after the normal certificate verification checks. The only check you’re performing above and beyond the normal checks is to verify the client’s hostname with the leaf certificate.

The _leaf certificate_ is the last certificate in the certificate chain given to the server by the client. The leaf certificate contains the client’s public key. All other certificates in the chain are intermediate certificates used to verify the authenticity of the leaf certificate and culminate with the certificate authority’s certificate. You’ll find each leaf certificate at index 0 in each `verifiedChains` slice. In other words, you can find the leaf certificate of the first chain at `verifiedChains[0][0]`. If the server calls your function assigned to the `VerifyPeerCertificate` method, the leaf certificate in the first chain exists at a minimum.

Create a new `x509.VerifyOptions` object and modify the `KeyUsages` method to indicate you want to perform client authentication 2. Then, assign the server pool to the `Roots` method 3. The server uses this pool as its trusted certificate source during verification.

Now, extract the client’s IP address from the connection object in the `*tls.ClientHelloInfo` object named `hello` passed into [Listing 11-18](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-18)’s `GetConfigForClient` method. Use the IP address to perform a reverse DNS lookup 4 to consider any hostnames assigned to the client’s IP address. If this lookup fails or returns an empty slice, the way you handle that situation is up to you. If you’re relying on the client’s hostname for authentication and the reverse lookup fails, you cannot authenticate the client. But if you’re using the client’s IP address only in the certificate’s common name or alternative names, then a reverse lookup failure is inconsequential. For demonstration purposes, we’ll consider a failed reverse lookup to equate to a failed test. At minimum, you append the client’s IP address to the `hostnames` slice.

All that’s left to do is loop through each verified chain, assign a new intermediate certificate pool to `opts.Intermediates`, add all certificates but the leaf certificate to the intermediate certificate pool 5, and attempt to verify the client 6. If verification returns a `nil` error, you authenticated the client. If you fail to verify each hostname with each leaf certificate, return an error to indicate that client authentication failed. The client will receive an error, and the server will terminate the connection.

Now that the server’s TLS configuration properly authenticates client certificates, continue with the server implementation in [Listing 11-20](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-20).

```
--snip--

    serverAddress := "localhost:44443"
    server := NewTLSServer(ctx, serverAddress, 0, 1serverConfig)
    done := make(chan struct{})

    go func() {
        err := server.ListenAndServeTLS("serverCert.pem", "serverKey.pem")
        if err != nil &&!strings.Contains(err.Error(),
            "use of closed network connection") {
            t.Error(err)
            return
        }
        done <- struct{}{}
    }()
 2 server.Ready()
```

Listing 11-20: Starting the TLS server (_tls_mutual_test.go_)

Create a new TLS server instance, making sure to pass in the TLS configuration you just created 1. Call its `ListenAndServeTLS` method in a goroutine and make sure to wait until the server is ready for connections 2 before proceeding.

Now that the server implementation is ready, let’s move on to the client portion of the test. [Listing 11-21](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-21) implements a TLS client that can present _clientCert.pem_ upon request by the server.

```
--snip--

    clientPool, err := caCertPool(1"serverCert.pem")
    if err != nil {
        t.Fatal(err)
    }

    clientCert, err := tls.LoadX509KeyPair("clientCert.pem", "clientKey.pem")
    if err != nil {
        t.Fatal(err)
    }

    conn, err := tls.Dial("tcp", serverAddress, &tls.Config{
     2 Certificates:     []tls.Certificate{clientCert},
        CurvePreferences: []tls.CurveID{tls.CurveP256},
        MinVersion:       tls.VersionTLS13,
     3 RootCAs:          clientPool,
    })
    if err != nil {
        t.Fatal(err)
    }
```

Listing 11-21: Pinning the server certificate to the client (_tls_mutual_test.go_)

The client retrieves a new certificate pool populated with the server’s certificate 1. The client then uses the certificate pool in the `RootCAs` field of its TLS configuration 3, meaning the client will trust only server certificates signed by _serverCert.pem_. You also configure the client with its own certificate 2 to present to the server upon request.

It’s worth noting that the client and server have not initialized a TLS session yet. They haven’t completed the TLS handshake. If `tls.Dial` returns an error, it isn’t because of an authentication issue but more likely a TCP connection issue. Let’s continue with the client code to initiate the handshake (see [Listing 11-22](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c11.xhtml#listing11-22)).

```
--snip--

    hello := []byte("hello")
    _, err = conn.Write(hello)
    if err != nil {
        t.Fatal(err)
    }

    b := make([]byte, 1024)
    n, err := 1conn.Read(b)
    if err != nil {
        t.Fatal(err)
    }

    if actual := b[:n]; !bytes.Equal(hello, actual) {
        t.Fatalf("expected %q; actual %q", hello, actual)
    }

    err = conn.Close()
    if err != nil {
        t.Fatal(err)
    }

    cancel()
    <-done
}
```

Listing 11-22: TLS handshake completes as you interact with the connection (_tls_mutual_test.go_)

The first read from, or write to, the socket connection automatically initiates the handshake process between the client and the server. If the server rejects the client certificate, the read call 1 will return a _bad certificate_ error. But if you created appropriate certificates and properly pinned them, both the client and the server are happy, and this test passes.

## What You’ve Learned

Transport Layer Security provides both authentication and encrypted communication between a client and a server. The server presents a certificate, signed by certificate authority, to a client as part of the TLS handshake process. The client verifies the certificate’s signatory. If a third party, trusted by the client, signed the server’s certificate, the server is authentic in the eyes of the client. From that point forward, the client and server communicate using symmetric-key cryptography.

By default, Go’s TLS configuration uses the operating system’s trusted certificate storage. This storage typically consists of certificates from the world’s foremost trusted certificate authorities. However, we can modify the TLS configuration to trust specific keys, a process known as key pinning.

We can also modify a server’s TLS configuration to require a certificate from the client. The server would then use this certificate to authenticate the client in the same manner the client authenticates the server. This process is known as mutual TLS authentication.

TLS 1.3 provides forward secrecy for all communication between a client and server. This means that compromising one session does not compromise any other session. Both the client and server generate per-session public- and private-key pairs. They also exchange an ephemeral shared secret as part of the handshake process. Once the session ends, the client and server shall purge the shared secret and their temporary key pairs. An attacker who was able to capture the shared secret and session traffic would be able to decrypt only that session’s traffic. An attacker could not use the shared secret from one session to decrypt traffic from any other session.

Even though TLS is ubiquitous and secures much of the world’s digital communication, attackers can compromise it. Part of a certificate authority’s job is to verify that the entity requesting a certificate for a specific domain name owns the domain name. If attackers dupe a certificate authority, or the certificate authority otherwise makes a mistake and issues a fraudulent certificate, the owner of the fraudulent certificate could masquerade as Google, for example, and trick people into divulging sensitive information.

Another attack vector includes fooling a client into adding the attacker’s certificate into the client’s trusted certificate storage. The attacker could then issue and sign any certificate they want, and the client would inherently trust that the attacker is who their certificate claims them to be.

An attacker could also compromise a server and intercept TLS session keys and secrets, or even capture traffic at the application later after the server has decrypted it.

Overall, however, these attacks are rare, and TLS succeeds at achieving its goals of authentication and encrypted communication.