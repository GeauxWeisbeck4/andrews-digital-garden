---
id: 01JC4FZV144VXFSZWK4T7C7FCQ
modified: 2024-11-12T17:53:14-05:00
title: Chapter 5 - Unreliable UDP Communication
tags:
  - go-lang
  - networking
  - programming
  - books
  - no-starch-press
---
# 5  
UNRELIABLE UDP COMMUNICATION

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

Although most networking applications take advantage of TCP’s reliability and flow control, the less popular User Datagram Protocol (UDP) is nonetheless an important part of the TCP/IP stack. UDP is a simple protocol with minimal features. Some applications do not require TCP’s feature set and session overhead. Those applications, like domain name resolution services, opt to use UDP instead.

This chapter starts by comparing UDP to TCP, focusing on scenarios where UDP may be a better choice over TCP. Then, you’ll learn how to send and receive UDP packets in Go. Finally, you’ll learn why it’s best to limit the size of UDP packets you send across a network and how to determine an optimal packet size.

## Using UDP: Simple and Unreliable

UDP is unreliable because it does not include many of the mechanisms that make TCP so trustworthy. It provides little more than a socket address (an IP address and port). In fact, the protocol is so simple that RFC 768 describes the entire thing in about three pages. Unlike TCP, UDP does not provide session support or even confirm that the destination is accessible; it simply makes a best-effort attempt to deliver the packet. Recipients do not automatically acknowledge UDP packets, so UDP has no inherent delivery confirmation. UDP does not manage congestion, control data flow, or retransmit packets. Lastly, UDP does not guarantee that the destination receives packets in the order they originate. UDP is simply a conduit between applications and the IP layer. This simplicity is what makes UDP fast and attractive for some applications.

UDP has a few strengths over TCP. Whereas TCP must establish a session with each individual node in a group before it can transmit data, UDP can send a single packet to a group of nodes without duplicating the packet, a process known as _multicasting_. UDP is also capable of broadcasting packets to all members of a subnet since it doesn’t need to establish a session between each node.

UDP is ideal when missing packets aren’t detrimental to overall communication because the most recently received packets can take the place of earlier, dropped packets. Weather data is a good example of this. If you’re tracking a tornado in your area by streaming weather data to your phone, you aren’t as concerned about dropped packets indicating the tornado’s location two minutes ago if you’ve received packets giving you the tornado’s current location.

You should consider using UDP in your application if it doesn’t require all the features TCP provides. For most network applications, TCP is the right protocol choice. But UDP is an option if its speed and simplicity better fit your use case and the reliability trade-offs aren’t detrimental.

