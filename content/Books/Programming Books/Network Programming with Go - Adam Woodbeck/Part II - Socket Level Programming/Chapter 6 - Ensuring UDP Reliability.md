---
id: 01JC4FZV144VXFSZWK4T7C7FCQ
modified: 2024-11-12T17:53:58-05:00
title: Chapter 6 - Ensuring UDP Reliability
tags:
  - go-lang
  - networking
  - programming
  - books
  - no-starch-press
---
# 6  
ENSURING UDP RELIABILITY

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

Chapter 5 introduced basic network applications using UDP and demonstrated the flexibility of Go’s `net` package and interfaces for writing portable code. This chapter picks up where the last one left off to introduce one method of ensuring reliability when communicating over UDP.

This chapter starts by introducing an application protocol built on top of UDP. We’ll cover a subset of types used by this protocol and demonstrate how they are used to reliably transfer data. We’ll then implement a server that allows clients to download files using the application protocol. Finally, we’ll download a file from our server and verify its integrity.

## Reliable File Transfers Using TFTP

As discussed in the preceding chapter, UDP is inherently unreliable. That means it’s your application’s job to make the UDP connection reliable. Since we spent the last chapter covering UDP and how it’s best used in situations that require a subset of TCP features, it’s only appropriate that we look at an example of such an application-level protocol.

The _Trivial File Transfer Protocol (TFTP)_ is an example of an application protocol that ensures reliable data transfers over UDP. It allows two nodes to transfer files over UDP by implementing a subset of the features that make TCP reliable. A TFTP server implements ordered packet delivery, acknowledgments, and retransmissions. To distill this example down to the essential bits, your server allows clients to download binary data only. It does not support uploads, American Standard Code for Information Interchange (ASCII) transfers, or some of the later additions to TFTP specified outside RFC 1350. Your server expediently serves the same file, no matter what file the client requests, in the name of simplicity.

