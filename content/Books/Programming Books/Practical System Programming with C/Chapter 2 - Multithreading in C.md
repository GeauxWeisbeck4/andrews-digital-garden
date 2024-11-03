---
id: 01JBK6NJATR8BQK19CE0PJ9TMG
modified: 2024-11-01T08:48:02-04:00
title: Chapter 2 - Multithreading in C
tags:
  - c-lang
  - multithreading
  - linux
  - systems-programming
  - books
---
# 2. Multithreading in C

Sri Manikanta Palakollu[1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Aff2) 

(1)

freelance, Hanuman Junction, Hanuman Junction, 521105, Andhra Pradesh, India

Multithreading is a program’s ability to execute multiple threads simultaneously to maximize the utilization of the CPU. Multithreading helps achieve concurrency. Concurrenscy is parallelly executing multiple threads at the same time. In this chapter, you learn about the following topics with practical coding.

- Introduction to threads and thread behavior
    
- The difference between threads and processes
    
- Concurrency
    
- Parallelism
    
- Introduction to multithreading
    
- Importance of multithreading
    
- Multithreading API in C
    
- Creating multithreading programs in C
    
- Practical examples of multithreading
    
- Multithreading use cases
    

## Introduction to Threads

A thread is a lightweight process that shares a common address space with the owner process. Threads are very helpful in performing parallel programming tasks to achieve concurrency. Applications such as video editing software, web servers, online conferencing software, and text editors use multiple threads to do their jobs more efficiently. A thread is a small segment in a process, as shown in Figure [2-1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Fig1).

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig1_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig1_HTML.jpg)

Figure 2-1

Multithreaded process

Every thread has a program counter and stack space for storing an activity. The usage of threads greater benefits than processes. The following are the main advantages of using threads.

- They are easy to create and handle.
    
- They achieve concurrency in parallel programming.
    
- They reduce context switching time in an operating system.
    
- They can effectively utilize multiprocessor architecture.
    
- Thread communication is much faster than process communication because threads share common address space.
    
- They increase the overall performance of a system.
    

## Thread Classification

Threads are classified into two types: user-level threads and kernel-level threads.

### User-Level Threads

The user creates user-level threads with thread libraries rather than system calls. These threads are independent of the kernel. The user does thread management according to his needs and requirements. For example, creating a thread is done by the user. The thread library performs thread management in the user space. It doesn’t depend on the operating system’s system calls.

The creation of user-level threads is much faster than kernel-level threads. User-level threads don’t depend on hardware utility. Context switching is easier with user-level threads because it is done in the user space with a thread library. But, when it comes to kernel-level threads, context switching is done in a kernel space. In a kernel space, there might be a situation where more than one thread is in an active state at a particular time. If the multiple threads are in a kernel space, then it takes some extra time for context switching.

In general, context switching in user-level threads is faster than in kernel-level threads, but this may change based on the situation. User-level threads are represented by a program counter. Some good examples of user-level threads are POSIX threads and Java threads.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig2_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig2_HTML.jpg)

Figure 2-2

User-level threads working mechanism

The following are the advantages of user-level threads.

- They are easy to create.
    
- They are platform-independent, which means they can run on any operating system.
    
- Kernel-mode privileges are not required for thread switching.
    
- Context switching is very easy for an operating system.
    
- They don’t depend upon the system hardware.
    

The following are the disadvantages of user-level threads.

- Multiprocessing is very difficult because it is independent of the kernel. When you want to perform a multiprocessing task in an operating system, kernel support is required to execute the task. This is impossible with these threads.
    
- They require nonblocking I/O calls; otherwise, the entire process may be blocked in the kernel.
    

### Kernel-Level Threads

Kernel-level threads are created by the operating system directly. The kernel does thread management with a thread table that helps the kernel monitor all the activities done by a thread in the system. Some good examples of kernel-level threads are Win32 and Solaris.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig3_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig3_HTML.jpg)

Figure 2-3

Kernel-level threads working mechanism

The following are the advantages of kernel-level threads.

- Multiprocessing is done very easily.
    
- They work on the blocking I/O protocol.
    

The following are the disadvantages of kernel-level threads.

- They are slow and inefficient compared to user-level threads.
    
- They are very hard to create and manage.
    
- Multiple switching is required to transfer control from one thread to another thread.
    

