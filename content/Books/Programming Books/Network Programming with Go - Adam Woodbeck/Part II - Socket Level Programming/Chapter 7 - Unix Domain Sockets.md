---
id: 01JC4FZV144VXFSZWK4T7C7FCQ
modified: 2024-11-12T17:54:39-05:00
title: Chapter 7 - Unix Domain Sockets
tags:
  - go-lang
  - networking
  - programming
  - books
  - no-starch-press
---
# 7  
UNIX DOMAIN SOCKETS

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

So far in this book, we’ve discussed communications between nodes on a network. But not all network programming occurs exclusively between separate nodes. Your applications may sometimes need to communicate with services, such as a database, hosted on the same node.

One way to connect your application to a database running on the same system would be to send data to the node’s IP address or localhost address—commonly 127.0.0.1—and the database’s port number. However, there’s another way: using Unix domain sockets. The _Unix domain socket_ is a communication method that uses the filesystem to determine a packet’s destination address, allowing services running on the same node to exchange data with one another, a process known as _inter-process communication (IPC)_.

This chapter first defines exactly what Unix domain sockets are and how you can control read and write access to them. Next, you’ll explore the three types of Unix domain sockets available through Go’s `net` package and write a simple echo server in each of them. Finally, you’ll write a service that uses Unix domain sockets to authenticate clients based on their user and group ID information.

## NOTE

Not all operating systems support the three types of Unix domain sockets. This chapter uses build constraints and special filename suffixes to identify the target platforms for each code listing.

## What Are Unix Domain Sockets?

In Chapter 2, I defined a network socket as an IP address and port number. Socket addressing allows individual services on the same node, at the same IP address, to listen for incoming traffic. To illustrate the importance of socket addressing, just imagine how inefficient having a single phone line at a large corporation would be. If you wanted to speak to someone, you’d best hope the phone wasn’t already in use. That’s why, to alleviate the congestion, most corporations assign an extension number to each employee. This allows you to contact the person you want to speak to by dialing the company’s phone number (which is like the node’s IP address) followed by the employee’s extension (which is like the port number). Just as phone numbers and extensions allow you to individually call every single person at a corporation, the IP address and port number of a socket address allow you to communicate with every single service listening to each socket address on a node.

_Unix domain sockets_ apply the socket-addressing principle to the filesystem: each Unix domain socket has an associated file on the filesystem, which corresponds to a network socket’s IP address and port number. You can communicate with a service listening to the socket by reading from and writing to this file. Likewise, you can leverage the filesystem’s ownership and permissions to control read and write access to the socket. Unix domain sockets increase efficiency by bypassing the operating system’s network stack, eliminating the overhead of traffic routing. For the same reasons, you won’t need to worry about fragmentation or packet ordering when using Unix domain sockets. If you choose to forgo Unix domain sockets and exclusively use network sockets when communicating with local services (for example, to connect your application to a local database, a memory cache, and so on), you ignore significant security advantages and performance gains.

Though this system brings distinct advantages, it comes with a caveat: Unix domain sockets are local to the node using them, so you cannot use them to communicate with other nodes, as you can with network sockets. Therefore, Unix domain sockets may not be a good fit if you anticipate moving a service to another node or require maximum portability for your application. To maintain communication, you’d have to first migrate to a network socket.

## Binding to Unix Domain Socket Files

A Unix domain socket file is created when your code attempts to bind to an unused Unix domain socket address by using the `net.Listen`, `net.ListenUnix`, or `net.ListenPacket` functions. If the socket file for that address already exists, the operating system will return an error indicating that the address is in use. In most cases, simply removing the existing Unix domain socket file is enough to clear up the error. However, you should first make sure that the socket file exists not because a process is currently using that address but because you didn’t properly clean up the file from a defunct process.

If you wish to reuse a socket file, use the `net` package’s `FileListener` function to bind to an existing socket file. This function is beyond the scope of this book, but I encourage you to read its documentation.

### Changing a Socket File’s Ownership and Permissions

Once a service binds to the socket file, you can use Go’s `os` package to modify the file’s ownership and read/write permissions. Specifically, the `os.Chown` function allows you to modify the user and group that owns the file. Windows does not support this function, though this function is supported on Windows Subsystem for Linux (WSL), Linux, and macOS, among others outside the scope of this book. We’ll look at the lines of code that change file ownership and permissions now but cover them in context later in this chapter.

The following line instructs the operating system to update the user and group ownership of the given file:

```
err := os.Chown("/path/to/socket/file", 1-1, 2100)
```

The `os.Chown` function accepts three arguments: the path to a file, the user ID of the owner 1, and the group ID of the owner 2. A user or group ID of `-1` tells Go you want to maintain the current user or group ID. In this example, you want to maintain the socket file’s current user ID but set its group ID to 100, which here is assumed to be a valid group ID in the _/etc/group_ file.

Go’s `os/user` package includes functions to help you translate between user and group names and IDs. For example, this line of code uses the `LookupGroup` function to find the group ID for the _users_ group:

```
grp, err := user.LookupGroup("users")
```

Provided `user.LookupGroup` did not return an error, the `grp` variable’s `Gid` field contains the group ID for the _users_ group.

The `os.Chmod` function changes the file’s mode and the numeric notation of Unix-compatible permission bits. These bits inform the operating system of the file’s mode, the file’s user read/write/execute permissions, the file’s group read/write/execute permissions, and the read/write/execute permissions for any user not in the file’s group:

```
err := os.Chmod("/path/to/socket/file", os.ModeSocket|0660)
```

The `os.Chmod` function accepts a file path and an `os.FileMode`, which represents the file mode, the user permissions, the group permissions, and non-group user permissions. Since you’re dealing with a socket file, you should always set the `os.ModeSocket` mode on the file. You do that using a bitwise OR between the `os.ModeSocket` and the numeric file permission notation. Here, you’re passing the octal `0660`, which gives the user and group read and write access but prevents anyone outside the group from reading or writing to the socket. You can read more about `os.FileMode` in Go’s documentation at [https://golang.org/pkg/os/#FileMode](https://golang.org/pkg/os/#FileMode) and familiarize yourself with filesystem permissions numeric notation at [https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation](https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation).

### Understanding Unix Domain Socket Types

There are three types of Unix domain sockets: _streaming sockets_, which operate like TCP; _datagram sockets_, which operate like UDP; and _sequence packet sockets_, which combine elements of both. Go designates these types as `unix`, `unixgram`, and `unixpacket`, respectively. In this section, we’ll write echo servers that work with each of these types.

The `net.Conn` interface allows you to write code once and use it across multiple network types. It abstracts many of the differences between the network sockets used by TCP and UDP and Unix domain sockets, which means that you can take code written for communication over TCP, for example, and use it over a Unix domain socket by simply changing the address and network type.

#### The unix Streaming Socket

The streaming Unix domain socket works like TCP without the overhead associated with TCP’s acknowledgments, checksums, flow control, and so on. The operating system is responsible for implementing the streaming inter-process communication over Unix domain sockets in lieu of TCP.

To illustrate this type of Unix domain socket, let’s write a function that creates a generic stream-based echo server ([Listing 7-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-1)). You’ll be able to use this function with any streaming network type. That means you can use it to create a TCP connection to a different node, but you’ll also be able to use it with the `unix` type to communicate with a Unix socket address.

## NOTE

Linux, macOS, and Windows all support the unix network type.

```
package echo

import (
    "context"
    "net"
)

func 1streamingEchoServer(ctx context.Context, network string,
    addr string) (net.Addr, error) {
    s, err := 2net.Listen(network, addr)
    if err != nil {
        return nil, err
    }
```

Listing 7-1: Creating the streaming echo server function (_echo.go_)

The `streamingEchoServer` function 1 accepts a string representing a stream-based network and a string representing an address and returns an address object and an `error` interface. You should recognize these arguments and return types from earlier in the book.

Since you’ve made the echo server a bit more generic by accepting a context, a network string, and an address string, you can pass it any stream-based network type, such as `tcp`, `unix`, or `unixpacket`. The address would then need to correspond to the network type. The context is used for signaling the server to close. If the network type is `tcp`, the address string must be an IP address and port combination, such as 127.0.0.1:80. If the network type is `unix` or `unixpacket`, the address must be the path to a nonexistent file. The socket file will be created when the echo server binds to it 2. Then the server will start listening for incoming connections.

[Listing 7-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-2) completes the streaming echo server implementation.

```
--snip--

    go func() {
        go func() {
          1<-ctx.Done()
            _ = s.Close()
        }()

        for {
            conn, err := 2s.Accept()
            if err != nil {
                return
            }

            go func() {
                defer func() { _ = conn.Close() }()

                for {
                    buf := make([]byte, 1024)
                    n, err := 3conn.Read(buf)
                    if err != nil {
                        return
                    }

                    _, err = 4conn.Write(buf[:n])
                    if err != nil {
                        return
                    }
                }
            }()
        }
    }()

    return s.Addr(), nil
}
```

Listing 7-2: A stream-based echo server (_echo.go_)

A listener created with either `net.Listen` or `net.ListenUnix` will automatically remove the socket file when the listener exits. You can modify this behavior with `net.UnixListener`’s `SetUnlinkOnClose` method, though the default is ideal for most use cases. Unix domain socket files created with `net.ListenPacket` won’t be automatically removed when the listener exits, as you’ll see a little later in this chapter.

As before, you spin off the echo server in its own goroutine so it can asynchronously accept connections. Once the server accepts a connection 2, you start a goroutine to echo incoming messages. Since you’re using the `net.Conn` interface, you can use its `Read`3 and `Write`4 methods to communicate with the client no matter whether the server is communicating over a network socket or a Unix domain socket. Once the caller cancels the context 1, the server closes.

[Listing 7-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-3) tests the streaming echo server over a Unix domain socket using the `unix` network type.

```
package echo

import (
    "bytes"
    "context"
    "fmt"
    "io/ioutil"
    "net"
    "os"
    "path/filepath"
    "testing"
)

func TestEchoServerUnix(t *testing.T) {
    dir, err := 1ioutil.TempDir("", "echo_unix")
    if err != nil {
        t.Fatal(err)
    }
    defer func() {
        if rErr := 2os.RemoveAll(dir); rErr != nil {
            t.Error(rErr)
        }
    }()

    ctx, cancel := context.WithCancel(context.Background())
    3 socket := filepath.Join(dir, fmt.Sprintf("%d.sock", os.Getpid()))
    rAddr, err := streamingEchoServer(ctx, "unix", socket)
    if err != nil {
        t.Fatal(err)
    }

    err = 4os.Chmod(socket, os.ModeSocket|0666)
    if err != nil {
        t.Fatal(err)
    }
```

Listing 7-3: Setting up an echo server test over a unix domain socket (_echo_test.go_)

You create a subdirectory in your operating system’s temporary directory named _echo_unix_1 that will contain the echo server’s socket file. The deferred call to `os.RemoveAll` cleans up after the server 2 by removing your temporary subdirectory when the test completes. You pass a socket file named _#.sock_3, where _#_ is the server’s process ID, saved in the temporary subdirectory (_/tmp/echo_unix/123.sock_) to the `streamingEchoServer` function. Finally, you make sure everyone has read and write access to the socket 4.

[Listing 7-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-4) makes a connection to the streaming echo server and sends a test.

```
--snip--

    conn, err := net.Dial("unix", 1rAddr.String())
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = conn.Close() }()

    msg := []byte("ping")
    2 for i := 0; i < 3; i++ { // write 3 "ping" messages
        _, err = conn.Write(msg)
        if err != nil {
            t.Fatal(err)
        }
    }

    buf := make([]byte, 1024)
    n, err := 3conn.Read(buf) // read once from the server
    if err != nil {
        t.Fatal(err)
    }

    expected := 4bytes.Repeat(msg, 3)
    if !bytes.Equal(expected, buf[:n]) {
        t.Fatalf("expected reply %q; actual reply %q", expected,
            buf[:n])
    }

    _ = closer.Close()
    <-done
}
```

Listing 7-4: Streaming data over a Unix domain socket (_echo_test.go_)

You dial the server by using the familiar `net.Dial` function. It accepts the `unix` network type and the server’s address, which is the full path to the Unix domain socket file 1.

You write three ping messages to the echo server before reading the first response 2. The reasoning for sending three separate pings will be clear when you explore the `unixpacket` type later in this chapter. When you read the first response 3 with a buffer large enough to store the three messages you just sent, you receive all three ping messages 4 in a single read as the string `pingpingping`. Remember, a stream-based connection does not delineate messages. The onus is on you to determine where one message stops and another one starts when you read a series of bytes from the server.

#### The unixgram Datagram Socket

Next let’s create an echo server that will communicate using datagram-based network types, such as `udp` and `unixgram`. Whether you’re communicating over UDP or a `unixgram` socket, the server you’ll write looks essentially the same. The difference is, you will need to clean up the socket file with a `unixgram` listener, as you’ll see in [Listing 7-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-5).

## NOTE

Windows and Windows Subsystem for Linux do not support unixgram domain sockets.

```
--snip--

func datagramEchoServer(ctx context.Context, network string,
    addr string) (net.Addr, error) {
    s, err := 1net.ListenPacket(network, addr)
    if err != nil {
        return nil, err
    }

    go func() {
        go func() {
            <-ctx.Done()
            _ = s.Close()
            if network == "unixgram" {
                _ = 2os.Remove(addr)
            }
        }()

        buf := make([]byte, 1024)
        for {
            n, clientAddr, err := s.ReadFrom(buf)
            if err != nil {
                return
            }

            _, err = s.WriteTo(buf[:n], clientAddr)
            if err != nil {
                return
            }
        }
    }()

    return s.LocalAddr(), nil
}
```

Listing 7-5: A datagram-based echo server (_echo.go_)

You call `net.ListenPacket`1, which returns a `net.PacketConn`. As mentioned earlier in this chapter, since you don’t use `net.Listen` or `net.ListenUnix` to create the listener, Go won’t clean up the socket file for you when your server is done with it. You must make sure you remove the socket file yourself, 2 or subsequent attempts to bind to the existing socket file will fail.

Since the `unixgram` network type doesn’t work on Windows, [Listing 7-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-6) uses a build constraint to make sure this code does not run on Windows and then imports the necessary packages.

```
// +build darwin linux

package echo

import (
    "bytes"
    "context"
    "fmt"
    "io/ioutil"
    "net"
    "os"
    "path/filepath"
    "testing"
)
```

Listing 7-6: Building constraints and imports for macOS and Linux (_echo_posix_test.go_)

The build constraint tells Go to include this code only if it’s running on a macOS or Linux operating system. Granted, Go supports other operating systems, many of which may offer `unixgram` support, that are outside the scope of this book. This build constraint does not take those other operating systems into account, and I encourage you to test this code on your target operating system.

With the build constraint in place, you can add the test in [Listing 7-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-7).

```
--snip--

func TestEchoServerUnixDatagram(t *testing.T) {
    dir, err := ioutil.TempDir("", "echo_unixgram")
    if err != nil {
        t.Fatal(err)
    }
    defer func() {
        if rErr := os.RemoveAll(dir); rErr != nil {
            t.Error(rErr)
        }
    }()

    ctx, cancel := context.WithCancel(context.Background())
    1 sSocket := filepath.Join(dir, fmt.Sprintf("s%d.sock", os.Getpid()))
    serverAddr, err := datagramEchoServer(ctx, "unixgram", sSocket)
    if err != nil {
        t.Fatal(err)
    }
    defer cancel()

    err = os.Chmod(sSocket, os.ModeSocket|0622)
    if err != nil {
        t.Fatal(err)
    }
```

Listing 7-7: Instantiating the datagram-based echo server (_echo_posix_test.go_)

Just as with UDP connections, both the server and the client must bind to an address so they can send and receive datagrams. The server has its own socket file 1 that is separate from the client’s socket file in [Listing 7-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-8).

```
--snip--

    1 cSocket := filepath.Join(dir, fmt.Sprintf("c%d.sock", os.Getpid()))
    client, err := net.ListenPacket("unixgram", cSocket)
    if err != nil {
        t.Fatal(err)
    }
    2 defer func() { _ = client.Close() }()

    err = 3os.Chmod(cSocket, os.ModeSocket|0622)
    if err != nil {
        t.Fatal(err)
    }
```

Listing 7-8: Instantiating the datagram-based client (_echo_posix_test.go_)

The call to `os.Remove` in [Listing 7-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-5)’s `datagramEchoServer` function cleans up the socket file when the server closes. The client has some additional housecleaning, so you make the client clean up its own socket file 1 when it’s done listening to it. Thankfully, this is taken care of for you by the call to `os.RemoveAll` to remove your temporary subdirectory in [Listing 7-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-7). Otherwise, you would need to add a call to `os.Remove` to remove the client’s socket file in the `defer`2. Also, the server should be able to write to the client’s socket file as well as its own socket file, or the server won’t be able to reply to messages. In this example, you set very permissive permissions so all users can write to the socket 3.

Now that the server and client are instantiated, [Listing 7-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-9) tests the difference between a streaming echo server and a datagram echo server.

```
--snip--

    msg := []byte("ping")
    for i := 0; i < 3; i++ { // write 3 "ping" messages
        _, err = 1client.WriteTo(msg, serverAddr)
        if err != nil {
            t.Fatal(err)
        }
    }

    buf := make([]byte, 1024)
    for i := 0; i < 3; i++ { // read 3 "ping" messages
        n, addr, err := 2client.ReadFrom(buf)
        if err != nil {
            t.Fatal(err)
        }

        if addr.String() != serverAddr.String() {
            t.Fatalf("received reply from %q instead of %q", 
                addr, serverAddr)
        }

        if !bytes.Equal(msg, buf[:n]) {
            t.Fatalf("expected reply %q; actual reply %q", msg,
                buf[:n])
        }
    }
}
```

Listing 7-9: Using unixgram sockets to echo messages (_echo_posix_test.go_)

You write three ping messages to the server 1 before reading the first datagram. You then perform three reads 2 with a buffer large enough to fit all three ping messages. As expected, `unixgram` sockets maintain the delineation between messages; you sent three messages and read three replies. Compare this to the `unix` socket type in Listings 7-3 and 7-4, where you sent three messages and received all three replies with a single read from the connection.

#### The unixpacket Sequence Packet Socket

The _sequence packet socket_ type is a hybrid that combines the session-oriented connections and reliability of TCP with the clearly delineated datagrams of UDP. However, sequence packet sockets discard unrequested data in each datagram. If you read 32 bytes of a 50-byte datagram, for example, the operating system discards the 18 unrequested bytes.

Of the three Unix domain socket types, `unixpacket` enjoys the least cross-platform support. Coupled with `unixpacket`’s hybrid behavior and discarding of unrequested data, `unix` or `unixgram` are better suited for most applications. You are unlikely to find sequence packet sockets in use over the internet. It was largely used in old X.25 telecommunication networks, some types of financial transactions, and AX.25 used in amateur radio.

The test code in [Listing 7-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-10) sets up a demonstration of `unixpacket` sockets.

## NOTE

Windows, WSL, and macOS do not support unixpacket domain sockets.

```
package echo

import (
    "bytes"
    "context"
    "fmt"
    "io/ioutil"
    "net"
    "os"
    "path/filepath"
    "testing"
)

func TestEchoServerUnixPacket(t *testing.T) {
    dir, err := ioutil.TempDir("", "echo_unixpacket")
    if err != nil {
        t.Fatal(err)
    }
    defer func() {
        if rErr := os.RemoveAll(dir); rErr != nil {
            t.Error(rErr)
        }
    }()

    ctx, cancel := context.WithCancel(context.Background())
    socket := filepath.Join(dir, fmt.Sprintf("%d.sock", os.Getpid()))
    rAddr, err := streamingEchoServer(ctx, "unixpacket", socket)
    if err != nil {
        t.Fatal(err)
    }
    defer cancel()

    err = os.Chmod(socket, os.ModeSocket|0666)
    if err != nil {
        t.Fatal(err)
    }
```

Listing 7-10: Instantiating a packet-based streaming echo server (_echo_linux_test.go_)

Notice first that you save this code in a file called _echo_linux_test.go._ The __linux_test.go_ suffix is a build constraint informing Go that it should include this file only when tests are invoked on Linux.

[Listing 7-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-11) dials the echo server and sends a series of ping messages.

```
--snip--

    conn, err := 1net.Dial("unixpacket", rAddr.String())
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = conn.Close() }()

    msg := []byte("ping")
    2 for i := 0; i < 3; i++ { // write 3 "ping" messages
        _, err = conn.Write(msg)
        if err != nil {
            t.Fatal(err)
        }
    }

    buf := make([]byte, 1024)
    3 for i := 0; i < 3; i++ { // read 3 times from the server
        n, err := conn.Read(buf)
        if err != nil {
            t.Fatal(err)
        }

        if !bytes.Equal(msg, buf[:n]) {
            t.Errorf("expected reply %q; actual reply %q", msg, buf[:n])
        }
    }
```

Listing 7-11: Using a unixpacket socket to echo messages (_echo_linux_test.go_)

Since `unixpacket` is session oriented, you use `net.Dial`1 to initiate a connection with the server. You do not simply write to the server’s address, as you would if the network type were datagram based.

You can see the distinction between the `unix` and `unixpacket` socket types by writing three ping messages to the server 2 before reading the first reply. Whereas a `unix` socket type would return all three ping messages with a single read, `unixpacket` acts just like other datagram-based network types and returns one message for each read operation 3.

[Listing 7-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-12) illustrates how `unixpacket` discards unrequested data in each datagram.

```
--snip--

    for i := 0; i < 3; i++ { // write 3 more "ping" messages
        _, err = conn.Write(msg)
        if err != nil {
            t.Fatal(err)
        }
    }

    1 buf = make([]byte, 2)    // only read the first 2 bytes of each reply
    for i := 0; i < 3; i++ { // read 3 times from the server
        n, err := conn.Read(buf)
        if err != nil {
            t.Fatal(err)
        }

        if !bytes.Equal(2msg[:2], buf[:n]) {
            t.Errorf("expected reply %q; actual reply %q", msg[:2],
                buf[:n])
        }
    }
}
```

Listing 7-12: Discarding unread bytes (_echo_linux_test.go_)

This time around, you reduce your buffer size to 2 bytes 1 and read the first 2 bytes of each datagram. If you were using a streaming network type like `tcp` or `unix`, you would expect to read `pi` for the first read and `ng` for the second read. But `unixpacket` discards the `ng` portion of the `ping` message because you requested only the first 2 bytes—`pi`. Therefore, you make sure you’re receiving only the first 2 bytes of the datagram with each read 2.

## Writing a Service That Authenticates Clients

On Linux systems, Unix domain sockets allow you to glean details about the process on the other end of a socket—your peer—by receiving the credentials from your peer’s operating system. You can use this information to authenticate your peer on the other side of the Unix domain socket and deny access if the peer’s credentials don’t meet your criteria. For instance, if the user _davefromaccounting_ connects to your administrative service through a Unix domain socket, the peer credentials might indicate that you should deny access; Dave should be crunching numbers, not sending bits to your administrative service.

You can create a service that allows connections only from specific users or any user in a specific group found in the _/etc/groups_ file. Each named group in the _/etc/groups_ file has a corresponding group ID number. When a client connects to your Unix domain socket, you can request the peer credentials and compare the client’s group ID in the peer credentials to the group ID of any allowed groups. If the client’s group ID matches one of the allowed group IDs, you can consider the client authenticated. Go’s standard library has useful support for working with Linux groups, which you’ll use in “Writing the Service” on page 156.

### Requesting Peer Credentials

The process of requesting peer credentials isn’t exactly straightforward. You cannot simply request the peer credentials from the connection object itself. Rather, you need to use the `golang.org/x/sys/unix` package to request peer credentials from the operating system, which you can retrieve using the following command:

```
go get -u golang.org/x/sys/unix
```

[Listing 7-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-13) shows a function that accepts a Unix domain socket connection and denies access if the peer isn’t a member of specific groups.

## NOTE

The code in [Listings 7-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-13) through [7-16](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-16) works on Linux systems only.

```
package auth

import (
    "log"
    "net"
    "golang.org/x/sys/unix"
)

func Allowed(conn *net.UnixConn, groups map[string]struct{}) bool {
    if conn == nil || groups == nil || len(groups) == 0 {
        return false
    }

    file, _ := 1conn.File()
    defer func(){ _ = file.Close() }()

    var (
        err   error
        ucred *unix.Ucred
    )

    for {
        ucred, err = 2unix.GetsockoptUcred(int(3file.Fd()), unix.SOL_SOCKET,
            unix.SO_PEERCRED)
        if err == unix.EINTR {
            continue // syscall interrupted, try again
        }
        if err != nil {
            log.Println(err)
            return false
        }

        break
    }

    u, err := 4user.LookupId(string(ucred.Uid))
    if err != nil {
        log.Println(err)
        return false
    }

    gids, err := 5u.GroupIds()
    if err != nil {
        log.Println(err)
        return false
    }

    for _, gid := range gids {
        if _, ok := 6groups[gid]; ok {
            return true
        }
    }

    return false
}
```

Listing 7-13: Retrieving the peer credentials for a socket connection (_creds/auth/allowed_linux.go_)

To retrieve the peer’s Unix credentials, you first grab the underlying file object from `net.UnixConn`1, the object that represents your side of the Unix domain socket connection. It’s analogous to `net.TCPConn` of a TCP connection in Go. Since you need to extract the file descriptor details from the connection, you cannot simply rely on the `net.Conn` interface that you receive from the listener’s `Accept` method. Instead, your `Allowed` function requires the caller to pass in a pointer to the underlying `net.UnixConn` object, typically returned from the listener’s `AcceptUnix` method. You’ll see this method in action in the next section.

You can then pass the file object’s descriptor 3, the protocol-level `unix.SOL_SOCKET`, and the option name `unix.SO_PEERCRED` to the `unix.GetsockoptUcred` function 2. Retrieving socket options from the Linux kernel requires that you specify both the option you want and the level at which the option resides. The `unix.SOL_SOCKET` tells the Linux kernel you want a socket-level option, as opposed to, for example, `unix.SOL_TCP`, which indicates TCP-level options. The `unix.SO_PEERCRED` constant tells the Linux kernel that you want the peer credentials option. If the Linux kernel finds the peer credentials option at the Unix domain socket level, `unix.GetsockoptUcred` returns a pointer to a valid `unix.Ucred` object.

The `unix.Ucred` object contains the peer’s process, user, and group IDs. You pass the peer’s user ID to the `user.LookupId` function 4. If successful, you then retrieve a list of group IDs from the user object 5. The user can belong to more than one group, and you want to consider each one for access. Finally, you check each group ID against a map of allowed groups 6. If any one of the peer’s group IDs is in your map, you return `true`, allowing the peer to connect.

This example is largely didactic. You can achieve similar results by assigning group ownership to the socket file, as we discussed in “Changing a Socket File’s Ownership and Permissions” on page 143. However, knowledge of group membership could be used for access control and other security decisions within your application.

### Writing the Service

Let’s now use this function in a service that you can run from the command line. This service will accept one or more group names found in the Linux operating system’s _/etc/group_ file as arguments on the command line and begin listening to a Unix domain socket file. The service will allow clients to connect only if they are a member of one of the groups specified on the command line. Clients can then make a Unix domain socket connection to the service. The service will retrieve the peer credentials of the client and either allow the client to remain connected, if the client is a member of one of the allowed groups, or immediately disconnect the unauthorized client. The service doesn’t do anything beyond authenticating the client’s group ID.

In [Listing 7-14](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-14), you specify the imports you’ll need and create a meaningful usage message for the service.

```
package main

import (
    "flag"
    "fmt"
    "log"
    "net"
    "os"
    "os/signal"
    "os/user"
    "path/filepath"

    "github.com/awoodbeck/gnp/ch07/creds/auth"
)

func init() {
    flag.Usage = func() {
        _, _ = fmt.Fprintf(flag.CommandLine.Output(),
            "Usage:\n\t%s 1<group names>\n", filepath.Base(os.Args[0]))
        flag.PrintDefaults()
    }
}
```

Listing 7-14: Expecting group names on the command line (_creds/creds.go_)

Our application expects a series of group names as arguments 1. You’ll add the group ID for each group name to the map of allowed groups. The code in [Listing 7-15](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-15) parses these group names.

```
--snip--

func parseGroupNames(args []string) map[string]struct{} {
    groups := make(map[string]struct{})

    for _, arg := range args {
        grp, err := 1user.LookupGroup(arg)
        if err != nil {
            log.Println(err)
            continue
        }

        groups[2grp.Gid] = struct{}{}
    }

    return groups
}
```

Listing 7-15: Parsing group names into group IDs (_creds/creds.go_)

The `parseGroupNames` function accepts a string slice of group names, retrieves the group information for each group name 1, and inserts each group ID into the `groups` map 2.

[Listing 7-16](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-16) ties the last few listings together into a service that you can connect to from the command line.

```
--snip--

func main() {
    flag.Parse()

    groups := parseGroupNames(flag.Args())
    socket := filepath.Join(os.TempDir(), "creds.sock")
    addr, err := net.ResolveUnixAddr("unix", socket)
    if err != nil {
        log.Fatal(err)
    }

    s, err := net.ListenUnix("unix", addr)
    if err != nil {
        log.Fatal(err)
    }

    c := make(chan os.Signal, 1)
    signal.Notify(c, 1os.Interrupt)
    2 go func() {
        <-c
        _ = s.Close()
    }()

    fmt.Printf("Listening on %s ...\n", socket)

    for {
        conn, err := 3s.AcceptUnix()
        if err != nil {
            break
        }
        if 4auth.Allowed(conn, groups) {
            _, err = conn.Write([]byte("Welcome\n"))
            if err == nil {
                // handle the connection in a goroutine here
                continue
            }
        } else {
            _, err = conn.Write([]byte("Access denied\n"))
        }
        if err != nil {
            log.Println(err)
        }
        _ = conn.Close()
    }
}
```

Listing 7-16: Authorizing peers based on their credentials (_creds/creds.go_ continued)

You start by parsing the command line arguments to create the map of allowed group IDs. You then create a listener on the _/tmp/creds.sock_ socket. The listener accepts connections by using `AcceptUnix`3 so a `*net.UnixConn` is returned instead of the usual `net.Conn`, since your `auth.Allowed` function requires a `*net.UnixConn` type as its first argument. You then determine whether the peer’s credentials are allowed 4. Allowed peers stay connected. Disallowed peers are immediately disconnected.

Since you’ll execute this service on the command line, you’ll stop the service by sending an interrupt signal, usually with the CTRL-C key combination. However, this signal abruptly terminates the service before Go has a chance to clean up the socket file, despite your diligent use of `net.ListenUnix`. Therefore, you need to listen for this signal 1 and spin off a goroutine in which you gracefully close the listener after receiving the signal 2. This will make sure Go properly cleans up the socket file.

### Testing the Service with Netcat

Netcat is a popular command line utility that allows you to make TCP, UDP, and Unix domain socket connections. You’ll use it to test the service from the command line. You can likely find Netcat in your Linux distribution’s package manager. For example, to install the OpenBSD rewrite of Netcat on Debian 10, run the following command:

```
$ sudo apt install netcat-openbsd
```

The command uses the `sudo` command line utility to run `apt install netcat-openbsd` masquerading as the `root` user. CentOS 8.1 offers Nmap’s Netcat replacement. Run this command to install it:

```
$ sudo dnf install nmap-ncat
```

Once it’s installed, you should find the _nc_ binary in your `PATH` environment variable.

Before you can connect to your credential-checking service, you need to run the service so that it binds to a socket file:

```
$ cd $GOPATH/src/github.com/awoodbeck/gnp/ch07/creds
$ go run . -- users staff
Listening on /tmp/creds.sock …
```

In this example, you allow connections from any peer in the `users` or `staff` groups. The service will deny access to any peers who are not in at least one of these groups. If these groups do not exist in your Linux distribution, choose any group in the _/etc/groups_ file. The service is listening to the _/tmp/creds.sock_ socket file, which is the address you give to Netcat.

Next, you need a way of changing your group ID so that you can test whether the service denies access to clients you haven’t allowed. Currently, the service is running with your user and group IDs, since you started the service. Therefore, it will accept all your connections, since the service allows its own group (which is our group) to authenticate, per the `groups` map in [Listing 7-15](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c07.xhtml#listing7-15). To change your group when initiating the socket connection with your service, you can use the `sudo` command line utility.

Since using `sudo` requires escalated privileges, you are usually prompted for your password when you attempt to do so. I’ve omitted password prompts from the following examples, but expect to be prompted for your password on `sudo`’s first invocation:

```
$ sudo -g staff -- nc -U /tmp/creds.sock
Welcome
^C
$
```

Using `sudo`, you modify your group by passing the group name to the `-g` flag. In this case, you set your group to `staff`. Then you execute the `nc` command. The `-U` flag tells Netcat to make a Unix domain socket connection to the _/tmp/creds.sock_ file.

Since the `staff` group is one of the allowed groups, you receive the `Welcome` message upon connecting. You terminate your side of the connection by pressing CTRL-C.

If you repeat the test with a disallowed group, you should receive the opposite result:

```
$ sudo -g nogroup -- nc -U /tmp/creds.sock
Access denied
$
```

This time, you use the group `nogroup`, which the service doesn’t allow. As expected, you immediately receive the `Access denied` message, and the server side of the socket terminates your connection.

## What You’ve Learned

You started this chapter with a look at Unix domain sockets. A Unix domain socket is a file-based communication method for processes running on the same node. Two or more processes, such as a local database server and client, can send and receive data through a Unix domain socket. Since Unix domain sockets rely on the filesystem for addressing, you can leverage filesystem ownership and permissions to control access to processes communicating over Unix domain sockets.

You then learned about the types of Unix domain sockets that Go supports: `unix`, `unixgram`, and `unixpacket`. Go makes communication over Unix domain sockets relatively painless and handles many of the details for you, particularly if you stick to the `net` package’s interfaces. For example, code written for use over a stream-based TCP network will also work with little modification over the stream-based `unix` domain socket, albeit only for local process communication. Likewise, code written for use over a UDP network can be leveraged by the `unixgram` domain socket. You also touched on the hybrid Unix domain socket type, `unixpacket`, and learned that its drawbacks don’t outweigh its benefits for most applications, particularly with respect to cross-platform support. The other two Unix domain socket types are better options for most use cases.

This chapter introduced peer credentials and showed how you could use them to authenticate client connections. You can go beyond file-based access restrictions to a Unix domain socket and request details about the client on the other side of the Unix domain socket.

You should now be equipped to determine where Unix domain sockets best fit into your network stack.