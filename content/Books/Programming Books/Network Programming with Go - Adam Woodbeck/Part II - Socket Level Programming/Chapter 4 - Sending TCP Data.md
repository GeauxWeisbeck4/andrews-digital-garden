---
id: 01JC4FZV144VXFSZWK4T7C7FCQ
modified: 2024-11-07T19:05:04-05:00
title: Chapter 4 - Sending TCP Data
tags:
  - go-lang
  - networking
  - programming
  - books
  - no-starch-press
---
# 4  
SENDING TCP DATA

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

Now that you know how to properly establish and gracefully terminate TCP connections in Go, it’s time to put that knowledge to use by transmitting data. This chapter covers various techniques for sending and receiving data over a network using TCP.

We’ll talk about the most common methods of reading data from network connections. You’ll create a simple messaging protocol that allows you to transmit dynamically sized payloads between nodes. You’ll then explore the networking possibilities afforded by the `net.Conn` interface. The chapter concludes with a deeper dive into the `TCPConn` object and insidious TCP networking problems that Go developers may experience.

## Using the net.Conn Interface

Most of the network code in this book uses Go’s `net.Conn` interface whenever possible, because it provides the functionality we need for most cases. You can write powerful network code using the `net.Conn` interface without having to assert its underlying type, ensuring your code is compatible across operating systems and allowing you to write more robust tests. (You will learn how to access `net.Conn`’s underlying type to use its more advanced methods later in this chapter.) The methods available on `net.Conn` cover most use cases.

The two most useful `net.Conn` methods are `Read` and `Write`. These methods implement the `io.Reader` and `io.Writer` interfaces, respectively, which are ubiquitous in the Go standard library and ecosystem. As a result, you can leverage the vast amounts of code written for those interfaces to create incredibly powerful network applications.

You use `net.Conn`’s `Close` method to close the network connection. This method will return `nil` if the connection successfully closed or an error otherwise. The `SetReadDeadline` and `SetWriteDeadline` methods, which accept a `time.Time` object, set the absolute time after which reads and writes on the network connection will return an error. The `SetDeadline` method sets both the read and write deadlines at the same time. As discussed in “Implementing Deadlines” on page 62, deadlines allow you to control how long a network connection may remain idle and allow for timely detection of network connectivity problems.

## Sending and Receiving Data

Reading data from a network connection and writing data to it is no different from reading and writing to a file object, since `net.Conn` implements the `io.ReadWriteCloser` interface used to read and write to files. In this section, you’ll first learn how to read data into a fixed-size buffer. Next, you’ll learn how to use `bufio.Scanner` to read data from a network connection until it encounters a specific delimiter. You’ll then explore TLV, an encoding method that enables you to define a basic protocol to dynamically allocate buffers for varying payload sizes. Finally, you’ll see how to handle errors when reading from and writing to network connections.

### Reading Data into a Fixed Buffer

TCP connections in Go implement the `io.Reader` interface, which allows you to read data from the network connection. To read data from a network connection, you need to provide a buffer for the network connection’s `Read` method to fill.

The `Read` method will populate the buffer to its capacity if there is enough data in the connection’s receive buffer. If there are fewer bytes in the receive buffer than the capacity of the buffer you provide, `Read` will populate the given buffer with the data and return instead of waiting for more data to arrive. In other words, `Read` is not guaranteed to fill your buffer to capacity before it returns. [Listing 4-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-1) demonstrates the process of reading data from a network connection into a byte slice.

```
package main

import (
    "crypto/rand"
    "io"
    "net"
    "testing"
)

func TestReadIntoBuffer(t *testing.T) {
    1 payload := make([]byte, 1<<24) // 16 MB
    _, err := rand.Read(payload)   // generate a random payload
    if err != nil {
        t.Fatal(err)
    }

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
        defer conn.Close()

        2 _, err = conn.Write(payload)
        if err != nil {
            t.Error(err)
        }
    }()

    conn, err := net.Dial("tcp", listener.Addr().String())
    if err != nil {
        t.Fatal(err)
    }

    buf := make([]byte, 31<<19) // 512 KB

    for {
        4 n, err := conn.Read(buf)
        if err != nil {
            if err != io.EOF {
                t.Error(err)
            }
            break
        }

        t.Logf("read %d bytes", n) // buf[:n] is the data read from conn
    }

    conn.Close()
}
```

Listing 4-1: Receiving data over a network connection (_read_test.go_)

You need something for the client to read, so you create a 16MB payload of random data 1—more data than the client can read in its chosen buffer size of 512KB 3 so that it will make at least a few iterations around its `for` loop. It’s perfectly acceptable to use a larger buffer or a smaller payload and read the entirety of the payload in a single call to `Read`. Go correctly processes the data regardless of the payload and receive buffer sizes.

You then spin up the listener and create a goroutine to listen for incoming connections. Once accepted, the server writes the entire payload to the network connection 2. The client then reads up to the first 512KB from the connection 4 before continuing around the loop. The client continues to read up to 512KB at a time until either an error occurs or the client reads the entire 16MB payload.

### Delimited Reading by Using a Scanner

Reading data from a network connection by using the method I just showed means your code needs to make sense of the data it receives. Since TCP is a stream-oriented protocol, a client can receive a stream of bytes across many packets. Unlike sentences, binary data doesn’t include inherent punctuation that tells you where one message starts and stops.

If, for example, your code is reading a series of email messages from a server, your code will have to inspect each byte for delimiters indicating the boundaries of each message in the stream of bytes. Alternatively, your client may have an established protocol with the server whereby the server sends a fixed number of bytes to indicate the payload size the server will send next. Your code can then use this size to create an appropriate buffer for the payload. You’ll see an example of this technique a little later in this chapter.

However, if you choose to use a delimiter to indicate the end of one message and the beginning of another, writing code to handle edge cases isn’t so simple. For example, you may read 1KB of data from a single `Read` on the network connection and find that it contains two delimiters. This indicates that you have two complete messages, but you don’t have enough information about the chunk of data following the second delimiter to know whether it is also a complete message. If you read another 1KB of data and find no delimiters, you can conclude that this entire block of data is a continuation of the last message in the previous 1KB you read. But what if you read 1KB of nothing but delimiters?