## Threads vs. Processes

Even though threads are small segments inside a process, there are differences between threads and processes in terms of parameters. A thread can do all the tasks that a process can do, but the major difference between these two is that a process is a program or software that is executing. Threads are a lightweight segment of a process. A process can have multiple child processes. A process and a thread have a similar life cycle that consists of five stages.

1. 1.
    
    New
    
2. 2.
    
    Ready
    
3. 3.
    
    Wait
    
4. 4.
    
    Running
    
5. 5.
    
    Terminated
    

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig4_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig4_HTML.jpg)

Figure 2-4

Process and thread life cycle

The diagram highlights the five stages of the thread and process life cycle. Now let’s discuss each stage.

### New State

In the new state, a new process or thread is created and added to the queue. If it is a process, it is created by a system call based on user input and then added to the queue. Generally, a process is created with fork(). Threads are created based on the programmer’s code and added to the queue explicitly. In a new state, the thread or process has been created, but it is not running.

### Ready State

In the ready state, a process or a thread is ready for execution.

### Wait State

In the wait state, a process or thread has been blocked for some reason and for some amount of time. When it resumes, it goes to the ready state; otherwise, the thread/process is terminated.

### Running State

The running state describes the execution of the process/thread. In this state, if the sleep method is called on a thread/process, it goes to the wait state.

### Terminated

When the execution of a process or thread has been completed, it moves to the termination state.

The differences between threads and processes are based on certain characteristics, as shown in Table [2-1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Tab1).

Table 2-1

Relationship Between Threads and Processes

|Characteristics|Threads|Processes|
|---|---|---|
|**Definition**|A thread is a lightweight segment that is a part of a process.|A process is any program or software that is executing.|
|**Creation Time**|Threads usually take very little time to create since they are lightweight.|A process requires more time to create because the process is heavier than a thread.|
|**Termination Time**|A thread requires little time to terminate because of its simple nature.|A process requires more time to terminate because of its complex structure.|
|**Resource Usage**|A thread needs a minimal number of resources to do its task.|A process uses more resources than threads.|
|**Memory Sharing**|Threads share memory with other threads based on the task to perform.|All the processes created in an operating system are isolated. They don’t communicate with any other processes.|
|**Communication**|Threads can communicate with other threads within the same process more effectively than a process.|Processes are less efficient than threads.|
|**Context Switching**|A thread requires less time for context switching in an OS. It is less expensive.|A process requires more time for context switching because of its heaviness. It is more expensive in processes.|
|**Management**|Threads do not depend on any OS system calls.|A process depends on system calls.|

## Introduction to Multithreading

In your operating system, generally, you can perform multiple tasks at the same time. For example, listening to music on iTunes while writing code in a text editor is considered multitasking. In multitasking, computer applications execute different tasks simultaneously. Multithreading is very similar to multitasking, but the key difference is that multitasking works on the process, whereas multithreading works on threads.

### Multitasking Architecture

When the CPU executes multiple tasks with the process by switching between activities in a minimal amount of time, multitasking enables users to interact with several applications at the same time. In multitasking architecture, the execution of a task is done by the processor by sharing memory space and allocation for each task.

In the architecture shown in Figure [2-5](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Fig5), an operating system is running four different applications (i.e., a text editor, iTunes, Google Chrome, and Keynote software). Consider a situation where the user is writing code using a text editor while listening to music on iTunes. He has a list of features that need to be implemented, which is available in a Keynote file, so he opens the Keynote application. In the middle of development, he wants to refer to official documentation regarding the application, so he opens Google Chrome. Four applications are running parallel on his operating system. This situation is called _multitasking_.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig5_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig5_HTML.jpg)

Figure 2-5

Multitasking architecture

### Multithreading Architecture

In a multithreading architecture, a single process creates multiple threads to execute a task. In this architecture, common memory space and allocation is shared by all the threads. A multithreading architecture occurs within a process based on multiple threads.

Let’s consider a situation in which a user is writing code in Visual Studio Code (a.k.a. VS Code). To develop code in VS Code, the operating system creates a single process for that task. A process creates multiple threads to effectively execute a task. A new thread is created for each of the following: when you open an integrated terminal in VS Code, when you open IntelliSense to write code, and when you format the code. In multithreading, some threads communicate with each other based on the situation. So, a thread is created to perform a task more effectively and without any lags during its execution.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig6_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig6_HTML.jpg)

