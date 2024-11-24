---
id: 01JCH9M36H0FKW8464BY85DZQ2
title: Chapter 12 - Data Serialization
modified: 2024-11-12T17:59:12-05:00
---
# 12  
DATA SERIALIZATION

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

A sizable portion of our work as developers involves integrating our network services with existing services, including legacy or third-party ones implemented in languages other than Go. These services must communicate by exchanging bytes of data in a way that is meaningful to both the sender and receiver, despite the different programming languages they’re using. To do this, the sender converts data into bytes using a standard format and transfers the bytes over the network to the receiver. If the receiver understands the format used by the sender, it can convert the bytes back into structured data. This process of transforming structured data into successive bytes is known as _data serialization_.

Services can use data serialization to convert structured data to a series of bytes suitable for transport over a network or for persistence to storage. No matter whether the serialized data came from a network or disk, any code that understands the serialization format should be able to deserialize it to reconstruct a clone of the original object.

While writing this chapter, I initially struggled to explain the concept of data serialization. Then I realized we serialize data as we speak. Electrical impulses in my brain form words. My brain instructs my voice box to _serialize_ these words into sound waves, which travel through the air and reach your ear. The sound waves vibrate your eardrum, which in turn transmits the vibrations to your inner ear. Hair-like structures in your inner ear deserialize these vibrations into electrical signals that your brain interprets as the original words I formed in my brain. We’ve just communicated using the serialization format of English, since it’s a format we both understand.

You already have a bit of experience serializing data in your Go code too. The type-length-value binary encoding you learned in Chapter 4 and the JavaScript Object Notation (JSON) you posted over HTTP in Chapter 8 are examples of translating objects into well-known data serialization formats. We also PEM-encoded the certificates and private keys in Chapter 11 for persistence to disk.

This chapter will take a deeper dive into using data serialization for the purposes of storing data or sending between systems, which can make it accessible to services written in other languages. We could cover many data serialization formats, but we’ll focus on the three that get the most use in Go network programming: JSON, protocol buffers, and Gob. We’ll also spend some time on how to execute code on remote machines using a framework named gRPC. By the end of this chapter, you will know how to serialize data for storage or transmission and decode that data into meaningful data structures. You should be able to use techniques in this chapter to build services that can exchange complex data over a network or write code to communicate with existing network services.

## Serializing Objects

Objects or structured data cannot traverse a network connection as is. In other words, you cannot pass in an object to `net.Conn`’s `Write` method, since it accepts only a byte slice. Therefore, you need to serialize the object to a byte slice, which you can then pass to the `Write` method. Thankfully, Go makes this easy.

Go’s standard library includes excellent support for popular data serialization formats in its `encoding` package. You’ve already used `encoding/binary` to serialize numbers into byte sequences, `encoding/json` to serialize objects into JSON for submission over HTTP, and `encoding/pem` to serialize TLS certificates and private keys to files. (Anytime you encounter a function or method whose name includes _encode_ or _marshal_, it likely serializes data. Likewise, _decode_ and _unmarshal_ are synonymous with deserializing data.)

This section will build an application that serializes data into three binary encoding formats: JSON, protocol buffers, and Gob. Since I often have trouble keeping track of housework, the application will document chores to do around the house. The application’s state will persist between executions because you don’t want it to forget the chores when it exits. You’ll serialize each task to a file and use your app to update it as needed.

To keep this program simple, you need a description of the chore and a way to determine whether it’s complete. [Listing 12-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-1) defines a new package with a type that represents a household chore.

```
package housework

type Chore struct {
    Complete    bool
    Description string
}
```

Listing 12-1: A type to represent a household chore (_housework/housework.go_)

Go’s JSON and Gob encoding packages can serialize exported struct fields only, so you define `Chore` as a struct, making sure to export its fields. The `Complete` field’s value will be `true` if you’ve completed the chore. The `Description` field is the human-readable description of the chore.

You could use struct tags to instruct the encoders on how to treat each field, if necessary. For example, you could place the struct tag `` `json:"-"` `` on the `Complete` field to tell Go’s JSON encoder to ignore this field instead of encoding it. Since you’re perfectly happy to pass along all field values, you omit struct tags.

Once you’ve defined a chore structure, you can use it in an application that tracks chores on the command line. This application should show a list of chores and their status, allow you to add chores to the list, and mark chores as complete. [Listing 12-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-2) includes the initial code for this housework application, including its command line usage details.