UDP’s packet structure consists of an 8-byte header and a payload. The header contains 2 bytes for the source port, 2 bytes for the destination port, 2 bytes for the packet length in bytes, and a 2-byte checksum. The minimum packet length is 8 bytes to account for the header and an empty payload. [Figure 5-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#figure5-1) illustrates the organization of a UDP packet.

![f05001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c05/f05001.png)

Figure 5-1: UDP packet header and payload

Although the maximum packet length is 65,535 bytes, application layer protocols often limit the packet length to avoid fragmentation, which we discuss in “Avoiding Fragmentation” on page 115.

## Sending and Receiving UDP Data

When it comes to sending and receiving data, UDP is uncivilized compared to TCP. For example, let’s assume your neighbor baked you a pie and wants to give it to you. Using TCP to communicate is like your neighbor shouting a greeting from her window (her socket address) to your open window (your socket address). You hear her greeting and return a greeting of your own (the TCP handshake). Your neighbor then delivers your pie. You accept it and thankfully acknowledge the pie (the data transfer). You then both exchange farewells and go about your business (the termination). By contrast, using UDP to communicate is like your neighbor abruptly throwing the pie at your window, whether it’s open or not, and awaiting no confirmation that you received it.

The section “Using the net.Conn Interface” on page 74 introduced the `net.Conn` interface for handling stream-oriented connections, such as TCP, between a client and a server. But this interface isn’t ideal for UDP connections because UDP is not a stream-oriented protocol. UDP does not maintain a session or involve a handshake process like TCP. UDP does not have the concept of acknowledgments, retransmissions, or flow control.

Instead, UDP primarily relies on the packet-oriented `net.PacketConn` interface. We’ll discuss a use case for `net.Conn` with UDP later in this chapter, but `net.PacketConn` is the better choice for most UDP applications.

### Using a UDP Echo Server

Sending and receiving UDP packets is a nearly identical process to sending and receiving TCP packets. But since UDP doesn’t have session support, you must be able to handle an additional return value, the sender’s address, when reading data from the connection object, as shown in [Listing 5-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-1)’s UDP echo server implementation.

```
package echo

import (
    "context"
    "net"
)

func echoServerUDP(1ctx context.Context, addr string) (net.Addr, error) {
    s, err := 2net.ListenPacket("udp", addr)
    if err != nil {
        return nil, fmt.Errorf("binding to udp %s: %w", addr, err)
    }

    3 go func() {
        go func() {
            4 <-ctx.Done()
            _ = s.Close()
        }()

        buf := make([]byte, 1024)
        for {
            n, 5clientAddr, err := 6s.ReadFrom(buf) // client to server
            if err != nil {
                return
            }

            _, err = 7s.WriteTo(buf[:n], 8clientAddr) // server to client
            if err != nil {
                return
            }
        }
    }()

    return s.LocalAddr(), nil
}
```

Listing 5-1: A simple UDP echo server implementation (_echo.go_)

This code allows you to spin up a UDP server that will echo any UDP packets it receives to the sender. You’ll make use of this code quite a bit in this chapter, so it behooves you to understand what’s happening here.

The function receives a context 1 to allow cancellation of the echo server by the caller and a string address in the familiar _host:port_ format. It returns a `net.Addr` interface and an `error` interface. The caller uses the `net.Addr` interface to address messages to the echo server. The returned `error` interface is not `nil` if anything goes wrong while instantiating the echo server.

You create a UDP connection for your server with a call to `net.ListenPacket`2, which returns a `net.PacketConn` interface and an `error` interface. The `net.ListenPacket` function is analogous to the `net.Listen` function you used to create a TCP listener in Chapters 3 and 4, except `net.ListenPacket` exclusively returns a `net.PacketConn` interface.

A goroutine manages the asynchronous echoing of messages by your echo server 3. A second goroutine blocks on the context’s `Done` channel 4. Once the caller cancels the context, receiving on the `Done` channel unblocks and the server is closed, tearing down both this goroutine and the parent goroutine 3.

To read from the UDP connection, you pass a byte slice to the `ReadFrom` method 6. This returns the number of bytes read, the sender’s address, and an error interface. Notice there is no `Accept` method on your UDP connection as there is with the TCP-based listeners in the previous chapters. This is because UDP doesn’t use a handshake process. Here, you simply create a UDP connection listening to a UDP port and read any incoming messages. Since you don’t have the luxury of a proper introduction and an established session, you rely on the returned address 5 to determine which node sent you the message.

To write a UDP packet, you pass a byte slice and a destination address 8 to the connection’s `WriteTo` method 7. The `WriteTo` method returns the number of bytes written and an error interface. Just as with reading data, you need to tell the `WriteTo` method where to send the packet, because you do not have an established session with a remote node. In [Listing 5-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-1), you write the message to the original sender. But you could just as easily forward the message onto another node using your existing UDP connection object. You do not have to establish a new UDP connection object to forward on the message as you would if you were using TCP.

### Receiving Data from the Echo Server

Now that you are familiar with the UDP-based echo server, let’s have a look at some client code that interacts with the echo server. [Listing 5-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-2) shows a simple interaction with the echo server.

```
package echo

import (
    "bytes"
    "context"
    "net"
    "testing"
)

func TestEchoServerUDP(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    1 serverAddr, err := echoServerUDP(ctx, "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }
    defer cancel()

    2 client, err := net.ListenPacket("udp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = client.Close() }()

    msg := []byte("ping")
    _, err = 3client.WriteTo(msg, serverAddr)
    if err != nil {
        t.Fatal(err)
    }

    buf := make([]byte, 1024)
    n, 4addr, err := 5client.ReadFrom(buf)
    if err != nil {
        t.Fatal(err)
    }

    if addr.String() != serverAddr.String() {
        t.Fatalf("received reply from %q instead of %q", addr, serverAddr)
    }

    if !bytes.Equal(msg, buf[:n]) {
        t.Errorf("expected reply %q; actual reply %q", msg, buf[:n])
    }
}
```

Listing 5-2: Sending UDP packets to the echo server and receiving replies (_echo_test.go_)

You pass along a context and the address string to the `echoServer` function and receive the server’s address 1 object. You defer a call to the context’s `cancel` function, which signals the server to exit and close its goroutines. In a real-world application, using a context for cancellation of long-running processes is useful to make sure you aren’t leaking resources like memory or unnecessarily keeping files open.

You instantiate the client’s `net.PacketConn`2 in the same way that you instantiated the echo server’s `net.PacketConn`. The `net.ListenPacket` function creates the connection object for both the client and the server. Here, too, you need to tell the client where to send its message with each invocation of its `WriteTo` method 3. After sending the message to the echo server, the client should immediately receive a message via its `ReadFrom` method 5. You can examine the address 4 returned by the `ReadFrom` method to confirm that the echo server sent the message.

It’s important to note that the test in [Listing 5-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-2) can fail under certain circumstances. Even though you’re reading packets from and writing packets to a computer’s local network stack, the packets are still subject to all of the conditions that make UDP unreliable over inter-node networks. For example, full send or receive buffers, or the lack of available RAM, can result in dropped packets; large UDP packets may be subject to fragmentation (discussed later in this chapter); and operating systems using multiple threads to deliver UDP packets may deliver the packets out of order.

### Every UDP Connection Is a Listener

Recall from Chapter 3 that Go’s `net` package distinguishes between a TCP connection object (`TCPConn`) and a TCP listener (`TCPListener`). The TCP listener is what accepts the connection and returns an object that represents the listener’s side of the connection so that the listener can then send a message to the client.

There is no UDP equivalent of the `TCPListener` because UDP lacks sessions. This means your code has a bit more accounting to do when it receives packets. You need to verify the sender’s address, because you can no longer trust that all incoming packets to a connection object are from the same sender.

The next few listings are part of a test that a single UDP connection object can receive packets from more than one sender. [Listing 5-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-3) spins up an echo server and a client for the test.

```
package echo

import (
    "bytes"
    "context"
    "net"
    "testing"
)

func TestListenPacketUDP(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    1 serverAddr, err := echoServerUDP(ctx, "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }
    defer cancel()

    2 client, err := net.ListenPacket("udp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = client.Close() }()
```

Listing 5-3: Creating an echo server and client (_listen_packet_test.go_)

You start by creating the echo server 1 and client connection 2. [Listing 5-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-4) adds a second network connection to interact with the client.

```
--snip--

    1 interloper, err := net.ListenPacket("udp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    interrupt := []byte("pardon me")
    2 n, err := interloper.WriteTo(interrupt, client.LocalAddr())
    if err != nil {
        t.Fatal(err)
    }
    _ = interloper.Close()

    if l := len(interrupt); l != n {
        t.Fatalf("wrote %d bytes of %d", n, l)
    }
```

Listing 5-4: Adding an interloper and interrupting the client with a message (_listen_packet_test.go_)

You then create a new UDP connection 1 meant to interlope on the client and echo server and interrupt the client 2. This message should queue up in the client’s receive buffer.

The client sends its ping message to the echo server and reconciles the replies in [Listing 5-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-5).

```
--snip--

    ping := []byte("ping")
    _, err = 1client.WriteTo(ping, serverAddr)
    if err != nil {
        t.Fatal(err)
    }

    buf := make([]byte, 1024)
    n, addr, err := 2client.ReadFrom(buf)
    if err != nil {
        t.Fatal(err)
    }

    if !bytes.Equal(3interrupt, buf[:n]) {
        t.Errorf("expected reply %q; actual reply %q", interrupt, buf[:n])
    }

    if addr.String() != interloper.LocalAddr().String() {
        t.Errorf("expected message from %q; actual sender is %q",
            interloper.LocalAddr(), addr)
    }

    n, addr, err = client.ReadFrom(buf)
    if err != nil {
        t.Fatal(err)
    }

    if !bytes.Equal(4ping, buf[:n]) {
        t.Errorf("expected reply %q; actual reply %q", ping, buf[:n])
    }

    5 if addr.String() != serverAddr.String() {
        t.Errorf("expected message from %q; actual sender is %q",
            serverAddr, addr)
    }
}
```

Listing 5-5: Receiving UDP packets from multiple senders at once (_listen_packet_test.go_)

Meanwhile, the client writes a ping message to the echo server 1 and promptly reads an incoming message 2. What’s unique about the UDP client connection is it first reads the interruption message from the interloping connection 3 and then the reply from the echo server 4. Were this a TCP connection, the client would have never received the interloper’s message. As such, your code should always verify the sender of each packet it reads by evaluating the second return value 5 from the `ReadFrom` method, the sender’s address.

### Using net.Conn in UDP

You can establish a UDP connection that implements the `net.Conn` interface so that your code behaves indistinguishably from a TCP `net.Conn`. You do so by passing `udp` as the first argument to the `net.Dial` function used in the preceding two chapters. Using `net.Conn` with your UDP-based connections can prevent interlopers from sending you messages and eliminate the need to check the sender’s address on every reply you receive.

[Listing 5-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-6) creates the UDP-based `net.Conn` and demonstrates how `net.Conn` encapsulates the implementation details of UDP to emulate a stream-oriented network connection.

```
package echo

import (
    "bytes"
    "context"
    "net"
    "testing"
    "time"
)

func TestDialUDP(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    1 serverAddr, err := echoServerUDP(ctx, "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }
    defer cancel()

    client, err := 2net.Dial("udp", serverAddr.String())
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = client.Close() }()
```

Listing 5-6: Creating an echo server and client (_dial_test.go_)

The client side of a connection can leverage the stream-oriented functionality of `net.Conn` over UDP, but the UDP listener must still use `net.PacketConn`. You spawn an instance of the echo server 1 for the purpose of sending a reply to the client. You then dial the echo server over UDP by passing `udp` as the first argument to `net.Dial`2. Unlike TCP, the echo server receives no traffic upon calling `net.Dial` because no handshake is necessary.

[Listing 5-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-7) interrupts the client by sending a message to it before the echo server sends its reply.

```
--snip--

    interloper, err := net.ListenPacket("udp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    interrupt := []byte("pardon me")
    1 n, err := interloper.WriteTo(interrupt, client.LocalAddr())
    if err != nil {
        t.Fatal(err)
    }
    _ = interloper.Close()

    if l := len(interrupt); l != n {
        t.Fatalf("wrote %d bytes of %d", n, l)
    }
```

Listing 5-7: Interrupting the client (_dial_test.go_)

Just as in [Listing 5-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-4), you send a message to the client from an interloping connection 1.

[Listing 5-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-8) details the difference between a UDP connection using `net.Conn` and one using `net.PacketConn`, as in [Listing 5-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-5).

```
--snip--

    ping := []byte("ping")
    _, err = 1client.Write(ping)
    if err != nil {
        t.Fatal(err)
    }

    buf := make([]byte, 1024)
    n, err = 2client.Read(buf)
    if err != nil {
        t.Fatal(err)
    }

    if !bytes.Equal(ping, buf[:n]) {
        t.Errorf("expected reply %q; actual reply %q", ping, buf[:n])
    }

    err = 3client.SetDeadline(time.Now().Add(time.Second))
    if err != nil {
        t.Fatal(err)
    }

    _, err = 4client.Read(buf)
    if err == nil {
        t.Fatal("unexpected packet")
    }
}
```

Listing 5-8: Using `net.Conn` to manage UDP traffic (_dial_test.go_)

The client sends a ping message to the echo server by using `net.Conn`’s `Write` method 1. The `net.Conn` client will write its messages to the address specified in the `net.Dial` call. You do not need to specify a destination address for every packet you send using the client connection. Likewise, you read packets using the client’s `Read` method 2. The client reads packets only from the sender address specified in the `net.Dial` call, as you would expect using a stream-oriented connection object. The client never reads the message sent by the interloping connection. To make sure, you set an ample deadline 3 and attempt to read another message 4.

For your purposes, using `net.Conn` over `net.PacketConn` may make your UDP connection code cleaner. Just be aware of the trade-offs. Using `net.Conn` with UDP does not offer the same functionality as you would expect when using `net.Conn` with TCP. For example, a UDP-based `net.Conn`’s `Write` method will not return an error if the destination failed to receive the packet. The onus is still on your application code to confirm delivery when using UDP.

## Avoiding Fragmentation

_Fragmentation_ is a Layer 3 IP process of splitting a packet into smaller pieces suitable for efficient transmission over a network. All network media have packet size limitations known as the _maximum transmission unit (MTU)_. Packets larger than the medium’s maximum transmission unit require fragmentation so that each fragment is less than or equal to the medium’s MTU before nodes pass them over the medium. Once the fragments reach their destination, the operating system reassembles each packet and presents the packet to your application.

But fragments can corrupt or fail to reach their destination for one reason or another. This is a significant consideration if you’re using UDP because, unlike TCP, UDP does not gracefully recover from missing or corrupt data. If an operating system fails to receive even a single fragment, the sender must retransmit the entire UDP packet. As you can imagine, retransmitting large packets is woefully inefficient. Although there are numerous approaches to mitigating the effects of fragmentation, we’ll attempt to avoid it altogether. We’ll focus on a straightforward way to identify the MTU between your computer and a destination node, and then use those results to inform your choice of payload size to avoid fragmentation.

You can use the `ping` command to determine the MTU between your computer and a destination node. The `ping` command allows you to send an ICMP packet of a specific size with a flag set to inform nodes not to fragment it. If the packet reaches a node that needs to fragment the packet because of its size, the node will see the _do not fragment_ flag and respond with an ICMP message informing you that the packet is too large.

The following example sends these pings over Ethernet, which has a minimum MTU of 46 bytes and a maximum MTU of 1,500 bytes, per its specification. If any hop between your computer and its destination has an MTU of less than 1,500 bytes, your packet will fragment. Let’s confirm that with the `ping` command on Linux ([Listing 5-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-9)).

```
$ ping -M 1do -s 21500 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 1500(31528) bytes of data.
ping: sendmsg: 4Message too long
```

Listing 5-9: Pinging 1.1.1.1 with a payload size of 1,500 bytes on Linux

You set the `-M` flag 1 to `do`, which sets the prohibit fragmentation option, and set the `-s` flag 2 to `1500`, which sets a payload of 1,500 bytes. Since you aren’t accounting for the packet’s header size, this should exceed the Ethernet MTU. As expected, you get a notification that the packet needs fragmentation 4. You also see that the total packet size is 1,528 bytes 3. The extra 28 bytes is the sum of the 8-byte ICMP header and the 20-byte IP header. Any payload you specify should account for the overall header size as well.

As you can see, you never received a reply from 1.1.1.1 in [Listing 5-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-9) because the packet you sent was too big to traverse each hop without requiring fragmentation. Instead, the `ping` command informed you that your message was too long.

Let’s try again and subtract 28 bytes from the payload ([Listing 5-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c05.xhtml#listing5-10)).

```
$ ping -M do -s 1472 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 1472(1500) bytes of data.
1480 bytes from 1.1.1.1: icmp_seq=1 ttl=59 time=11.8 ms
```

Listing 5-10: Pinging 1.1.1.1 with a payload size of 1472 bytes on Linux

That’s more like it. You confirmed that the MTU between this computer and 1.1.1.1 over the internet is set to 1,500 bytes. This is the maximum packet size you can send on the network before it will require fragmentation. Thankfully, the UDP header is also 8 bytes, so the `ping` command gives accurate results despite using ICMP. Accounting for the headers, your maximum UDP payload size is 1,472 bytes to avoid fragmentation.

The equivalent `ping` command on Windows is the following:

```
C:\>ping -f -l 1500 1.1.1.1
```

The `-f` flag instructs nodes to not fragment the packet, and the `-l` flag sets the packet size to the given integer in bytes.

On macOS, the `ping` command looks like this:

```
$ ping -D -s 1500 1.1.1.1
```

The `-D` flag sets the no fragmentation flag, and the `-s` flag specifies the payload size.

Keep in mind that the MTU from your computer may differ from the examples in this chapter because of MTU settings either on your network or between your computer and the destination you choose to ping. I recommend you experiment with the `ping` command to determine the MTU from your computer to various hosts on the internet and see if you find any differences.

## What You’ve Learned

UDP is a minimalistic, datagram-based protocol that favors speed over reliability by eschewing many of TCP’s flow control and reliability features. UDP is ideal when speed and simplicity are required and the potential for data loss is acceptable, such as with live video streaming.

Since UDP is not session based, there is no concept of a UDP listener that accepts a connection after establishing a session. Rather, you create a network connection by using `net.ListenPacket`, which returns a `net.PacketConn` interface. Your code can then read any incoming messages, or datagrams, from the `net.PacketConn` interface since every `net.PacketConn` listens for incoming messages.

Fragmentation is a serious consideration when using UDP. It’s important to avoid fragmentation of your UDP packets whenever possible to help ensure delivery. The `ping` command can help you derive the appropriate maximum transmission unit between your computer and your destination network. Since ICMP packet headers used by the `ping` command are the same size as UDP headers, you can use that knowledge to easily determine the payload size threshold at which fragmentation will occur. Aside from managing fragmentation by appropriately sizing your payloads, your code must manage acknowledgments and retransmissions to ensure reliability.