If this is starting to sound a bit complex, it’s because you must account for data across multiple `Read` calls and handle any errors along the way. Anytime you’re tempted to roll your own solution to such a problem, check the standard library to see if a tried-and-true implementation already exists. In this case, `bufio.Scanner` does what you need.

The `bufio.Scanner` is a convenient bit of code in Go’s standard library that allows you to read delimited data. The `Scanner` accepts an `io.Reader` as its input. Since `net.Conn` has a `Read` method that implements the `io.Reader` interface, you can use the `Scanner` to easily read delimited data from a network connection. [Listing 4-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-2) sets up a listener to serve up delimited data for later parsing by `bufio.Scanner`.

```
package main

import (
    "bufio"
    "net"
    "reflect"
    "testing"
)

const 1payload = "The bigger the interface, the weaker the abstraction."

func TestScanner(t *testing.T) {
    listener, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    go func() {
        conn, err := listener.Accept()
        if err != nil {
            t.Error(err)
            return
        }
        defer conn.Close()

        _, err = conn.Write([]byte(payload))
        if err != nil {
            t.Error(err)
        }
    }()

--snip--
```

Listing 4-2: Creating a test to serve up a constant payload (_scanner_test.go_)

This listener should look familiar by now. All it’s meant to do is serve up the payload 1. [Listing 4-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-3) uses `bufio.Scanner` to read a string from the network, splitting each chunk by whitespace.

```
--snip--

    conn, err := net.Dial("tcp", listener.Addr().String())
    if err != nil {
        t.Fatal(err)
    }
    defer conn.Close()

    1 scanner := bufio.NewScanner(conn)
    scanner.Split(bufio.ScanWords)

    var words []string

    2 for scanner.Scan() {
        words = append(words, 3scanner.Text())
    }

    err = scanner.Err()
    if err != nil {
        t.Error(err)
    }

    expected := []string{"The", "bigger", "the", "interface,", "the", 
        "weaker", "the", "abstraction."}

    if !reflect.DeepEqual(words, expected) {
        t.Fatal("inaccurate scanned word list")
    }
    4 t.Logf("Scanned words: %#v", words)
}
```

Listing 4-3: Using bufio.Scanner to read whitespace-delimited text from the network (_scanner_test.go_)

Since you know you’re reading a string from the server, you start by creating a `bufio.Scanner` that reads from the network connection 1. By default, the scanner will split data read from the network connection when it encounters a newline character (`\n`) in the stream of data. Instead, you elect to have the scanner delimit the input at the end of each word by using `bufio.ScanWords`, which will split the data when it encounters a word border, such as whitespace or sentence-terminating punctuation.

You keep reading data from the scanner as long as it tells you it’s read data from the connection 2. Every call to `Scan` can result in multiple calls to the network connection’s `Read` method until the scanner finds its delimiter or reads an error from the connection. It hides the complexity of searching for a delimiter across one or more reads from the network connection and returning the resulting messages.

The call to the scanner’s `Text` method returns the chunk of data as a string—a single word and adjacent punctuation, in this case—that it just read from the network connection 3. The code continues to iterate around the `for` loop until the scanner receives an `io.EOF` or other error from the network connection. If it’s the latter, the scanner’s `Err` method will return a non-`nil` error. You can view the scanned words 4 by adding the `-v` flag to the `go test` command.

### Dynamically Allocating the Buffer Size

You can read data of variable length from a network connection, provided that both the sender and receiver have agreed on a protocol for doing so. The _type-length-value__(TLV)_ encoding scheme is a good option. TLV encoding uses a fixed number of bytes to represent the type of data, a fixed number of bytes to represent the value size, and a variable number of bytes to represent the value itself. Our implementation uses a 5-byte header: 1 byte for the type and 4 bytes for the length. The TLV encoding scheme allows you to send a type as a series of bytes to a remote node and constitute the same type on the remote node from the series of bytes.

[Listing 4-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-4) defines the types that our TLV encoding protocol will accept.

```
package main

import (
    "bytes"
    "encoding/binary"
    "errors"
    "fmt"
    "io"
)

const (
    1 BinaryType uint8 = iota + 1
    2 StringType

    3 MaxPayloadSize uint32 = 10 << 20 // 10 MB
)

var ErrMaxPayloadSize = errors.New("maximum payload size exceeded")

type 4Payload interface {
    fmt.Stringer
    io.ReaderFrom
    io.WriterTo
    Bytes() []byte
}
```

Listing 4-4: The message struct implements a simple protocol (_types.go_).

You start by creating constants to represent each type you will define. In this example, you will create a `BinaryType`1 and a `StringType`2. After digesting the implementation details of each type, you should be able to create types that fit your needs. For security purposes that we’ll discuss in just a moment, you must define a maximum payload size 3.

You also define an interface named `Payload`4 that describes the methods each type must implement. Each type must have the following methods: `Bytes`, `String`, `ReadFrom`, and `WriteTo`. The `io.ReaderFrom` and `io.WriterTo` interfaces allow your types to read from readers and write to writers, respectively. You have some flexibility in this regard. You could just as easily make the Payload implement the `encoding.BinaryMarshaler` interface to marshal itself to a byte slice and the `encoding.BinaryUnmarshaler` interface to unmarshal itself from a byte slice. But the byte slice is one level removed from the network connection, so you’ll keep the `Payload` interface as is. Besides, you’ll use the binary encoding interfaces in the next chapter.

You now have the foundation built to create TLV-based types. [Listing 4-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-5) details the first type, `Binary`.

```
--snip--

type 1Binary []byte

func (m Binary) 2Bytes() []byte  { return m }
func (m Binary) 3String() string { return string(m) }

func (m Binary) 4WriteTo(w io.Writer) (int64, error) {
    err := 5binary.Write(w, binary.BigEndian, BinaryType) // 1-byte type
    if err != nil {
        return 0, err
    }
    var n int64 = 1

    err = 6binary.Write(w, binary.BigEndian, uint32(len(m))) // 4-byte size
    if err != nil {
        return n, err
    }
    n += 4

    o, err := 7w.Write(m) // payload

    return n + int64(o), err
}
```

Listing 4-5: Creating the Binary type (_types.go_)