Figure 2-6

Multithreading architecture

### Importance of Multithreading

A program executing a task with a single thread is not always effective, especially for video editing software, code editors, and so forth. To use powerful applications that have many tasks to do simultaneously within an application, then multithreading is very helpful.

High-level programming languages like C, C++, Java, and Python have a single thread by default—without creating anything. In the C language, the main function creates a single thread in the background by default; it works as a background thread, and it does its job as assigned by the compiler. Executing higher-order tasks with a single thread is not efficient, which leads to the need for multithreading.

The importance of multithreading in modern application development is due to its advantages over a single-threaded architecture. Multithreading architecture has many benefits, which are discussed next.

#### Efficient Resource Sharing

An application that executes a task may require common resources to share among multiple threads. This is done with standard techniques, such as message passing and shared memory (which are covered in upcoming chapters), which are very efficient in multithreading because of the common memory address space.

#### Application Scalability

Multithreading increases the scalability of the application because the application’s subactivities are easily performed with multiple threads. For instance, a single-threaded application runs only on a single processor regardless of the number of processors. But a multithreaded application utilizes the multiple cores that are available in a machine. So, a multithreaded application can increase the parallelism in an operating system that has multiple core CPUs.

#### Responsiveness

Multithreading operations are performed in an application. All the internal threads work together to provide efficiency and a good user experience by increasing the responsiveness of the application.

#### Efficient Memory Utilization

In a multithreading environment, there is no need to create a separate memory for each thread. All threads share the same address space, so they use the memory effectively without creating new allocation spaces.

#### Efficient CPU Utilization

The creation of multiple threads within a process to execute a single application task increases speed and performance. Threads within a process utilize the improved CPU resources to perform the assigned task. Because threads use resources that are allocated by the process, indirectly, they are using better CPU resources, which results in efficient CPU utilization.

## Concurrency

Concurrency is a mechanism that decreases the response time of the system by using a single processing unit. In concurrency, a major task is divided into subtasks that execute simultaneously but not at the same time. A good example of concurrency is having multiple applications, like a Chrome browser, a video editor, and iTunes running at the same time in an operating system.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig7_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig7_HTML.jpg)

Figure 2-7

Concurrency mechanism process execution time

In Figure [2-7](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Fig7), five processes are executing simultaneously but not at the same time. You can observe the gap between the process executions. Concurrency creates an illusion that all processes are running simultaneously at the same time, but concurrency hides the latency time between process executions.

## Parallelism

Parallelism is a mechanism that increases computational speed by using multiple processors. In parallelism, tasks execute simultaneously and at the same time. A good example of parallelism is running a video editor that has many tasks to perform simultaneously.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig8_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig8_HTML.jpg)

Figure 2-8

Parallelism mechanism process execution time

In Figure [2-8](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Fig8), all the processes are executing simultaneously and at the same time. This simultaneous execution results in an increase in the system’s speed.

## Support of Multithreading in C

The C programming language does not have built-in library support for multithreaded programming. Even though C is a general-purpose programming language that is widely used in embedded systems, system programming, and so forth, some vendors have developed libraries that deal with multithreading to achieve parallelism and concurrency.

The library to develop portable multithreaded applications is pthread.h; that is, the POSIX thread library. POSIX stands for _portable operating system interface_. POSIX threads are lightweight and designed to be very easy to implement. The pthread.h library is an external third-party library that helps you effectively do tasks.

Note

You can develop multithreaded programs with the pthread.h library in all Unix-based operating systems but not for Windows. If you want to develop an application that works on Windows, the windows.h library is very effective for multithreading-supported Windows operating systems.

The following are the functions in the pthread.h library that create, manipulate, and exit the threads.

- pthread_create
    
- pthread_join
    
- pthread_self
    
- pthread_equal
    
- pthread_exit
    
- pthread_cancel
    
- pthread_detach
    

### pthread_create

pthread_create creates a new thread with a thread descriptor. A descriptor is an information container of the thread state, execution status, the process that it belongs to, related threads, stack reference information, and thread-specific resource information allocated by the process. This function takes four arguments as parameters. The return type of this function is an integer.

The following shows the syntax.