Please keep in mind that TFTP is not appropriate for secure file transmission. Though it adds reliability to UDP connections, it does not support encryption or authentication. If your application requires communication over UDP, you may want to use WireGuard ([https://github.com/WireGuard/wireguard-go/](https://github.com/WireGuard/wireguard-go/)), an application that allows for secure communication over UDP.

The next few sections will implement a read-only TFTP server to teach you the basics of adding reliability to UDP. By _read-only_, I mean your server will allow clients to only download files, not upload them. You will start by defining the subset of constants and types your TFTP server supports. You will encapsulate as much of the type-related logic in each type’s methods. You’ll then implement the TFTP server portion of the code that will interact with clients and use the types we define to facilitate reliable file transfers.

## TFTP Types

Your TFTP server will accept read requests from the client, send data packets, transmit error packets, and accept acknowledgments from the client. To do this, you must define a few types in your code to represent client requests, transmitted data, acknowledgments, and errors. [Listing 6-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-1) outlines key types used to cap packet sizes, identify operations, and codify various errors.

```
package tftp

import (
    "bytes"
    "encoding/binary"
    "errors"
    "io"
    "strings"
)

const (
    DatagramSize = 1516 // the maximum supported datagram size
    BlockSize = 2DatagramSize – 4 // the DatagramSize minus a 4-byte header
)

3 type OpCode uint16

const (
    OpRRQ OpCode = iota + 1
    _            // no WRQ support
    OpData
    OpAck
    OpErr
)

4 type ErrCode uint16

const (
    ErrUnknown ErrCode = iota
    ErrNotFound
    ErrAccessViolation
    ErrDiskFull
    ErrIllegalOp
    ErrUnknownID
    ErrFileExists
    ErrNoUser
)
```

Listing 6-1: Types and codes used by the TFTP server (_types.go_)

TFTP limits datagram packets to 516 bytes or fewer to avoid fragmentation. You define two constants to enforce the datagram size limit 1 and the maximum data block size 2. The maximum block size is the datagram size minus a 4-byte header. The first 2 bytes of a TFTP packet’s header is an operation code 3.

Each operation code is a 2-byte, unsigned integer. Your server supports four operations: a read request (RRQ), a data operation, an acknowledgment, and an error. Since your server is read-only, you skip the write request (WRQ) definition.

As with the operation codes, you define a series of unsigned 16-bit integer error codes 4 per the RFC. Although you don’t use all error codes in your server since it allows only downloads, a client could return these error codes in lieu of an acknowledgment packet.

The following sections detail the types that implement your server’s four supported operations.

### Read Requests

The server receives a _read request_ packet when the client wants to download a file. The server must then respond with either a data packet or an error packet, both of which you’ll look at in the next few sections. Either packet serves as an acknowledgment to the client that the server received the read request. If the client does not receive a data or error packet, it may retransmit the read request until the server responds or the client gives up.

[Figure 6-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#figure6-1) illustrates the structure of a read request packet.

![f06001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c06/f06001.png)

Figure 6-1: Read request packet structure

The read request packet consists of a 2-byte operation code, a filename, a null byte, a mode, and a trailing null byte. An _operation code_ is an integer that is unique to each of your operation types. Each type’s operation code corresponds to the integer detailed in RFC 1350. For example, a read request’s operation code is 1. The filename and mode are strings of varying lengths. The mode indicates to the server how it should send the file: netascii or octet. If a client requests a file using the _netascii_ mode, the client must convert the file to match its own line-ending format. For our purposes, you will accept only the _octet_ mode, which tells the server to send the file in a binary format, or as is.

[Listing 6-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-2) is a continuation of [Listing 6-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-1). Here, you define the read request and its method that allows the server to marshal the request into a slice of bytes in preparation for writing to a network connection.

```
--snip--

1 type ReadReq struct {
    Filename string
    Mode     string
}

// Although not used by our server, a client would make use of this method.
func (q ReadReq) MarshalBinary() ([]byte, error) {
    mode := "octet"
    if q.Mode != "" {
        mode = q.Mode
    }

    // operation code + filename + 0 byte + mode + 0 byte
    cap := 2 + 2 + len(q.Filename) + 1 + len(q.Mode) + 1

    b := new(bytes.Buffer)
    b.Grow(cap)

    err := 2binary.Write(b, binary.BigEndian, OpRRQ) // write operation code
    if err != nil {
        return nil, err
    }

    _, err = b.WriteString(q.Filename) // write filename
    if err != nil {
        return nil, err
    }

    err = 3b.WriteByte(0) // write 0 byte
    if err != nil {
        return nil, err
    }

    _, err = b.WriteString(mode) // write mode
    if err != nil {
        return nil, err
    }

    err = 3b.WriteByte(0) // write 0 byte
    if err != nil {
        return nil, err
    }

    return b.Bytes(), nil
}
```

Listing 6-2: Read request and its binary marshaling method (_types.go_ continued)

The struct representing your read request 1 needs to keep track of the filename and the mode. You insert the operation code 2 and null bytes 3 into the buffer while marshaling the packet to a byte slice.

[Listing 6-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-3) continues where [Listing 6-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-2) left off and rounds out the read request’s implementation by defining a method that allows the server to unmarshal a read request from a byte slice, typically read from a network connection with a client.

```
--snip--

func (q *ReadReq) 1UnmarshalBinary(p []byte) error {
    r := bytes.NewBuffer(p)

    var code OpCode

    err := 2binary.Read(r, binary.BigEndian, &code) // read operation code
    if err != nil {
        return err
    }

    if code != OpRRQ {
        return errors.New("invalid RRQ")
    }

    q.Filename, err = 3r.ReadString(0) // read filename
    if err != nil {
        return errors.New("invalid RRQ")
    }

    q.Filename = 4strings.TrimRight(q.Filename, "\x00") // remove the 0-byte
    if len(q.Filename) == 0 {
        return errors.New("invalid RRQ")
    }

    q.Mode, err = r.ReadString(0) // read mode
    if err != nil {
        return errors.New("invalid RRQ")
    }

    q.Mode = strings.TrimRight(q.Mode, "\x00") // remove the 0-byte
    if len(q.Mode) == 0 {
        return errors.New("invalid RRQ")
    }

    actual := strings.ToLower(q.Mode) // enforce octet mode
    if actual != "octet" {
        return errors.New("only binary transfers supported")
    }

    return nil
}
```

Listing 6-3: Read request type implementation (_types.go_ continued)

Your TFTP server’s read request, data, acknowledgment, and error packets all implement the `encoding.BinaryMarshaler` and `encoding.BinaryUnmarshaler` interfaces. These methods allow your types to marshal themselves to a binary format suitable for transmission over the network and from network bytes back into the original types. For example, the read request type can marshal itself into a byte slice that matches the read request format showed in [Figure 6-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#figure6-1) by using its `MarshalBinary` method from [Listing 6-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-2). Likewise, it can constitute itself from a byte slice read from the network using its `UnmarshalBinary` method 1. Although your server does not send a read request and make use of its `MarshalBinary` method, I encourage you to write a TFTP client that will marshal a read request to its binary form as you progress through this chapter. I leave it as an exercise for you to implement.

The `UnmarshalBinary` method returns `nil` only if the given byte slice matches the read request format. If you are unsure of whether a given byte slice is a read request, you can pass the byte slice to this method and make that determination based on the return value. You will see this in action when you look at the server code.

The `UnmarshalBinary` method reads in the first 2 bytes 2 and confirms the operation code is that of a read request. It then reads all bytes up to the first null byte 3 and strips the null byte delimiter 4. The resulting string of bytes represents the filename. Similarly, you read in the mode, returning `nil` if everything is as expected. The server can then use the populated `ReadReq` to retrieve the requested file for the client.

### Data Packets

Clients receive _data packets_ in response to their read requests, provided the server was able to retrieve the requested file. The server sends the file in a series of data packets, each of which has an assigned block number, starting at 1 and incrementing with every subsequent data packet. The block number allows the client to properly order the received data and account for duplicates.

All data packets have a payload of 512 bytes except for the last packet. The client continues to read data packets until it receives a data packet whose payload is less than 512 bytes, indicating the end of the transmission. At any point, the client can return an error packet in place of an acknowledgment, and the server can return an error packet instead of a data packet. An error packet immediately terminates the transfer.

[Figure 6-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#figure6-2) shows the format of a data packet.

![f06002](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c06/f06002.png)

Figure 6-2: Data packet structure

Like the read request packet, the data packet’s first 2 bytes contain its operation code. The next 2 bytes represent the block number. The remaining bytes, up to 512, are the payload.

The server requires an acknowledgment from the client after each data packet. If the server does not receive a timely acknowledgment or an error from the client, the server will retry the transmission until it receives a reply or exhausts its number of retries. [Figure 6-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#figure6-3) illustrates the initial communication between a client downloading a file from a TFTP server.

![f06003](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c06/f06003.png)

Figure 6-3: Downloading a file by using the Trivial File Transfer Protocol

Once the client has sent the initial read request packet, the server responds with the first block of data. Next, the client acknowledges receipt of block 1. The server receives the acknowledgment and replies with the second block of data. But in this contrived example, the server does not receive a timely reply from the client, so it resends block 2. The client receives block 2 and sends its acknowledgment. This back-and-forth continues until the server sends the last block with a payload of fewer than 512 bytes.

[Listing 6-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-4) details the data type that is used for the actual data transfer.

```
--snip--

1 type Data struct {
    Block   uint16
    Payload io.Reader
}

2 func (d *Data) MarshalBinary() ([]byte, error) {
    b := new(bytes.Buffer)
    b.Grow(DatagramSize)

    d.Block++ // block numbers increment from 1

    err := binary.Write(b, binary.BigEndian, OpData) // write operation code
    if err != nil {
        return nil, err
    }

    err = binary.Write(b, binary.BigEndian, d.Block) // write block number
    if err != nil {
        return nil, err
    }

    // write up to BlockSize worth of bytes
    _, err = 3io.CopyN(b, d.Payload, BlockSize)
    if err != nil && err != io.EOF {
        return nil, err
    }

    return b.Bytes(), nil
}
```

Listing 6-4: Data type and its binary marshaling method (_types.go_ continued)

The `Data` struct 1 keeps track of the current block number and the data source. In this case, your payload is an `io.Reader` instead of a byte slice, the reasoning being that an `io.Reader` allows greater flexibility about where you retrieve the payload. You could just as easily use an `*os.File` object to read a file from the filesystem as you could use a `net.Conn` to read the data from another network connection. The `io.Reader` interface gives you options that a simple byte slice doesn’t. You’re relying on the reader to keep track of the bytes left to read, eliminating a lot of code you’d otherwise have to write.

Every call to `MarshalBinary`2 will return 516 bytes per call at most by relying on the `io.CopyN` function 3 and the `BlockSize` constant. Since you want `MarshalBinary` to modify the state, you need to use a pointer receiver. The intention is that the server can keep calling this method to get sequential blocks of data, each with an increasing block number, from the `io.Reader` until it exhausts the reader. Just like the client, the server needs to monitor the packet size returned by this method. When the packet size is less than 516 bytes, the server knows it received the last packet and should stop calling `MarshalBinary`. You’ll see this in action in the server code later in this chapter.

You may have recognized the potential for an integer overflow of the 16-bit, unsigned block number. If you send a payload larger than about 33.5MB (65,535 × 512 bytes), the block number will overflow back to 0. Your server will happily continue sending data packets, but the client may not be as graceful handling the overflow. You should consider mitigating overflow risks by limiting the file size the TFTP server will support so as not to trigger the overflow, recognizing that an overflow can occur and determining whether it is acceptable to the client, or using a different protocol altogether.

[Listing 6-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-5) finishes up the data type implementation with its binary unmarshaling method. This method follows the code in [Listing 6-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-4).

```
--snip--

func (d *Data) UnmarshalBinary(p []byte) error {
    1 if l := len(p); l < 4 || l > DatagramSize {
        return errors.New("invalid DATA")
    }

    var opcode

    err := 2binary.Read(bytes.NewReader(p[:2]), binary.BigEndian, &opcode)
    if err != nil || opcode != OpData {
        return errors.New("invalid DATA")
    }

    err = 3binary.Read(bytes.NewReader(p[2:4]), binary.BigEndian, &d.Block)
    if err != nil {
        return errors.New("invalid DATA")
    }

    d.Payload = 4bytes.NewBuffer(p[4:])

    return nil
}
```

Listing 6-5: Data type implementation (_types.go_ continued)

To unmarshal data, you perform an initial sanity check 1 to determine whether the packet size is within the expected bounds, making it worth reading the remaining bytes. You then read the operation code 2 and check it, then the block number 3. Finally, you stuff the remaining bytes into a new buffer 4 and assign it to the `Payload` field.

The client uses the block number to send a corresponding acknowledgment to the server and to properly order this block of data among the other received blocks of data.

### Acknowledgments

_Acknowledgment packets_ are only 4 bytes long, as shown in [Figure 6-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#figure6-4).

![f06004](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c06/f06004.png)

Figure 6-4: Acknowledgment packet structure

As in the other types, the first 2 bytes represent the operation code. The final 2 bytes contain the number of the acknowledged block.

[Listing 6-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-6) shows the entire implementation of the acknowledgment type, which follows [Listing 6-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-5)’s code.

```
--snip--

1 type Ack uint16

func (a Ack) MarshalBinary() ([]byte, error) {
    cap := 2 + 2 // operation code + block number

    b := new(bytes.Buffer)
    b.Grow(cap)

    err := binary.Write(b, binary.BigEndian, OpAck) // write operation code
    if err != nil {
        return nil, err
    }

    err = binary.Write(b, binary.BigEndian, a) // write block number
    if err != nil {
        return nil, err
    }

    return b.Bytes(), nil
}

func (a *Ack) UnmarshalBinary(p []byte) error {
    var code OpCode

    r := bytes.NewReader(p)

    err := binary.Read(r, binary.BigEndian, &code) // read operation code
    if err != nil {
        return err
    }

    if code != OpAck {
        return errors.New("invalid ACK")
    }

    return binary.Read(r, binary.BigEndian, a) // read block number
}
```

Listing 6-6: Acknowledgment type implementation (_types.go_ continued)

You represent an acknowledgment packet by using a 16-bit, unsigned integer 1. This integer is set to the acknowledged block number. The `MarshalBinary` and `UnmarshalBinary` methods should look familiar by this point. They handle marshaling the operation code and block number to a byte slice and populating an `Ack` object from bytes read from the network, respectively.

### Handling Errors

In TFTP, clients and servers convey errors by using an _error packet_, illustrated in [Figure 6-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#figure6-5).

![f06005](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c06/f06005.png)

Figure 6-5: Error packet structure

Error packets consist of a 2-byte operation code, a 2-byte error code, an error message of variable length, and a terminating null byte.

[Listing 6-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-7) details the error type and its binary marshal method, a continuation of [Listing 6-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-6).

```
--snip--

1 type Err struct {
    Error   ErrCode
    Message string
}

func (e Err) MarshalBinary() ([]byte, error) {
    // operation code + error code + message + 0 byte
    cap := 2 + 2 + len(e.Message) + 1

    b := new(bytes.Buffer)
    b.Grow(cap)

    err := binary.Write(b, binary.BigEndian, OpErr) // write operation code
    if err != nil {
        return nil, err
    }

    err = binary.Write(b, binary.BigEndian, e.Error) // write error code
    if err != nil {
        return nil, err
    }

    _, err = b.WriteString(e.Message) // write message
    if err != nil {
        return nil, err
    }

    err = b.WriteByte(0) // write 0 byte
    if err != nil {
        return nil, err
    }

    return b.Bytes(), nil
}
```

Listing 6-7: Error type used for conveying errors between the client and server (_types.go_ continued)

Like the read request, the error type 1 contains the minimum data required to craft an error packet: an error code and an error message. The `MarshalBinary` method populates a bytes buffer following the byte sequence detailed in [Figure 6-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#figure6-5).

[Listing 6-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-8) completes the error type implementation with its binary unmarshaler method. This code is appended to the code in [Listing 6-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-7).

```
--snip--

func (e *Err) UnmarshalBinary(p []byte) error {
    r := bytes.NewBuffer(p)

    var code OpCode

    err := 1binary.Read(r, binary.BigEndian, &code) // read operation code
    if err != nil {
        return err
    }

    if code != OpErr {
        return errors.New("invalid ERROR")
    }

    err = 2binary.Read(r, binary.BigEndian, &e.Error) // read error message
    if err != nil {
        return err
    }

    e.Message, err = 3r.ReadString(0)
    e.Message = 4strings.TrimRight(e.Message, "\x00") // remove the 0-byte

    return err
}
```

Listing 6-8: Error type’s binary unmarshaler implementation (_types.go_ continued)

The `UnmarshalBinary` method is quite simple in that it reads and verifies the operation code 1, consumes the error code 2 and error message 3, and strips the trailing null byte 4.

## The TFTP Server

Now you’ll write the server code, which will use the types you defined to interact with TFTP clients.

### Writing the Server Code

[Listing 6-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-9) describes your server type and the methods that allow it to serve incoming requests. The fact that your packet types implement the `encoding.BinaryMarshaler` and `encoding.BinaryUnmarshaler` interfaces means that your server code can act as a conduit between the network interface and these types, leading to simpler code. All your server must concern itself with is transferring byte slices between your types and the network connection. The logic in the type interfaces takes care of the rest.

```
package tftp

import (
    "bytes"
    "errors"
    "fmt"
    "log"
    "net"
    "time"
)

type Server struct {
    1 Payload []byte // the payload served for all read requests
    2 Retries uint8 // the number of times to retry a failed transmission
    3 Timeout time.Duration // the duration to wait for an acknowledgment
}

func (s Server) ListenAndServe(addr string) error {
    conn, err := net.ListenPacket("udp", addr)
    if err != nil {
        return err
    }
    defer func() { _ = conn.Close() }()

    log.Printf("Listening on %s ...\n", conn.LocalAddr())

    return s.Serve(conn)
}

func (s *Server) 4Serve(conn net.PacketConn) error {
    if conn == nil {
        return errors.New("nil connection")
    }

    if s.Payload == nil {
        return errors.New("payload is required")
    }

    if s.Retries == 0 {
        s.Retries = 10
    }

    if s.Timeout == 0 {
        s.Timeout = 6 * time.Second
    }

    var rrq ReadReq

    for {
        buf := make([]byte, DatagramSize)

        _, addr, err := conn.ReadFrom(buf)
        if err != nil {
            return err
        }

        err = 5rrq.UnmarshalBinary(buf)
        if err != nil {
            log.Printf("[%s] bad request: %v", addr, err)
            continue
        }

        6 go s.handle(addr.String(), rrq)
    }
}
```

Listing 6-9: Server type implementation (_server.go_)

Our server maintains a payload 1 that it returns for every read request, a record of the number of times to attempt packet delivery 2, and a time-out duration between each attempt 3. The server’s `Serve` method accepts a `net.PacketConn` and uses it to read incoming requests 4. Closing the network connection will cause the method to return.

The server reads up to 516 bytes from its connection and attempts to unmarshal the bytes to a `ReadReq` object 5. Since your server is read-only, it’s interested only in servicing read requests. If the data read from the connection is a read request, the server passes it along to a handler method in a goroutine 6. We’ll define that next.

### Handling Read Requests

The handler ([Listing 6-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-10)) accepts read requests from the client and replies with the server’s payload. It uses the features you built into your TFTP server’s type system to improve the reliability of the data transfer over UDP. The handler sends one data packet and waits for an acknowledgment from the client before sending another data packet. It also attempts to retransmit the current data packet when it fails to receive a timely reply from the client.

```
--snip--

1 func (s Server) handle(clientAddr string, rrq ReadReq) {
    log.Printf("[%s] requested file: %s", clientAddr, rrq.Filename)

    conn, err := 2net.Dial("udp", clientAddr)
    if err != nil {
        log.Printf("[%s] dial: %v", clientAddr, err)
        return
    }
    defer func() { _ = conn.Close() }()

    var (
        ackPkt  Ack
        errPkt  Err
        dataPkt = 3Data{Payload: bytes.NewReader(s.Payload)}
        buf     = make([]byte, DatagramSize)
    )

NEXTPACKET:
    4 for n := DatagramSize; n == DatagramSize; {
        data, err := dataPkt.MarshalBinary()
        if err != nil {
            log.Printf("[%s] preparing data packet: %v", clientAddr, err)
            return
        }

    RETRY:
        5 for i := s.Retries; i > 0; i-- {
            6 n, err = conn.Write(data) // send the data packet
            if err != nil {
                log.Printf("[%s] write: %v", clientAddr, err)
                return
            }

            // wait for the client's ACK packet
            _ = conn.SetReadDeadline(time.Now().Add(s.Timeout))

            _, err = conn.Read(buf)
            if err != nil {
                if nErr, ok := err.(net.Error); ok && nErr.Timeout() {
                    continue RETRY
                }

                log.Printf("[%s] waiting for ACK: %v", clientAddr, err)
                return
            }

            switch {
            case ackPkt.UnmarshalBinary(buf) == nil:
                7 if uint16(ackPkt) == dataPkt.Block {
                    // received ACK; send next data packet
                    continue NEXTPACKET
                }
            case errPkt.UnmarshalBinary(buf) == nil:
                log.Printf("[%s] received error: %v",
                    clientAddr, errPkt.Message)
                return
            default:
                log.Printf("[%s] bad packet", clientAddr)
            }
        }

        log.Printf("[%s] exhausted retries", clientAddr)
        return
    }

    log.Printf("[%s] sent %d blocks", clientAddr, dataPkt.Block)
}
```

Listing 6-10: Handling read requests (_server.go_ continued)

This handler is a method 1 on your `Server` type that accepts a client address and a read request. It’s defined as a method because you need access to the `Server`’s fields. You then initiate a connection with the client by using `net.Dial`2. The resulting UDP connection object created with `net.Dial`, if you remember, will read only packets from the client, freeing you from having to check the sender address on every `Read` call. You prepare a data object 3 by using the server’s payload, then enter a `for` loop to send each data packet 4. This `for` loop will continue looping as long as the data packet size is equal to 516 bytes.

After marshaling the data object to a byte slice, you enter the `for` loop 5 meant to resend the data packet until you either exhaust the number of retries or successfully deliver the data packet. Writing the data packet to the network connection 6 updates the `n` loop variable with the number of bytes sent. If this value is 516 bytes, you iterate again when control passes back to the `for` loop 4 labeled `NEXTPACKET`. If this value is less than 516 bytes, you break out of the loop.

Before you determine whether the transfer is complete, you must first verify that the client successfully received the last data packet. You read bytes from the client and attempt to unmarshal them into an `Ack` object or `Err` object. If you successfully unmarshal them into an `Err` object, you know the client returned an error, so you should log that fact and return early. An early return from this handler means the transmission terminated short of sending the entire payload. For our purposes, this is unrecoverable. The client would need to re-request the file to initiate another transfer.

If you successfully unmarshal the bytes into an `Ack` object, you can then check the object’s `Block` value to determine whether it matches the block number of the current data packet 7. If so, you iterate around the `for` loop 4 and send the next packet. If not, you iterate around the inner `for` loop 5 and resend the current data packet.

### Starting the Server

To start your TFTP server, you need to give the server two things: a file (its payload) and an address on which to listen for incoming requests ([Listing 6-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-11)).

```
package main

import (
    "flag"
    "io/ioutil"
    "log"

    "github.com/awoodbeck/gnp/ch06/tftp"
)

var (
    address = flag.String("a", "127.0.0.1:69", "listen address")
    payload = flag.String("p", "payload.svg", "file to serve to clients")
)

func main() {
    flag.Parse()

    p, err := 1ioutil.ReadFile(*payload)
    if err != nil {
        log.Fatal(err)
    }

    s := 2tftp.Server{Payload: p}
    3 log.Fatal(s.ListenAndServe(*address))
}
```

Listing 6-11: Command line TFTP server implementation (_tftp.go_)

Once you’ve read the file 1 that your TFTP server will serve into a byte slice, you instantiate the server and assign the byte slice to the server’s `Payload` field 2. The last step is calling its `ListenAndServe` method to establish the UDP connection on which it will listen for requests. The `ListenAndServe` method 3 calls the server’s `Serve` method for you, which listens on the network connection for incoming requests. The server will continue to run until you terminate it with a CTRL-C on the command line.

## Downloading Files over UDP

Now let’s try to download a file from the server you just wrote. First, you need to make sure you have a TFTP client installed. Windows has a native TFTP client that you can install through the Programs and Features section of the Control Panel by clicking the Turn Windows features on or off link. Select the **TFTP Client** checkbox and click the **OK** button to install it. Most Linux distributions have a TFTP client available for installation through the distribution’s package manager, and macOS has a TFTP client installed by default.

This example uses Windows 10. Start by running the TFTP server by running the code in [Listing 6-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-11) in a terminal:

```
Microsoft Windows [Version 10.0.18362.449]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\User\gnp\ch06\tftp\tftp>go run tftp.go
2006/01/02 15:04:05 Listening on 127.0.0.1:69 ...
```

The server should bind to UDP port 69 on 127.0.0.1 by default. Port 69 is a privileged port, and you may need root permissions on Linux. You may need to first build the binary by using `go build tftp.go` and then run the resulting binary by using the `sudo` command to bind to port 69: `sudo ./tftp`. The TFTP server should log a message to standard output that indicates it’s listening.

From a separate terminal, execute the TFTP client, making sure to pass the `-i` argument to tell the server you wish to initiate a binary (octet) transfer. Remember, your TFTP server doesn’t care what the source filename is because it returns the same payload regardless of the requested filename. You’ll use _test.svg_ in this example:

```
Microsoft Windows [Version 10.0.18362.449]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\User>tftp -i 127.0.0.1 GET test.svg
Transfer successful: 75352 bytes in 1 second(s), 75352 bytes/s
```

Almost immediately upon pressing ENTER, the client should report the transfer was successful. The TFTP server’s terminal should show its progress as well:

```
Microsoft Windows [Version 10.0.18362.449]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\User\gnp\ch06\tftp\tftp>go run tftp.go
2006/01/02 15:04:05 Listening on 127.0.0.1:69 ...
2006/01/02 15:04:05 [127.0.0.1:57944] requested file: test.svg
2006/01/02 15:04:05 [127.0.0.1:57944] sent 148 blocks
```

You can confirm that the downloaded file is the same as the payload provided to the TFTP server by comparing _test.svg_’s checksum with the checksum of the server’s _payload.svg_. A _checksum_ is a calculated value used to verify the integrity of a file. If two files are identical, they will have equivalent checksums. Linux and macOS both have various command line utilities for generating checksums, but you’ll use a pure Go implementation, as shown in [Listing 6-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-12).

```
package main

import (
    "crypto/sha512"
    "flag"
    "fmt"
    "io/ioutil"
    "os"
)

func init() {
    flag.Usage = func() {
        fmt.Printf("Usage: %s file...\n", os.Args[0])
        flag.PrintDefaults()
    }
}

func main() {
    flag.Parse()
    for _, file := range 1flag.Args() {
        fmt.Printf("%s  %s\n", checksum(file), file)
    }
}

func checksum(file string) string {
    b, err := 2ioutil.ReadFile(file)
    if err != nil {
        return err.Error()
    }

    return fmt.Sprintf("%x", 3sha512.Sum512_256(b))
}
```

Listing 6-12: Generating SHA512/256 checksums for given command line arguments (_sha512-256sum.go_)

This bit of code will accept one or more file paths as command line arguments 1 and generate SHA512/256 checksums 3 from their contents 2.

A SHA512/256 checksum is a SHA512 checksum truncated to 256 bits. Calculating SHA512 on a 64-bit machine is faster than calculating a SHA256 checksum, because the SHA512 computation uses 64-bit words, whereas SHA256 uses 32-bit words. By truncating SHA512 to 256 bits, you eliminate a length extension hashing attack that SHA512 is vulnerable to by itself. SHA512/256 isn’t necessary here since you’re not using the checksum beyond verifying the integrity of a file, but you should be familiar with it, and it should be on your short list of hashing algorithms.

You can use the code from [Listing 6-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-12) in [Listing 6-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#listing6-13) to verify that the file you downloaded (_test.svg_) is identical to the file the server sent (_payload.svg_). You’ll continue to use Windows as your target platform, but the code will work on Linux and macOS without changes:

```
Microsoft Windows [Version 10.0.18362.449]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\User\dev\gnp\ch06>go build sha512-256sum\sha512-256sum.go

C:\Users\User\dev\gnp\ch06>sha512-256sum \Users\User\test.svg

\Users\User\test.svg =>
1 3f5794c522e83b827054183658ce63cb701dc49f4e59335f08b5c79c56873969

C:\Users\User\dev\gnp\ch06>sha512-256sum tftp\tftp\payload.svg

tftp\tftp\payload.svg =>
2 3f5794c522e83b827054183658ce63cb701dc49f4e59335f08b5c79c56873969
```

Listing 6-13: Generating SHA512/256 checksums for _test.svg_ and _payload.svg_

As you can see, the _test.svg_ checksum 1 is equal to the _payload.svg_ checksum 2.

In this case, the _test.svg_ file is an image of a gopher from Egon Elbre’s excellent _gophers_ repository on GitHub ([https://github.com/egonelbre/gophers/](https://github.com/egonelbre/gophers/)). If you opened the file in a web browser, you’d see the image in [Figure 6-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c06.xhtml#figure6-6).

Although you transferred the payload over localhost and don’t expect data loss or corruption, the client and server still acknowledged every data packet, ensuring the proper delivery of the payload.

![f06006](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c06/f06006.png)

Figure 6-6: Downloaded payload from the TFTP server

## What You’ve Learned

UDP can be made reliable at the application level, as evident by the Trivial File Transfer Protocol. TFTP uses a combination of data packet sequence numbers and acknowledgments to ensure that the client and server agree on all transferred data, redelivering packets as necessary.

Liberal use of Go’s binary marshaling and unmarshaling interfaces allow you to implement types that make communication using TFTP straightforward. Each TFTP type meant for delivery over UDP implements the `encoding.BinaryMarshaler` interface to marshal its data into a format suitable for writing to a network connection. Likewise, each type you expect to read from a network connection should implement the `encoding.BinaryUnmarshaler` interface. Successfully unmarshaling binary data to your custom type allows you to determine what binary data was received and that it is correct.