The `Binary` type 1 is a byte slice; therefore, its `Bytes` method 2 simply returns itself. Its `String` method 3 casts itself as a `string` before returning. The `WriteTo` method accepts an `io.Writer` and returns the number of bytes written to the writer and an `error` interface 4. The `WriteTo` method first writes the 1-byte type to the writer 5. It then writes the 4-byte length of the `Binary` to the writer 6. Finally, it writes the `Binary` value itself 7.

[Listing 4-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-6) rounds out the `Binary` type with its `ReadFrom` method.

```
--snip--

func (m *Binary) ReadFrom(r io.Reader) (int64, error) {
    var typ uint8
    err := 1binary.Read(r, binary.BigEndian, &typ) // 1-byte type
    if err != nil {
        return 0, err
    }
    var n int64 = 1
    if typ != 2BinaryType {
        return n, errors.New("invalid Binary")
    }

    var size uint32
    err = 3binary.Read(r, binary.BigEndian, &size) // 4-byte size
    if err != nil {
        return n, err
    }
    n += 4
    4 if size > MaxPayloadSize {
        return n, ErrMaxPayloadSize
    }

    5 *m = make([]byte, size)
    o, err := 6r.Read(*m) // payload

    return n + int64(o), err
}
```

Listing 4-6: Completing the Binary type’s implementation (_types.go_)

The `ReadFrom` method reads 1 1 byte from the reader into the `typ` variable. It next verifies 2 that the type is `BinaryType` before proceeding. Then it reads 3 the next 4 bytes into the `size` variable, which sizes the new `Binary` byte slice 5. Finally, it populates the `Binary` byte slice 6.

Notice that you enforce a maximum payload size 4. This is because the 4-byte integer you use to designate the payload size has a maximum value of 4,294,967,295, indicating a payload of over 4GB. With such a large payload size, it would be easy for a malicious actor to perform a denial-of-service attack that exhausts all the available random access memory (RAM) on your computer. Keeping the maximum payload size reasonable makes memory exhaustion attacks harder to execute.

[Listing 4-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-7) introduces the `String` type, which, like `Binary`, implements the `Payload` interface.

```
--snip--

type String string

func (m String) 1Bytes() []byte  { return []byte(m) }
func (m String) 2String() string { return string(m) }

func (m String) 3WriteTo(w io.Writer) (int64, error) {
    err := 4binary.Write(w, binary.BigEndian, StringType) // 1-byte type
    if err != nil {
        return 0, err
    }
    var n int64 = 1

    err = binary.Write(w, binary.BigEndian, uint32(len(m))) // 4-byte size
    if err != nil {
        return n, err
    }
    n += 4

    o, err := 5w.Write([]byte(m)) // payload

    return n + int64(o), err
}
```

Listing 4-7: Creating the String type (_types.go_)

The `String` implementation’s `Bytes` method 1 casts the `String` to a byte slice. The `String` method 2 casts the `String` type to its base type, `string`. The `String` type’s `WriteTo` method 3 is like `Binary`’s `WriteTo` method except the first byte written 4 is the `StringType` and it casts the `String` to a byte slice before writing it to the writer 5.

[Listing 4-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-8) finishes up the `String` type’s `Payload` implementation.

```
--snip--

func (m *String) ReadFrom(r io.Reader) (int64, error) {
    var typ uint8
    err := binary.Read(r, binary.BigEndian, &typ) // 1-byte type
    if err != nil {
        return 0, err
    }
    var n int64 = 1
    if typ != 1StringType {
        return n, errors.New("invalid String")
    }

    var size uint32
    err = binary.Read(r, binary.BigEndian, &size) // 4-byte size
    if err != nil {
        return n, err
    }
    n += 4

    buf := make([]byte, size)
    o, err := r.Read(buf) // payload
    if err != nil {
        return n, err
    }
    2 *m = String(buf)

    return n + int64(o), nil
}
```

Listing 4-8: Completing the String type’s implementation (_types.go_)

Here, too, `String`’s `ReadFrom` method is like `Binary`’s `ReadFrom` method, with two exceptions. First, the method compares the `typ` variable against the `StringType`1 before proceeding. Second, the method casts the value read from the reader to a `String`2.

All that’s left to implement is a way to read arbitrary data from a network connection and use it to constitute one of our two types. For that, we turn to [Listing 4-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-9).

```
--snip--

func 1decode(r io.Reader) (Payload, error) {
    var typ uint8
    err := 2binary.Read(r, binary.BigEndian, &typ)
    if err != nil {
        return nil, err
    }

    3 var payload Payload

    switch 4typ {
    case BinaryType:
        payload = new(Binary)
    case StringType:
        payload = new(String)
    default:
        return nil, errors.New("unknown type")
    }

    _, err = payload.ReadFrom(
        5 io.MultiReader(bytes.NewReader([]byte{typ}), r))
    if err != nil {
        return nil, err
    }

    return payload, nil
}
```

Listing 4-9: Decoding bytes from a reader into a Binary or String type (_types.go_)

The `decode` function 1 accepts an `io.Reader` and returns a `Payload` interface and an `error` interface. If `decode` cannot decode the bytes read from the reader into a `Binary` or `String` type, it will return an error along with a nil `Payload`.

You must first read a byte from the reader 2 to determine the type and create a `payload` variable 3 to hold the decoded type. If the type you read from the reader is an expected type constant 4, you assign the corresponding type to the payload variable.

You now have enough information to finish decoding the binary data from the reader into the `payload` variable by using its `ReadFrom` method. But you have a problem here. You cannot simply pass the reader to the `ReadFrom` method. You’ve already read a byte from it corresponding to the type, yet the `ReadFrom` method expects the first byte it reads to be the type as well. Thankfully, the `io` package has a helpful function you can use: `MultiReader`. We cover `io.MultiReader` in more detail later in this chapter, but here you use it to concatenate the byte you’ve already read with the reader 5. From the `ReadFrom` method’s perspective, it will read the bytes in the sequence it expects.

Although the use of `io.MultiReader` shows you how to inject bytes back into a reader, it isn’t optimal in this use case. The proper fix is to remove each type’s need to read the first byte in its `ReadFrom` method. Then, the `ReadFrom` method would read only the 4-byte size and the payload, eliminating the need to inject the type byte back into the reader before passing it on to `ReadFrom`. As an exercise, I recommend you refactor the code to eliminate the need for `io.MultiReader`.

