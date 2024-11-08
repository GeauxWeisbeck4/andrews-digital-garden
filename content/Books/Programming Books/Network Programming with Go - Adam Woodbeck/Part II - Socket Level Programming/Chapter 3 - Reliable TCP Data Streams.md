---
id: 01JC4FZV144VXFSZWK4T7C7FCQ
modified: 2024-11-07T18:40:55-05:00
title: Chapter 3 - Reliable TCP Data Streams
tags:
  - go-lang
  - networking
  - programming
  - books
  - no-starch-press
---
# 3  
RELIABLE TCP DATA STREAMS

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

TCP allows you to reliably stream data between nodes on a network. This chapter takes a deeper dive into the protocol, focusing on the aspects directly influenced by the code we’ll write to establish TCP connections and transmit data over those connections. This knowledge should help you debug network-related issues in your programs.

We’ll start by covering the TCP handshake process, its sequence numbers, acknowledgments, retransmissions, and other features. Next, we’ll implement the steps of a TCP session in Go, from dialing, listening, and accepting to the session termination. Then, we’ll discuss time-outs and temporary errors, how to detect them, and how to use them to keep our users happy. Finally, we’ll cover the early detection of unreliable network connections. Go’s standard library allows you to write robust TCP-based networking applications. But it doesn’t hold your hand. If you aren’t mindful of managing incoming data or properly closing connections, you’ll experience insidious bugs in your programs.

## What Makes TCP Reliable?

TCP is reliable because it overcomes the effects of packet loss or receiving packets out of order. _Packet loss_ occurs when data fails to reach its destination—typically because of data transmission errors (such as wireless network interference) or network congestion. _Network congestion_ happens when nodes attempt to send more data over a network connection than the connection can handle, causing the nodes to discard the excess packets. For example, you can’t send data at a rate of 1 gigabit per second (Gbps) over a 10 megabit-per-second (Mbps) connection. The 10Mbps connection quickly becomes saturated, and nodes involved in the flow of the data drop the excess data.

TCP adapts its data transfer rate to make sure it transmits data as fast as possible while keeping dropped packets to a minimum, even if the network conditions change—for example, the Wi-Fi signal fades, or the destination node becomes overwhelmed with data. This process, called _flow control_, does its best to make up for the deficiencies of the underlying network media. TCP cannot send good data over a bad network and is at the mercy of the network hardware.

TCP also keeps track of received packets and retransmits unacknowledged packets, as necessary. Recipients can also receive packets out of sequence if, for example, data is rerouted in transit. Remember from Chapter 2 that routing protocols use metrics to determine how to route packets. These metrics may change as network conditions change. There is no guarantee that all packets you send take the same route for the duration of the TCP session. Thankfully, TCP organizes unordered packets and processes them in sequence.

Together with flow control and retransmission, these properties allow TCP to overcome packet loss and facilitate the delivery of data to the recipient. As a result, TCP eliminates the need for you to concern yourself with these errors. You are free to focus on the data you send and receive.

## Working with TCP Sessions

A _TCP session_ allows you to deliver a stream of data of any size to a recipient and receive confirmation that the recipient received the data. This saves you from the inefficiency of sending a large amount of data across a network, only to find out at the end of the transmission that the recipient didn’t receive it.

Much like the occasional head nod that people use to indicate they’re listening to someone speaking, streaming allows you to receive feedback from the recipient while the transfer is taking place so that you can correct any errors in real time. In fact, you can think of a TCP session as you would a conversation between two nodes. It starts with a greeting, progresses into the conversation, and concludes with a farewell.

As we discuss the specifics of TCP, I want you to understand that Go takes care of the implementation details for you. Your code will take advantage of the `net` package’s interfaces when working with TCP connections.

### Establishing a Session with the TCP Handshake

