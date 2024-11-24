---
id: 01JCGMSB5B8MX85QRRMPT9YY7Z
title: Chapter 1 - Why Go?
modified: 2024-11-12T15:15:59-05:00
---
# 2

# Refreshing Concurrency and Parallelism

This chapter will explore goroutines at the core of Go’s concurrency. You will learn how they function, distinguish between concurrency and parallelism, manage currently running goroutines, handle data race issues, use channels for communication, and use **Channel** states and signaling to maximize their potential. Mastering these concepts is essential to write efficient and error-free Go code.

In this chapter, we’re going to cover the following main topics:

- Understanding goroutines
- Managing data races
- Making sense of channels
- The guarantee of delivery
- State and signaling

# Technical requirements

You can find this chapter’s source code at [https://github.com/PacktPublishing/System-Programming-Essentials-with-Go/tree/main/ch2](https://github.com/PacktPublishing/System-Programming-Essentials-with-Go/tree/main/ch2).

# Understanding goroutines

Goroutines are functions created and scheduled to be run independently by the Go scheduler. The Go scheduler is responsible for the management and execution of goroutines.

Behind the scenes, we have a complex algorithm to make goroutines work. Fortunately, in Golang, we can achieve this highly complex operation with simplicity using the **go** keyword.

NOTE

If you are accustomed to a language that has the **async**/**await** feature, you probably are used to deciding your function beforehand. It will be used concurrently to change the function signature to sign that the function can be paused/resumed. Calling this function also needs a special notation. When using goroutines, there is no need to change the function signature.

In the following snippets, we have a main function calling sequentially the **say** function, passing as an argument **"hello"** and **"****world"**, respectively:

func main() {
  say(«hello»)
  say(«world»)
}

The **say** function receives a string as a parameter and iterates five times. For each iteration, we make the function sleep for 500 milliseconds and print the **s** parameter immediately after:

func say(s string) {
  for i := 1; i < 5; i++ {
     time.Sleep(500 * time.Millisecond)
     fmt.Println(s)
  }
}

When we execute the program, it should print the following output:

hello
hello
hello
hello
hello
world
world
world
world
world

Now, we introduce the **go** keyword right before the first call to the **say** function to introduce concurrency in our program:

func main() {
  go say(«hello»)
  say(«world»)
}

The output should alternate between **hello** and **world**.

So, we can achieve the same result if we create a goroutine for the second function call, right?

func main() {
  say(«hello»)
  go say(«world»)
}

Let’s see the results of the program now:

hello
hello
hello
hello

Wait! Something is wrong here. What did we do wrong? The main function and the goroutine seem out of sync.

We didn’t do anything wrong. That is the expected behavior. When you take a closer look at the first program, the goroutine is fired, and the second call of **say** executes in the context of the main function sequentially.

In other words, the program should wait for the function to terminate to reach the end of the **main** function. For the second program, we have the opposite behavior. The first call is a normal function call, so it prints five times as expected, but when the second goroutine is fired, there is no following instruction on the main function, so the program terminates.

Although the behavior is correct from the perspective of how the program works, this is not our intention. We need a way to synchronize the **wait** for all the goroutines in this group of executions before giving the **main** function a chance to terminate. In situations such as this, we can leverage Go’s construct, the **sync** package, called **WaitGroup**.

## WaitGroup

**WaitGroup**, as the name suggests, is a Go standard library mechanism that allows us to wait for a group of goroutines until they finish explicitly.

No particular factory function exists to create them, since their zero-value is already a valid usable state. Since **WaitGroup** has been created, we need to control how many goroutines we are waiting for. We can use the **Add()** method to inform the group.

How can we inform the group that we have completed one of the routines? It couldn’t be more intuitive. We can achieve this using the **Done()** method.

In the following example, we introduce the wait group to make our program output the messages as intended:

func main() {
  wg := sync.WaitGroup{}
  wg.Add(2)
  go say(«world», &wg)
  go say("hello", &wg)
  wg.Wait()
}

We create the **WaitGroup** ( **wg := sync.WaitGroup{}**) and declare that two goroutines participate in this group (**wg.Add(2)**).

In the last line of the program, we explicitly hold the execution with the **Wait()** method to avoid the program termination.

To make our function interact with **Waitgroup**, we need to send a reference to this group. Once we have its reference, the function can defer, calling **Done()**, to ensure that we signal correctly for our group every time the function is complete.

This is the new **say** function:

func say(s string, wg *sync.WaitGroup) {
  defer wg.Done()
  for i := 0; i < 5; i++ {
     fmt.Println(s)
  }
}

We don’t need to rely on **time.Sleep()**, so this version doesn’t have it.

Now, we can control our group of goroutines. Let’s deal with one central worrisome issue in concurrent programming – state.

## Changing shared state

Imagine a scenario where two diligent workers are tasked with packing items into boxes in a busy warehouse. Each worker fills a fixed number of things into packets, and we must keep track of the total number of items packed.

This seemingly straightforward task, analogous to concurrent programming, can quickly become a nightmare when not handled properly. With proper synchronization, the workers may avoid intentionally interfering with each other’s work, leading to incorrect results and unpredictable behavior. It’s a classic example of a data race, a common challenge in concurrent programming.

The following code will walk you through an analogy where two warehouse workers face a data race issue while packing items into boxes. We’ll first present the code without proper synchronization, demonstrating the data race problem. Then, we’ll modify the code to address the issue, ensuring that the workers collaborate smoothly and accurately.

Let’s step into the bustling warehouse and witness firsthand the challenges of concurrency and the importance of synchronization in this example:

package main
import (
     "fmt"
     "sync"
)
func main() {
     fmt.Println("Total Items Packed:", PackItems(0))
}
func PackItems(totalItems int) int {
     const workers = 2
     const itemsPerWorker = 1000
     var wg sync.WaitGroup
     itemsPacked := 0
     for i := 0; i < workers; i++ {
          wg.Add(1)
          go func(workerID int) {
               defer wg.Done()
               // Simulate the worker packing items into boxes.
               for j := 0; j < itemsPerWorker; j++ {
                      itemsPacked = totalItems
                    // Simulate packing an item.
                    itemsPacked++
               // Update the total items packed without proper synchronization.
               totalItems = itemsPacked
               }
          }(i)
     }
     // Wait for all workers to finish.
     wg.Wait()
     return totalItems
}

The **main** function starts by calling the **PackItems** function with an initial **totalItems** value of 0.

In the **PackItems** function, there are two constants defined:

- **workers**: The number of worker goroutines (set to 2)
- **itemsPerWorker**: The number of items each worker should pack into boxes (set to 1,000)

**WaitGroup** named **wg** is created to wait for all worker goroutines to finish before returning the final **totalItems** value.

A loop runs **workers** times, where each iteration starts a new goroutine to simulate a worker packing items into boxes. Inside the goroutine, the following steps are performed:

1. A worker ID is passed to the goroutine as an argument.
2. The **defer wg.Done()** statement ensures that the wait group is decremented when the goroutine exits.
3. An **itemsPacked** variable is initialized with the current value of **totalItems** to keep track of the items packed by this worker.
4. A loop runs **itemsPerWorker** times, simulating the process of packing items into boxes. However, there’s no actual packing happening;the loop’s just incrementing the **itemsPacked** variable.
5. In the last step in the inner loop, **totalItems** receive the altered value of the **itemsPacked** variable, which contains the number of items packed by the worker.
6. **This is where the synchronization issue occurs**. The worker attempts to update the **totalItems** variable by adding the **itemsPacked** value to it.

Since multiple goroutines attempt to modify **totalItems** concurrently without proper synchronization, a data race occurs, leading to unpredictable and incorrect results.

### Nondeterministic results

Consider this alternative **main** function:

func main() {
     times := 0
     for {
          times++
          counter := PackItems(0)
          if counter != 2000 {
               log.Fatalf("it should be 2000 but found %d on execution %d", counter, times)
          }
     }
}

The program constantly runs the **PackItems** function until the expected result of 2,000 is not achieved. Once this occurs, the program will display the incorrect value returned by the function and the number of attempts it took to reach that point.

Because of the non-deterministic nature of the Go scheduler, the result would be right _most of the time_. This code would need a lot of runs to reveal its synchronization flaw.

In a single execution, I needed more than 16,000 iterations:

it should be 2000 but found 1170 on execution 16421

YOUR TURN!

Experiment running the code on your machine. How many iterations did your code need to fail?

If you’re using your personal computer, there are likely many tasks being performed, but your machine probably has a lot of unused resources. However, it’s important to consider the amount of noise on shared nodes in a cluster if you’re running programs in cloud environments with containers. By “noise,” I mean the work done on the host machine while running your program. It may be just as idle as your local experiment. Still, it’s likely being used to its full potential in a cost-effective scenario where every core and memory is utilized.

This scenario of a constant contest for resources makes our schedule much more inclined to choose another workload instead of just continuing to run our goroutine.

In the following example, we call the **runtime.Gosched** function to emulate noise. The idea is to give a hint to the Go scheduler, saying, “_Hey! Maybe it is a good moment to_ _pause me_”:

for j := 0; j < itemsPerWorker; j++ {
    itemsPacked = totalItems
    runtime.Gosched() // emulating noise!
    itemsPacked++
    totalItems = itemsPacked
}

Running the main function again, we can see that the erroneous results occur much faster than before. In my execution, for example, I need just four iterations:

it should be 2000 but found 1507 on execution 4

Unfortunately, the code is still buggy. How can we anticipate that? At this point, you should have guessed that Go tools have the answer, and you’re right again. We can manage data races on our tests.

# Managing data races

When multiple goroutines access shared data or resources concurrently, a “race condition” can occur. As we can attest, this type of concurrency bug can lead to unpredictable and undesirable behavior. The Go test tool has a built-in feature called **Go race detection** that can detect and identify race conditions in your Go code.

So, let’s create a **main_test.go** file with a simple test case:

package main
import (
     "testing"
)
func TestPackItems(t *testing.T) {
     totalItems := PackItems(2000)
     expectedTotal := 2000
     if totalItems != expectedTotal {
          t.Errorf("Expected total: %d, Actual total: %d", expectedTotal, totalItems)
     }
}

Now, let’s use the race detector:

go test -race

The result in the console will be something like this:

==================
WARNING: DATA RACE
Read at 0x00c00000e288 by goroutine 9:
  example1.PackItems.func1()
      /tmp/main.go:35 +0xa8
  example1.PackItems.func2()
      /tmp/main.go:45 +0x47
Previous write at 0x00c00000e288 by goroutine 8:
  example1.PackItems.func1()
      /tmp/main.go:39 +0xba
  example1.PackItems.func2()
      /tmp/main.go:45 +0x47
// Other lines omitted for brevity

The output can be quite intimidating at first glance, but the most revealing information initially is the message **WARNING:** **DATA RACE**.

To fix the synchronization issue in this code, we should use synchronization mechanisms to protect access to the **totalItems** variable. Without proper synchronization, concurrent writes to shared data can lead to race conditions and unexpected results.

We have used **WaitGroup** from the **sync** package. Let’s explore more synchronization mechanisms to ensure the program’s correctness.

## Atomic operations

It’s heartbreaking that the term “atomic” in Go doesn’t involve physically manipulating atoms, like in physics or chemistry. It would be fascinating to have that capability in programming; instead, atomic operations in Go are focused on synchronizing and managing concurrency among goroutines using the sync/atomic package.

Go offers atomic operations to load, store, add, and **CAS** (**compare and swap**) for certain types, such as **int32**, **int64**, **uint32**, **uint64**, **uintptr**, **float32**, and **float64**. Atomic operations can’t be directly performed on arbitrary data structures.

Let’s change our program using the atomic package. First, we should import it:

import (
     "fmt"
     "sync"
     "sync/atomic"
)

Instead of updating **totalItems** directly, we will leverage the **AddInt32** function to guarantee the synchronization:

for j := 0; j < itemsPerWorker; j++ {
    atomic.AddInt32(&totalItems, int32(itemsPacked))
}

If we check for data races again, no problem will be reported.

Atomic structures are great when we need to synchronize a single operation, but when we want to synchronize a block of code, other tools are a better fit, such as mutexes.

## Mutexes

Ah, mutexes! They’re like the bouncers at a party for goroutines. Imagine a bunch of these little Go creatures trying to dance around with shared data. It’s all fun and games until chaos breaks loose, and you have a goroutine traffic jam with data spills all over the place!

Do not worry, as mutexes swoop in like the dance-floor supervisors, ensuring that only one groovy goroutine can bust a move in the critical section at a time. They’re like the rhythm keepers of concurrency, ensuring that everyone takes turns and nobody steps on each other’s toes.

You can create a mutex by declaring a variable of type **sync.Mutex**. A mutex allows us to protect a critical section of code, using the **Lock()** and **Unlock()** methods. When a goroutine calls **Lock()**, it acquires the mutex lock, and any other goroutines attempting to call **Lock()** will be blocked until the lock is released with **Unlock()**.

Here is the code for our program using mutex:

package main
import (
     "fmt"
     "sync"
)
func main() {
      m := sync.Mutex{}
     fmt.Println("Total Items Packed:", PackItems(&m, 0))
}
func PackItems(m *sync.Mutex, totalItems int) int {
     const workers = 2
     const itemsPerWorker = 1000
     var wg sync.WaitGroup
     for i := 0; i < workers; i++ {
          wg.Add(1)
          go func(workerID int) {
               defer wg.Done()
               for j := 0; j < itemsPerWorker; j++ {
                    m.Lock()
                    itemsPacked := totalItems
                   itemsPacked++
                      totalItems = itemsPacked
                    m.Unlock()
               }
          }(i)
     }
     // Wait for all workers to finish.
     wg.Wait()
     return totalItems
}

In this example, we lock a block of code handling to change our shared state, and when we’re done, we unlock the mutex.

If the mutexes ensure the correctness handling shared state, you could consider two options:

- You could use lock and unlock for every critical line
- You could simply lock in the beginning of the function and defer the unlock

Yes, you could! Sadly, there is a catch in both approaches. We introduce latency indiscriminately. To make my point, let’s benchmark the second approach versus the original use of mutex.

Let’s create a second version of the function using multiple calls to lock/unlock, called **MultiplePackItems**, where everything remains the same except the function name and the inner loop.

Here is the inner loop:

for j := 0; j < itemsPerWorker; j++ {
    m.Lock()
    itemsPacked = totalItems
    m.Unlock()
    m.Lock()
    itemsPacked++
    m.Unlock()
    m.Lock()
    totalItems = itemsPacked
    m.Unlock()
}

Let’s look at the performance of both options running a benchmark test:

Benchmark-8                   36546             32629 ns/op
BenchmarkMultipleLocks-8      13243             91246 ns/op

The version with multiple locks is approximately **~64%** slower than the first one in terms of the time taken per operation.

BENCHMARKS

We’ll cover in detail benchmarks and other techniques of performance measurement in [_Chapter 6_](https://learning.oreilly.com/library/view/system-programming-essentials/9781837634132/B21662_06.xhtml#_idTextAnchor145), _Analyzing Performance_.

These examples show goroutines performing their tasks independently, without collaborating with each other. However, in many cases, our tasks require exchanging information or signals to make decisions, such as starting or stopping a procedure.

When exchanging information is crucial, we can use a flagship tool in Go called a channel.

# Making sense of channels

Welcome to the channel carnival!

Imagine Go channels as magical, clown-sized pipes that allow circus performers (goroutines) to pass around juggling balls (data) while making sure nobody drops the ball – quite literally!

## How to use channels

To use channels, we need to use a built-in function called **make()**, informing what type of data we’re interested in passing using this channel:

 make(Chan T)

If we want a channel of **string**, we should declare the following:

 make (chan string)

We can inform a capacity. Channels with capacity are called buffered channels. We won’t bother going into detail about capacity for now. We create an unbuffered channel when we don’t inform the capacity.

## An unbuffered channel

An unbuffered channel is a way to communicate between multiple goroutines, and it needs to respect a simple rule – the goroutine that wants to send in the channel and the one that wants to receive should be **ready** at the same time.

Think of this as a “trust fall” exercise. The sender and receiver must trust each other fully, ensuring the safety of the data, just like acrobats trust their partners to catch them mid-air.

Abstract? Let’s explore this concept with examples.

First, let’s send information to a channel with no receiver:

package main
func main() {
    c := make(chan string)
    c <- "message"
}

When we execute, the console will print something like the following:

fatal error: all goroutines are sleep – dead lock!
goroutine 1 [chan send]:
main.main()

Let’s break down this output.

**all goroutines are sleep – deadlock!** is the main error message. It tells us that all goroutines in our program are in a **sleep** state, which implies that they are waiting for some event or resource to become available. However, because all of them are waiting and cannot make any progress, your program has encountered a deadlock situation.

**goroutine 1 [chan send]:** is the part of the message that provides additional information about the specific goroutine that has encountered the deadlock. In this case, it’s **goroutine 1**, and it was involved in a channel send operation (**chan send**).

This deadlock occurs because the execution is paused, waiting for another goroutine to receive the information, but there’s none.

DEADLOCKS

A deadlock is a condition where two or more processes or goroutines are unable to proceed because they are all waiting for something that will never happen.

Now, we can try the opposite; in the next example, we want to receive from a channel with no sender:

package main
func main() {
    c := make(chan string)
    fmt.Println(<- c )
}

The output in the console is very similar, except that now, the error is about receiving:

fatal error: all goroutines are sleep – dead lock!
goroutine 1 [chan receive]:
main.main()

Now, following the rule is as simple as sending and receiving simultaneously. So, declaring both will be sufficient:

package main
func main() {
    c := make(chan string)
    c <- "message" // Sending
    fmt.Println(<- c ) // Receiving
}

It’s a good idea, but unfortunately, it doesn’t work, as we can see in the following output:

fatal error: all goroutines are sleep – dead lock!
goroutine 1 [chan send]:
main.main()

If we’re following the rule, why is it not working?

Well, we’re not exactly following the rule. The rule states that the goroutine that wants to send in the channel and the one that wants to receive should be _ready_ at the same time.

The important thing to take note of is the final part – _ready at the_ _same time_.

Since the code runs sequentially, line by line, when we try to send **c <- "message"**, the program waits for the receiver to receive the message. We need to make these two parties send and receive the message simultaneously. We can use our concurrent programming knowledge to make this happen.

Let’s add goroutines to the mix, using the circus analogy. We’ll introduce a function, **throwBalls**, that will expect the color of the balls to be thrown (**color**) and the channel (**balls**) where it should receive these throws:

package main
import "fmt"
func main() {
    balls := make(chan string)
    go throwBalls("red", balls)
    fmt.Println(<-balls, "received!")
}
func throwBalls(color string, balls chan string) {
    fmt.Printf("throwing the %s ball\n", color)
    balls <- color
}

Here, we have three major steps:

1. We create an unbuffered string channel named **balls**.
2. A goroutine is launched inline using the **throwBalls** function to send “red” into the channel.
3. The main function receives and prints the value received from the channel.

The output for this example is as follows:

throwing the red ball
red received!

We did it! We successfully passed information between goroutines using channels!

But what happens when we send one more ball? Let’s try it with a green ball:

func main() {
    balls := make(chan string)
    go throwBalls("red", balls)
    go throwBalls("green", balls)
    fmt.Println(<-balls, "received!")
}

The output shows just one ball being received. What happened?

throwing the red ball
red received!

RED OR GREEN?

Since we’re launching more than one goroutine, the scheduler will elect arbitrarily what should execute first. Therefore, you can see green or red randomly running the code.

We can fix the issue by putting in one more **print** statement received from the channel:

func main() {
    balls := make(chan string)
    go throwBalls("red", balls)
    go throwBalls("green", balls)
    fmt.Println(<-balls, "received!")
    fmt.Println(<-balls, "received!")
}

Although it works, it’s not the most elegant solution. We could have trouble with deadlocks again if we have more receivers than senders:

func main() {
    balls := make(chan string)
    go throwBalls("red", balls)
    go throwBalls("green", balls)
    fmt.Println(<-balls, "received!")
    fmt.Println(<-balls, "received!")
    fmt.Println(<-balls, "received!")
}

The last print will await forever, causing another deadlock.

If we want to make code work with any number of balls, we should stop adding more and more lines and replace them all with the **range** keyword.

### Iterating over a channel

The mechanism used to iterate over the values sent through a channel is the **range** keyword.

Let’s change the code to iterate over the channel values:

func main() {
    balls := make(chan string)
    go throwBalls("red", balls)
    go throwBalls("green", balls)
    for color := range balls {
         fmt.Println(color, "received!")
    }
}

We can happily check the console to see the balls received elegantly, but wait – all the goroutines are asleep! Deadlock again?

This error occurs when we iterate over channels and the range expects a channel to be closed to stop the iteration.

### Closing a channel

To close a channel, we need to call the built-in **close** function, passing the channel:

close(balls)

OK, we can now guarantee that the channel is closed. Let’s change the code by adding the **close** call between the senders and **range**:

go throwBalls("green", balls)
close(balls)
for color := range balls {

You may have noticed that if the range stops when the channel is closed, with this code, the range will never run once the channel has closed.

We need to orchestrate this group of tasks, and yes, you’re right – we’re using **WaitGroup** to save us again. This time, we don’t want to taint the **throwBalls** signature to receive our **WaitGroup**, so we’ll create inline anonymous functions to keep our functions unaware of the concurrency. Additionally, we want to close the channel when we have the guarantee that all the tasks are done. We infer this with the **Wait()** method from our **WaitGroup**.

Here is our **main** function:

func main() {
    balls := make(chan string)
    wg := sync.WaitGroup{}
    wg.Add(2)
    go func() {
        defer wg.Done()
        throwBalls("red", balls)
    }()
    go func() {
        defer wg.Done()
        throwBalls("green", balls)
    }()
    go func() {
        wg.Wait()
        close(balls)
    }()
    for color := range balls {
        fmt.Println(color, "received!")
    }
}

Phew! This time, the output is correctly shown:

throwing the green ball
green received!
throwing the red ball
red received!

What a ride, huh? But wait! We still need to explore the buffered channels!

## Buffered channels

It’s analogy time!

These are the channels where clowns come into play! Imagine a clown car with a limited number of seats (capacity). Clowns (senders) can hop in and out of the car, dropping juggling balls (data) into it.

We want to create a program with buffered channels that simulate a circus car ride, where clowns try to get into a clown car (limited to three clowns at a time) with balloons. The driver controls the car and manages the clowns’ rides while the clowns attempt to get in. If the car is full, they wait and print a message. After all the clowns are done, the program waits for the car driver to finish and then prints that the circus car ride is over.

If a clown tries to stuff too many juggling balls into the car, it’s as hilarious as a car overflowing with clowns and juggling balls, creating a comical spectacle!

First, let’s create the program structure to receive our senders and receivers:

package main
import (
    "fmt"
    "sync"
    "time"
)
func main() {
    clownChannel := make(chan int, 3)
    clowns := 5
    // senders and receivers logic here!
    var wg sync.WaitGroup
    wg.Wait()
    fmt.Println("Circus car ride is over!")
}

Here is the driver’s goroutine (receiver):

go func() {
        defer close(clownChannel)
        for clownID := range clownChannel {
            balloon := fmt.Sprintf("Balloon %d", clownID)
            fmt.Printf("Driver: Drove the car with %s inside\n", balloon)
            time.Sleep(time.Millisecond * 500)
            fmt.Printf("Driver: Clown finished with %s, the car is ready for more!\n", balloon)
        }
    }()

We add the clown logic (sender) just below the rider’s block:

for clown := 1; clown <= clowns; clown++ {
    wg.Add(1)
    go func(clownID int) {
        defer wg.Done()
        balloon := fmt.Sprintf("Balloon %d", clownID)
        fmt.Printf("Clown %d: Hopped into the car with %s\n", clownID, balloon)
        select {
            case clownChannel <- clownID:
                fmt.Printf("Clown %d: Finished with %s\n", clownID, balloon)
            default:
                fmt.Printf("Clown %d: Oops, the car is full, can't fit %s!\n", clownID, balloon)
        }
    }(clown)
}

Running the code, we can see all the trouble that the clowns are making:

Clown 1: Hopped into the car with Balloon 1
Clown 1: Finished with Balloon 1
Driver: Drove the car with Balloon 1 inside
Clown 2: Hopped into the car with Balloon 2
Clown 2: Finished with Balloon 2
Clown 5: Hopped into the car with Balloon 5
Clown 5: Finished with Balloon 5
Clown 3: Hopped into the car with Balloon 3
Clown 3: Finished with Balloon 3
Clown 4: Hopped into the car with Balloon 4
Clown 4: Oops, the car is full, can't fit Balloon 4!
Circus car ride is over!

SELECT

The **select** statement allows us to wait on multiple communication channels and select the first one that becomes ready, effectively allowing us to perform non-blocking operations on channels.

When working with channels, it’s easy to get caught up in comparing message queues and channels, but there may be better ways to understand them. The channel internals are ring buffers, and this information can be confusing and unhelpful when choosing the program design. By prioritizing an understanding of signaling and the guaranteed delivery of messages, you’d be better equipped to work efficiently with channels.

# The guarantee of delivery

The main difference between buffered and unbuffered channels is the guarantee of delivery.

As we saw earlier, the unbuffered channels always guarantee delivery, since they only send a message when the receiver is ready. Conversely, the buffered channels can’t ensure message delivery because they can “buffer” an arbitrary number of messages before the synchronization step becomes mandatory. Therefore, the reader could fail to read a message from the channel buffer.

The most considerable side effect of choosing between them is how much latency you can afford to introduce to your program.

## Latency

Latency in the context of concurrent programming refers to the time it takes for a piece of data to travel from a sender (goroutine) to a receiver (goroutine) through a channel.

In Go channels, latency is influenced by several factors:

- **Buffering**: Buffering can reduce latency when the sender and receiver are not perfectly synchronized.
- **Blocking**: Unbuffered channels block the sender and receiver until they are ready to communicate, leading to potentially higher latency. Buffered channels allow the sender to continue without immediate synchronization, potentially reducing latency.
- **Goroutine scheduling**: The latency in channel communication also depends on how the Go runtime schedules goroutines. Factors such as the number of available CPU cores and the scheduling algorithm influence how quickly goroutines can be executed.

### Choosing a channel type

As a rule of thumb, we consider an unbuffered channel a strong choice for the following scenarios:

- **Guaranteed delivery**: Provide a guarantee that the value being sent is received by another goroutine. This is especially useful in scenarios where you need to ensure data integrity and that no data is lost.
- **One-to-one communication**: Unbuffered channels are best suited for one-to-one communication between goroutines.
- **Load balancing**: Unbuffered channels can be used to implement load-balancing patterns, ensuring that work is distributed evenly among worker goroutines.

Conversely, buffered channels offer the following:

- **Asynchronous communication**: Buffered channels allow for asynchronous communication between goroutines. When sending data on a buffered channel, the sender won’t block until the data is received, if there is space in the channel’s buffer. This can improve throughput in certain scenarios.
- **Reducing contention**: In scenarios where you have multiple senders and receivers, using a buffered channel can reduce contention. For example, in a producer-consumer pattern, you can use a buffered channel to allow producers to keep producing without waiting for consumers to catch up.
- **Preventing deadlocks**: Buffered channels can help prevent goroutine deadlocks by allowing a certain level of buffering, which can be useful when you have unpredictable variations in a workload.
- **Batch processing**: Buffered channels can be used for batch processing or pipelining where data is produced at one rate and consumed at another rate.

Now that we’ve covered the key aspects of latency and how it impacts channel communication in concurrent programming, let’s shift our focus to another critical aspect – state and signaling. Understanding the semantics of state and signaling is essential to avoid common pitfalls and make informed design decisions.

# State and signaling

Exploring the semantics of state and signaling puts you ahead of the curve in avoiding more straightforward bugs or making good design choices.

## State

Although Go eased the adoption of concurrency with channels, there are some characteristics and pitfalls.

We should remember that channels have three states – nil, open (empty, not empty), and closed. These states strongly relate to what we can and cannot do with channels, whether from the sender’s or receiver’s perspective.

Consider a channel when you want to read from:

- Reading to a **write-only** channel results in a compilation error
- If the channel is **nil**, reading from it indefinitely blocks your goroutine until it is initialized
- Reading will be blocked in an **open** and **empty** channel until data is available
- In an **open** and **not empty** channel, reading will return data
- If the channel is **closed**, reading it will return the default value for its type and **false** to indicate closure

Writing also has its nuances:

- Writing to a **read-only** channel results in a compilation error
- Writing to a **nil** channel block until it’s initialized
- Writing to an **open** and **full** channel blocks until there’s space
- In an **open** and **not full** channel, writing is successful
- Writing on a **closed** channel leads to a panic

Closing a channel depends on its state:

- Closing an **open channel with data** allows reads until drained, and then returns the default value.
- Closing an **open empty channel** immediately closes it, and reads also return the default value.
- Attempting to close an **already closed channel** results in a **panic**.
- Closing a read-only channel results in a compilation error.

## Signaling

Signaling between goroutines is an everyday use case for channels. You can use channels to coordinate and synchronize the execution of different goroutines by sending signals or messages between them.

Here is a simple example of how to use a Go channel to signal between two goroutines:

package main
import (
    "fmt"
    "sync"
)
func main() {
    signalChannel := make(chan bool)
    var wg sync.WaitGroup
    wg.Add(1)
    go func() {
        defer wg.Done()
        fmt.Println("Goroutine 1 is waiting for a signal...")
        <-signalChannel
        fmt.Println("Goroutine 1 received the signal and is now doing something.")
    }()
    wg.Add(1)
    go func() {
        defer wg.Done()
        fmt.Println("Goroutine 2 is about to send a signal.")
        signalChannel <- true
        fmt.Println("Goroutine 2 sent the signal.")
    }()
    wg.Wait()
    fmt.Println("Both goroutines have finished.")
}

In this snippet, we create a channel called **signalChannel** to signal between the two goroutines. **Goroutine 1** waits for a signal on the channel using **<-signalChannel**, and **Goroutine 2** sends a signal using **signalChannel <-** **true**.

The **sync.WaitGroup** ensures that we wait for both goroutines to finish before printing **"Both goroutines** **have finished."**.

When you run this program, you’ll see that **Goroutine 1** waits for the signal from **Goroutine 2** and then proceeds with its task.

Go channels are a flexible way to synchronize and coordinate complex interactions between goroutines. They can be used to implement concurrency patterns producer-consumer or fan-out/fan-in.

## Choosing your synchronization mechanism

Are channels always the answer? Definitely not! We can use mutexes or channels to solve the same problem. How do we choose? Prefer pragmatism. When mutexes make your solution easy to read and maintain, don’t think twice and go with mutexes!

If you have trouble choosing between them, here is an opinionated guideline.

Use channels when you need to do the following:

- Pass the ownership of data
- Distribute units of work
- Communicate results in an asynchronous way

Use mutexes when you’re handling the following:

- Caches
- Shared state

Alright, let’s wrap things up and recap what we’ve covered in this chapter.

# Summary

In this chapter, we learned about the functioning of goroutines, their simplicity, and the importance of synchronization using **WaitGroup**. We also became aware of the difficulties in managing shared state, using a warehouse analogy to explain data races. Additionally, we were introduced to Go’s race detection tool to identify race conditions, the significance of communication channels, and their potential pitfalls.

Now that our concurrency knowledge is refreshed, let’s explore in the next chapter interactions with an operational system using system calls.