Let’s see the `decode` function in action in the form of a test. [Listing 4-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-10) illustrates how you can send your two distinct types over a network connection and properly decode them back into their original type on the receiver’s end.

```
package main

import (
    "bytes"
    "encoding/binary"
    "net"
    "reflect"
    "testing"
)

func TestPayloads(t *testing.T) {
    b1 := 1Binary("Clear is better than clever.")
    b2 := Binary("Don't panic.")
    s1 := 2String("Errors are values.")
    payloads := 3[]Payload{&b1, &s1, &b2}

    listener, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    go func() {
        conn, err := listener.Accept()
        if err != nil {
            t.Error(err)
            return
        }
        defer conn.Close()

        for _, p := range payloads {
            _, err = 4p.WriteTo(conn)
            if err != nil {
                t.Error(err)
                break
            }
        }
    }()

--snip--
```

Listing 4-10: Creating the TestPayloads test (_types_test.go_)

Your test should first create at least one of each type. You create two `Binary` types 1 and one `String` type 2. Next, you create a slice of `Payload` interfaces and add pointers to the `Binary` and `String` types you created 3. You then create a listener that will accept a connection and write each type in the `payloads` slice to it 4.

This is a good start. Let’s finish up the client side of the test in [Listing 4-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-11).

```
--snip--

    conn, err := 1net.Dial("tcp", listener.Addr().String())
    if err != nil {
        t.Fatal(err)
    }
    defer conn.Close()

    for i := 0; i < len(payloads); i++ {
        actual, err := 2decode(conn)
        if err != nil {
            t.Fatal(err)
        }

        3 if expected := payloads[i]; !reflect.DeepEqual(expected, actual) {
            t.Errorf("value mismatch: %v != %v", expected, actual)
            continue
        }

        4 t.Logf("[%T] %[1]q", actual)
    }
}
```

Listing 4-11: Completing the TestPayloads test (_types_test.go_)

You know how many types to expect in the payloads slice, so you initiate a connection to the listener 1 and attempt to decode each one 2. Finally, your test compares the type you decoded with the type the server sent 3. If there’s any discrepancy with the variable type or its contents, the test fails. You can run the test with the `-v` flag to see the type and its value 4.

Let’s make sure the `Binary` type enforces the maximum payload size in [Listing 4-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-12).

```
--snip--

func TestMaxPayloadSize(t *testing.T) {
    buf := new(bytes.Buffer)
    err := buf.WriteByte(BinaryType)
    if err != nil {
        t.Fatal(err)
    }

    err = binary.Write(buf, binary.BigEndian, 1uint32(1<<30)) // 1 GB
    if err != nil {
        t.Fatal(err)
    }

    var b Binary
    _, err = b.ReadFrom(buf)
    2 if err != ErrMaxPayloadSize {
        t.Fatalf("expected ErrMaxPayloadSize; actual: %v", err)
    }
}
```

Listing 4-12: Testing the maximum payload size (_types_test.go_)

This test starts with the creation of a `bytes.Buffer` containing the `BinaryType` byte and a 4-byte, unsigned integer indicating the payload is 1GB 1. When this buffer is passed to the `Binary` type’s `ReadFrom` method, you receive the `ErrMaxPayloadSize` error in return 2. The test cases in Listings 4-10 and 4-11 should cover the use case of a payload that is less than the maximum size, but I encourage you to modify this test to make sure that’s the case.

### Handling Errors While Reading and Writing Data

Unlike writing to file objects, writing to network connections can be unreliable, especially if your network connection is spotty. Files don’t often return errors while you’re writing to them, but the receiver on the other end of a network connection may abruptly disconnect before you write your entire payload.

Not all errors returned when reading from or writing to a network connection are permanent. The connection can recover from some errors. For example, writing data to a network connection where adverse network conditions delay the receiver’s ACK packets, and where your connection times out while waiting to receive them, can result in a temporary error. This can occur if someone temporarily unplugs a network cable between you and the receiver. In that case, the network connection is still active, and you can either attempt to recover from the error or gracefully terminate your end of the connection.

[Listing 4-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-13) illustrates how to check for temporary errors while writing data to a network connection.

```
var (
    err error
    n int
    i = 7 // maximum number of retries
)

1 for ; i > 0; i-- {
    n, err = 2conn.Write(3[]byte("hello world"))
    if err != nil {
        if nErr, ok := 4err.(net.Error); ok && 5nErr.Temporary() {
            log.Println("temporary error:", nErr)
            time.Sleep(10 * time.Second)
            continue
        }
       6 return err
    }
    break
}

if i == 0 {
    return errors.New("temporary write failure threshold exceeded")
}

log.Printf("wrote %d bytes to %s\n", n, conn.RemoteAddr())
```

Listing 4-13: Sending the string "hello world" over the connection

Since you might receive a transient error when writing to a network connection, you might need to retry a write operation. One way to account for this is to encapsulate the code in a `for` loop 1. This makes it easy to retry the write operation, if necessary.

To write to the connection, you pass a byte slice 3 to the connection’s `Write` method 2 as you would to any other `io.Writer`. This returns the number of bytes written and an `error` interface. If the `error` interface is not `nil`, you check whether the error implements the `net.Error` interface by using a type assertion 4 and check whether the error is temporary 5. If the `net.Error`’s `Temporary` method returns `true`, the code makes another write attempt by iterating around the `for` loop. If the error is permanent, the code returns the error 6. A successful write breaks out of the loop.

## Creating Robust Network Applications by Using the io Package

In addition to interfaces common in Go code, such as `io.Reader` and `io.Writer`, the `io` package provides several useful functions and utilities that make the creation of robust network applications easy. In this section, you’ll learn how to use the `io.Copy`, `io.MultiWriter`, and `io.TeeReader` functions to proxy data between connections, log network traffic, and ping hosts when firewalls attempt to keep you from doing so.

### Proxying Data Between Connections

One of the most useful functions from the `io` package, the `io.Copy` function reads data from an `io.Reader` and writes it to an `io.Writer`. This is useful for creating a _proxy_, which, in this context, is an intermediary that transfers data between two nodes. Since `net.Conn` includes both `io.Reader` and `io.Writer` interfaces, and `io.Copy` writes whatever it reads from an `io.Reader` to an `io.Writer`, you can easily create a proxy between network connections, such as the one you define in the `proxyConn` function in [Listing 4-14](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-14). This function copies any data sent from the source node to the destination node, and vice versa.