int pthread_create(pthread_t *thread,

                   const pthread_attr_t *attr,

                   void * (*start_routine)(void *),

                   void *arg);

The following describes the parameters.

- pthread_t is a thread descriptor variable that takes the thread descriptor, has an argument, and returns the thread ID, which is an unsigned long integer.
    
- pthread_attr_t is an argument that determines all the properties assigned to a thread. If it is a normal default thread, then you set the attribute value to NULL; otherwise, the argument is changed based on the programmer’s requirements.
    
- start_routine is an argument that points to the subroutines that execute by thread. The return type for this parameter is an void type because it typecasts return types explicitly. This argument takes a single value as a parameter. If you want to pass multiple arguments, a heterogeneous datatype should be passed that might be a struct.
    
- args is a parameter that depends on the previous parameter; it takes multiple parameters as an argument.
    

### pthread_join

This function waits for the termination of another thread. It takes two parameters as arguments and returns the integer type. It returns 0 on successful termination and –1 if any failure occurs.

The following shows the syntax.

int pthread_join(pthread_t, *thread,

                 void **thread_return)

The following describes the parameters.

- thread takes the ID of the thread that is currently waiting for termination
    
- thread_return is an argument that points to the exit status of the termination thread, which is a NULL value.
    

### pthread_self

This function returns the thread ID of the currently running thread. The return type of this thread is an integer or the thread_t descriptor. It takes zero parameters as arguments.

The following shows the syntax.

pthread_t pthread_self()

or

int pthread_self()

### pthread_equal

This function checks whether two threads are equal or not. If the two threads are equal, then the function returns a nonzero value. If the threads are not equal, then it is zero. It takes two parameters as arguments and returns the integer as output.

The following shows the syntax.

int pthread_equal(pthread_t thread1,

                  pthread_t thread2);

The following describes the parameters.

- thread1 and thread2 are the IDs for the first and second thread, respectively.
    

### pthread_exit

This function terminates a calling thread. It takes one argument as a parameter and returns nothing.

The following shows the syntax.

void pthread_exit(void *retval);

The following describes the parameters.

- retval is the return value of a thread that you want to detach it.
    

### pthread_cancel

This function is used for thread cancellation. It takes one parameter as an argument and returns an integer value.

The following shows the syntax.

int pthread_cancel(pthread_t thread);

The following describes the parameter.

- pthread is the thread ID of the thread that you want to cancel.
    

### pthread_detach

This function detaches a thread in a detached state. It takes a thread descriptor as an argument and returns the integer value as output.

The following shows the syntax.

int pthread_detach(pthread_t thread);

The following describes the parameter.

- thread is a descriptor variable that is passed as an ID, which you want to detach it.
    

These functions are the most common functions in multithreading operations.

## Creating Threads

As discussed earlier, the pthread_create function creates threads. This section deals with thread creation and how to execute multithreaded programs. It examines the weird behavior of multithreaded programs and how to overcome ambiguous output during the development process. A simple multithreaded program can be created in simple three steps.

1. 1.
    
    Import the required libraries. Our program includes the headers that are necessary for the operation.
    
    **#include<stdio.h>   // Standard I/O Routines Library**
    
    **#include<unistd.h>  // Unix Standard Library**
    
    **#include<pthread.h> // POSIX Thread Creation Library**
    
2. 2.
    
    Develop the thread function to make it multithreaded. The thread function must have a return type as a pointer.
    
    **void *customThreadFunction(){**
    
       **for(int i = 0; i < 15; i++){**
    
           **printf("I am a Custom Thread Function Created By Programmer.\n");**
    
           **sleep(1);**
    
       **}**
    
       **return NULL;**
    
    **}**
    

In this custom thread function, a for loop is written, which iterates 15 times [0–14] and prints the statement using the printf function. It sleeps for one second after the iteration of every print statement. sleep() is available in the unistd.h library.

1. 3.
    
    In this step, the main function comes into the picture. In the main function, you need to create a thread descriptor variable and method. The following program tells you whether a thread is created successfully or not. The pthread_create function returns the status codes 0 and 1 for success or failure. If the thread is successful, the thread function executes; otherwise, you exit out of the program.
    
    In the pthread_create function , the first argument is the address of the thread descriptor variable. Since this deals with the custom default threads, don’t bother with the second argument; take it as a NULL value. The third value is a custom thread function that is executed in the thread. The last argument is also NULL because, in this custom thread function, there aren’t any arguments to pass.
    