```
package main

import (
    "flag"
    "fmt"
    "log"
    "os"
    "path/filepath"
    "strconv"
    "strings"

    "github.com/awoodbeck/gnp/ch12/housework"
 1 storage "github.com/awoodbeck/gnp/ch12/json"
    // storage "github.com/awoodbeck/gnp/ch12/gob"
    // storage "github.com/awoodbeck/gnp/ch12/protobuf"
)

var dataFile string

func init() {
    flag.StringVar(&dataFile, "file", "housework.db", "data file")

    flag.Usage = func() {
        fmt.Fprintf(flag.CommandLine.Output(),
         2 `Usage: %s [flags] [add chore, ...|complete #]
    add         add comma-separated chores
    complete    complete designated chore

Flags:
`, filepath.Base(os.Args[0]))
        flag.PrintDefaults()
    }
}
```

Listing 12-2: Initial housework application code (_cmd/housework.go_)

This bit of code sets up the command line arguments and their usage 2: you can specify the `add` argument, followed by a comma-separated list of chores to add to the list, or you can pass the argument `complete` and a chore number to mark the chore as complete. In the absence of command line options, the app will display the current list of chores.

Since the ultimate purpose of this application is to demonstrate data serialization, you’ll use multiple serialization formats to store the data. This should show you how easy it is to switch between various formats. To prepare for that, you include `import` statements for those formats 1. This will make it easier for you to switch between the formats later.

Let’s write the code to load chores from storage (see [Listing 12-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-3)).

```
--snip--

func load() ([]*housework.Chore, error) {
    if _, err := os.Stat(dataFile); 1os.IsNotExist(err) {
        return make([]*housework.Chore, 0), nil
    }

    df, err := 2os.Open(dataFile)
    if err != nil {
        return nil, err
    }
    defer func() {
        if err := df.Close(); err != nil {
            fmt.Printf("closing data file: %v", err)
        }
    }()

    return 3storage.Load(df)
}
```

Listing 12-3: Deserializing chores from a file (_cmd/housework.go_)

This function returns a slice of pointers to `housework.Chore` structs from [Listing 12-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-1). If the data file does not exist 1, you exit early, returning an empty slice. This default case will occur when you first run the application.

If the app finds a data file, you open it 2 and pass it along to the storage’s `Load` function 3, which expects an `io.Reader`. You used the same pattern of accepting an interface and returning a concrete type in previous chapters.

[Listing 12-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-4) defines a function that flushes the chores in memory to your storage for persistence.

```
--snip--

func flush(chores []*housework.Chore) error {
    df, err := 1os.Create(dataFile)
    if err != nil {
        return err
    }
    defer func() {
        if err := df.Close(); err != nil {
            fmt.Printf("closing data file: %v", err)
        }
    }()

    return 2storage.Flush(df, chores)
}
```

Listing 12-4: Flushing chores to the storage (_cmd/housework.go_)

Here, you create a new file or truncate the existing file 1 and pass the file pointer and slice of chores to the storage’s `Flush` function 2. This function accepts an `io.Writer` and your slice. There’s certainly room for improvement in the way you handle the existing serialized file. But for the purposes of demonstration, this will suffice.

You need a way to display the chores on the command line. [Listing 12-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-5) adds such a function to your application.

```
--snip--

func list() error {
    chores, err := 1load()
    if err != nil {
        return err
    }

    if len(chores) == 0 {
        fmt.Println("You're all caught up!")
        return nil
    }

    fmt.Println("#\t[X]\tDescription")
    for i, chore := range chores {
        c := " "
        if chore.Complete {
            c = "X"
        }
        fmt.Printf("%d\t[%s]\t%s\n", i+1, c, chore.Description)
    }

    return nil
}
```

Listing 12-5: Printing the list of chores to standard output (_cmd/housework.go_)

First, you load the list of chores from storage 1. If there are no chores in your list, you simply print as much to standard output. Otherwise, you print a header and the list of chores, which looks something like this (see [Listing 12-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-6)).

```
#       [X]     Description
1       [ ]     Mop floors
2       [ ]     Clean dishes
3       [ ]     Mow the lawn
```

Listing 12-6: Example output of the list function with three chores in the list

The first column represents the chore number. You can reference this number to mark the chore complete, which will add an X between its brackets in the second column. The third column describes the chore.

[Listing 12-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-7) implements the `add` function, which allows you to add chores to the list.

```
--snip--

func add(s string) error {
    chores, err := 1load()
    if err != nil {
        return err
    }

    for _, chore := range 2strings.Split(s, ",") {
        if desc := strings.TrimSpace(chore); desc != "" {
            chores = append(chores, &housework.Chore{
                Description: desc,
            })
        }
    }

    return 3flush(chores)
}
```

Listing 12-7: Adding chores to the list (_cmd/housework.go_)

Unlike a long-running service, this application’s lifetime starts when you execute it on the command line and ends when it completes whatever task you ask it to perform. Therefore, because you want your list of chores to persist between executions of the application, you need to store the chore state on disk. In other words, you retrieve the chores from storage 1, modify them, and flush the changes to storage 3. The changes persist until the next time you run the app.

You want the option to add more than one chore at a time, so you split the incoming chore description by commas 2 and append each chore to the slice. Granted, this keeps you from using commas in individual chore descriptions, so the members of your household will have to keep their requests short (which isn’t all bad, in my opinion). As an exercise, figure out a way around this limitation. One approach may be to use a different delimiter, but keep in mind, whatever you choose as a delimiter may have significance on the command line. Another approach may be to add support for quoted strings containing commas.

The last piece of this puzzle is my favorite part about working on chores: marking them as complete (see [Listing 12-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-8)).

```
--snip--

func complete(s string) error {
    i, err := strconv.Atoi(s)
    if err != nil {
        return err
    }

    chores, err := load()
    if err != nil {
        return err
    }

    if i < 1 || i > len(chores) {
        return fmt.Errorf("chore %d not found", i)
    }

 1 chores[i-1].Complete = true

    return flush(chores)
}
```

Listing 12-8: Marking a chore as complete (_cmd/housework.go_)

The `complete` function accepts the command line argument representing the chore you want to complete and converts it to an integer. I find I’m more efficient if I perform tasks one by one, so I’ll have you mark only one complete at a time. You then load the chores from storage and make sure the integer is within range. If so, you mark the chore complete. Since you’re numbering chores starting with 1 when displaying the list, you need to account for placement in the slice by subtracting 1 1. Finally, you flush the chores to storage.

Now, let’s tie everything together by implementing the app’s `main` function in [Listing 12-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-9).

```
--snip--

func main() {
    flag.Parse()

    var err error

    switch strings.ToLower(flag.Arg(0)) {
    case "add":
        err = add(strings.Join(flag.Args()[1:], " "))
    case "complete":
        err = complete(flag.Arg(1))
    }

    if err != nil {
        log.Fatal(err)
    }

    err = list()
    if err != nil {
        log.Fatal(err)
    }
}
```

Listing 12-9: The main logic of the housework application (_cmd/housework.go_)

You’ve put as much logic in the previous functions as possible, so this `main` function is quite minimal. You check the first argument to determine whether it’s an expected subcommand. If so, you call the appropriate function. You call the `list` function if `err` is still nil after accounting for the optional subcommand and its arguments.

All that’s left to do now is implement the storage `Load` and `Flush` functions for JSON, Gob, and protocol buffers.

### JSON

JSON is a common, human-readable, text-based data serialization format that uses key-value pairs and arrays to represent complex data structures. Most contemporary languages offer official library support for JSON, which is one reason it’s the customary encoding format for RESTful APIs.

JSON’s types include strings, Booleans, numbers, arrays, key-value objects, and nil values specified by the keyword _null_. JSON numbers do not differentiate between floating-points and integers. You can read more about Go’s JSON implementation at [https://blog.golang.org/json](https://blog.golang.org/json).

Let’s look at what the contents of the _housework.db_ file would be if we JSON-encoded the chores from [Listing 12-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-6). I’ve formatted the JSON for easier reading in [Listing 12-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-10), though you could use the encoder’s `SetIndent` method to do it for you.

```
1[
2 {
    "Complete": false,
    "Description": "Mop floors"
  },
  {
    "Complete": false,
    "Description": "Clean dishes"
  },
  {
    "Complete": false,
    "Description": "Mow the lawn"
  }
]
```

Listing 12-10: Formatted contents of the _housework.db_ file after serializing the chores to JSON

As you can see, the JSON is an array of objects 1, and each object 2 includes `Complete` and `Description` fields and corresponding values.

[Listing 12-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-11) details the JSON storage implementation using Go’s `encoding/json` package.

```
package json

import (
    "encoding/json"
    "io"

    "github.com/awoodbeck/gnp/ch12/housework"
)

func Load(r io.Reader) ([]*housework.Chore, error) {
    var chores []*housework.Chore

    return chores, 1json.NewDecoder(r).Decode(&chores)
}

func Flush(w io.Writer, chores []*housework.Chore) error {
    return 2json.NewEncoder(w).Encode(chores)
}
```

Listing 12-11: JSON storage implementation (_json/housework.go_)

The `Load` function passes the `io.Reader` to the `json.NewDecoder` function 1 and returns a decoder. You then call the decoder’s `Decode` method, passing it a pointer to the `chores` slice. The decoder reads JSON from the `io.Reader`, deserializes it, and populates the `chores` slice.

The `Flush` function accepts an `io.Writer` and a `chores` slice. It then passes the `io.Writer` to the `json.NewEncoder` function 2, which returns an encoder. You pass the `chores` slice to the encoder’s `Encode` function, which serializes the `chores` slice into JSON and writes it to the `io.Writer`.

Now that you’ve implemented a JSON package that can serve as storage for your application, let’s try it out in [Listing 12-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-12).

```
$ go run cmd/housework.go
You're all caught up!
$ go run cmd/housework.go add Mop floors, Clean dishes, Mow the lawn
#       [X]     Description
1       [ ]     Mop floors
2       [ ]     Clean dishes
3       [ ]     Mow the lawn
$ go run cmd/housework.go complete 2
#       [X]     Description
1       [ ]     Mop floors
2       [X]     Clean dishes
3       [ ]     Mow the lawn
$ cat housework.db
[{"Complete":false,"Description":"Mop floors"},
{"Complete":true,"Description":"Clean dishes"},
{"Complete":false,"Description":"Mow the lawn"}]
```

Listing 12-12: Testing the housework application with JSON storage on the command line

Your first execution of the app lets you know you have nothing in your list of chores. You then add three comma-separated chores and complete the second one. Looks good. Notice also that the _housework.db_ file contains readable JSON (to see this on Windows, use the `type` command instead of `cat`). Let’s modify this application to use a binary encoding format native to Go.

### Gob

_Gob_, as in “gobs of binary data,” is a binary serialization format native to Go. Engineers on the Go team developed Gob to combine the efficiency of protocol buffers, arguably the most popular binary serialization format, with JSON’s ease of use. For example, protocol buffers don’t let us simply instantiate a new encoder and throw data structures at it, as you did in the JSON example in [Listing 12-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-11). On the other hand, Gob functions much the same way as the JSON encoder, in that Gob can intelligently infer an object’s structure and serialize it.

If you’re interested in exploring the motivation and finer points of Gob, give Rob Pike’s “Gobs of Data” blog post a read ([https://blog.golang.org/gob](https://blog.golang.org/gob)). In the meantime, let’s implement our storage backend in Gob (see [Listing 12-13](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-13)).

```
package gob

import (
    "encoding/gob"
    "io"

    "github.com/awoodbeck/gnp/ch12/housework"
)

func Load(r io.Reader) ([]*housework.Chore, error) {
    var chores []*housework.Chore

    return chores, gob.NewDecoder(r).Decode(&chores)
}

func Flush(w io.Writer, chores []*housework.Chore) error {
    return gob.NewEncoder(w).Encode(chores)
}
```

Listing 12-13: Gob storage implementation (_gob/housework.go_)

If you’re looking at this code and observing that it replaces all occurrences of `json` from [Listing 12-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-11) with `gob`, you’re not wrong. In Go, Gob is as easy to use as JSON, since it infers what it needs to encode from the object itself. You’ll see how this differs from protocol buffers in the next section.

All that’s left to do is swap out the JSON storage implementation for the Gob one by modifying the imports from [Listing 12-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-2) ([Listing 12-14](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-14)).

```
--snip--
    "github.com/awoodbeck/gnp/ch12/housework"
 1 // storage "github.com/awoodbeck/gnp/ch12/json"
 2 storage "github.com/awoodbeck/gnp/ch12/gob"
    // storage "github.com/awoodbeck/gnp/ch12/protobuf"
--snip--
```

Listing 12-14: Swapping the JSON storage package for the Gob storage package (_cmd/housework.go_)

Comment out the JSON storage package import 1 in [Listing 12-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-2) and uncomment the Gob storage package one 2.

Since your current _housework.db_ file contains JSON, it isn’t compatible with Gob. Therefore, the housework application will throw an error when attempting to decode it using Gob. Remove the _housework.db_ file and retest the application (see [Listing 12-15](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-15)).

```
$ rm housework.db
$ go run cmd/housework.go
You're all caught up!
$ go run cmd/housework.go add Mop floors, Clean dishes, Mow the lawn
#       [X]     Description
1       [ ]     Mop floors
2       [ ]     Clean dishes
3       [ ]     Mow the lawn
$ go run cmd/housework.go complete 2
#       [X]     Description
1       [ ]     Mop floors
2       [X]     Clean dishes
3       [ ]     Mow the lawn
$ hexdump -c housework.db
0000000  \r 377 203 002 001 002 377 204  \0 001 377 202  \0  \0   ) 377
0000010 201 003 001 002 377 202  \0 001 002 001  \b   C   o   m   p   l
0000020   e   t   e 001 002  \0 001  \v   D   e   s   c   r   i   p   t
0000030   i   o   n 001  \f  \0  \0  \0   1 377 204  \0 003 002  \n   M
0000040   o   p       f   l   o   o   r   s  \0 001 001 001  \f   C   l
0000050   e   a   n       d   i   s   h   e   s  \0 002  \f   M   o   w
0000060       t   h   e       l   a   w   n  \0                        
000006a
```

Listing 12-15: Testing the housework application with Gob storage on the command line

Everything still works as expected. Using the `hexdump` tool, you can see that the _housework.db_ file now includes binary data. It’s certainly not human-readable as JSON was in [Listing 12-12](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-12), but Go happily deserializes the Gob-encoded data, even though it’s harder for us to make sense of it. (My Windows friends can find a `hexdump` binary at _https://www.di-mgt.com.au/hexdump-for-windows.html_, though you’ll have to use the `-C` flag to get the same effect.)

If you are communicating with other Go services that support Gob, I recommend you use Gob over JSON. Go’s `encoding/gob` is more performant than `encoding/json`. Gob encoding is also more succinct, in that Gob uses less data to represent objects than JSON does. This can make a difference when storing or transmitting serialized data

Now that you have a taste for serializing data using `encoding/json` and `encoding/gob`, let’s add protocol buffer support to your storage backend.

### Protocol Buffers

Like Gob, _protocol buffers_ use binary encoding to store or exchange information across various platforms. It’s faster and more succinct than Go’s JSON encoding. Unlike Gob and like JSON, protocol buffers are language neutral and enjoy wide support among popular programming languages. This makes them ideally suited for using in Go-based services that you hope to integrate with services written in other programming languages. This chapter assumes you’re using the _proto3_ version of the format.

Protocol buffers use a definition file, conventionally suffixed with _.proto_, to define messages. _Messages_ describe the structured data you want to serialize for storage or transmission. For example, a protocol buffer message representing the `Chore` type looks like the definition in [Listing 12-16](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-16).

```
message Chore {
  bool complete = 1;
  string description = 2;
}
```

Listing 12-16: Protocol buffer message definition representing a Chore type

You define a new message by using the `message` keyword, followed by the unique name of the message. Convention dictates you use Pascal case. (_Pascal casing_ is a style of code formatting in which you concatenate capitalized words: _ThisIsPascalCasing_.) You then add fields to the `Chore` message. Each field definition includes a type, a snake-cased name, and a field number unique to the message. (_Snake casing_ is like Pascal casing except the first word is lowercase: _thisIsSnakeCasing_.) The field’s type and number identify the field in the binary payload, so these must not change once used or you’ll break backward compatibility. However, it’s fine to add new messages and message fields to an existing _.proto_ file.

Speaking of backward compatibility, it’s a good practice to treat your protocol buffer definitions as you would an API. If third parties use your protocol buffer definition to communicate with your service, consider versioning your definition; this allows you to create new versions anytime you need to break backward compatibility. Your development can move forward with the latest version while clients continue to use the previous version until they’re ready to target the latest version. You’ll see one method of versioning the protocol buffer definitions later in this section.

You’ll have to compile the _.proto_ file to generate Go code from it. This code allows you to serialize and deserialize the messages defined in the _.proto_ file. Third parties that want to exchange messages with you can use the same _.proto_ file to generate code for their target programming language. The resulting code can exchange messages with you too. Therefore, before you can start using protocol buffers in earnest, you must install the protocol buffer compiler and its Go generation module. Your operating system’s package manager may allow you to easily install the protocol buffer compiler. For example, on Debian 10, run the following:

```
$ sudo apt install protobuf-compiler
```

On macOS with Homebrew, run this:

```
$ brew install protobuf
```

On Windows, download the latest protocol buffer compiler ZIP file from [https://github.com/protocolbuffers/protobuf/releases/](https://github.com/protocolbuffers/protobuf/releases/), extract it, and add its _bin_ subdirectory to your `PATH`. You should now be able to run the _protoc_ binary on your command line.

A simple `go get` will install the protocol buffer’s Go generator on your system. Make sure you have the resulting _protoc-gen-go_ binary in your `PATH` or _protoc_ won’t recognize the plug-in:

```
$ GO111MODULE=on go get -u github.com/golang/protobuf/protoc-gen-go
```

Now that you’ve installed the protocol buffer compiler and Go generation module, let’s create a new _.proto_ file for your housework application (see [Listing 12-17](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-17)). You’ll create this file in _housework/v1/housework.proto_. The `v1` in the path stands for _version 1_ and allows you to introduce future versions of this package.

```
1 syntax = "proto3";
2 package housework;

3 option go_package = "github.com/awoodbeck/gnp/ch12/housework/v1/housework";

message Chore {
  bool complete = 1;
  string description = 2;
}

message Chores {
4repeated Chore chores = 1;
}
```

Listing 12-17: Protocol buffer definition for your housework application (_housework/v1/housework.proto_)

First, you specify that you’re using the proto3 syntax 1 and that you want any generated code to have the package name `housework`2. Next, you add a `go_package` option 3 with the full import path of the generated module. Then you define the `Chore` message and a second message named `Chores` that represents a collection of `Chore` messages based on the `repeated` field type 4.

Now, let’s compile the _.proto_ file to Go code:

```
$ protoc 1--go_out=. 2--go_opt=paths=source_relative housework/v1/housework.proto
```

You call _protoc_ with flags indicating you want to generate Go code 1 from the _housework/v1/housework.proto_ file you created in [Listing 12-17](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-17) and output the generated code to the _.proto_ file’s current directory, with relative paths 2.

If you receive the following error indicating _protoc_ cannot find the _protoc-gen-go_ binary, make sure _protoc-gen-go_’s location (likely _$GOPATH/bin_) is in your `PATH` environment variable:

```
protoc-gen-go: program not found or is not executable
--go_out: protoc-gen-go: Plugin failed with status code 1.
```

If _protoc_ is happy with the _.proto_ file and successfully generated the Go code, you’ll find that a new file named _housework/v1/housework.pb.go_ exists with these first few lines, though the version numbers may differ. I’ll use the `head` command on Linux/macOS to print the first seven lines:

```
$ head -n 7 housework/v1/housework.pb.go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
//      protoc-gen-go v1.25.0
//      protoc        v3.6.1
// source: housework/v1/housework.proto

package housework
```

As the comments indicate, you shouldn’t edit this module. Instead, make any necessary changes to the _.proto_ file and recompile it.

Now that you’ve generated a Go module from the _.proto_ file, you can put it to effective use in [Listing 12-18](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-18) by implementing your storage backend with protocol buffers.

```
package protobuf

import (
    "io"
    "io/ioutil"

    "google.golang.org/protobuf/proto"

 1 "github.com/awoodbeck/gnp/ch12/housework/v1"
)

func Load(r io.Reader) ([]*housework.Chore, error) {
    b, err := ioutil.ReadAll(r)
    if err != nil {
        return nil, err
    }

    var chores housework.Chores

    return chores.Chores, proto.Unmarshal(b, &chores)
}

func Flush(w io.Writer, chores []*housework.Chore) error {
    b, err := proto.Marshal(2&housework.Chores{Chores: chores})
    if err != nil {
        return err
    }

    _, err = w.Write(b)

    return err
}
```

Listing 12-18: Protocol buffers storage implementation (_protobuf/housework.go_)

Instead of relying on the `housework` package from [Listing 12-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-1), as you did when working with JSON and Gob, you import version 1 of the _protoc_-generated package, which you also named `housework`1. The generated `Chores` type 2 is a struct with a `Chores` field, which itself is a slice of `Chore` pointers. Also, Go’s protocol buffers package doesn’t implement encoders and decoders. Therefore, you must marshal objects to bytes, write them to the `io.Writer`, and unmarshal bytes from the `io.Reader` yourself.

Revisit the code in [Listing 12-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-2) and plug in the protocol buffers implementation with the two simple changes shown in [Listing 12-19](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-19).

```
--snip--
 1 "github.com/awoodbeck/gnp/ch12/housework/v1"
    // storage "github.com/awoodbeck/gnp/ch12/json"
    // storage "github.com/awoodbeck/gnp/ch12/gob"
 2 storage "github.com/awoodbeck/gnp/ch12/protobuf"
--snip--
```

Listing 12-19: Swapping the JSON storage package for the protobuf storage package (_cmd/housework.go_)

You replace the housework package from [Listing 12-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-1) with your generated package 1, make sure to comment out the `json` and `gob` imports, and uncomment the `protobuf` storage import 2. The actual functionality of the housework application remains unchanged.

## Transmitting Serialized Objects

Although you may sometimes need to serialize and store objects locally, you’re more likely to build a network service that serializes data. For example, an online store may have a web service that communicates with inventory, user accounting, billing, shipping, and notification services to facilitate customer orders. If these services ran on the same server, you’d have to buy a larger server to scale the online store as business grows. Another approach would be to run each service on its own server and increase the number of servers. But then you’d have a new problem: how can you share data among services when they no longer reside on a single server and so can’t access the same memory?

Large technology companies facilitate this with _remote procedure calls (RPCs)_, a technique by which a client can transparently call a subroutine on a server as if the call were local. From your application’s perspective, RPC services take code that appears to run locally and distribute it over a network. Your code may call a function that transparently relays a message to a server. The server would locally execute the function, then respond with the results, which your code receives as the function’s return value. As far as your code is concerned, the function call is local, despite RPC’s transparent interaction with the server. This approach allows you to scale services across servers while abstracting the details away from your code. In other words, your code functions the same no matter whether the function call runs on the same computer or on one over the network.

Most companies now implement RPC with _gRPC_, a cross-platform framework that leverages HTTP/2 and protocol buffers. Let’s use it here to build something more sophisticated than an app to keep track of the housework you have yet to do. You’ll write a service that can send tasks to Rosie, the robotic maid from the classic animated series _The Jetsons_, who can take over your domestic responsibilities. Granted, she won’t be available until the year 2062, but you can get a head start on the code.

### Connecting Services with gRPC

The gRPC framework is a collection of libraries that abstracts many of the RPC implementation details. It is platform neutral and programming-language agnostic; you can use it to integrate a Go service running on Windows with a Rust service running on Linux, for example. Now that you know how to work with protocol buffers, you have the foundation needed to add gRPC support to your application. You’ll reuse the _.proto_ file from the previous section.

First, make sure your gRPC package is up-to-date:

```
$ go get -u google.golang.org/grpc
```

Next, get the appropriate module for generating gRPC Go code:

```
$ go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc
```

The protocol buffer compiler includes a gRPC module. This module will output Go code that lets you easily add gRPC support. First, you need to add definitions to the _.proto_ file. [Listing 12-20](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-20) defines a new service and two additional messages.

```
--snip--

service RobotMaid {
1   rpc Add (Chores) returns (Response);
  rpc Complete (CompleteRequest) returns (Response);
  rpc List (Empty) returns (Chores);
}

message CompleteRequest {
  int32 2chore_number = 1;
}

3 message Empty {}

message Response {
  string message = 1;
}
```

Listing 12-20: Additional definitions to support a gRPC RobotMaid service (_housework/v1/housework.proto_)

The service needs to support the same three calls you used on the command line: `add`, `complete`, and `list`. You define a new service named `RobotMaid`, then add three RPC methods to it. These RPC methods correspond to the functions defined in Listings 12-5, 12-7, and 12-8 for use on the command line: the `list`, `add`, and `complete` functions, respectively. Instead of calling these functions locally, you’ll call the corresponding method on the `RobotMaid` to execute these commands via RPC. You prefix each method with the `rpc` keyword and follow this with the Pascal-cased method name. Next, you write the request message type in parentheses, the `returns` keyword, and the return message type in parentheses 1.

The `List` method doesn’t require any user input, but as in the command line application, you still must provide a request message type for it, even if it’s nil. In gRPC, the message type equivalent to nil is an empty message, which you call `Empty`3.

Until you’ve had the opportunity to add proper artificial intelligence (AI) to the robot, you’ll need to tell Rosie when her current chore is complete so she can move on to the next one. For this, you add a new message that informs her of the completed chore number 2. Since you expect feedback from Rosie, you also add a response message that contains a string.

Now compile the _.proto_ file to use the new service and messages. Tell _protoc_ to use the _protoc__-gen-go-grpc_ binary, which must also be in your `PATH` environment variable, like this:

```
$ protoc 1--go-grpc_out=. 2--go-grpc_opt=paths=source_relative \
housework/v1/housework.proto
```

The `--go-grpc_out` flag 1 invokes the _protoc-gen-go-grpc_ binary to add gRPC support to the generated code. This binary generates the relevant gRPC service code for you and writes gRPC-specific code to the _housework/v1/housework_grpc.pb.go_ file since you told _protoc-gen-go-grpc_ to use relative paths 2. You can now use the generated code to build a gRPC server and client.

### Creating a TLS-Enabled gRPC Server

Now let’s implement a gRPC client and server. By default, gRPC requires a secure connection, so you’ll add TLS support to your server. You’ll use the server’s _cert.pem_ and _key.pem_ files created in the preceding chapter for your gRPC server and pin the server’s certificate to your client. See the “Generating Certificates for Authentication” section on page 256 for details.

You’ll leverage the Go code generated by your _.proto_ file to define a new `RobotMaid` client and server and use the client to communicate with the server over the network using gRPC. First, let’s create the server portion of your robot maid. The `RobotMaidServer` interface generated from your _.proto_ file looks like this:

```
type RobotMaidServer interface {
    Add(context.Context, *Chores) (*Response, error)
    Complete(context.Context, *CompleteRequest) (*Response, error)
    List(context.Context, *empty.Empty) (*Chores, error)
    mustEmbedUnimplementedRobotMaidServer()
}
```

You’ll implement this interface in [Listing 12-21](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-21) by creating a new type named `Rosie`.

```
package main

import (
    "context"
    "fmt"
    "sync"

    "github.com/awoodbeck/gnp/ch12/housework/v1"
)

type Rosie struct {
    mu sync.Mutex
 1 chores []*housework.Chore
}

func (r *Rosie) Add(_ context.Context, chores *housework.Chores) (
    *housework.Response, error) {
    r.mu.Lock()
    r.chores = append(r.chores, chores.Chores...)
    r.mu.Unlock()

    return 2&housework.Response{Message: "ok"}, nil
}

func (r *Rosie) Complete(_ context.Context,
    req *housework.CompleteRequest) (*housework.Response, error) {
    r.mu.Lock()
    defer r.mu.Unlock()

    if r.chores == nil || req.ChoreNumber < 1 ||
        int(req.ChoreNumber) > len(r.chores) {
        return nil, fmt.Errorf("chore %d not found", req.ChoreNumber)
    }

    r.chores[req.ChoreNumber-1].Complete = true

    return &housework.Response{Message: "ok"}, nil
}

func (r *Rosie) List(_ context.Context, _ *housework.Empty) (
    *housework.Chores, error) {
    r.mu.Lock()
    defer r.mu.Unlock()

    if r.chores == nil {
        r.chores = make([]*housework.Chore, 0)
    }

    return &housework.Chores{Chores: r.chores}, nil
}

func (r *Rosie) Service() *housework.RobotMaidService {
    return 3&housework.RobotMaidService{
        Add:      r.Add,
        Complete: r.Complete,
        List:     r.List,
    }
}
```

Listing 12-21: Building a RobotMaid-compatible type named Rosie (_server/rosie.go_)

The new `Rosie` struct keeps its list of chores in memory 1, guarded by a mutex, since more than one client can concurrently use the service. The `Add`, `Complete`, and `List` methods all return either a response message type 2 or an error, both of which ultimately make their way back to the client. The `Service` method returns a pointer to a new `housework.RobotMaidService` instance 3 where `Rosie`’s `Add`, `Complete`, and `List` methods map to their corresponding method on the new instance.

Now, let’s set up a new gRPC server instance by using the `Rosie` struct ([Listing 12-22](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-22)).

```
package main

import (
    "crypto/tls"
    "flag"
    "fmt"
    "log"
    "net"

    "google.golang.org/grpc"

    "github.com/awoodbeck/gnp/ch12/housework/v1"
)

var addr, certFn, keyFn string

func init() {
    flag.StringVar(&addr, "address", "localhost:34443", "listen address")
    flag.StringVar(&certFn, "cert", "cert.pem", "certificate file")
    flag.StringVar(&keyFn, "key", "key.pem", "private key file")
}

func main() {
    flag.Parse()

    server := 1grpc.NewServer()
    rosie := new(Rosie)
  2housework.RegisterRobotMaidServer(server, 3rosie.Service())

    cert, err := tls.LoadX509KeyPair(certFn, keyFn)
    if err != nil {
        log.Fatal(err)
    }

    listener, err := net.Listen("tcp", addr)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Listening for TLS connections on %s ...", addr)
    log.Fatal(
     4 server.Serve(
         5 tls.NewListener(
                listener,
                &tls.Config{
                    Certificates:             []tls.Certificate{cert},
                    CurvePreferences:         []tls.CurveID{tls.CurveP256},
                    MinVersion:               tls.VersionTLS12,
                    PreferServerCipherSuites: true,
                },
            ),
        ),
    )
}
```

Listing 12-22: Creating a new gRPC server using Rosie (_server/server.go_)

First, you retrieve a new server instance 1. You pass it and a new `*housework.RobotMaidService` from `Rosie`’s `Service`3 method to the `RegisterRobotMaidServer` function 2 in the generated gRPC code. This registers `Rosie`’s `RobotMaidService` implementation with the gRPC server. You must do this before you call the server’s `Serve` method 4. You then load the server’s key pair and create a new TLS listener 5, which you pass to the server when calling `Serve`.

Now that you have a gRPC server implementation, let’s work on the client.

### Creating a gRPC Client to Test the Server

The client-side code isn’t much different from what you wrote in the “Serializing Objects” section on page 270. The main difference is that you need to instantiate a new gRPC client and modify the `add`, `complete`, and `list` functions to use it. Remember, you can implement the client portion in a separate programming language if the programming language offers `protobuf` support. You can generate code for your target language from your _.proto_ file with the expectation it will work seamlessly with your server in [Listing 12-22](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-22).

[Listing 12-23](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-23) details the changes to [Listing 12-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-2) necessary to add gRPC support to the housework application.

```
package main

import (
 1 "context"
 2 "crypto/tls"
 3 "crypto/x509"
    "flag"
    "fmt"
 4 "io/ioutil"
    "log"
    "os"
    "path/filepath"
    "strconv"
    "strings"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials"

    "github.com/awoodbeck/gnp/ch12/housework/v1"
)

var addr, caCertFn string

func init() {
 5 flag.StringVar(&addr, "address", "localhost:34443", "server address")
 6 flag.StringVar(&caCertFn, "ca-cert", "cert.pem", "CA certificate")

    flag.Usage = func() {
        fmt.Fprintf(flag.CommandLine.Output(),
            `Usage: %s [flags] [add chore, ...|complete #]
    add         add comma-separated chores
    complete    complete designated chore

Flags:
`, filepath.Base(os.Args[0]))
        flag.PrintDefaults()
    }
}
```

Listing 12-23: Initial gRPC client code for our housework application (_client/client.go_)

Aside from all the new imports 1234, you add flags for the gRPC server address 5 and its certificate 6.

[Listing 12-24](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-24) uses the gRPC client to list the current chores.

```
--snip--

func list(ctx context.Context, client housework.RobotMaidClient) error {
    chores, err := client.List(ctx, 1new(housework.Empty))
    if err != nil {
        return err
    }

    if len(chores.Chores) == 0 {
        fmt.Println("You have nothing to do!")
        return nil
    }

    fmt.Println("#\t[X]\tDescription")
    for i, chore := range chores.Chores {
        c := " "
        if chore.Complete {
            c = "X"
        }
        fmt.Printf("%d\t[%s]\t%s\n", i+1, c, chore.Description)
    }

    return nil
}
```

Listing 12-24: Using the gRPC client to list the current chores (_client/client.go_)

This code is quite like [Listing 12-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-5), except you’re asking the gRPC client for the list of chores, which retrieves them from the server. You need to pass along an empty message to make gRPC happy 1.

[Listing 12-25](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-25) uses the gRPC client to add new chores to the gRPC server’s chores.

```
--snip--

func add(ctx context.Context, client housework.RobotMaidClient,
    s string) error {
    chores := new(housework.Chores)

    for _, chore := range strings.Split(s, ",") {
        if desc := strings.TrimSpace(chore); desc != "" {
            chores.Chores = append(chores.Chores, &housework.Chore{
                Description: desc,
            })
        }
    }

    var err error
    if len(chores.Chores) > 0 {
        _, 1err = client.Add(ctx, chores)
    }

    return err
}
```

Listing 12-25: Adding new chores using the gRPC client (_client/client.go_)

As you did in the previous section, you parse the comma-separated list of chores. Instead of flushing these chores to disk, you pass them along to the gRPC client. The gRPC client transparently sends them to the gRPC server and returns the response to you. Since you know `Rosie` returns a non-nil error when the `Add` call fails, you return the error 1 as the result of the `add` function.

In [Listing 12-26](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-26), you write the code to mark chores complete. Doing this over gRPC requires a bit less code than [Listing 12-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-8) since most of the logic is in the server.

```
--snip--

func complete(ctx context.Context, client housework.RobotMaidClient,
    s string) error {
    i, err := strconv.Atoi(s)
    if err == nil {
        _, err = client.Complete(ctx,
            &housework.CompleteRequest{1ChoreNumber: int32(i)})
    }

    return err
}
```

Listing 12-26: Marking chores complete by using the gRPC client (_client/client.go_)

Notice the _protoc-gen-go_ module, which converts the snake-cased `chore_number` field in [Listing 12-20](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-20) to Pascal case in the generated Go code 1. You must also convert the `int` returned by `strconv.Atoi` to an `int32` before assigning it to the complete request message’s chore number since `ChoreNumber` is an `int32`.

[Listing 12-27](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-27) creates a new gRPC connection and pins the server’s certificate to its TLS configuration.

```
--snip--

func main() {
    flag.Parse()

    caCert, err := ioutil.ReadFile(caCertFn)
    if err != nil {
        log.Fatal(err)
    }
    certPool := x509.NewCertPool()
    if ok := certPool.AppendCertsFromPEM(caCert); !ok {
        log.Fatal("failed to add certificate to pool")
    }

    conn, err := 1grpc.Dial(
        addr,
     2 grpc.WithTransportCredentials(
         3 credentials.NewTLS(
                &tls.Config{
                    CurvePreferences: []tls.CurveID{tls.CurveP256},
                    MinVersion:       tls.VersionTLS12,
                    RootCAs:          certPool,
                },
            ),
        ),
    )
    if err != nil {
        log.Fatal(err)
    }
```

Listing 12-27: Creating a new gRPC connection using TLS and certificate pinning (_client/client.go_)

On the client side, you first create a new gRPC network connection 1 and then use the network connection to instantiate a new gRPC client. For most use cases, you can simply pass the address to `grpc.Dial`. But you want to pin the server’s certificate to the client connection. Therefore, you need to explicitly pass in a `grpc.DialOption` with the appropriate TLS credentials. This involves using the `grpc.WithTransportCredentials` function 2 to return the `grpc.DialOption` and the `credentials.NewTLS` function 3 to create the transport credentials from your TLS configuration. The result is a gRPC network connection that speaks TLS with the server and authenticates the server by using the pinned certificate.

You use this gRPC network connection to instantiate a new gRPC client in [Listing 12-28](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-28).

```
--snip--

    rosie := 1housework.NewRobotMaidClient(conn)
    ctx := context.Background()

    switch strings.ToLower(flag.Arg(0)) {
    case "add":
        err = add(ctx, rosie, strings.Join(flag.Args()[1:], " "))
    case "complete":
        err = complete(ctx, rosie, flag.Arg(1))
    }

    if err != nil {
        log.Fatal(err)
    }

    err = list(ctx, rosie)
    if err != nil {
        log.Fatal(err)
    }
}
```

Listing 12-28: Instantiating a new gRPC client and making calls (_client/client.go_)

Aside from instantiating a new gRPC client from the gRPC network connection 1, this bit of code doesn’t vary much from [Listing 12-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c12.xhtml#listing12-9). The difference, of course, lies in the fact that any interaction with the list of chores transparently takes place over a TLS connection to the gRPC server.

Give it a try. In one terminal, start the server:

```
$ go run server/server.go server/rosie.go -cert server/cert.pem -key server/key.pem
Listening for TLS connections on localhost:34443 ...
```

And in another terminal, run the client:

```
$ go run client/client.go -ca-cert server/cert.pem
You have nothing to do!
$ go run client/client.go -ca-cert server/cert.pem add Mop floors, Wash dishes
#       [X]     Description
1       [ ]     Mop floors
2       [ ]     Wash dishes
$ go run client/client.go -ca-cert server/cert.pem complete 2
#       [X]     Description
1       [ ]     Mop floors
2       [X]     Wash dishes
```

Of course, restarting the server wipes out the list of chores. I’ll leave it as an exercise for you to implement persistence on the server. One approach is to have the server load the chores from and flush chores to disk as you did earlier in this chapter.

## What You’ve Learned

Data serialization allows you to exchange data in a platform-neutral and language-neutral way. You can also serialize data for long-term storage, retrieve and deserialize the data, and pick up where your application left off.

JSON is arguably the most popular text-based data serialization format. Contemporary programming languages offer good support for JSON, which is one reason for its ubiquity in RESTful APIs. Go offers good support for binary-based data serialization formats as well, including Gob, which is nearly a drop-in replacement for JSON. Gob is Go’s native binary data serialization format, and it’s designed to be efficient and easy to use.

If you’re looking for a binary data serialization format with wider support, consider protocol buffers. Google designed protocol buffers to facilitate the exchange of serialized binary data across its supported platforms and programming languages. Many contemporary programming languages currently offer support for protocol buffers. Although protocol buffers aren’t the same drop-in replacement in Go as Gob is for JSON, Go has excellent protocol buffer support, nonetheless. You first need to add definitions that define the data structures you intend to serialize in a _.proto_ file. You then use the protocol buffer compiler and its Go module to generate Go code that corresponds to your defined data structures. Finally, you use the generated code to serialize your data structures into protocol buffer-formatted binary data and deserialize that binary data back into your data structures.

The gRPC framework is a high-performance, platform-neutral standard for making distributed function calls across a network. The _RPC_ in _gRPC_ stands for _remote procedure call_, which is a technique for transparently calling a function on a remote system and receiving the result as if you had executed the function on your local system. gRPC uses protocol buffers as its underlying data serialization format. Go’s protocol buffer module allows you to easily add gRPC support by defining services in your _.proto_ file and leveraging the generated code. This lets you quickly and efficiently stand up distributed services or integrate with existing gRPC services.