```
package main

import (
    "io"
    "net"
)

func proxyConn(source, destination string) error {
    connSource, err := 1net.Dial("tcp", source)
    if err != nil {
        return err
    }
    defer connSource.Close()

    connDestination, err := 2net.Dial("tcp", destination)
    if err != nil {
        return err
    }
    defer connDestination.Close()

    // connDestination replies to connSource
    3 go func() { _, _ = io.Copy(connSource, connDestination) }()

    // connSource messages to connDestination
    4 _, err = io.Copy(connDestination, connSource)

    return err
}
```

Listing 4-14: Proxying data between two network connections (_proxy_conn.go_)

The `io.Copy` function does all the heavy input/output (I/O) lifting for you. It takes an `io.Writer` as its first argument and an `io.Reader` as its second argument. It then writes, to the writer, everything it reads from the reader until the reader returns an `io.EOF`, or, alternately, either the reader or writer returns an error. The `io.Copy` function returns an error only if a non-`io.EOF` error occurred during the copy, because `io.EOF` means it has read all the data from the reader.

You start by creating a connection to the `source` node 1 and a connection to the `destination` node 2. Next, you run `io.Copy` in a goroutine, reading from `connDestination` and writing to `connSource`3 to handle any replies. You don’t need to worry about leaking this goroutine, since `io.Copy` will return when either connection is closed. Then, you make another call to `io.Copy`, reading from `connSource` and writing to `connDestination`4. Once this call returns and the function returns, each connection’s `Close` method runs, which causes `io.Copy` to return, terminating its goroutine 3. As a result, the data is proxied between network connections as if they had a direct connection to one another.

## NOTE

Since Go version 1.11, if you use io.Copy or io.CopyN when the source and destination are both *net.TCPConn objects, the data never enters the user space on Linux, thereby causing the data transfer to occur more efficiently. Think of it as the Linux kernel reading from one socket and writing to the other without the data needing to interact directly with your Go code. io.CopyN functions like io.Copy except it copies up to _n_ bytes. We’ll use io.CopyN in the next chapter.

[Listing 4-15](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-15) illustrates how to use a slight variation of the `proxyConn` function. Whereas [Listing 4-14](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-14)’s `proxyConn` function established network connections and proxied traffic between them, [Listing 4-15](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-15)’s `proxy` function proxies data between an `io.Reader` and an `io.Writer`, making it applicable to more than just network connections and much easier to test.

```
package main

import (
    "io"
    "net"
    "sync"
    "testing"
)

1 func proxy(from io.Reader, to io.Writer) error {
    fromWriter, fromIsWriter := from.(io.Writer)
    toReader, toIsReader := to.(io.Reader)

    if toIsReader && fromIsWriter {
        // Send replies since "from" and "to" implement the
        // necessary interfaces.
        go func() { _, _ = io.Copy(fromWriter, toReader) }()
    }

    _, err := io.Copy(to, from)

    return err
}
```

Listing 4-15: Proxy data between a reader and writer (_proxy_test.go_)

This `proxy` function 1 is a bit more useful in that it accepts the ubiquitous `io.Reader` and `io.Writer` interfaces instead of `net.Conn`. Because of this change, you could proxy data from a network connection to `os.Stdout`, `*bytes.Buffer`, `*os.File`, or any number of objects that implement the `io.Writer` interface. Likewise, you could read bytes from any object that implements the `io.Reader` interface and send them to the writer. This implementation of `proxy` supports replies if the _from_ reader implements the `io.Writer` interface and the _to_ writer implements the `io.Reader` interface.

[Listing 4-16](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-16) creates a test to make sure the proxy functions as you expect.

```
--snip--

func TestProxy(t *testing.T) {
    var wg sync.WaitGroup

    // server listens for a "ping" message and responds with a
    // "pong" message. All other messages are echoed back to the client.
    1 server, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    wg.Add(1)

    go func() {
        defer wg.Done()

        for {
            conn, err := server.Accept()
            if err != nil {
                return
            }

            go func(c net.Conn) {
                defer c.Close()

                for {
                    buf := make([]byte, 1024)
                    n, err := c.Read(buf)
                    if err != nil {
                        if err != io.EOF {
                            t.Error(err)
                        }

                        return
                    }

                    switch msg := string(buf[:n]); msg {
                    case "ping":
                        _, err = c.Write([]byte("pong"))
                    default:
                        _, err = c.Write(buf[:n])
                    }

                    if err != nil {
                        if err != io.EOF {
                            t.Error(err)
                        }

                        return
                    }
                }
            }(conn)
        }
    }()

--snip--
```

Listing 4-16: Creating the listener (_proxy_test.go_)

You start by initializing a server 1 that listens for incoming connections. It reads bytes from each connection, replies with the string `"pong"` when it receives the string `"ping`,`"` and echoes any other message it receives.

[Listing 4-17](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-17) continues the test implementation.

```
--snip--

    // proxyServer proxies messages from client connections to the
    // destinationServer. Replies from the destinationServer are proxied
    // back to the clients.
    1 proxyServer, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        t.Fatal(err)
    }

    wg.Add(1)

    go func() {
        defer wg.Done()

        for {
            conn, err := 2proxyServer.Accept()
            if err != nil {
                return
            }

            go func(from net.Conn) {
                defer from.Close()

                to, err := 3net.Dial("tcp",
                    server.Addr().String())
                if err != nil {
                    t.Error(err)
                    return
                }

                defer to.Close()

                err = 4proxy(from, to)
                if err != nil && err != io.EOF {
                    t.Error(err)
                }
            }(conn)
        }
    }()

--snip--
```

Listing 4-17: Set up the proxy between the client and server (_proxy_test.go_)

You then set up a proxy server 1 that handles the message passing between the client and the destination server. The proxy server listens for incoming client connections. Once a client connection accepts 2, the proxy establishes a connection to the destination server 3 and starts proxying messages 4. Since the proxy server passes two `net.Conn` objects to `proxy`, and `net.Conn` implements the `io.ReadWriter` interface, the server proxies replies automatically. Then `io.Copy` writes to the `Write` method of the destination `net.Conn` everything it reads from the `Read` method of the origin `net.Conn`, and vice versa for replies from the destination to the origin.