**int main(){**

       **pthread_t thread;   // Thread Descriptor**

       **int status;         // Status Variable to store the Status of the thread.**

       **status = pthread_create(&thread, NULL, customThreadFunction, NULL);**

       **/*  status = 0 ==> If thread is created Sucessfully.**

           **status = 1 ==> If thread is unable to create.   */**

       **if(!status){**

           **printf("Custom Created Successfully.\n");**

       **}else{**

           **printf("Unable to create the Custom Thread.\n");**

           **return 0;**

       **}**

       **// Main Function For loop**

       **for(int i = 0; i < 15; i++){**

           **printf("I am the process thread created by compiler By default.\n");**

           **sleep(1);**

       **}**

       **return 0;**

    **}**

After the compilation is done, run the ./a.out command, which gives the output for your program.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig9_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig9_HTML.jpg)

Figure 2-9

The output of the thread creation program

The output of my program may differ from yours. This is the ambiguity that is hidden inside multithreaded programs.

Next, let’s learn how to overcome this ambiguity by using different functions in our program.

## Practical Examples of Multithreading

### Thread Termination

Termination of a thread is mandatory in certain situations. In our program, let’s terminate the thread after the three iterations. It is done using the pthread_exit function, which takes a single argument. Let’s pass that argument as NULL since this deals with the default threads. You can see the difference in the thread termination in the output.

**#include<stdio.h>   // Standard I/O Routines Library**

**#include<unistd.h>  // Unix Standard Library**

**#include<pthread.h> // POSIX Thread Creation Library**

**void *customThreadFunction(){**

   **for(int i = 0; i < 5; i++){**

       **printf("I am a Custom Thread Function Created By Programmer.\n");**

       **sleep(1);**

       **if(i == 3){**

           **printf("My** **JOB** **is Done. I am now being terminated by programmer.\n");**

           **pthread_exit(NULL);**

       **}**

   **}**

   **return NULL;**

**}**

**int main(){**

   **pthread_t thread;   // Thread Descriptor**

   **pthread_create(&thread, NULL, customThreadFunction, NULL);**

   **for(int i = 0; i < 5; i++){**

       **printf("I am the process thread created by compiler By default.\n");**

       **sleep(1);**

   **}**

   **return 0;**

**}**

The output of the program is shown in Figure [2-10](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Fig10).

### Thread Equal Property

If you want to check whether two threads are equal or not, use the thread_equal function, which checks the equality condition.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig10_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig10_HTML.jpg)

Figure 2-10

The output of the thread termination program

**#include<stdio.h>   // Standard I/O Routines Library**

**#include<unistd.h>  // Unix Standard Library**

**#include<pthread.h> // POSIX Thread Creation Library**

**void *customThreadFunction(){**

   **printf("This is my custom thread\n");**

   **return NULL;**

**}**

**int main(){**

   **pthread_t thread1, thread2;**

   **pthread_create(&thread1, NULL, customThreadFunction, NULL);**

   **pthread_create(&thread2, NULL, customThreadFunction, NULL);**

   **if(pthread_equal(thread1, thread2)){**

       **printf("Two threads are Equal..!\n");**

   **}else{**

       **printf("Two threads are not equal\n");**

   **}**

   **return 0;**

**}**

The output is shown in Figure [2-11](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Fig11).

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig11_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig11_HTML.jpg)

Figure 2-11

The output of the thread equal program

### Passing a Single Argument to a Thread Function

Passing arguments to the thread function is done with a few changes to the pthread_create function arguments, as shown in the following code.

**#include <stdio.h>**

**#include <pthread.h>**

**void *sayGreetings(void *input) {**

   **printf("Hello %s\n", (char *)input);**

   **pthread_exit(NULL);**

**}**

**int main() {**

   **char name[50];**

   **printf("Enter your name: \n");**

   **fgets(name,50, stdin);**

   **pthread_t thread;**

   **pthread_create(&thread, NULL, sayGreetings, name);**

   **pthread_join(thread, NULL);**

   **return 0;**

**}**

The pthread_create function takes four arguments as parameters. In general, if a custom thread does not take any arguments, then you should pass fourth parameter as a NULL value. If your custom thread function requires a single parameter as an argument, then you pass that variable to the fourth variable. Passing multiple arguments is discussed in the next example.

