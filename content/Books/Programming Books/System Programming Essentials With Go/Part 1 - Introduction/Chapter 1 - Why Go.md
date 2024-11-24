---
id: 01JCGMSB5B8MX85QRRMPT9YY7Z
title: Chapter 1 - Why Go?
modified: 2024-11-12T11:55:03-05:00
---
# 1

# Why Go?

At some point in your programming journey, your programs performed I/O-related tasks such as creating and removing files and directories. They may have orchestrated the creation of new processes and the execution of other programs or even facilitated communication between threads and processes on the same computer and between processes on different computers connected via a network.

When our programs center on using a low-level set of tasks, we categorize them as system programming.

It is alleged that system programming is tedious. But I do not see it this way at all! In fact, it is quite the opposite – an enjoyable and entertaining experience. It is like being a magician. You get to control the operating system and hardware, and you can make things happen that would be impossible in other languages.

In this chapter, we discuss why Go is an excellent fit for building efficient, high-performance system software to support real-world scenarios.

In this chapter, we are going to cover the following main topics:

- Choosing Go
- Concurrency and goroutines
- Interacting with the OS
- Tooling
- Cross-platform development with Go

By the end of this chapter, you will have a grasp of where Go is in the ecosystem of system programming, the importance of the Go concurrency model to build efficient and high-performance system software, how Go chooses to interact with the OS, the Go approach to cross-platform development, and the main commands in the Go built-in tooling.

# Choosing Go

There are plenty of languages in the system programming space nowadays: some are well established, such as C and C++; some form a wave of newcomers, such as Zig, Rust, and Odin; and others claim the title of “C/C++ killer,” with their pledges of impressive performance.

Sure, we can use all of them and achieve outstanding results. Still, we could fall into hidden traps such as a steep learning curve, high cognitive load, a lack of community and support, inconsistent APIs with constant breaking changes, and a lack of adoption.

Go’s design philosophy emphasizes simplicity, expressiveness, robustness, and efficiency. Its support for concurrency and strong dependency management, as well as its focus on composition, make it a compelling choice for system programming. Its creators aimed to build a language that provides powerful building blocks without unnecessary complexity, which makes writing, reading, understanding, and maintaining system-level code easier. People with programming experience usually take two weeks to get acquainted with Go. While they may not be considered experts, they can confidently read standard Go code and write basic to medium-complexity programs without struggle.

Also, Go is excellent for system programming because the language has a Unix-minded design by checking all the boxes for simplicity. Many programmers who are proficient in Python and Ruby often transition to Go, as it allows them to retain their level of expressiveness while achieving improved performance and the capability to work with concurrency.

It is worth noting that Go’s philosophy doesn’t prioritize zero cost in terms of CPU usage. Instead, the language aims to reduce the effort demanded from programmers, which is considered more significant and, as a by-product, makes the experience enjoyable.