[Listing 4-18](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-18) implements the client portion of the test.

```
--snip--

    conn, err := net.Dial("tcp", proxyServer.Addr().String())
    if err != nil {
        t.Fatal(err)
    }

    1 msgs := []struct{ Message, Reply string }{
        {"ping", "pong"},
        {"pong", "pong"},
        {"echo", "echo"},
        {"ping", "pong"},
    }

    for i, m := range msgs {
        _, err = conn.Write([]byte(m.Message))
        if err != nil {
            t.Fatal(err)
        }

        buf := make([]byte, 1024)

        n, err := conn.Read(buf)
        if err != nil {
            t.Fatal(err)
        }

        actual := string(buf[:n])
        t.Logf("%q -> proxy -> %q", m.Message, actual)

        if actual != m.Reply {
            t.Errorf("%d: expected reply: %q; actual: %q",
                i, m.Reply, actual)
        }
    }

    _ = conn.Close()
    _ = proxyServer.Close()
    _ = server.Close()

    wg.Wait()
}
```

Listing 4-18: Proxying data from an upstream server to a downstream server (_proxy_test.go_)

You run the proxy through a series of tests 1 to verify that your ping messages result in pong replies and that the destination echoes anything else you send. The output should look like the following:

```
$ go test 1-race -v proxy_test.go
=== RUN   TestProxy
--- PASS: TestProxy (0.00s)
    proxy_test.go:138: "ping" -> proxy -> "pong"
    proxy_test.go:138: "pong" -> proxy -> "pong"
    proxy_test.go:138: "echo" -> proxy -> "echo"
    proxy_test.go:138: "ping" -> proxy -> "pong"
PASS
ok      command-line-arguments  1.018s
```

I’m in the habit of running my tests with the `-race` flag 1 to enable the race detector. The race detector can help alert you to data races that need your attention. Although not necessary for this test, enabling it is a good habit.

### Monitoring a Network Connection

The `io` package includes useful tools that allow you to do more with network data than just send and receive it using connection objects. For example, you could use `io.MultiWriter` to write a single payload to multiple network connections. You could also use `io.TeeReader` to log data read from a network connection. [Listing 4-19](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-19) gives an example of using the `io.TeeReader` and `io.MultiWriter` to log all network traffic on a TCP listener.

```
package main

import (
    "io"
    "log"
    "net"
    "os"
)

// Monitor embeds a log.Logger meant for logging network traffic.
type Monitor struct {
    *log.Logger
}

// Write implements the io.Writer interface.
func (m *Monitor) 1Write(p []byte) (int, error) {
    return len(p), m.Output(2, string(p))
}

func ExampleMonitor() {
    2 monitor := &Monitor{Logger: log.New(os.Stdout, "monitor: ", 0)}

    listener, err := net.Listen("tcp", "127.0.0.1:")
    if err != nil {
        monitor.Fatal(err)
    }

    done := make(chan struct{})

    go func() {
        defer close(done)

        conn, err := listener.Accept()
        if err != nil {
            return
        }
        defer conn.Close()

        b := make([]byte, 1024)
        3 r := io.TeeReader(conn, monitor)

        n, err := r.Read(b)
        if err != nil && err != io.EOF {
            monitor.Println(err)
            return
        }

        4 w := io.MultiWriter(conn, monitor)

        _, err = w.Write(b[:n]) // echo the message
        if err != nil && err != io.EOF {
            monitor.Println(err)
            return
        }
    }()

--snip--
```

Listing 4-19: Using io.TeeReader and io.MultiWriter to capture a network connection’s input and output (_monitor_test.go_)

You create a new struct named `Monitor` that embeds a `log.Logger` for the purposes of logging the server’s network traffic. Since the `io.TeeReader` and the `io.MultiWriter` expect an `io.Writer`, the monitor implements the `io.Writer` interface 1.

You start by creating an instance of `Monitor`2 that writes to `os.Stdout`. You use the monitor in conjunction with the connection object in an `io.TeeReader`3. This results in an `io.Reader` that will read from the network connection and write all input to the monitor before passing along the input to the caller. Likewise, you log server output by creating an `io.MultiWriter`4, writing to the network connection and the monitor.

[Listing 4-20](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-20) details the client portion of the example and its output.

```
--snip--

    conn, err := net.Dial("tcp", listener.Addr().String())
    if err != nil {
        monitor.Fatal(err)
    }

    _, err = 1conn.Write([]byte("Test\n"))
    if err != nil {
        monitor.Fatal(err)
    }

    _ = conn.Close()
    <-done

    // 2Output:
    // monitor: Test
    // monitor: Test
}
```

Listing 4-20: The client implementation and example output (_monitor_test.go_)

When you send the message `Test\n` 1, it’s logged to `os.Stdout` twice 2: once when you read the message from the connection, and again when you echo the message back to the client. If you want to get fancy, you could decorate the log entries to differentiate between incoming and outgoing data. One way to do this would be to create an object that implements the `io.Writer` interface and embeds the monitor. When its `Write` method is called, it prepends the data with the prefix before passing the data along to the monitor’s `Write` method.

Although using the `io.TeeReader` and the `io.MultiWriter` in this fashion is powerful, it isn’t without a few caveats. First, both the `io.TeeReader` and the `io.MultiWriter` will block while writing to your writer. Your writer will add latency to the network connection, so be mindful not to block too long. Second, an error returned by your writer will cause the `io.TeeReader` or `io.MultiWriter` to return an error as well, halting the flow of network data. If you don’t want your use of these objects to potentially interrupt network data flow, I strongly recommend you implement a reader that always returns a `nil` error and logs its underlying error in a manner that’s actionable.

For example, you can modify `Monitor`’s `Write` method to always return a `nil` error:

```
func (m *Monitor) Write(p []byte) (int, error) {
    err := m.Output(2, string(p))
    if err != nil {
        log.Println(err) // use the log package’s default Logger
    }

    return len(p), nil
}
```

The `Monitor` attempts to write the byte slice to its embedded logger. Failing that, it writes the error to the `log` package’s default logger and returns a `nil` error to `io.TeeReader` and `io.MultiWriter` in [Listing 4-19](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-19) so as not to interrupt the flow of data.

### Pinging a Host in ICMP-Filtered Environments