A TCP connection uses a three-way handshake to introduce the client to the server and the server to the client. The handshake creates an established TCP session over which the client and server exchange data. [Figure 3-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#figure3-1) illustrates the three messages sent in the handshake process.

![f03001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c03/f03001.png)

Figure 3-1: The three-way handshake process leading to an established TCP session

Before it can establish a TCP session, the server must listen for incoming connections. (I use the terms _server_ and _client_ in this chapter to refer to the listening node and dialing node, respectively. TCP itself doesn’t have a concept of a client and server, but an established session between two nodes, whereby one node reaches out to another node to establish the session.)

As the first step of the handshake, the client sends a packet with the _synchronize (SYN) flag_ to the server. This SYN packet informs the server of the client’s capabilities and preferred window settings for the rest of the conversation. We’ll discuss the receive window shortly. Next, the server responds with its own packet, with both the _acknowledgment (ACK)_ and SYN flags set. The ACK flag tells the client that the server acknowledges receipt of the client’s SYN packet. The server’s SYN packet tells the client what settings it’s agreed to for the duration of the conversation. Finally, the client replies with an ACK packet to acknowledge the server’s SYN packet, completing the three-way handshake.

Completion of the three-way handshake process establishes the TCP session, and nodes may then exchange data. The TCP session remains idle until either side has data to transmit. Unmanaged and lengthy idle TCP sessions may result in wasteful consumption of memory. We’ll cover techniques for managing idle connections in your code later in this chapter.

When you initiate a connection in your code, Go will return either a connection object or an error. If you receive a connection object, the TCP handshake succeeded. You do not need to manage the handshake yourself.

### Acknowledging Receipt of Packets by Using Their Sequence Numbers

Each TCP packet contains a _sequence number_, which the receiver uses to acknowledge receipt of each packet and properly order the packets for presentation to your Go application ([Figure 3-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#figure3-2)).

![f03002](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c03/f03002.png)

Figure 3-2: Client and server exchanging sequence numbers

The client’s operating system determines the initial sequence number (X in [Figure 3-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#figure3-2)) and sends it to the server in the client’s SYN packet during the handshake. The server acknowledges receipt of the packet by including this sequence number in its ACK packet to the client. Likewise, the server shares its generated sequence number Y in its SYN packet to the client. The client replies with its ACK to the server.

An ACK packet uses the sequence number to tell the sender, “I’ve received all packets up to and including the packet with this sequence number.” One ACK packet can acknowledge the receipt of one or more packets from the sender. The sender uses the sequence number in the ACK packet to determine whether it needs to retransmit any packets. For example, if a sender transmits a bunch of packets with sequence numbers up through 100 but then receives an ACK from the receiver with sequence number 90, the sender knows it needs to retransmit packets from sequence numbers 91 to 100.

While writing and debugging network programs, it’s often necessary to view the traffic your code sends and receives. To capture and inspect TCP packets, I strongly recommend you familiarize yourself with Wireshark ([https://www.wireshark.org/](https://www.wireshark.org/)). This program will go a long way toward helping you understand how your code influences the data sent over the network. To learn more, see _Practical Packet Analysis_, 3rd Edition, by Chris Sanders (No Starch, 2017).

If you view your application’s network traffic in Wireshark, you may notice _selective acknowledgments (SACKs)_. These are ACK packets used to acknowledge the receipt of a _subset_ of sent packets. For example, let’s assume the sender transmitted a hundred packets but only packets 1 to 59 and 81 to 100 made it to the receiver. The receiver could send a SACK to inform the sender what subset of packets it received.

Here again, Go handles the low-level details. Your code will not need to concern itself with sequence numbers and acknowledgments.

### Receive Buffers and Window Sizes

Since TCP allows a single ACK packet to acknowledge the receipt of more than one incoming packet, the receiver must advertise to the sender how much space it has available in its receive buffer before it sends an acknowledgment. A _receive buffer_ is a block of memory reserved for incoming data on a network connection. The receive buffer allows the node to accept a certain amount of data from the network without requiring an application to immediately read the data. Both the client and the server maintain their own per-connection receive buffer. When your Go code reads data from a network connection object, it reads the data from the connection’s receive buffer.

ACK packets include a particularly important piece of information: the _window size_, which is the number of bytes the sender can transmit to the receiver without requiring an acknowledgment. If the client sends an ACK packet to the server with a window size of 24,537, the server knows it can send 24,537 bytes to the client before expecting the client to send another ACK packet. A window size of zero indicates that the receiver’s buffer is full and can no longer receive additional data. We’ll discuss this scenario a bit later in this chapter.

Both the client and the server keep track of each other’s window size and do their best to completely fill each other’s receive buffers. This method—of receiving the window size in an ACK packet, sending data, receiving an updated window size in the next ACK, and then sending more data—is known as a _sliding window_, as shown in [Figure 3-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#figure3-3). Each side of the connection offers up a window of data that can it can receive at any one time.

![f03003](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c03/f03003.png)

Figure 3-3: A client’s ACKs advertising the amount of data it can receive

In this snippet of communication, the client sends an ACK for previously received data. This ACK includes a window size of 3,072 bytes. The server now knows that it can send up to 3,072 bytes before it requires an ACK from the client. The server sends three packets with 1,024 bytes each to fill the client’s receive buffer. The client then sends another ACK with an updated window size of 2,048 bytes. This means that the application running on the client read 2,048 bytes from the receive buffer before the client sent its acknowledgment to the server. The server then sends two more packets of 1,024 bytes to fill the client’s receive buffer and waits for another ACK.

Here again, all you need to concern yourself with is reading and writing to the connection object Go gives you when you establish a TCP connection. If something goes wrong, Go will surely let you know by returning an error.

### Gracefully Terminating TCP Sessions

Like the handshake process, gracefully terminating a TCP session involves exchanging a sequence of packets. Either side of the connection may initiate the termination sequence by sending a _finish (FIN)_ packet. In [Figure 3-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#figure3-4), the client initiates the termination by sending a FIN packet to the server.

![f03004](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c03/f03004.png)

Figure 3-4: The client initiates a TCP session termination with the server.

The client’s connection state changes from ESTABLISHED to FIN_WAIT_1, which indicates the client is in the process of tearing down the connection from its end and is waiting for the server’s acknowledgment. The server acknowledges the client’s FIN and changes its connection state from ESTABLISHED to CLOSE_WAIT. The server sends its own FIN packet, changing its state to LAST_ACK, indicating it’s waiting for a final acknowledgment from the client. The client acknowledges the server’s FIN and enters a TIME_WAIT state, whose purpose is to allow the client’s final ACK packet to reach the server. The client waits for twice the maximum segment lifetime (the segment lifetime arbitrarily defaults to two minutes, per RFC 793, but your operating system may allow you to tweak this value), then changes its connection state to CLOSED without any further input required from the server. The _maximum segment lifetime_ is the duration a TCP segment can remain in transit before the sender considers it abandoned. Upon receiving the client’s last ACK packet, the server immediately changes its connection state to CLOSED, fully terminating the TCP session.

Like the initial handshake, Go handles the details of the TCP connection teardown process when you close a connection object.

### Handling Less Graceful Terminations

Not all connections politely terminate. In some cases, the application that opened a TCP connection may crash or abruptly stop running for some reason. When this happens, the TCP connection is immediately closed. Any packets sent from the other side of the former connection will prompt the closed side of the connection to return a _reset (RST) packet_. The RST packet informs the sender that the receiver’s side of the connection closed and will no longer accept data. The sender should close its side of the connection knowing the receiver ignored any packets it did not acknowledge.

Intermediate nodes, such as firewalls, can send RST packets to each node in a connection, effectively terminating the socket from the middle.

## Establishing a TCP Connection by Using Go’s Standard Library

The `net` package in Go’s standard library includes good support for creating TCP-based servers and clients capable of connecting to those servers. Even so, it’s your responsibility to make sure you handle the connection appropriately. Your software should be attentive to incoming data and always strive to gracefully shut down the connection. Let’s write a TCP server that can listen for incoming TCP connections, initiate connections from a client, accept and asynchronously handle each connection, exchange data, and terminate the connection.

### Binding, Listening for, and Accepting Connections

To create a TCP server capable of listening for incoming connections (called a _listener_), use the `net.Listen` function. This function will return an object that implements the `net.Listener` interface. [Listing 3-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-1) shows the creation of a listener.

```
package ch03

import (
    "net"
    "testing"
)

func TestListener(t *testing.T) {
  1listener, err := net.Listen("2tcp", "3127.0.0.1:0")
    if err != nil {
        t.Fatal(err)
    }
   4  defer func() { _ = listener.Close() }()

    t.Logf("bound to %q", 5listener.Addr())
}
```

Listing 3-1: Creating a listener on 127.0.0.1 using a random port (_listen_test.go_)

The `net.Listen` function accepts a network type 2 and an IP address and port separated by a colon 3. The function returns a `net.Listener` interface 1 and an `error` interface. If the function returns successfully, the listener is bound to the specified IP address and port. _Binding_ means that the operating system has exclusively assigned the port on the given IP address to the listener. The operating system allows no other processes to listen for incoming traffic on bound ports. If you attempt to bind a listener to a currently bound port, `net.Listen` will return an error.

You can choose to leave the IP address and port parameters empty. If the port is zero or empty, Go will randomly assign a port number to your listener. You can retrieve the listener’s address by calling its `Addr` method 5. Likewise, if you omit the IP address, your listener will be bound to all unicast and anycast IP addresses on the system. Omitting both the IP address and port, or passing in a colon for the second argument to `net.Listen`, will cause your listener to bind to all unicast and anycast IP addresses using a random port.

In most cases, you should use `tcp` as the network type for `net.Listener`’s first argument. You can restrict the listener to just IPv4 addresses by passing in `tcp4` or exclusively bind to IPv6 addresses by passing in `tcp6`.

You should always be diligent about closing your listener gracefully by calling its `Close` method 4, often in a `defer` if it makes sense for your code. Granted, this is a test case, and Go will tear down the listener when the test completes, but it’s good practice nonetheless. Failure to close the listener may lead to memory leaks or deadlocks in your code, because calls to the listener’s `Accept` method may block indefinitely. Closing the listener immediately unblocks calls to the `Accept` method.

[Listing 3-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-2) demonstrates how a listener can accept incoming TCP connections.

```
   1  for {
      2conn, err := 3listener.Accept()
        if err != nil {
            return err
        }

       4  go func(c net.Conn) {
       5      defer c.Close()

            // Your code would handle the connection here.
        }(conn)
    }
```

Listing 3-2: Accepting and handling incoming TCP connection requests

Unless you want to accept only a single incoming connection, you need to use a `for` loop 1 so your server will accept each incoming connection, handle it in a goroutine, and loop back around, ready to accept the next connection. Serially accepting connections is perfectly acceptable and efficient, but beyond that point, you should use a goroutine to handle each connection. You could certainly write serialized code after accepting a connection if your use case demands it, but it would be woefully inefficient and fail to take advantage of Go’s strengths. We start the `for` loop by calling the listener’s `Accept` method 2. This method will block until the listener detects an incoming connection and completes the TCP handshake process between the client and the server. The call returns a `net.Conn` interface 3 and an `error`. If the handshake failed or the listener closed, for example, the error interface would be non-`nil`.

The connection interface’s underlying type is a pointer to a `net.TCPConn` object because you’re accepting TCP connections. The connection interface represents the server’s side of the TCP connection. In most cases, `net.Conn` provides all methods you’ll need for general interactions with the client. However, the `net.TCPConn` object provides additional functionality we’ll cover in Chapter 4 should you require more control.

To concurrently handle client connections, you spin off a goroutine to asynchronously handle each connection 4 so your listener can ready itself for the next incoming connection. Then you call the connection’s `Close` method 5 before the goroutine exits to gracefully terminate the connections by sending a FIN packet to the server.

### Establishing a Connection with a Server

From the client’s side, Go’s standard library `net` package makes reaching out and establishing a connection with a server a simple matter. [Listing 3-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-3) is a test that demonstrates the process of initiating a TCP connection with a server listening to 127.0.0.1 on a random port.

```
package ch03

import (
    "io"
    "net"
    "testing"
)

func TestDial(t *testing.T) {
    // Create a listener on a random port.
    listener, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    done := make(chan struct{})
   1  go func() {
        defer func() { done <- struct{}{} }()

        for {
            conn, err := 2listener.Accept()
            if err != nil {
                t.Log(err)
                return
            }

           3  go func(c net.Conn) {
                defer func() {
                    c.Close()
                    done <- struct{}{}
                }()

                buf := make([]byte, 1024)
                for {
                    n, err := 4c.Read(buf)
                    if err != nil {
                        if err != io.EOF {
                            t.Error(err)
                        }
                        return
                    }

                    t.Logf("received: %q", buf[:n])
                }
            }(conn)
        }
    }()

  5conn, err := net.Dial("6tcp", 7listener.Addr().String())
    if err != nil {
        t.Fatal(err)
    }

   8  conn.Close()
    <-done
   9  listener.Close()
    <-done
}
```

Listing 3-3: Establishing a connection to 127.0.0.1 (_dial_test.go_)

You start by creating a listener on the IP address 127.0.0.1, which the client will connect to. You omit the port number altogether, so Go will randomly pick an available port for you. Then, you spin off the listener in a goroutine 1 so you can work with the client’s side of the connection later in the test. The listener’s goroutine contains code like [Listing 3-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-2)’s for accepting incoming TCP connections in a loop, spinning off each connection into its own goroutine. (We often call this goroutine a _handler_. I’ll explain the implementation details of the handler shortly, but it will read up to 1024 bytes from the socket at a time and log what it received.)

The standard library’s `net`.`Dial` function is like the `net.Listen` function in that it accepts a network 6 like `tcp` and an IP address and port combination 7—in this case, the IP address and port of the listener to which it’s trying to connect. You can use a hostname in place of an IP address and a service name, like _http_, in place of a port number. If a hostname resolves to more than one IP address, Go will attempt a connection to each one in order until a connection succeeds or all IP addresses have been exhausted. Since IPv6 addresses include colon delimiters, you must enclose an IPv6 address in square brackets. For example, `"[2001:ed27::1]:https"` specifies port 443 at the IPv6 address 2001:ed27::1. `Dial` returns a connection object 5 and an `error` interface value.

Now that you’ve established a successful connection to the listener, you initiate a graceful termination of the connection from the client’s side 8. After receiving the FIN packet, the `Read` method 4 returns the `io.EOF` error, indicating to the listener’s code that you closed your side of the connection. The connection’s handler 3 exits, calling the connection’s `Close` method on the way out. This sends a FIN packet to your connection, completing the graceful termination of the TCP session.

Finally, you close the listener 9. The listener’s `Accept` method 2 immediately unblocks and returns an error. This error isn’t necessarily a failure, so you simply log it and move on. It doesn’t cause your test to fail. The listener’s goroutine 1 exits, and the test completes.

#### Understanding Time-outs and Temporary Errors

In a perfect world, your connection attempts will immediately succeed, and all read and write attempts will never fail. But you need to hope for the best and prepare for the worst. You need a way to determine whether an error is temporary or something that warrants termination of the connection altogether. The `error` interface doesn’t provide enough information to make that determination. Thankfully, Go’s `net` package provides more insight if you know how to use it.

Errors returned from functions and methods in the `net` package typically implement the `net.Error` interface, which includes two notable methods: `Timeout` and `Temporary`. The `Timeout` method returns `true` on Unix-based operating systems and Windows if the operating system tells Go that the resource is temporarily unavailable, the call would block, or the connection timed out. We’ll touch on time-outs and how you can use them to your advantage a bit later in this chapter. The `Temporary` method returns `true` if the error's `Timeout` function returns `true`, the function call was interrupted, or there are too many open files on the system, usually because you’ve exceeded the operating system’s resource limit.

Since the functions and methods in the `net` package return the more general `error` interface, you’ll see the code in this chapter use type assertions to verify you received a `net.Error`, as in [Listing 3-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-4).

```
if nErr, ok := err.(net.Error); ok && !nErr.Temporary() { return err }
```

Listing 3-4: Asserting a net.Error to check whether the error was temporary

Robust network code won’t rely exclusively on the `error` interface. Rather, it will readily use `net.Error`’s methods, or even dive in further and assert the underlying `net.OpError` struct, which contains more details about the connection, such as the operation that caused the error, the network type, the source address, and more. I encourage you to read the `net.OpError` documentation (available at [https://golang.org/pkg/net/#OpError/](https://golang.org/pkg/net/#OpError/)) to learn more about specific errors beyond what the `net.Error` interface provides.

#### Timing Out a Connection Attempt with the DialTimeout Function

Using the `Dial` function has one potential problem: you are at the mercy of the operating system to time out each connection attempt. For example, if you use the `Dial` function in an interactive application and your operating system times out connection attempts after two hours, your application’s user may not want to wait that long, much less give your app a five-star rating.

To keep your applications predictable and your users happy, it’d be better to control time-outs yourself. For example, you may want to initiate a connection to a low-latency service that responds quickly if it’s available. If the service isn’t responding, you’ll want to time out quickly and move onto the next service.

One solution is to explicitly define a per-connection time-out duration and use the `DialTimeout` function instead. [Listing 3-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-5) implements this solution.

```
package ch03

import (
    "net"
    "syscall"
    "testing"
    "time"
)

func 1DialTimeout(network, address string, timeout time.Duration,
) (net.Conn, error) {
    d := net.Dialer{
       2  Control: func(_, addr string, _ syscall.RawConn) error {
            return &net.DNSError{
                Err:         "connection timed out",
                Name:        addr,
                Server:      "127.0.0.1",
                IsTimeout:   true,
                IsTemporary: true,
            }
        },
        Timeout: timeout,
    }
    return d.Dial(network, address)
}

func TestDialTimeout(t *testing.T) {
    c, err := DialTimeout("tcp", "10.0.0.1:http", 35*time.Second)
    if err == nil {
        c.Close()
        t.Fatal("connection did not time out")
    }
    nErr, ok := 4err.(net.Error)
    if !ok {
        t.Fatal(err)
    }
    if 5!nErr.Timeout() {
        t.Fatal("error is not a timeout")
    }
}
```

Listing 3-5: Specifying a time-out duration when initiating a TCP connection (_dial_timeout_test.go_)

Since the `net.DialTimeout` function 1 does not give you control of its `net.Dialer` to mock the dialer’s output, you’re using our own implementation that matches the signature. Your `DialTimeout` function overrides the `Control` function 2 of the `net.Dialer` to return an error. You’re mocking a DNS time-out error.

Unlike the `net.Dial` function, the `DialTimeout` function includes an additional argument, the time-out duration 3. Since the time-out duration is five seconds in this case, the connection attempt will time out if a connection isn’t successful within five seconds. In this test, you dial 10.0.0.0, which is a non-routable IP address, meaning your connection attempt assuredly times out. For the test to pass, you need to first use a type assertion to verify you’ve received a `net.Error`4 before you can check its `Timeout` method 5.

If you dial a host that resolves to multiple IP addresses, Go starts a connection race between each IP address, giving the primary IP address a head start. The first connection to succeed persists, and the remaining contenders cancel their connection attempts. If all connections fail or time out, `net.DialTimeout` returns an error.

#### Using a Context with a Deadline to Time Out a Connection

A more contemporary solution to timing out a connection attempt is to use a context from the standard library’s `context` package. A _context_ is an object that you can use to send cancellation signals to your asynchronous processes. It also allows you to send a cancellation signal after it reaches a deadline or after its timer expires.

All cancellable contexts have a corresponding `cancel` function returned upon instantiation. The `cancel` function offers increased flexibility since you can optionally cancel the context before the context reaches its deadline. You could also pass along its `cancel` function to hand off cancellation control to other bits of your code. For example, you could monitor for specific signals from your operating system, such as the one sent to your application when a user presses the CTRL-C key combination, to gracefully abort connection attempts and tear down existing connections before terminating your application.

[Listing 3-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-6) illustrates a test that accomplishes the same functionality as `DialTimeout`, using `context` instead.

```
package ch03

import (
    "context"
    "net"
    "syscall"
    "testing"
    "time"
)

func TestDialContext(t *testing.T) {
    1 dl := time.Now().Add(5 * time.Second)
    2 ctx, cancel := context.WithDeadline(context.Background(), dl)
    3 defer cancel()

    var d net.Dialer // DialContext is a method on a Dialer
    d.Control = 4func(_, _ string, _ syscall.RawConn) error {
        // Sleep long enough to reach the context's deadline.
        time.Sleep(5*time.Second + time.Millisecond)
        return nil
    }
    conn, err := d.DialContext(5ctx, "tcp", "10.0.0.0:80")
    if err == nil {
        conn.Close()
        t.Fatal("connection did not time out")
    }
    nErr, ok := err.(net.Error)
    if !ok {
        t.Error(err)
    } else {
        if !nErr.Timeout() {
            t.Errorf("error is not a timeout: %v", err)
        }
    }
   6  if ctx.Err() != context.DeadlineExceeded {
        t.Errorf("expected deadline exceeded; actual: %v", ctx.Err())
    }
}
```

Listing 3-6: Using a context with a deadline to time out the connection attempt (_dial_context_test.go_)

Before you make a connection attempt, you create the context with a deadline of five seconds into the future 1, after which the context will automatically cancel. Next, you create the context and its `cancel` function by using the `context.WithDeadline` function 2, setting the deadline in the process. It’s good practice to defer the `cancel` function 3 to make sure the context is garbage collected as soon as possible. Then, you override the dialer’s `Control` function 4 to delay the connection just long enough to make sure you exceed the context’s deadline. Finally, you pass in the context as the first argument to the `DialContext` function 5. The sanity check 6 at the end of the test makes sure that reaching the deadline canceled the context, not an erroneous call to `cancel`.

As with `DialTimeout`, if a host resolves to multiple IP addresses, Go starts a connection race between each IP address, giving the primary IP address a head start. The first connection to succeed persists, and the remaining contenders cancel their connection attempts. If all connections fail or the context reaches its deadline, `net.Dialer.DialContext` returns an error.

#### Aborting a Connection by Canceling the Context

Another advantage to using a context is the `cancel` function itself. You can use it to cancel the connection attempt on demand, without specifying a deadline, as shown in [Listing 3-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-7).

```
package ch03

import (
    "context"
    "net"
    "syscall"
    "testing"
    "time"
)

func TestDialContextCancel(t *testing.T) {
    ctx, cancel := 1context.WithCancel(context.Background())
    sync := make(chan struct{})

   2  go func() {
        defer func() { sync <- struct{}{} }()

        var d net.Dialer
        d.Control = func(_, _ string, _ syscall.RawConn) error {
            time.Sleep(time.Second)
            return nil
        }
        conn, err := d.DialContext(ctx, "tcp", "10.0.0.1:80")
        if err != nil {
            t.Log(err)
            return
        }

        conn.Close()
        t.Error("connection did not time out")
    }()

   3  cancel()
    <-sync

    if ctx.Err() != 4context.Canceled {
        t.Errorf("expected canceled context; actual: %q", ctx.Err())
    }
}
```

Listing 3-7: Directly canceling the context to abort the connection attempt

(_dial_cancel_test.go_)

Instead of creating a context with a deadline and waiting for the deadline to abort the connection attempt, you use `context.WithCancel` to return a context and a function to cancel the context 1. Since you’re manually canceling the context, you create a closure and spin it off in a goroutine to handle the connection attempt 2. Once the dialer is attempting to connect to and handshake with the remote node, you call the `cancel` function 3to cancel the context. This causes the `DialContext` method to immediately return with a non-`nil` error, exiting the goroutine. You can check the context’s `Err` method to make sure the call to cancel was what resulted in the canceled context, as opposed to a deadline in [Listing 3-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-6). In this case, the context’s `Err` method should return a `context.Canceled` error 4.

#### Canceling Multiple Dialers

You can pass the same context to multiple `DialContext` calls and cancel all the calls at the same time by executing the context’s `cancel` function. For example, let’s assume you need to retrieve a resource via TCP that is on several servers. You can asynchronously dial each server, passing each dialer the same context. You can then abort the remaining dialers after you receive a response from one of the servers.

In [Listing 3-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-8), you pass the same context to multiple dialers. When you receive the first response, you cancel the context and abort the remaining dialers.

```
package ch03

import (
    "context"
    "net"
    "sync"
    "testing"
    "time"
)

func TestDialContextCancelFanOut(t *testing.T) {
   1  ctx, cancel := context.WithDeadline(
        context.Background(),
        time.Now().Add(10*time.Second),
    )

    listener, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }
    defer listener.Close()

   2  go func() {
            // Only accepting a single connection.
            conn, err := listener.Accept()
            if err == nil {
                conn.Close()
            }
    }()

   3  dial := func(ctx context.Context, address string, response chan int,
        id int, wg *sync.WaitGroup) {
        defer wg.Done()

        var d net.Dialer
        c, err := d.DialContext(ctx, "tcp", address)
        if err != nil {
            return
        }
        c.Close()

        select {
        case <-ctx.Done():
        case response <- id:
        }
    }

    res := make(chan int)
    var wg sync.WaitGroup

   4  for i := 0; i < 10; i++ {
        wg.Add(1)
        go dial(ctx, listener.Addr().String(), res, i+1, &wg)
    }

   5  response := <-res
    cancel()
    wg.Wait()
    close(res)

    if ctx.Err() != 6context.Canceled {
        t.Errorf("expected canceled context; actual: %s",
            ctx.Err(),
        )
    }

    t.Logf("dialer %d retrieved the resource", response)
}
```

Listing 3-8: Canceling all outstanding dialers after receiving the first response (_dial_fanout_test.go_)

You create a context by using `context.WithDeadline`1 because you want to have three potential results when checking the context’s `Err` method: `context.Canceled`, `context.DeadlineExceeded`, or `nil`. You expect `Err` will return the `context.Canceled` error, since your test aborts the dialers with a call to the `cancel` function.

First, you need a listener. This listener accepts a single connection and closes it after the successful handshake 2. Next, you create your dialers. Since you’re spinning up multiple dialers, it makes sense to abstract the dialing code to its own function 3. This anonymous function dials out to the given address by using `DialContext`. If it succeeds, it sends the dialer’s ID across the response channel, provided you haven’t yet canceled the context. You spin up multiple dialers by calling `dial` in separate goroutines using a `for` loop 4. If `dial` blocks on the call to `DialContext` because another dialer won the race, you cancel the context, either by way of the `cancel` function or the deadline, causing the dial function to exit early. You use a wait group to make sure the test doesn’t proceed until all `dial` goroutines terminate after you cancel the context.

Once the goroutines are running, one will win the race and make a successful connection to the listener. You receive the winning dialer’s ID on the `res` channel 5, then abort the losing dialers by canceling the context. At this point, the call to `wg.Wait` blocks until the aborted dialer goroutines return. Finally, you make sure it was your call to `cancel` that caused the cancelation of the context 6. Calling `cancel` does not guarantee that `Err` will return `context.Canceled`. The deadline can cancel the context, at which point calls to `cancel` become a no-op and `Err` will return `context.DeadlineExceeded`. In practice, the distinction may not matter to you, but it’s there if you need it.

### Implementing Deadlines

Go’s network connection objects allow you to include deadlines for both read and write operations. Deadlines allow you to control how long network connections can remain idle, where no packets traverse the connection. You can control the `Read` deadline by using the `SetReadDeadline` method on the connection object, control the `Write` deadline by using the `SetWriteDeadline` method, or both by using the `SetDeadline` method. When a connection reaches its read deadline, all currently blocked and future calls to a network connection’s `Read` method immediately return a time-out error. Likewise, a network connection’s `Write` method returns a time-out error when the connection reaches its write deadline.

Go’s network connections don’t set any deadline for reading and writing operations by default, meaning your network connections may remain idle for a long time. This could prevent you from detecting network failures, like an unplugged cable, in a timely manner, because it’s tougher to detect network issues between two nodes when no traffic is in flight.

The server in [Listing 3-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-9) implements a deadline on its connection object.

```
package ch03

import (
    "io"
    "net"
    "testing"
    "time"
)

func TestDeadline(t *testing.T) {
    sync := make(chan struct{})

    listener, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    go func() {
        conn, err := listener.Accept()
        if err != nil {
            t.Log(err)
            return
        }
        defer func() {
            conn.Close()
            close(sync) // read from sync shouldn't block due to early return
        }()

       1  err = conn.SetDeadline(time.Now().Add(5 * time.Second))
        if err != nil {
            t.Error(err)
            return
        }

        buf := make([]byte, 1)
        _, err = conn.Read(buf) // blocked until remote node sends data
        nErr, ok := err.(net.Error)
        if !ok || 2!nErr.Timeout() {
            t.Errorf("expected timeout error; actual: %v", err)
        }

        sync <- struct{}{}

       3  err = conn.SetDeadline(time.Now().Add(5 * time.Second))
        if err != nil {
            t.Error(err)
            return
        }

        _, err = conn.Read(buf)
        if err != nil {
            t.Error(err)
        }
    }()

    conn, err := net.Dial("tcp", listener.Addr().String())
    if err != nil {
        t.Fatal(err)
    }
    defer conn.Close()

    <-sync
    _, err = conn.Write([]byte("1"))
    if err != nil {
        t.Fatal(err)
    }

    buf := make([]byte, 1)
    _, err = conn.Read(buf) // blocked until remote node sends data
    if err != 4io.EOF {
        t.Errorf("expected server termination; actual: %v", err)
    }
}
```

Listing 3-9: A server-enforced deadline terminates the network connection (_deadline_test.go_).

Once the server accepts the client’s TCP connection, you set the connection’s read deadline 1. Since the client won’t send data, the call to `Read` will block until the connection exceeds the read deadline. After five seconds, `Read` returns an error, which you verify is a time-out 2. Any future reads to the connection object will immediately result in another time-out error. However, you can restore the functionality of the connection object by pushing the deadline forward again 3. After you’ve done this, a second call to `Read` succeeds. The server closes its end of the network connection, which initiates the termination process with the client. The client, currently blocked on its `Read` call, returns `io.EOF`4 when the network connection closes.

We typically use deadlines to provide a window of time during which the remote node can send data over the network connection. When you read data from the remote node, you push the deadline forward. The remote node sends more data, and you push the deadline forward again, and so on. If you don’t hear from the remote node in the allotted time, you can assume that either the remote node is gone and you never received its FIN or that it is idle.

#### Implementing a Heartbeat

For long-running network connections that may experience extended idle periods at the application level, it’s wise to implement a heartbeat between nodes to advance the deadline. This allows you to quickly identify network issues and promptly reestablish a connection as opposed to waiting to detect the network error when your application goes to transmit data. In this way, you can help make sure your application always has a good network connection when it needs it.

For our purposes, a _heartbeat_ is a message sent to the remote side with the intention of eliciting a reply, which we can use to advance the deadline of our network connection. Nodes send these messages at a regular interval, like a heartbeat. Not only is this method portable over various operating systems, but it also makes sure the application using the network connection is responding, since the application implements the heartbeat. Also, this technique tends to play well with firewalls that may block TCP keepalives. We’ll discuss keepalives in Chapter 4.

To start, you’ll need a bit of code you can run in a goroutine to ping at regular intervals. You don’t want to needlessly ping the remote node when you recently received data from it, so you need a way to reset the ping timer. [Listing 3-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-10) is a simple implementation from a file named _ping.go_ that meets those requirements.

I use _ping_ and _pong_ messages in my heartbeat examples, where the reception of a ping message—the challenge—tells the receiver it should reply with a pong message—the response. The challenge and response messages are arbitrary. You could use anything you want to here, provided the remote node knows your intention is to elicit its reply.

```
package ch03

import (
    "context"
    "io"
    "time"
)

const defaultPingInterval = 30 * time.Second

func Pinger(ctx context.Context, w io.Writer, reset <-chan time.Duration) {
    var interval time.Duration
    select {
    case <-ctx.Done():
        return
   1  case interval = <-reset: // pulled initial interval off reset channel
    default:
    }
    if interval <= 0 {
        interval = defaultPingInterval
    }

   2  timer := time.NewTimer(interval)
    defer func() {
        if !timer.Stop() {
            <-timer.C
        }
    }()

    for {
        select {
       3  case <-ctx.Done():
            return
       4  case newInterval := <-reset:
            if !timer.Stop() {
                <-timer.C
            }
            if newInterval > 0 {
                interval = newInterval
            }
       5  case <-timer.C:
            if _, err := w.Write([]byte("ping")); err != nil {
                // track and act on consecutive timeouts here
                return
            }
        }

       6  _ = timer.Reset(interval)
    }
}
```

Listing 3-10: A function that pings a network connection at a regular interval (_ping.go_)

The `Pinger` function writes ping messages to a given writer at regular intervals. Because it’s meant to run in a goroutine, `Pinger` accepts a context as its first argument so you can terminate it and prevent it from leaking. Its remaining arguments include an `io.Writer` interface and a channel to signal a timer reset. You create a buffered channel and put a duration on it to set the timer’s initial interval 1. If the interval isn’t greater than zero, you use the default ping interval.

You initialize the timer to the interval 2 and set up a deferred call to drain the timer’s channel to avoid leaking it, if necessary. The endless `for` loop contains a `select` statement, where you block until one of three things happens: the context is canceled, a signal to reset the timer is received, or the timer expires. If the context is canceled 3, the function returns, and no further pings will be sent. If the code selects the `reset` channel 4, you shouldn’t send a ping, and the timer resets 6 before iterating on the `select` statement again.

If the timer expires 5, you write a ping message to the writer, and the timer resets before the next iteration. If you wanted, you could use this `case` to keep track of any consecutive time-outs that occur while writing to the writer. To do this, you could pass in the context’s `cancel` function and call it here if you reach a threshold of consecutive time-outs.

[Listing 3-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-11) illustrates how to use the `Pinger` function introduced in [Listing 3-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-10) by giving it a writer and running it in a goroutine. You can then read pings from the reader at the expected intervals and reset the ping timer with different intervals.

```
package ch03

import (
    "context"
    "fmt"
    "io"
    "time"
)

func ExamplePinger() {
    ctx, cancel := context.WithCancel(context.Background())
    r, w := io.Pipe() // in lieu of net.Conn
    done := make(chan struct{})
   1  resetTimer := make(chan time.Duration, 1)
    resetTimer <- time.Second // initial ping interval

    go func() {
        Pinger(ctx, w, resetTimer)
        close(done)
    }()

    receivePing := func(d time.Duration, r io.Reader) {
        if d >= 0 {
            fmt.Printf("resetting timer (%s)\n", d)
            resetTimer <- d
        }

        now := time.Now()
        buf := make([]byte, 1024)
        n, err := r.Read(buf)
        if err != nil {
            fmt.Println(err)
        }

        fmt.Printf("received %q (%s)\n",
            buf[:n], time.Since(now).Round(100*time.Millisecond))
    }

   2  for i, v := range []int64{0, 200, 300, 0, -1, -1, -1} {
        fmt.Printf("Run %d:\n", i+1)
        receivePing(time.Duration(v)*time.Millisecond, r)
    }

    cancel()
    <-done // ensures the pinger exits after canceling the context

    // Output:
   3  // Run 1:
    // resetting timer (0s)
    // received "ping" (1s)
   4  // Run 2:
    // resetting timer (200ms)
    // received "ping" (200ms)
   5  // Run 3:
    // resetting timer (300ms)
    // received "ping" (300ms)
   6  // Run 4:
    // resetting timer (0s)
    // received "ping" (300ms)
   7  // Run 5:
    // received "ping" (300ms)
    // Run 6:
    // received "ping" (300ms)
    // Run 7:
    // received "ping" (300ms)

}
```

Listing 3-11: Testing the pinger and resetting its ping timer interval (_ping_example_test.go_)

In this example, you create a buffered channel 1 that you’ll use to signal a reset of the `Pinger`’s timer. You put an initial ping interval of one second on the `resetTimer` channel before passing the channel to the `Pinger` function. You’ll use this duration to initialize the `Pinger`’s timer and dictate when to write the ping message to the writer.

You run through a series of millisecond durations in a loop 2, passing each to the `receivePing` function. This function resets the ping timer to the given duration and then waits to receive the ping message on the given reader. Finally, it prints to stdout the time it takes to receive the ping message. Go checks stdout against the expected output in the example.

During the first iteration 3, you pass in a duration of zero, which tells the `Pinger` to reset its timer by using the previous duration—one second in this example. As expected, the reader receives the ping message after one second. The second iteration 4 resets the ping timer to 200 ms. Once this expires, the reader receives the ping message. The third run resets the ping timer to 300 ms 5, and the ping arrives at the 300 ms mark.

You pass in a zero duration for run 4 6, preserving the 300 ms ping timer from the previous run. I find the technique of using zero durations to mean “use the previous timer duration” useful because I do not need to keep track of the initial ping timer duration. I can simply initialize the timer with the duration I want to use for the remainder of the TCP session and reset the timer by passing in a zero duration every time I need to preempt the transmission of the next ping message. Changing the ping timer duration in the future involves the modification of a single line as opposed to every place I send on the `resetTimer` channel.

Runs 5 to 7 7 simply listen for incoming pings without resetting the ping timer. As expected, the reader receives a ping at 300 ms intervals for the last three runs.

With [Listing 3-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-10) saved to a file named _ping.go_ and [Listing 3-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-11) saved to a file named _ping_example_test.go_, you can run the example by executing the following:

```
$ go test ping.go ping_example_test.go
```

#### Advancing the Deadline by Using the Heartbeat

Each side of a network connection could use a `Pinger` to advance its deadline if the other side becomes idle, whereas the previous examples showed only a single side using a `Pinger`. When either node receives data on the network connection, its ping timer should reset to stop the delivery of an unnecessary ping. [Listing 3-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c03.xhtml#listing3-12) is a new file named _ping_test.go_ that shows how you can use incoming messages to advance the deadline.

```
package ch03

import (
    "context"
    "io"
    "net"
    "testing"
    "time"
)

func TestPingerAdvanceDeadline(t *testing.T) {
    done := make(chan struct{})
    listener, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    begin := time.Now()
    go func() {
        defer func() { close(done) }()
        conn, err := listener.Accept()
        if err != nil {
            t.Log(err)
            return
        }
        ctx, cancel := context.WithCancel(context.Background())
        defer func() {
            cancel()
            conn.Close()
        }()

        resetTimer := make(chan time.Duration, 1)
        resetTimer <- time.Second
        go Pinger(ctx, conn, resetTimer)

        err = conn.SetDeadline(time.Now().Add(15 * time.Second))
        if err != nil {
            t.Error(err)
            return
        }

        buf := make([]byte, 1024)
        for {
            n, err := conn.Read(buf)
            if err != nil {
                return
            }
            t.Logf("[%s] %s", 
                time.Since(begin).Truncate(time.Second), buf[:n])

           2  resetTimer <- 0
            err = 3conn.SetDeadline(time.Now().Add(5 * time.Second))
            if err != nil {
                t.Error(err) 
                return
            }
        }
    }()

    conn, err := net.Dial("tcp", listener.Addr().String())
    if err != nil {
        t.Fatal(err)
    }
    defer conn.Close()

    buf := make([]byte, 1024)
   4  for i := 0; i < 4; i++ { // read up to four pings
        n, err := conn.Read(buf)
        if err != nil {
            t.Fatal(err)
        }
        t.Logf("[%s] %s", time.Since(begin).Truncate(time.Second), buf[:n])
    }
    _, err = 5conn.Write([]byte("PONG!!!")) // should reset the ping timer
    if err != nil {
        t.Fatal(err)
    }
   6  for i := 0; i < 4; i++ { // read up to four more pings
        n, err := conn.Read(buf)
        if err != nil {
            if err != io.EOF {
                t.Fatal(err)
            }
            break
        }
        t.Logf("[%s] %s", time.Since(begin).Truncate(time.Second), buf[:n])
    }
    <-done
    end := time.Since(begin).Truncate(time.Second)
    t.Logf("[%s] done", end)
    if end != 79*time.Second {
        t.Fatalf("expected EOF at 9 seconds; actual %s", end)
    }
}
```

Listing 3-12: Receiving data advances the deadline (_ping_test.go_)

You start a listener that accepts a connection, spins off a `Pinger` set to ping every second, and sets the initial deadline to five seconds 1. From a client’s perspective, it receives four pings followed by an `io.EOF` when the server reaches its deadline and terminates its side of the connection. However, a client can advance the server’s deadline by sending the server data 5 before the server reaches its deadline.

If the server reads data from its connection, it can be confident the network connection is still good. Therefore, it can inform the `Pinger` to reset 2 its timer and push the connection’s deadline forward 3. To preempt the termination of the socket, the client listens for four ping messages 4 from the server before sending an emphatic pong message 5. This should buy the client five more seconds until the server reaches its deadline. The client reads four more pings 6 and then waits for the inevitable. You check that a total of nine seconds 7 has elapsed by the time the server terminates the connection, indicating the client’s pong successfully triggered the reset of the ping timer.

In practice, this method of advancing the ping timer cuts down on the consumption of bandwidth by unnecessary pings. There is rarely a need to challenge the remote side of a network connection if you just received data on the connection.

The strings `"ping"` and `"pong"` are arbitrary. You could use smaller payloads, such as a single byte, for the same purpose, provided both sides of the network connection agree upon what values constitute a ping and a pong.

## What You’ve Learned

We covered a lot of ground in this chapter. We started with a dive into TCP’s handshake, sequences, and acknowledgments, the sliding window, and connection terminations. Then, we covered the process of establishing TCP connections using Go’s standard library. We talked about temporary errors, time-outs, listening for incoming connections, and dialing remote services. Finally, we covered techniques to help you detect and timely correct network integrity issues.

I strongly recommend picking up _Practical Packet Analysis_ by Chris Sanders (No Starch Press, 2017) and installing Wireshark. Manipulating your network code and seeing how it affects TCP traffic in Wireshark is a fantastic way to gain a deeper understanding of both TCP and Go’s networking packages. The next chapter covers sending and receiving data over TCP connections. Wireshark will help you gain a deeper understanding of data you send, including each payload’s effects on the sliding window. Familiarizing yourself with it now will pay dividends.