Figure [2-12](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Fig12) shows the output.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig12_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig12_HTML.jpg)

Figure 2-12

Thread argument program output

### Passing Multiple Arguments as Parameters

If you want to pass multiple arguments for a custom thread function, then you use heterogeneous data types (i.e., structures that collect all your required data into one variable, which pass as a fourth argument in the pthread_create function).

**#include <stdio.h>**

**#include<stdlib.h>**

**#include <pthread.h>**

**// Data Collector.**

**struct arguments {**

   **char* name;**

   **int age;**

   **char *bloodGroup;**

**};**

**// Thread Function**

**void *sayGreetings(void *data) {**

   **printf("Name: %s", ((struct arguments*)data)->name);**

   **printf("Age: %d\n", ((struct arguments*)data)->age);**

   **printf("Blood Group: %s\n", ((struct arguments*)data)->bloodGroup);**

   **return NULL;**

**}**

**int main() {**

   **struct arguments *person = (struct arguments *)malloc(sizeof(struct arguments));**

   **printf("This is a Simple Data Collection Application\n");**

   **char bloodGroup[5], name[50];**

   **int age;**

   **printf("Enter the name of the person: ");**

   **fgets(name, 50, stdin);**

   **printf("Enter the age of the person: ");**

   **scanf("%d",&age);**

   **printf("Enter the person's Blood Group: ");**

   **scanf("%s", bloodGroup);**

   **person->name = name;**

   **person->age = age;**

   **person->bloodGroup = bloodGroup;**

   **pthread_t thread;**

   **pthread_create(&thread, NULL, sayGreetings, (void *)person);**

   **pthread_join(thread, NULL);**

   **return 0;**

**}**

This example creates a pointer that points to a struct, which is cast to a void pointer as (void *) and passes to the pthread_thread function.

Figure [2-13](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Fig13) shows the output.

![../images/497677_1_En_2_Chapter/497677_1_En_2_Fig13_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_2_Chapter/497677_1_En_2_Fig13_HTML.jpg)

Figure 2-13

Thread multiple argument output

## The Relationship Between Threads and the CPU

Table [2-2](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_2_Chapter.xhtml#Tab2) defines the relationship between threads and the CPU.

Table 2-2

Relationship Between Thread and CPU

|Parameters|Thread|CPU|
|---|---|---|
|**Definition**|A thread is a small segment in a process. It is a virtual component that manages tasks in an operating system.|The CPU is a hardware component that contains multiple cores. A core is a single computing component that helps the CPU execute and read program instructions. The number of cores is directly proportional to the processing speed.|
|**Work**|The process assigns a thread’s work.|The tasks are assigned by the threads to process.|
|**Task**|The task of the threads is to achieve concurrency.|The task of the CPU is to multitask and multiprogram.|
|**Dependent**|It is dependent on the CPU.|It is dependent on the core (i.e., internal component).|
|**Processing Units**|It requires multiple processing units.|It requires a single processing unit.|
|**Benefits**|Threads improve the throughput and computation speed.|It performs arithmetical and logical operations in a system.|
|**Example**|Running a visual code editor application is a multithreaded-based process|Running multiple applications, like a browser, code editor, and iTunes, at the same time.|

## Multithreading Use Cases

There are a lot of practical use cases that implement multithreading. All the topics in this chapter used socket programming, which is discussed in Chapter [8](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_8_Chapter.xhtml). But briefly, in socket programming, multithreading is helpful for listening to requests from various clients. This section looks at things that use multithreading. Here are some of the applications that use it.

- Web crawler applications
    
- Online booking applications, which runs on PHP or Java
    
- 3D games
    
- Integrated development environments
    
- Video editing software
    
- Text editors
    
- Word processors
    

And the list goes on, but here are a few types of applications that internally use multithreading. There are certain issues that programmers face during the usage of multithreading, but handling these issues is done with synchronization. All these topics are covered in the upcoming chapters.

## Summary

In this chapter, you were introduced to threads on an operating system. The chapter also discussed the following.

- The life cycle of a process and thread
    
- The importance of multithreading and its support in C
    
- Creating a thread in C and examples in the pthread.h library
    
- The differences between a CPU and threads
    
- Multithreading use cases