In “The Internet Control Message Protocol” on page 31, you learned that ICMP is a protocol that gives you feedback about local network conditions. One of its most common uses is to determine whether a host is online by issuing a ping request and receiving a pong reply from the host. Most operating systems have a built-in `ping` command that sends an ICMP echo request to a destination IP address. Once the host responds with an ICMP echo reply, `ping` prints the duration between sending the ping and receiving the pong.

Unfortunately, many internet hosts filter or block ICMP echo replies. If a host filters `pongs`, the `ping` erroneously reports that the remote system is unavailable. One technique you can use instead is to establish a TCP connection with the remote host. If you know that the host listens for incoming TCP connections on a specific port, you can use this knowledge to confirm that the host is available, because you can establish a TCP connection only if the host is up and completes the handshake process.

[Listing 4-21](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-21) shows a small application that reports the time it takes to establish a TCP connection with a host on a specific port.

```
package main

import (
    "flag"
    "fmt"
    "net"
    "os"
    "time"
)

1 var (
    count    = flag.Int("c", 3, "number of pings: <= 0 means forever")
    interval = flag.Duration("i", time.Second, "interval between pings")
    timeout  = flag.Duration("W", 5*time.Second, "time to wait for a reply")
)

func init() {
    flag.Usage = func() {
        fmt.Printf("Usage: %s [options] host:port\nOptions:\n", os.Args[0])
        flag.PrintDefaults()
    }
}

--snip--
```

Listing 4-21: The command line flags for the ping command (_ping.go_)

This example starts by defining a few command line options 1 that mimic a subset of the functionality provided by the `ping` command on Linux.

[Listing 4-22](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-22) adds the `main` function.

```
--snip--

func main() {
    flag.Parse()

    if flag.NArg() != 1 {
        fmt.Print("host:port is required\n\n")
        flag.Usage()
        os.Exit(1)
    }

    target := flag.Arg(0)
    fmt.Println("PING", target)

    if *count <= 0 {
        fmt.Println("CTRL+C to stop.")
    }

    msg := 0

    for (*count <= 0) || (msg < *count) {
        msg++
        fmt.Print(msg, " ")

        start := time.Now()
        1 c, err := net.DialTimeout("tcp", target, *timeout)
        2 dur := time.Since(start)

        if err != nil {
            fmt.Printf("fail in %s: %v\n", dur, err)
            if nErr, ok := err.(net.Error); !ok || 3!nErr.Temporary() {
                os.Exit(1)
            }
        } else {
            _ = c.Close()
            fmt.Println(dur)
        }

        time.Sleep(*interval)
    }
}
```

Listing 4-22: Reporting the time to establish a TCP socket to a given host and port (_ping.go_)

You attempt to establish a connection to a remote host’s TCP port 1, setting a reasonable time-out duration if the remote host doesn’t respond. You keep track of the time it takes to complete the TCP handshake and consider this duration 2 the ping interval between your host and the remote host. If you encounter a temporary error (for example, a time-out), you’ll continue trying, and you’ll exit if the error is permanent 3. This is handy if you restart a TCP service and want to monitor its progress in restarting. Initially, the code in [Listing 4-22](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-22) will report time-out errors, but it will eventually start printing valid results when the service is again listening on the specific port.

It’s important to understand that system admins could consider the code in [Listing 4-22](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-22) abusive, especially if you specify a large ping count. That’s because you aren’t simply asking the remote host to send an echo reply using ICMP. Instead, you’re rapidly establishing and tearing down a TCP connection with every interval. Establishing a TCP connection has more overhead than an ICMP echo request and response. I recommend that you use this method only when intermediate firewalls filter ICMP echo messages and, even then, with the permission of the system admin.

## Exploring Go’s TCPConn Object

For most use cases, the `net.Conn` interface will provide adequate functionality and the best cross-platform support for TCP sessions between nodes. But accessing the underlying `net.TCPConn` object allows fine-grained control over the TCP network connection should you need to do such things as modify the read and write buffers, enable keepalive messages, or change the behavior of pending data upon closing the connection. The `net.TCPConn` object is the concrete object that implements the `net.Conn` interface. Keep in mind that not all the following functionality may be available on your target operating system.

The easiest way to retrieve the `net.TCPConn` object is by using a type assertion. This works for connections where the underlying network is TCP:

```
tcpConn, ok := conn.(*net.TCPConn)
```

On the server side, you can use the `AcceptTCP` method on a `net.TCPListener`, as shown in [Listing 4-23](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-23), to retrieve the `net.TCPConn` object.

```
addr, err := net.ResolveTCPAddr("tcp", "127.0.0.1:")
if err != nil {
    return err
}

listener, err := net.ListenTCP("tcp", addr)
if err != nil {
    return err
}

tcpConn, err := listener.AcceptTCP()
```

Listing 4-23: Retrieving net.TCPConn from the listener

On the client side, use the `net.DialTCP` function, as shown in [Listing 4-24](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-24).

```
addr, err := net.ResolveTCPAddr("tcp", "www.google.com:http")
if err != nil {
    return err
}

tcpConn, err := net.DialTCP("tcp", nil, addr)
```

Listing 4-24: Using DialTCP to retrieve a net.TCPConn object

The next few sections cover useful methods on `net.TCPConn` that are unavailable on `net.Conn`. Some of these methods may not be available on your target operating system or may have hard limits imposed by the operating system. My advice is to use the following methods only when necessary. Altering these settings on the connection object from the operating system defaults may lead to network behavior that’s difficult to debug. For example, shrinking the read buffer on a network connection may lead to unexpected zero window issues unexplained by checking the operating system’s default read buffer value.

### Controlling Keepalive Messages

A _keepalive_ is a message sent over a network connection to check the connection’s integrity by prompting an acknowledgment of the message from the receiver. After an operating system–specified number of unacknowledged keepalive messages, the operating system will close the connection.

The operating system configuration dictates whether a connection uses keepalives for TCP sessions by default. If you need to enable keepalives on a `net.TCPConn` object, pass `true` to its `SetKeepAlive` method:

```
err := tcpConn.SetKeepAlive(true)
```

You also have control over how often the connection sends keepalive messages using the `SetKeepAlivePeriod` method. This method accepts a `time.Duration` that dictates the keepalive message interval:

```
err := tcpConn.SetKeepAlivePeriod(time.Minute)
```

Using deadlines advanced by a heartbeat is usually the better method for detecting network problems. As mentioned earlier in this chapter, deadlines provide better cross-platform support, traverse firewalls better, and make sure your application is actively managing the network connection.

### Handling Pending Data on Close

By default, if you’ve written data to `net.Conn` but the data has yet to be sent to or acknowledged by the receiver and you close the network connection, your operating system will complete the delivery in the background. If you don’t want this behavior, the `net.TCPConn` object’s `SetLinger` method allows you to tweak it:

```
err := tcpConn.SetLinger(-1) // anything < 0 uses the default behavior
```

With the linger disabled, it is possible that the server may receive the last portion of data you send along with your FIN when you close your connection. Since your call to `conn.Close` doesn’t block, you have no way of knowing whether the server received the data you just sent prior to your FIN. It’s possible the data sat in the server’s receive buffer and then the server crashed, taking your unacknowledged data and FIN with it. Lingering on the connection to give the server time to acknowledge the data may seem tempting. But this won’t solve your problem if the server crashes, as in the example. Also, some developers may argue that using linger for this purpose is a code smell. Your application should instead verify that the server received all data before tearing down its connection if this last bit of unacknowledged data is a concern.

If you wish to abruptly discard all unsent data and ignore acknowledgments of sent data upon closing the network connection, set the connection’s linger to zero:

```
err := tcpConn.SetLinger(0) // immediately discard unsent data on close
```

Setting linger to zero will cause your connection to send an RST packet when your code calls your connection’s `Close` method, aborting the connection and bypassing the normal teardown procedures.

If you’re looking for a happy medium and your operating system supports it, you can pass a positive integer _n_ to `SetLinger`. Your operating system will attempt to complete delivery of all outstanding data up to _n_ seconds, after which point your operating system will discard any unsent or unacknowledged data:

```
err := tcpConn.SetLinger(10) // discard unsent data after 10 seconds
```

If you feel compelled to modify your connection’s linger value, please read up on how your operating system handles lingering on network connections. When in doubt, use the default value.

### Overriding Default Receive and Send Buffers

Your operating system assigns read and write buffers to each network connection you create in your code. For most cases, those values should be enough. But in the event you want greater control over the read or write buffer sizes, you can tweak their value, as demonstrated in [Listing 4-25](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-25).

```
if err := tcpConn.SetReadBuffer(212992); err != nil {
    return err
}

if err := tcpConn.SetWriteBuffer(212992); err != nil {
    return err
}
```

Listing 4-25: Setting read and write buffer sizes on a TCP connection

The `SetReadBuffer` method accepts an integer representing the connection’s read buffer size in bytes. Likewise, the `SetWriteBuffer` method accepts an integer and sets the write buffer size in bytes on the connection. Keep in mind that you can’t exceed your operating system’s maximum value for either buffer size.

## Solving Common Go TCP Network Problems

Go doesn’t hold your hand when working with TCP network connections. As such, it’s possible to introduce bugs in your code that manifest as network errors. This section presents two common TCP networking issues: zero window errors and sockets stuck in the CLOSE_WAIT state.

### Zero Window Errors

We spent a bit of time in “Receive Buffers and Window Sizes” on page 48 discussing TCP’s sliding window and how the window size tells the sender how much data the receiver can accept before the next acknowledgment. A common workflow when reading from a network connection is to read some data from the connection, handle the data, read more data from the connection, handle it, and so on.

But what happens if you don’t read data from a network connection quickly enough? Eventually, the sender may fill the receiver’s receive buffer, resulting in a zero-window state. The receiver will not be able to receive data until the application reads data from the buffer. This most often happens when the handling of data read from a network connection blocks and the code never makes its way around to reading from the socket again, as shown in [Listing 4-26](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-26).

```
buf := make([]byte, 1024)

for {
    1 n, err := conn.Read(buf)
    if err != nil {
        return err
    }

    2 handle(buf[:n]) // BLOCKS!
}
```

Listing 4-26: Handling received data blocks preventing iteration around the loop

Reading data from the network connection 1 frees up receive buffer space. If the code blocks for an appreciable amount of time while handling the received data 2, the receive buffer may fill up. A full receive buffer isn’t necessarily bad. Zeroing the window is a way to _throttle_, or slow, the flow of data from the sender by creating backpressure on the sender. But if it’s unintended or prolonged, a zero window may indicate a bug in your code.

### Sockets Stuck in the CLOSE_WAIT State

In “Gracefully Terminating TCP Sessions” on page 50, I mentioned that the server side of a TCP network connection will enter the CLOSE_WAIT state after it receives and acknowledges the FIN packet from the client. If you see TCP sockets on your server that persist in the CLOSE_WAIT state, it’s likely your code is neglecting to properly call the `Close` method on its network connections, as in [Listing 4-27](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c04.xhtml#listing4-27).

```
for {
    conn, err := listener.Accept()
    if err != nil {
        return err
    }

    1 go func(c net.Conn) { // we never call c.Close() before returning!
        buf := make([]byte, 1024)

        for {
            n, err := c.Read(buf)
            if err != nil {
               2  return
            }

            handle(buf[:n])
        }
    }(conn)
}
```

Listing 4-27: Returning from a connection-handling goroutine without properly closing the connection

The listener handles each connection in its own goroutine 1. However, the goroutine fails to call the connection’s `Close` method before fully returning from the goroutine 2. Even a temporary error will cause the goroutine to return. And because you never close the connection, this will leave the TCP socket in the CLOSE_WAIT state. If the server attempted to send anything other than a FIN packet to the client, the client would respond with an RST packet, abruptly tearing down the connection. The solution is to make sure to defer a call to the connection’s `Close` method soon after creating the goroutine 1.

## What You’ve Learned

In this chapter, you first learned several methods of reading data from and writing data to a network connection, including the type-length-value encoding scheme. You built on this knowledge and learned an efficient way to proxy data between network connections. Next, you used a few `io` tools to monitor network traffic. Then, you used your knowledge of TCP handshakes to ping remote hosts in environments where ICMP echo requests and replies are filtered. Finally, the chapter wrapped up by covering the more platform-specific, yet powerful, methods provided by the `net.TCPConn` object and a few common connection-handling bugs.