One of the leading criticisms of using Go for system programming is the **garbage collector** (**GC**), specifically its pauses and explicit memory limits. If you still have this pet peeve with Go, don’t worry. In [_Chapter 6_](https://learning.oreilly.com/library/view/system-programming-essentials/9781837634132/B21662_06.xhtml#_idTextAnchor145), we’ll see that more granular memory management is available from Go 1.20 and above.

NOTE

In a GC pause worst-case scenario, the stop-the-world time is typically less than 100 microseconds.

# Concurrency and goroutines

One of the most essential features of Go is its concurrency model. Concurrency is the ability to run multiple tasks at the same time. In system programming, executing many tasks in parallel is essential for improving the performance and responsiveness of our programs.

## Concurrency

Real-time systems demand precision, with concurrency being a pivotal factor. These systems coordinate tasks with exceptional timing, particularly in scenarios where even milliseconds matter. Concurrency offers significant advantages by increasing throughput (a measure of how many units of information a system can process in a given amount of time) while decreasing task completion times. Real-life instances show how concurrency improves responsiveness, making systems more flexible and tasks more efficient. Moreover, concurrency’s isolation abilities guarantee data integrity by preventing interference.

System programming involves a diverse range of tasks, from CPU-bound to I/O-bound. Concurrency orchestrates this diversity by allowing CPU-bound tasks to progress while I/O-bound tasks await resources.

Later, in [_Chapter 10_](https://learning.oreilly.com/library/view/system-programming-essentials/9781837634132/B21662_10.xhtml#_idTextAnchor211), when we discuss distributed systems, the importance of concurrency will shine. It orchestrates tasks across an application or even different nodes in the network, which is ideal for managing large-scale concurrency.

## Goroutines

Go’s concurrency model relies on goroutines and channels. Goroutines are lightweight execution threads, often referred to as green threads. Creating them is cost-effective. Unlike conventional threads, they exhibit remarkable efficiency, enabling thousands of goroutines to run simultaneously on just a few OS threads.

Channels, on the other hand, provide a mechanism for goroutines to communicate and synchronize without resorting to locks. This approach is inspired by the **Communicating Sequential Process** (**CSP**) ([https://www.cs.cmu.edu/~crary/819-f09/Hoare78.pdf](https://www.cs.cmu.edu/~crary/819-f09/Hoare78.pdf)) formalism, emphasizing coordinated interactions between concurrent components.

Diverging from numerous other programming languages that depend on external libraries or threading constructs for concurrency, Go incorporates concurrency seamlessly into its core language design. This design decision leads to code that is not only easier to comprehend but also less susceptible to errors, as the complexities of threading are abstracted.

## CSP-inspired model

Go’s concurrency model draws inspiration from CSP, a formal language for describing concurrent systems. CSP focuses on communication and synchronization between concurrently executing entities. Unlike traditional multi-threaded programming, CSP and Go prioritize communication through channels instead of shared memory, reducing complexity and potential hazards. Synchronization and coordination are essential, with CSP using channels for process synchronization and Go using similar channels to coordinate goroutines. Safety and isolation are key, as both languages ensure safe interaction through channels, enhancing predictability and reliability. Go’s channels directly realize CSP’s communication-based approach, providing a safe way for goroutines to exchange data without the pitfalls of shared memory and locks.

## Share by communication

The famous Go proverb, “Don’t communicate by sharing memory, share memory by communicating” is often a source of discussion and misinterpretation. Still, reading it as “Share by communicating, not by locking” would be more precise, mainly because mainstream languages often rely on locks to protect shared data, leading to potential issues such as deadlocks and race conditions. Go encourages a different paradigm: _sharing data through channels by sending and receiving messages_. This “share by communicating” philosophy reduces the need for explicit locks and promotes a safer concurrency environment.

If you are into functional programming, I have great news for you. In Go, data is not implicitly shared between goroutines. In other words, the data is copied. Did you see the concern with data immutability? This stands in contrast to languages, where shared memory is the default mode of communication between threads. Go’s emphasis on explicit communication via channels helps avoid the unintended data sharing and race conditions that can arise in traditional threading models. Another benefit of this model is that there is no callback hell since every interaction with concurrent code is often read in a procedural manner. A regular Go function can be used in procedural code without tying the signatures with extra keywords.

NOTE

Callback hell, also known as the “pyramid of doom,” is a term used in programming to describe a situation where nested and interdependent callback functions make the code difficult to read, understand, and maintain. This typically occurs in asynchronous programming environments, such as JavaScript, where callbacks are used to handle asynchronous operations.

In the next chapter, we will refresh all concepts of concurrency and its building blocks to prepare you for interacting with the OS interfaces.

In addition to its concurrency model, Go also provides a way to interact with the operating system at a low level. This is essential for system programming, where you often need to control the OS and hardware.

# Interacting with the OS

Go’s approach to system calls is designed to be safe and efficient, especially in the context of its concurrency model.

In Go, system calls are comparatively lower-level in comparison to certain other programming languages. It can be helpful if you need fine-grained control over system resources, but it also means you’re dealing with more low-level details.

Making system calls often requires understanding the underlying operating system APIs and conventions. The side effect is that it can introduce a steeper learning curve if you are new to systems programming or lower-level development.

Not familiar with system calls? Fear not! The book’s second part will explore and experiment with them in detail to cover the main aspects we need to progress in our system programming journey.

# Tooling

Go is like a toolbox. It has everything we need to build great software, so we don’t need anything more than its standard tools to create our programs.

Let’s explore the principal tools that facilitate building, testing, running, error-checking, and code formatting.

## go build

The **go build** command is used to compile Go code into an executable binary that you can run.

Let’s see an example.

Assume you have a Go source file named **main.go** containing the following code:

package main
import "fmt"
func main() {
    fmt.Println("Hello, Go!")
}

You can compile it using the **go** **build** command:

go build main.go

This will generate an executable binary named **main** (or **main.exe** on Windows). You can then run the binary to see the output:

./main

## go test

The **go test** command is used to run tests on your Go code. It automatically finds test files and runs the associated test functions.

Here’s an example.

Assume you have a Go source file named **math.go** containing a function to add two numbers:

package math
func Add(a, b int) int {
    return a + b
}

You can create a test file named **math_test.go** to write tests for the **Add** function:

package math
import "testing"
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, but got %d", result)
    }
}

Run the tests using the **go** **test** command:

go test

## go run

The **go run** command allows you to run Go code directly without explicitly compiling it into an executable.

Let’s see this using an example.

Assume you have a Go source file named **hello.go** containing the following code:

package main
import "fmt"
func main() {
    fmt.Println("Hello, Go!")
}

You can directly run the code using the **go** **run** command:

go run hello.go

This will execute the code and print **Hello, Go!** to the console.

## go vet

We use the **go vet** command to check our Go code for potential errors or suspicious constructs. It employs heuristics that may not ensure all reports are actual issues, but it can uncover errors not caught by the compilers.

Here’s an example.

Assume you have a Go source file named **error.go** containing the following code with an intentional error:

package main
import "fmt"
func main() {
    movie_year := 1999
    movie_title := "The Matrix"
    fmt.Printf("In %s, %s was released.\n", movie_year, movie_title)
}

You can use the **go vet** command to check for errors:

go vet error.go

It might report a warning such as this: **Printf format %s has arg 1999 of wrong** **type int**.

## go fmt

The **go fmt** command is used to format your Go code according to the Go programming style guidelines. It automatically adjusts code indentation, spacing, and more.

Let’s see an example for this too.

Assume you have a Go source file named **unformatted.go** containing improperly formatted code:

package main
import "fmt"
func main() {
        msg:="Hello"
        fmt.Println(msg) 
}

You can format the code using the **go** **fmt** command:

go fmt unformatted.go

It will update the code to match the standard formatting conventions:

package main
import "fmt"
func main() {
        msg := "Hello"
        fmt.Println(msg)
}

Now that we have a good grasp of the basic tools, we can start familiarizing ourselves with Go’s cross-platform capabilities.

# Cross-platform development with Go

Cross-platform development with Go is a breeze. You can write code running on various operating systems and architectures with ease.

Cross-platform development with Go can be achieved by using the **GOOS** and **GOARCH** environment variables. The **GOOS** environment variable specifies the OS you want to target, and the **GOARCH** environment variable specifies your target architecture.

For example, assume you have a Go source file named **main.go**:

package main
import "fmt"
func main() {
    fmt.Println("This program runs in any OS!")
}

To compile code for Linux, you would set the **GOOS** environment variable to **linux** and the **GOARCH** environment variable to **amd64**:

GOOS=linux GOARCH=amd64 go build

This command will compile the code for Linux.

You can also use the **GOOS** and **GOARCH** environment variables to run code on different platforms. For example, to run the code you compiled for Linux on macOS, you would set the **GOOS** environment variable to **darwin** and the **GOARCH** environment variable to **amd64**.

GOOS=darwin GOARCH=amd64 go run

This command will run the code on macOS.

NOTE

Although Go strives for portability across various platforms, engaging with the OS via system calls inherently ties your code to specific OS features. Code that is heavily reliant on these operations might need conditional compilation or adjustments when aiming at different platforms.

Leveraging build flags in Go allows you to selectively compile specific sections of your code contingent upon particular conditions, such as the target OS or architecture.

This can be useful when creating programs that interface with the **golang.org/x/sys** package for Windows and Unix-like systems.

Assume that you have two Go source files named **main_windows.go** and **main_linux.go** and you want to use build tags to ensure code segmentation.

Here is an example of a code using build tags to segment for Windows:

// go:build windows
package main
import "fmt"
func main() {
    fmt.Println("This is Windows!")
}

We can do the same but aiming for Linux this time:

// go:build linux
package main
import "fmt"
func main() {
    fmt.Println("This is Linux!")
}

These are the commands to use to compile these programs, respectively:

GOOS=windows go build -o app.exe
GOOS=linux go build -o app

When we execute **app** within a Linux environment, it should print **This is Linux!**. Meanwhile, on a Windows system, running **app.exe** will display **This** **is Windows!**.

# Summary

This chapter provided a comprehensive guide to why Go is a top choice for system programming and insights into Go’s design philosophy, emphasizing simplicity, robustness, and efficiency. We learned about Go’s concurrency model, its approach to interacting with the OS, and how to interact with the tools for cross-platform development. These lessons are helpful as they equip us with the knowledge needed to write, read, and maintain system-level code with Go, enabling improved performance and the ability to work with concurrency.

In the next chapter, we explore the concurrency concepts, refreshing all related concepts and building blocks. It will prepare us for more advanced interactions with OS interfaces, enhancing our ability to create powerful and responsive programs.