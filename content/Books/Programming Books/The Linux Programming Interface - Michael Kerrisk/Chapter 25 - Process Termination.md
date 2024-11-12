---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 25 - Process Termination
modified: 2024-11-11T19:24:39-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **25**  
**PROCESS TERMINATION**

This chapter describes what happens when a process terminates. We begin by describing the use of _exit()_ and __exit()_ to terminate a process. We then discuss the use of exit handlers to automatically perform cleanups when a process calls _exit()_. We conclude by considering some interactions between _fork()_, _stdio_ buffers, and _exit()_.

### **25.1 Terminating a Process: __exit()_ and _exit()_**

A process may terminate in two general ways. One of these is _abnormal_ termination, caused by the delivery of a signal whose default action is to terminate the process (with or without a core dump), as described in [Section 20.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec01). Alternatively, a process can terminate _normally_, using the __exit()_ system call.

#include <unistd.h>  
  
void _exit(int status);

The _status_ argument given to __exit()_ defines the _termination status_ of the process, which is available to the parent of this process when it calls _wait()_. Although defined as an _int_, only the bottom 8 bits of _status_ are actually made available to the parent. By convention, a termination status of 0 indicates that a process completed successfully, and a nonzero status value indicates that the process terminated unsuccessfully. There are no fixed rules about how nonzero status values are to be interpreted; different applications follow their own conventions, which should be described in their documentation. SUSv3 specifies two constants, EXIT_SUCCESS (0) and EXIT_FAILURE (1), that are used in most programs in this book.

A process is always successfully terminated by __exit()_ (i.e., __exit()_ never returns).

Although any value in the range 0 to 255 can be passed to the parent via the _status_ argument to __exit()_, specifying values greater than 128 can cause confusion in shell scripts. The reason is that, when a command is terminated by a signal, the shell indicates this fact by setting the value of the variable _$?_ to 128 plus the signal number, and this value is indistinguishable from that yielded when a process calls __exit()_ with the same _status_ value.

Programs generally don’t call __exit()_ directly, but instead call the _exit()_ library function, which performs various actions before calling __exit()_.

#include <stdlib.h>  
  
void exit(int status);

The following actions are performed by _exit()_:

• Exit handlers (functions registered with _atexit()_ and _on_exit()_) are called, in reverse order of their registration ([Section 25.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch25.xhtml#ch25lev1sec03)).

• The _stdio_ stream buffers are flushed.

• The __exit()_ system call is invoked, using the value supplied in _status_.

Unlike __exit()_, which is UNIX-specific, _exit()_ is defined as part of the standard C library; that is, it is available with every C implementation.

One other way in which a process may terminate is to return from _main()_, either explicitly, or implicitly, by falling off the end of the _main()_ function. Performing an explicit _return n_ is generally equivalent to calling _exit(n)_, since the run-time function that invokes _main()_ uses the return value from _main()_ in a call to _exit()_.

There is one circumstance in which calling _exit()_ and returning from _main()_ are not equivalent. If any steps performed during exit processing access variables local to _main()_, then doing a return from _main()_ results in undefined behavior. For example, this could occur if a variable that is local to _main()_ is specified in a call to _setvbuf()_ or _setbuf()_ ([Section 13.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch13.xhtml#ch13lev1sec02)).

Performing a return without specifying a value, or falling off the end of the _main()_ function, also results in the caller of _main()_ invoking _exit()_, but with results that vary depending on the version of the C standard supported and the compilation options employed:

• In C89, the behavior in these circumstances is undefined; the program can terminate with an arbitrary _status_ value. This is the behavior that occurs by default with _gcc_ on Linux, where the exit status of the program is taken from some random value lying on the stack or in a particular CPU register. Terminating a program in this way should be avoided.

• The C99 standard requires that falling off the end of the main program should be equivalent to calling _exit(0)_. This is the behavior we obtain on Linux if we compile a program using _gcc –std=c99_.

### **25.2 Details of Process Termination**

During both normal and abnormal termination of a process, the following actions occur:

• Open file descriptors, directory streams ([Section 18.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch18.xhtml#ch18lev1sec08)), message catalog descriptors (see the _catopen(3)_ and _catgets(3)_ manual pages), and conversion descriptors (see the _iconv_open(3)_ manual page) are closed.

• As a consequence of closing file descriptors, any file locks ([Chapter 55](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch55.xhtml#ch55)) held by this process are released.

• Any attached System V shared memory segments are detached, and the _shm_nattch_ counter corresponding to each segment is decremented by one. (Refer to [Section 48.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch48.xhtml#ch48lev1sec08).)

• For each System V semaphore for which a _semadj_ value has been set by the process, that _semadj_ value is added to the semaphore value. (Refer to [Section 47.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch47.xhtml#ch47lev1sec08).)

• If this is the controlling process for a controlling terminal, then the SIGHUP signal is sent to each process in the controlling terminal’s foreground process group, and the terminal is disassociated from the session. We consider this point further in [Section 34.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch34.xhtml#ch34lev1sec06).

• Any POSIX named semaphores that are open in the calling process are closed as though _sem_close()_ were called.

• Any POSIX message queues that are open in the calling process are closed as though _mq_close()_ were called.

• If, as a consequence of this process exiting, a process group becomes orphaned and there are any stopped processes in that group, then all processes in the group are sent a SIGHUP signal followed by a SIGCONT signal. We consider this point further in [Section 34.7.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch34.xhtml#ch34lev2sec06).

• Any memory locks established by this process using _mlock()_ or _mlockall()_ ([Section 50.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch50.xhtml#ch50lev1sec02)) are removed.

• Any memory mappings established by this process using _mmap()_ are unmapped.

### **25.3 Exit Handlers**

Sometimes, an application needs to automatically perform some operations on process termination. Consider the example of an application library that, if used during the life of the process, needs to have some cleanup actions performed automatically when the process exits. Since the library doesn’t have control of when and how the process exits, and can’t mandate that the main program call a library-specific cleanup function before exiting, cleanup is not guaranteed to occur. One approach in such situations is to use an _exit handler_ (older System V manuals used the term _program termination routine_).

An exit handler is a programmer-supplied function that is registered at some point during the life of the process and is then automatically called during _normal_ process termination via _exit()_. Exit handlers are not called if a program calls __exit()_ directly or if the process is terminated abnormally by a signal.

To some extent, the fact that exit handlers are not called when a process is terminated by a signal limits their utility. The best we can do is to establish handlers for the signals that might be sent to the process, and have these handlers set a flag that causes the main program to call _exit()_. (Because _exit()_ is not one of the async-signal-safe functions listed in [Table 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table1), on [page 426](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#page_426), we generally can’t call it from a signal handler.) Even then, this doesn’t handle the case of SIGKILL, whose default action can’t be changed. This is one more reason we should avoid using SIGKILL to terminate a process (as noted in [Section 20.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec02)), and instead use SIGTERM, which is the default signal sent by the _kill_ command.

##### **Registering exit handlers**

The GNU C library provides two ways of registering exit handlers. The first method, specified in SUSv3, is to use the _atexit()_ function.

#include <stdlib.h>  
  
int atexit(void (*func)(void));

Returns 0 on success, or nonzero on error

The _atexit()_ function adds _func_ to a list of functions that are called when the process terminates. The function _func_ should be defined to take no arguments and return no value, thus having the following general form:

void  
func(void)  
{  
    /* Perform some actions */  
}

Note that _atexit()_ returns a nonzero value (not necessarily –1) on error.

It is possible to register multiple exit handlers (and even the same exit handler multiple times). When the program invokes _exit()_, these functions are called _in reverse order_ of registration. This ordering is logical because, typically, functions that are registered earlier are those that carry out more fundamental types of cleanups that may need to be performed after later-registered functions.

Essentially, any desired action can be performed inside an exit handler, including registering additional exit handlers, which are placed at the head of the list of exit handlers that remain to be called. However, if one of the exit handlers fails to return—either because it called __exit()_ or because the process was terminated by a signal (e.g., the exit handler called _raise()_)—then the remaining exit handlers are not called. In addition, the remaining actions that would normally be performed by _exit()_ (i.e., flushing _stdio_ buffers) are not performed.

SUSv3 states that if an exit handler itself calls _exit()_, the results are undefined. On Linux, the remaining exit handlers are invoked as normal. However, on some systems, this causes all of the exit handlers to once more be invoked, which can result in an infinite recursion (until a stack overflow kills the process). Portable applications should avoid calling _exit()_ inside an exit handler.

SUSv3 requires that an implementation allow a process to be able to register at least 32 exit handlers. Using the call _sysconf(_SC_ATEXIT_MAX)_, a program can determine the implementation-defined upper limit on the number of exit handlers that can be registered. (However, there is no way to find out how many exit handlers have already been registered.) By chaining the registered exit handlers in a dynamically allocated linked list, _glibc_ allows a virtually unlimited number of exit handlers to be registered. On Linux, _sysconf(_SC_ATEXIT_MAX)_ returns 2,147,483,647 (i.e., the maximum signed 32-bit integer). In other words, something else will break (e.g., lack of memory) before we reach the limit on the number of functions that can be registered.

A child process created via _fork()_ inherits a copy of its parent’s exit handler registrations. When a process performs an _exec()_, all exit handler registrations are removed. (This is necessarily so, since an _exec()_ replaces the code of the exit handlers along with the rest of the existing program code.)

We can’t deregister an exit handler that has been registered with _atexit()_ (or _on_exit()_, described below). However, we can have the exit handler check whether a global flag is set before it performs its actions, and disable the exit handler by clearing the flag.

Exit handlers registered with _atexit()_ suffer a couple of limitations. The first is that when called, an exit handler doesn’t know what status was passed to _exit()_. Occasionally, knowing the status could be useful; for example, we may like to perform different actions depending on whether the process is exiting successfully or unsuccessfully. The second limitation is that we can’t specify an argument to the exit handler when it is called. Such a facility could be useful to define an exit handler that performs different actions depending on its argument, or to register a function multiple times, each time with a different argument.

To address these limitations, _glibc_ provides a (nonstandard) alternative method of registering exit handlers: _on_exit()_.

#define _BSD_SOURCE          /* Or: #define _SVID_SOURCE */  
#include <stdlib.h>  
  
int on_exit(void (*func)(int, void *), void *arg);

Returns 0 on success, or nonzero on error

The _func_ argument of _on_exit()_ is a pointer to a function of the following type:

void  
func(int status, void *arg)  
{  
    /* Perform cleanup actions */  
}

When called, _func()_ is passed two arguments: the _status_ argument supplied to _exit()_, and a copy of the _arg_ argument supplied to _on_exit()_ at the time the function was registered. Although defined as a pointer type, _arg_ is open to programmer-defined interpretation. It could be used as a pointer to some structure; equally, through judicious use of casting, it could be treated as an integer or other scalar type.

Like _atexit()_, _on_exit()_ returns a nonzero value (not necessarily –1) on error.

As with _atexit()_, multiple exit handlers can be registered with _on_exit()_. Functions registered using _atexit()_ and _on_exit()_ are placed on the same list. If both methods are used in the same program, then the exit handlers are called in reverse order of their registration using the two methods.

Although more flexible than _atexit()_, _on_exit()_ should be avoided in programs intended to be portable, since it is not covered by any standards and is available on few other UNIX implementations.

##### **Example program**

[Listing 25-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch25.xhtml#ch25ex1) demonstrates the use of _atexit()_ and _on_exit()_ to register exit handlers. When we run this program, we see the following output:

$ ./exit_handlers  
on_exit function called: status=2, arg=20  
atexit function 2 called  
atexit function 1 called  
on_exit function called: status=2, arg=10

**Listing 25-1:** Using exit handlers

__________________________________________________ procexec/exit_handlers.c  
  
#define _BSD_SOURCE     /* Get on_exit() declaration from <stdlib.h> */  
#include <stdlib.h>  
#include "tlpi_hdr.h"  
  
static void  
atexitFunc1(void)  
{  
    printf("atexit function 1 called\n");  
}  
  
static void  
atexitFunc2(void)  
{  
    printf("atexit function 2 called\n");  
}  
  
static void  
onexitFunc(int exitStatus, void *arg)  
{  
    printf("on_exit function called: status=%d, arg=%ld\n",  
                exitStatus, (long) arg);  
}  
  
int  
main(int argc, char *argv[])  
{  
    if (on_exit(onexitFunc, (void *) 10) != 0)  
        fatal("on_exit 1");  
    if (atexit(atexitFunc1) != 0)  
        fatal("atexit 1");  
    if (atexit(atexitFunc2) != 0)  
        fatal("atexit 2");  
    if (on_exit(onexitFunc, (void *) 20) != 0)  
        fatal("on_exit 2");  
  
    exit(2);  
}  
__________________________________________________ procexec/exit_handlers.c

### **25.4 Interactions Between _fork()_, _stdio_ Buffers, and __exit()_**

The output yielded by the program in [Listing 25-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch25.xhtml#ch25ex2) demonstrates a phenomenon that is at first puzzling. When we run this program with standard output directed to the terminal, we see the expected result:

$ ./fork_stdio_buf  
Hello world  
Ciao

However, when we redirect standard output to a file, we see the following:

$ ./fork_stdio_buf > a  
$ cat a  
Ciao  
Hello world  
Hello world

In the above output, we see two strange things: the line written by _printf()_ appears twice, and the output of _write()_ precedes that of _printf()_.

**Listing 25-2:** Interaction of _fork()_ and _stdio_ buffering

_________________________________________________ procexec/fork_stdio_buf.c  
  
#include "tlpi_hdr.h"  
  
int  
main(int argc, char *argv[])  
{  
    printf("Hello world\n");  
    write(STDOUT_FILENO, "Ciao\n", 5);  
  
    if (fork() == -1)  
        errExit("fork");  
  
    /* Both child and parent continue execution here */  
    exit(EXIT_SUCCESS);  
}  
_________________________________________________ procexec/fork_stdio_buf.c

To understand why the message written with _printf()_ appears twice, recall that the _stdio_ buffers are maintained in a process’s user-space memory (refer to [Section 13.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch13.xhtml#ch13lev1sec02)). Therefore, these buffers are duplicated in the child by _fork()_. When standard output is directed to a terminal, it is line-buffered by default, with the result that the newline-terminated string written by _printf()_ appears immediately. However, when standard output is directed to a file, it is block-buffered by default. Thus, in our example, the string written by _printf()_ is still in the parent’s _stdio_ buffer at the time of the _fork()_, and this string is duplicated in the child. When the parent and the child later call _exit()_, they both flush their copies of the _stdio_ buffers, resulting in duplicate output.

We can prevent this duplicated output from occurring in one of the following ways:

• As a specific solution to the _stdio_ buffering issue, we can use _fflush()_ to flush the _stdio_ buffer prior to a _fork()_ call. Alternatively, we could use _setvbuf()_ or _setbuf()_ to disable buffering on the _stdio_ stream.

• Instead of calling _exit()_, the child can call __exit()_, so that it doesn’t flush _stdio_ buffers. This technique exemplifies a more general principle: in an application that creates child processes that don’t exec new programs, typically only one of the processes (most often the parent) should terminate via _exit()_, while the other processes should terminate via __exit()_. This ensures that only one process calls exit handlers and flushes _stdio_ buffers, which is usually desirable.

Other approaches that allow both the parent and child to call _exit()_ are possible (and sometimes necessary). For example, it may be possible to design exit handlers so that they operate correctly even if called from multiple processes, or to have the application install exit handlers only after the call to _fork()_. Furthermore, sometimes we may actually want all processes to flush their _stdio_ buffers after a _fork()_. In this case, we may choose to terminate the processes using _exit()_, or use explicit calls to _fflush()_ in each process, as appropriate.

The output of the _write()_ in the program in [Listing 25-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch25.xhtml#ch25ex2) doesn’t appear twice, because _write()_ transfers data directly to a kernel buffer, and this buffer is not duplicated during a _fork()_.

By now, the reason for the second strange aspect of the program’s output when redirected to a file should be clear. The output of _write()_ appears before that from _printf()_ because the output of _write()_ is immediately transferred to the kernel buffer cache, while the output from _printf()_ is transferred only when the _stdio_ buffers are flushed by the call to _exit()_. (In general, care is required when mixing _stdio_ functions and system calls to perform I/O on the same file, as described in [Section 13.7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch13.xhtml#ch13lev1sec07).)

### **25.5 Summary**

A process can terminate either abnormally or normally. Abnormal termination occurs on delivery of certain signals, some of which also cause the process to produce a core dump file.

Normal termination is accomplished by calling __exit()_ or, more usually, _exit()_, which is layered on top of __exit()_. Both __exit()_ and _exit()_ take an integer argument whose least significant 8 bits define the termination status of the process. By convention, a status of 0 is used to indicate successful termination, and a nonzero status indicates unsuccessful termination.

As part of both normal and abnormal process termination, the kernel performs various cleanup steps. Terminating a process normally by calling _exit()_ additionally causes exit handlers registered using _atexit()_ and _on_exit()_ to be called (in reverse order of registration), and causes _stdio_ buffers to be flushed.

##### **Further information**

Refer to the sources of further information listed in [Section 24.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch24.xhtml#ch24lev1sec06).

### **25.6 Exercise**

**25-1.**   If a child process makes the call _exit(–1)_, what exit status will be seen by the parent?

- Highlight
- Add Note
- Copy