---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 21 - Signals - Signal Handlers
modified: 2024-11-11T19:21:04-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **21**  
**SIGNALS: SIGNAL HANDLERS**

This chapter continues the description of signals begun in the previous chapter. It focuses on signal handlers, and extends the discussion started in [Section 20.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec04). Among the topics we consider are the following:

• how to design a signal handler, which necessitates a discussion of reentrancy and async-signal-safe functions;

• alternatives to performing a normal return from a signal handler, in particular, the use of a nonlocal goto for this purpose;

• handling of signals on an alternate stack;

• the use of the _sigaction()_ SA_SIGINFO flag to allow a signal handler to obtain more detailed information about the signal that caused its invocation; and

• how a blocking system call may be interrupted by a signal handler, and how the call can be restarted if desired.

### **21.1 Designing Signal Handlers**

In general, it is preferable to write simple signal handlers. One important reason for this is to reduce the risk of creating race conditions. Two common designs for signal handlers are the following:

• The signal handler sets a global flag and exits. The main program periodically checks this flag and, if it is set, takes appropriate action. (If the main program cannot perform such periodic checks because it needs to monitor one or more file descriptors to see if I/O is possible, then the signal handler can also write a single byte to a dedicated pipe whose read end is included among the file descriptors monitored by the main program. We show an example of this technique in [Section 63.5.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch63.xhtml#ch63lev2sec17).)

• The signal handler performs some type of cleanup and then either terminates the process or uses a nonlocal goto ([Section 21.2.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev2sec04)) to unwind the stack and return control to a predetermined location in the main program.

In the following sections, we explore these ideas, as well as other concepts that are important in the design of signal handlers.

#### **21.1.1 Signals Are Not Queued (Revisited)**

In [Section 20.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec10), we noted that delivery of a signal is blocked during the execution of its handler (unless we specify the SA_NODEFER flag to _sigaction()_). If the signal is (again) generated while the handler is executing, then it is marked as pending and later delivered when the handler returns. We also already noted that signals are not queued. If the signal is generated more than once while the handler is executing, then it is still marked as pending, and it will later be delivered only once.

That signals can “disappear” in this way has implications for how we design signal handlers. To begin with, we can’t reliably count the number of times a signal is generated. Furthermore, we may need to code our signal handlers to deal with the possibility that multiple events of the type corresponding to the signal have occurred. We’ll see an example of this when we consider the use of the SIGCHLD signal in [Section 26.3.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch26.xhtml#ch26lev2sec07).

#### **21.1.2 Reentrant and Async-Signal-Safe Functions**

Not all system calls and library functions can be safely called from a signal handler. To understand why requires an explanation of two concepts: reentrant functions and async-signal-safe functions.

##### **Reentrant and nonreentrant functions**

To explain what a reentrant function is, we need to first distinguish between single-threaded and multithreaded programs. Classical UNIX programs have a single _thread of execution_: the CPU processes instructions for a single logical flow of execution through the program. In a multithreaded program, there are multiple, independent, concurrent logical flows of execution within the same process.

In [Chapter 29](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch29.xhtml#ch29), we’ll see how to explicitly create programs that contain multiple threads of execution. However, the concept of multiple threads of execution is also relevant for programs that employ signal handlers. Because a signal handler may asynchronously interrupt the execution of a program at any point in time, the main program and the signal handler in effect form two independent (although not concurrent) threads of execution within the same process.

A function is said to be _reentrant_ if it can safely be simultaneously executed by multiple threads of execution in the same process. In this context, “safe” means that the function achieves its expected result, regardless of the state of execution of any other thread of execution.

The SUSv3 definition of a reentrant function is one “whose effect, when called by two or more threads, is guaranteed to be as if the threads each executed the function one after the other in an undefined order, even if the actual execution is interleaved.”

A function may be _nonreentrant_ if it updates global or static data structures. (A function that employs only local variables is guaranteed to be reentrant.) If two invocations of (i.e., two threads executing) the function simultaneously attempt to update the same global variable or data structure, then these updates are likely to interfere with each other and produce incorrect results. For example, suppose that one thread of execution is in the middle of updating a linked list data structure to add a new list item when another thread also attempts to update the same linked list. Since adding a new item to the list requires updating multiple pointers, if another thread interrupts these steps and updates the same pointers, chaos will result.

Such possibilities are in fact rife within the standard C library. For example, we already noted in [Section 7.1.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch07.xhtml#ch07lev2sec03) that _malloc()_ and _free()_ maintain a linked list of freed memory blocks available for reallocation from the heap. If a call to _malloc()_ in the main program is interrupted by a signal handler that also calls _malloc()_, then this linked list can be corrupted. For this reason, the _malloc()_ family of functions, and other library functions that use them, are nonreentrant.

Other library functions are nonreentrant because they return information using statically allocated memory. Examples of such functions (described elsewhere in this book) include _crypt()_, _getpwnam()_, _gethostbyname()_, and _getservbyname()_. If a signal handler also uses one of these functions, then it will overwrite information returned by any earlier call to the same function from within the main program (or vice versa).

Functions can also be nonreentrant if they use static data structures for their internal bookkeeping. The most obvious examples of such functions are the members of the _stdio_ library (_printf()_, _scanf()_, and so on), which update internal data structures for buffered I/O. Thus, when using _printf()_ from within a signal handler, we may sometimes see strange output—or even a program crash or data corruption—if the handler interrupts the main program in the middle of executing a call to _printf()_ or another _stdio_ function.

Even if we are not using nonreentrant library functions, reentrancy issues can still be relevant. If a signal handler updates programmer-defined global data structures that are also updated within the main program, then we can say that the signal handler is nonreentrant with respect to the main program.

If a function is nonreentrant, then its manual page will normally provide an explicit or implicit indication of this fact. In particular, watch out for statements that the function uses or returns information in statically allocated variables.

##### **Example program**

[Listing 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21ex1) demonstrates the nonreentrant nature of the _crypt()_ function ([Section 8.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch08.xhtml#ch08lev1sec05)). As command-line arguments, this program accepts two strings. The program performs the following steps:

1. Call _crypt()_ to encrypt the string in the first command-line argument, and copy this string to a separate buffer using _strdup()_.
    
2. Establish a handler for SIGINT (generated by typing _Control-C_). The handler calls _crypt()_ to encrypt the string supplied in the second command-line argument.
    
3. Enter an infinite for loop that uses _crypt()_ to encrypt the string in the first command-line argument and check that the returned string is the same as that saved in step 1.
    

In the absence of a signal, the strings will always match in step 3. However, if a SIGINT signal arrives and the execution of the signal handler interrupts the main program just after the execution of the _crypt()_ call in the for loop, but before the check to see if the strings match, then the main program will report a mismatch. When we run the program, this is what we see:

$ ./non_reentrant abc def  
Repeatedly type Control-C to generate SIGINT  
Mismatch on call 109871 (mismatch=1 handled=1)  
Mismatch on call 128061 (mismatch=2 handled=2)  
Many lines of output removed  
Mismatch on call 727935 (mismatch=149 handled=156)  
Mismatch on call 729547 (mismatch=150 handled=157)  
Type Control-\ to generate SIGQUIT  
Quit (core dumped)

Comparing the _mismatch_ and _handled_ values in the above output, we see that in the majority of cases where the signal handler is invoked, it overwrites the statically allocated buffer between the call to _crypt()_ and the string comparison in _main()_.

**Listing 21-1:** Calling a nonreentrant function from both _main()_ and a signal handler

____________________________________________________ signals/nonreentrant.c  
  
#define _XOPEN_SOURCE 600  
#include <unistd.h>  
#include <signal.h>  
#include <string.h>  
#include "tlpi_hdr.h"  
  
static char *str2;             /* Set from argv[2] */  
static volatile int handled = 0;        /* Counts number of calls to handler */  
  
static void  
handler(int sig)  
{  
    crypt(str2, "xx");  
    handled++;  
}  
  
int  
main(int argc, char *argv[])  
{  
    char *cr1;  
    int callNum, mismatch;  
    struct sigaction sa;  
  
    if (argc != 3)  
        usageErr("%s str1 str2\n", argv[0]);  
  
    str2 = argv[2];                      /* Make argv[2] available to handler */  
    cr1 = strdup(crypt(argv[1], "xx"));  /* Copy statically allocated string  
                                            to another buffer */  
    if (cr1 == NULL)  
        errExit("strdup");  
  
    sigemptyset(&sa.sa_mask);  
    sa.sa_flags = 0;  
    sa.sa_handler = handler;  
    if (sigaction(SIGINT, &sa, NULL) == -1)  
        errExit("sigaction");  
  
    /* Repeatedly call crypt() using argv[1]. If interrupted by a  
       signal handler, then the static storage returned by crypt()  
       will be overwritten by the results of encrypting argv[2], and  
       strcmp() will detect a mismatch with the value in 'cr1'. */  
  
    for (callNum = 1, mismatch = 0; ; callNum++) {  
        if (strcmp(crypt(argv[1], "xx"), cr1) != 0) {  
            mismatch++;  
            printf("Mismatch on call %d (mismatch=%d handled=%d)\n",  
                    callNum, mismatch, handled);  
        }  
    }  
}  
____________________________________________________ signals/nonreentrant.c

##### **Standard async-signal-safe functions**

An _async-signal-safe_ function is one that the implementation guarantees to be safe when called from a signal handler. A function is async-signal-safe either because it is reentrant or because it is not interruptible by a signal handler.

[Table 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table1) lists the functions that various standards require to be async-signal-safe. In this table, the functions whose names are not followed by a _v2_ or _v3_ were specified as async-signal-safe in POSIX.1-1990. SUSv2 added the functions marked _v2_ to the list, and those marked _v3_ were added by SUSv3. Individual UNIX implementations may make other functions async-signal-safe, but all standards-conformant UNIX implementations must ensure that at least these functions are async-signal-safe (if they are provided by the implementation; not all of these functions are provided on Linux).

SUSv4 makes the following changes to [Table 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table1):

• The following functions are removed: _fpathconf()_, _pathconf()_, and _sysconf()_.

• The following functions are added: _execl()_, _execv()_, _faccessat()_, _fchmodat()_, _fchownat()_, _fexecve()_, _fstatat()_, _futimens()_, _linkat()_, _mkdirat()_, _mkfifoat()_, _mknod()_, _mknodat()_, _openat()_, _readlinkat()_, _renameat()_, _symlinkat()_, _unlinkat()_, _utimensat()_, and _utimes()_.

**Table 21-1:** Functions required to be async-signal-safe by POSIX.1-1990, SUSv2, and SUSv3

|   |
|---|
|__Exit() (v3)_<br><br>__exit()_<br><br>_abort() (v3)_<br><br>_accept() (v3)_<br><br>_access()_<br><br>_aio_error() (v2)_<br><br>_aio_return() (v2)_<br><br>_aio_suspend() (v2)_<br><br>_alarm()_<br><br>_bind() (v3)_<br><br>_cfgetispeed()_<br><br>_cfgetospeed()_<br><br>_cfsetispeed()_<br><br>_cfsetospeed()_<br><br>_chdir()_<br><br>_chmod()_<br><br>_chown()_<br><br>_clock_gettime() (v2)_<br><br>_close()_<br><br>_connect() (v3)_<br><br>_creat()_<br><br>_dup()_<br><br>_dup2()_<br><br>_execle()_<br><br>_execve()_<br><br>_fchmod() (v3)_<br><br>_fchown() (v3)_<br><br>_fcntl()_<br><br>_fdatasync() (v2)_<br><br>_fork()_<br><br>_fpathconf() (v2)_<br><br>_fstat()_<br><br>_fsync() (v2)_<br><br>_ftruncate() (v3)_<br><br>_getegid()_<br><br>_geteuid()_<br><br>_getgid()_<br><br>_getgroups()_<br><br>_getpeername() (v3)_<br><br>_getpgrp()_<br><br>_getpid()_<br><br>_getppid()_<br><br>_getsockname() (v3)_<br><br>_getsockopt() (v3)_<br><br>_getuid()_<br><br>_kill()_<br><br>_link()_<br><br>_listen() (v3)_<br><br>_lseek()_<br><br>_lstat() (v3)_<br><br>_mkdir()_<br><br>_mkfifo()_<br><br>_open()_<br><br>_pathconf()_<br><br>_pause()_<br><br>_pipe()_<br><br>_poll() (v3)_<br><br>_posix_trace_event() (v3)_<br><br>_pselect() (v3)_<br><br>_raise() (v2)_<br><br>_read()_<br><br>_readlink() (v3)_<br><br>_recv() (v3)_<br><br>_recvfrom() (v3)_<br><br>_recvmsg() (v3)_<br><br>_rename()_<br><br>_rmdir()_<br><br>_select() (v3)_<br><br>_sem_post() (v2)_<br><br>_send() (v3)_<br><br>_sendmsg() (v3)_<br><br>_sendto() (v3)_<br><br>_setgid()_<br><br>_setpgid()_<br><br>_setsid()_<br><br>_setsockopt() (v3)_<br><br>_setuid()_<br><br>_shutdown() (v3)_<br><br>_sigaction()_<br><br>_sigaddset()_<br><br>_sigdelset()_<br><br>_sigemptyset()_<br><br>_sigfillset()_<br><br>_sigismember()_<br><br>_signal() (v2)_<br><br>_sigpause() (v2)_<br><br>_sigpending()_<br><br>_sigprocmask()_<br><br>_sigqueue() (v2)_<br><br>_sigset() (v2)_<br><br>_sigsuspend()_<br><br>_sleep()_<br><br>_socket() (v3)_<br><br>_sockatmark() (v3)_<br><br>_socketpair() (v3)_<br><br>_stat()_<br><br>_symlink() (v3)_<br><br>_sysconf()_<br><br>_tcdrain()_<br><br>_tcflow()_<br><br>_tcflush()_<br><br>_tcgetattr()_<br><br>_tcgetpgrp()_<br><br>_tcsendbreak()_<br><br>_tcsetattr()_<br><br>_tcsetpgrp()_<br><br>_time()_<br><br>_timer_getoverrun() (v2)_<br><br>_timer_gettime() (v2)_<br><br>_timer_settime() (v2)_<br><br>_times()_<br><br>_umask()_<br><br>_uname()_<br><br>_unlink()_<br><br>_utime()_<br><br>_wait()_<br><br>_waitpid()_<br><br>_write()_|

SUSv3 notes that all functions not listed in [Table 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table1) are considered to be unsafe with respect to signals, but points out that a function is unsafe only when invocation of a signal handler interrupts the execution of an unsafe function, and the handler itself also calls an unsafe function. In other words, when writing signal handlers, we have two choices:

• Ensure that the code of the signal handler itself is reentrant and that it calls only async-signal-safe functions.

• Block delivery of signals while executing code in the main program that calls unsafe functions or works with global data structures also updated by the signal handler.

The problem with the second approach is that, in a complex program, it can be difficult to ensure that a signal handler will never interrupt the main program while it is calling an unsafe function. For this reason, the above rules are often simplified to the statement that we must not call unsafe functions from within a signal handler.

If we set up the same handler function to deal with several different signals or use the SA_NODEFER flag to _sigaction()_, then a handler may interrupt itself. As a consequence, the handler may be nonreentrant if it updates global (or static) variables, even if they are not used by the main program.

##### **Use of _errno_ inside signal handlers**

Because they may update _errno_, use of the functions listed in [Table 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table1) can nevertheless render a signal handler nonreentrant, since they may overwrite the _errno_ value that was set by a function called from the main program. The workaround is to save the value of _errno_ on entry to a signal handler that uses any of the functions in [Table 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table1) and restore the _errno_ value on exit from the handler, as in the following example:

void  
handler(int sig)  
{  
    int savedErrno;  
  
    savedErrno = errno;  
  
    /* Now we can execute a function that might modify errno */  
  
    errno = savedErrno;  
}

##### **Use of unsafe functions in example programs in this book**

Although _printf()_ is not async-signal-safe, we use it in signal handlers in various example programs in this book. We do so because _printf()_ provides an easy and concise way to demonstrate that a signal handler has been called, and to display the contents of relevant variables within the handler. For similar reasons, we occasionally use a few other unsafe functions in signal handlers, including other _stdio_ functions and _strsignal()_.

Real-world applications should avoid calling non-async-signal-safe functions from signal handlers. To make this clear, each signal handler in the example programs that uses one of these functions is marked with a comment indicating that the usage is unsafe:

printf("Some message\n");         /* UNSAFE */

#### **21.1.3 Global Variables and the _sig_atomic_t_ Data Type**

Notwithstanding reentrancy issues, it can be useful to share global variables between the main program and a signal handler. This can be safe as long as the main program correctly handles the possibility that the signal handler may change the global variable at any time. For example, one common design is to make a signal handler’s sole action the setting of a global flag. This flag is periodically checked by the main program, which then takes appropriate action in response to the delivery of the signal (and clears the flag). When global variables are accessed in this way from a signal handler, we should always declare them using the volatile keyword (see [Section 6.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch06.xhtml#ch06lev1sec08)) in order to prevent the compiler from performing optimizations that result in the variable being stored in a register.

Reading and writing global variables may involve more than one machine-language instruction, and a signal handler may interrupt the main program in the middle of such an instruction sequence. (We say that access to the variable is _nonatomic_.) For this reason, the C language standards and SUSv3 specify an integer data type, _sig_atomic_t_, for which reads and writes are guaranteed to be atomic. Thus, a global flag variable that is shared between the main program and a signal handler should be declared as follows:

volatile sig_atomic_t flag;

We show an example of the use of the _sig_atomic_t_ data type in [Listing 22-5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex5), on [page 466](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#page_466).

Note that the C increment (++) and decrement (--) operators don’t fall within the guarantee provided for _sig_atomic_t_. On some hardware architectures, these operations may not be atomic (refer to [Section 30.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch30.xhtml#ch30lev1sec01) for more details). All that we are guaranteed to be safely allowed to do with a _sig_atomic_t_ variable is set it within the signal handler, and check it in the main program (or vice versa).

C99 and SUSv3 specify that an implementation should define two constants (in <stdint.h>), SIG_ATOMIC_MIN and SIG_ATOMIC_MAX, that define the range of values that may be assigned to variables of type _sig_atomic_t_. The standards require that this range be at least –127 to 127 if _sig_atomic_t_ is represented as a signed value, or 0 to 255 if it is represented as an unsigned value. On Linux, these two constants equate to the negative and positive limits for signed 32-bit integers.

### **21.2 Other Methods of Terminating a Signal Handler**

All of the signal handlers that we have looked at so far complete by returning to the main program. However, simply returning from a signal handler sometimes isn’t desirable, or in some cases, isn’t even useful. (We’ll see an example of where returning from a signal handler isn’t useful when we discuss hardware-generated signals in [Section 22.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec04).)

There are various other ways of terminating a signal handler:

• Use __exit()_ to terminate the process. Beforehand, the handler may carry out some cleanup actions. Note that we can’t use _exit()_ to terminate a signal handler, because it is not one of safe functions listed in [Table 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table1). It is unsafe because it flushes _stdio_ buffers prior to calling __exit()_, as described in [Section 25.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch25.xhtml#ch25lev1sec01).

• Use _kill()_ or _raise()_ to send a signal that kills the process (i.e., a signal whose default action is process termination).

• Perform a nonlocal goto from the signal handler.

• Use the _abort()_ function to terminate the process with a core dump.

The last two of these options are described in further detail in the following sections.

#### **21.2.1 Performing a Nonlocal Goto from a Signal Handler**

[Section 6.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch06.xhtml#ch06lev1sec08) described the use of _setjmp()_ and _longjmp()_ to perform a nonlocal goto from a function to one of its callers. We can also use this technique from a signal handler. This provides a way to recover after delivery of a signal caused by a hardware exception (e.g., a memory access error), and also allows us to catch a signal and return control to a particular point in a program. For example, upon receipt of a SIGINT signal (normally generated by typing _Control-C_), the shell performs a nonlocal goto to return control to its main input loop (and thus read a new command).

However, there is a problem with using the standard _longjmp()_ function to exit from a signal handler. We noted earlier that, upon entry to the signal handler, the kernel automatically adds the invoking signal, as well as any signals specified in the _act.sa_mask_ field, to the process signal mask, and then removes these signals from the mask when the handler does a normal return.

What happens to the signal mask if we exit the signal handler using _longjmp()_? The answer depends on the genealogy of the particular UNIX implementation. Under System V, _longjmp()_ doesn’t restore the signal mask, so that blocked signals are not unblocked upon leaving the handler. Linux follows the System V behavior. (This is usually not what we want, since it leaves the signal that caused invocation of the handler blocked.) Under BSD-derived implementations, _setjmp()_ saves the signal mask in its _env_ argument, and the saved signal mask is restored by _longjmp()_. (BSD-derived implementations also provide two other functions, __setjmp()_ and __longjmp()_, which have the System V semantics.) In other words, we can’t portably use _longjmp()_ to exit a signal handler.

If we define the _BSD_SOURCE feature test macro when compiling a program, then (the _glibc_) _setjmp()_ follows the BSD semantics.

Because of this difference in the two main UNIX variants, POSIX.1-1990 chose not to specify the handling of the signal mask by _setjmp()_ and _longjmp()_. Instead, it defined a pair of new functions, _sigsetjmp()_ and _siglongjmp()_, that provide explicit control of the signal mask when performing a nonlocal goto.

#include <setjmp.h>  
  
int sigsetjmp(sigjmp_buf env, int savesigs);

Returns 0 on initial call, nonzero on return via _siglongjmp()_

void siglongjmp(sigjmp_buf env, int val);

The _sigsetjmp()_ and _siglongjmp()_ functions operate similarly to _setjmp()_ and _longjmp()_. The only differences are in the type of the _env_ argument (_sigjmp_buf_ instead of _jmp_buf_) and the extra _savesigs_ argument to _sigsetjmp()_. If _savesigs_ is nonzero, then the process signal mask that is current at the time of the _sigsetjmp()_ call is saved in _env_ and restored by a later _siglongjmp()_ call specifying the same _env_ argument. If _savesigs_ is 0, then the process signal mask is not saved and restored.

The _longjmp()_ and _siglongjmp()_ functions are not listed among the async-signal-safe functions in [Table 21-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table1). This is because calling any non-async-signal-safe function after performing a nonlocal goto carries the same risks as calling that function from within the signal handler. Furthermore, if a signal handler interrupts the main program while it is part-way through updating a data structure, and the handler exits by performing a nonlocal goto, then the incomplete update may leave that data structure in an inconsistent state. One technique that can help to avoid problems is to use _sigprocmask()_ to temporarily block the signal while sensitive updates are being performed.

##### **Example program**

[Listing 21-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21ex2) demonstrates the difference in signal mask handling for the two types of nonlocal gotos. This program establishes a handler for SIGINT. The program is designed to allow either _setjmp()_ plus _longjmp()_ or _sigsetjmp()_ plus _siglongjmp()_ to be used to exit the signal handler, depending on whether the program is compiled with the USE_SIGSETJMP macro defined. The program displays the current settings of the signal mask both on entry to the signal handler and after the nonlocal goto has transferred control from the handler back to the main program.

When we build the program so that _longjmp()_ is used to exit the signal handler, this is what we see when we run the program:

$ make -s sigmask_longjmp         Default compilation causes setjmp() to be used  
$ ./sigmask_longjmp  
Signal mask at startup:  
                <empty signal set>  
Calling setjmp()  
Type Control-C to generate SIGINT  
Received signal 2 (Interrupt), signal mask is:  
                2 (Interrupt)  
After jump from handler, signal mask is:  
                2 (Interrupt)  
(At this point, typing Control-C again has no effect, since SIGINT is blocked)  
Type Control-\ to kill the program  
Quit

From the program output, we can see that, after a _longjmp()_ from the signal handler, the signal mask remains set to the value to which it was set on entry to the signal handler.

In the above shell session, we built the program using the makefile supplied with the source code distribution for this book. The _–s_ option tells _make_ not to echo the commands that it is executing. We use this option to avoid cluttering the session log. ([[Mecklenburg, 2005](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib68)] provides a description of the GNU _make_ program.)

When we compile the same source file to build an executable that uses _siglongjmp()_ to exit the handler, we see the following:

$ make -s sigmask_siglongjmp       Compiles using cc –DUSE_SIGSETJMP  
$ ./sigmask_siglongjmp  
Signal mask at startup:  
                <empty signal set>  
Calling sigsetjmp()  
Type Control-C  
Received signal 2 (Interrupt), signal mask is:  
                2 (Interrupt)  
After jump from handler, signal mask is:  
                <empty signal set>

At this point, SIGINT is not blocked, because _siglongjmp()_ restored the signal mask to its original state. Next, we type _Control-C_ again, so that the handler is once more invoked:

Type Control-C  
Received signal 2 (Interrupt), signal mask is:  
                2 (Interrupt)  
After jump from handler, signal mask is:  
                <empty signal set>  
Type Control-\ to kill the program  
Quit

From the above output, we can see that _siglongjmp()_ restores the signal mask to the value it had at the time of the _sigsetjmp()_ call (i.e., an empty signal set).

[Listing 21-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21ex2) also demonstrates a useful technique for use with a signal handler that performs a nonlocal goto. Because a signal can be generated at any time, it may actually occur before the target of the goto has been set up by _sigsetjmp()_ (or _setjmp()_). To prevent this possibility (which would cause the handler to perform a nonlocal goto using an uninitialized _env_ buffer), we employ a guard variable, _canJump_, to indicate whether the _env_ buffer has been initialized. If _canJump_ is false, then instead of doing a nonlocal goto, the handler simply returns. An alternative approach is to arrange the program code so that the call to _sigsetjmp()_ (or _setjmp()_) occurs before the signal handler is established. However, in complex programs, it may be difficult to ensure that these two steps are performed in that order, and the use of a guard variable may be simpler.

Note that using #ifdef was the simplest way of writing the program in [Listing 21-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21ex2) in a standards-conformant fashion. In particular, we could not have replaced the #ifdef with the following run-time check:

if (useSiglongjmp)  
    s = sigsetjmp(senv, 1);  
else  
    s = setjmp(env);  
if (s == 0)  
    ...

This is not permitted because SUSv3 doesn’t allow _setjmp()_ and _sigsetjmp()_ to be used within an assignment statement (see [Section 6.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch06.xhtml#ch06lev1sec08)).

**Listing 21-2:** Performing a nonlocal goto from a signal handler

__________________________________________________signals/sigmask_longjmp.c  
  
#define _GNU_SOURCE     /* Get strsignal() declaration from <string.h> */  
#include <string.h>  
#include <setjmp.h>  
#include <signal.h>  
#include "signal_functions.h"           /* Declaration of printSigMask() */  
#include "tlpi_hdr.h"  
  
static volatile sig_atomic_t canJump = 0;  
                        /* Set to 1 once "env" buffer has been  
                           initialized by [sig]setjmp() */  
#ifdef USE_SIGSETJMP  
static sigjmp_buf senv;  
#else  
static jmp_buf env;  
#endif  
  
static void  
handler(int sig)  
{  
    /* UNSAFE: This handler uses non-async-signal-safe functions  
       (printf(), strsignal(), printSigMask(); see [Section 21.1.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev2sec02)) */  
  
    printf("Received signal %d (%s), signal mask is:\n", sig,  
            strsignal(sig));  
    printSigMask(stdout, NULL);  
  
    if (!canJump) {  
        printf("'env' buffer not yet set, doing a simple return\n");  
        return;  
    }  
  
#ifdef USE_SIGSETJMP  
    siglongjmp(senv, 1);  
#else  
    longjmp(env, 1);  
#endif  
}  
  
int  
main(int argc, char *argv[])  
{  
    struct sigaction sa;  
  
    printSigMask(stdout, "Signal mask at startup:\n");  
  
    sigemptyset(&sa.sa_mask);  
    sa.sa_flags = 0;  
    sa.sa_handler = handler;  
    if (sigaction(SIGINT, &sa, NULL) == -1)  
        errExit("sigaction");  
  
#ifdef USE_SIGSETJMP  
    printf("Calling sigsetjmp()\n");  
    if (sigsetjmp(senv, 1) == 0)  
#else  
    printf("Calling setjmp()\n");  
    if (setjmp(env) == 0)  
#endif  
        canJump = 1;                    /* Executed after [sig]setjmp() */  
  
    else                                /* Executed after [sig]longjmp() */  
        printSigMask(stdout, "After jump from handler, signal mask is:\n" );  
  
    for (;;)                            /* Wait for signals until killed */  
        pause();  
}  
__________________________________________________signals/sigmask_longjmp.c

#### **21.2.2 Terminating a Process Abnormally: _abort()_**

The _abort()_ function terminates the calling process and causes it to produce a core dump.

#include <stdlib.h>  
  
void abort(void);

The _abort()_ function terminates the calling process by raising a SIGABRT signal. The default action for SIGABRT is to produce a core dump file and terminate the process. The core dump file can then be used within a debugger to examine the state of the program at the time of the _abort()_ call.

SUSv3 requires that _abort()_ override the effect of blocking or ignoring SIGABRT. Furthermore, SUSv3 specifies that _abort()_ must terminate the process unless the process catches the signal with a handler that doesn’t return. This last statement requires a moment’s thought. Of the methods of terminating a signal handler described in [Section 21.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev1sec02), the one that is relevant here is the use of a nonlocal goto to exit the handler. If this is done, then the effect of _abort()_ will be nullified; otherwise, _abort()_ always terminates the process. In most implementations, termination is guaranteed as follows: if the process still hasn’t terminated after raising SIGABRT once (i.e., a handler catches the signal and returns, so that execution of _abort()_ is resumed), _abort()_ resets the handling of SIGABRT to SIG_DFL and raises a second SIGABRT, which is guaranteed to kill the process.

If _abort()_ does successfully terminate the process, then it also flushes and closes _stdio_ streams.

An example of the use of _abort()_ is provided in the error-handling functions of [Listing 3-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch03.xhtml#ch3ex3), on [page 54](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch03.xhtml#page_54).

### **21.3 Handling a Signal on an Alternate Stack: _sigaltstack()_**

Normally, when a signal handler is invoked, the kernel creates a frame for it on the process stack. However, this may not be possible if a process attempts to extend the stack beyond the maximum possible size. For example, this may occur because the stack grows so large that it encounters a region of mapped memory ([Section 48.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch48.xhtml#ch48lev1sec05)) or the upwardly growing heap, or it reaches the RLIMIT_STACK resource limit ([Section 36.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36lev1sec03)).

When a process attempts to grow its stack beyond the maximum possible size, the kernel generates a SIGSEGV signal for the process. However, since the stack space is exhausted, the kernel can’t create a frame for any SIGSEGV handler that the program may have established. Consequently, the handler is not invoked, and the process is terminated (the default action for SIGSEGV).

If we instead need to ensure that the SIGSEGV signal is handled in these circumstances, we can do the following:

1. Allocate an area of memory, called an _alternate signal stack_, to be used for the stack frame of a signal handler.
    
2. Use the _sigaltstack()_ system call to inform the kernel of the existence of the alternate signal stack.
    
3. When establishing the signal handler, specify the SA_ONSTACK flag, to tell the kernel that the frame for this handler should be created on the alternate stack.
    

The _sigaltstack()_ system call both establishes an alternate signal stack and returns information about any alternate signal stack that is already established.

#include <signal.h>  
  
int sigaltstack(const stack_t *sigstack, stack_t *old_sigstack);

Returns 0 on success, or –1 on error

The _sigstack_ argument points to a structure specifying the location and attributes of the new alternate signal stack. The _old_sigstack_ argument points to a structure used to return information about the previously established alternate signal stack (if there was one). Either one of these arguments can be specified as NULL. For example, we can find out about the existing alternate signal stack, without changing it, by specifying NULL for the _sigstack_ argument. Otherwise, each of these arguments points to a structure of the following type:

typedef struct {  
    void  *ss_sp;        /* Starting address of alternate stack */  
    int    ss_flags;     /* Flags: SS_ONSTACK, SS_DISABLE */  
    size_t ss_size;      /* Size of alternate stack */  
} stack_t;

The _ss_sp_ and _ss_size_ fields specify the size and location of the alternate signal stack. When actually using the alternate signal stack, the kernel automatically takes care of aligning the value given in _ss_sp_ to an address boundary that is suitable for the hardware architecture.

Typically, the alternate signal stack is either statically allocated or dynamically allocated on the heap. SUSv3 specifies the constant SIGSTKSZ to be used as a typical value when sizing the alternate stack, and MINSIGSTKSZ as the minimum size required to invoke a signal handler. On Linux/x86-32, these constants are defined with the values 8192 and 2048, respectively.

The kernel doesn’t resize an alternate signal stack. If the stack overflows the space we have allocated for it, then chaos results (e.g., overwriting of variables beyond the limits of the stack). This is not usually a problem—because we normally use an alternate signal stack to handle the special case of the standard stack overflowing, typically only one or a few frames are allocated on the stack. The job of the SIGSEGV handler is either to perform some cleanup and terminate the process or to unwind the standard stack using a nonlocal goto.

The _ss_flags_ field contains one of the following values:

SS_ONSTACK

If this flag is set when retrieving information about the currently established alternate signal stack (_old_sigstack_), it indicates that the process is currently executing on the alternate signal stack. Attempts to establish a new alternate signal stack while the process is already running on an alternate signal stack result in an error (EPERM) from _sigaltstack()_.

SS_DISABLE

Returned in _old_sigstack_, this flag indicates that there is no currently established alternate signal stack. When specified in _sigstack_, this disables a currently established alternate signal stack.

[Listing 21-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21ex3) demonstrates the establishment and use of an alternate signal stack. After establishing an alternate signal stack and a handler for SIGSEGV, this program calls a function that infinitely recurses, so that the stack overflows and the process is sent a SIGSEGV signal. When we run the program, this is what we see:

$ ulimit -s unlimited  
$ ./t_sigaltstack  
Top of standard stack is near 0xbffff6b8  
Alternate stack is at          0x804a948-0x804cfff  
Call    1 - top of stack near 0xbff0b3ac  
Call    2 - top of stack near 0xbfe1714c  
Many intervening lines of output removed  
Call 2144 - top of stack near 0x4034120c  
Call 2145 - top of stack near 0x4024cfac  
Caught signal 11 (Segmentation fault)  
Top of handler stack near      0x804c860

In this shell session, the _ulimit_ command is used to remove any RLIMIT_STACK resource limit that may have been set in the shell. We explain this resource limit in [Section 36.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36lev1sec03).

**Listing 21-3:** Using _sigaltstack()_

___________________________________________________ signals/t_sigaltstack.c  
  
#define _GNU_SOURCE         /* Get strsignal() declaration from <string.h> */  
#include <string.h>  
#include <signal.h>  
#include "tlpi_hdr.h"  
  
static void  
sigsegvHandler(int sig)  
{  
    int x;  
  
    /* UNSAFE: This handler uses non-async-signal-safe functions  
       (printf(), strsignal(), fflush(); see [Section 21.1.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev2sec02)) */  
  
    printf("Caught signal %d (%s)\n", sig, strsignal(sig));  
    printf("Top of handler stack near     %10p\n", (void *) &x);  
    fflush(NULL);  
  
    _exit(EXIT_FAILURE);                /* Can't return after SIGSEGV */  
}  
  
static void             /* A recursive function that overflows the stack */  
overflowStack(int callNum)  
{  
    char a[100000];                     /* Make this stack frame large */  
  
    printf("Call %4d - top of stack near %10p\n", callNum, &a[0]);  
    overflowStack(callNum+1);  
}  
  
int  
main(int argc, char *argv[])  
{  
    stack_t sigstack;  
    struct sigaction sa;  
    int j;  
  
    printf("Top of standard stack is near %10p\n", (void *) &j);  
  
    /* Allocate alternate stack and inform kernel of its existence */  
  
    sigstack.ss_sp = malloc(SIGSTKSZ);  
    if (sigstack.ss_sp == NULL)  
        errExit("malloc");  
    sigstack.ss_size = SIGSTKSZ;  
    sigstack.ss_flags = 0;  
    if (sigaltstack(&sigstack, NULL) == -1)  
        errExit("sigaltstack");  
    printf("Alternate stack is at         %10p-%p\n",  
            sigstack.ss_sp, (char *) sbrk(0) - 1);  
  
    sa.sa_handler = sigsegvHandler;     /* Establish handler for SIGSEGV */  
    sigemptyset(&sa.sa_mask);  
    sa.sa_flags = SA_ONSTACK;           /* Handler uses alternate stack */  
    if (sigaction(SIGSEGV, &sa, NULL) == -1)  
        errExit("sigaction");  
  
    overflowStack(1);  
}  
___________________________________________________ signals/t_sigaltstack.c

### **21.4 The** SA_SIGINFO **Flag**

Setting the SA_SIGINFO flag when establishing a handler with _sigaction()_ allows the handler to obtain additional information about a signal when it is delivered. In order to obtain this information, we must declare the handler as follows:

void handler(int sig, siginfo_t *siginfo, void *ucontext);

The first argument, _sig_, is the signal number, as for a standard signal handler. The second argument, _siginfo_, is a structure used to provide the additional information about the signal. We describe this structure below. The last argument, _ucontext_, is also described below.

Since the above signal handler has a different prototype from a standard signal handler, C typing rules mean that we can’t use the _sa_handler_ field of the _sigaction_ structure to specify the address of the handler. Instead, we must use an alternative field: _sa_sigaction_. In other words, the definition of the _sigaction_ structure is somewhat more complex than was shown in [Section 20.13](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec13). In full, the structure is defined as follows:

struct sigaction {  
    union {  
        void (*sa_handler)(int);  
        void (*sa_sigaction)(int, siginfo_t *, void *);  
    } __sigaction_handler;  
    sigset_t   sa_mask;  
    int        sa_flags;  
    void     (*sa_restorer)(void);  
};  
  
/* Following defines make the union fields look like simple fields  
   in the parent structure */  
  
#define sa_handler __sigaction_handler.sa_handler  
#define sa_sigaction __sigaction_handler.sa_sigaction

The _sigaction_ structure uses a union to combine the _sa_sigaction_ and _sa_handler_ fields. (Most other UNIX implementations similarly use a union for this purpose.) Using a union is possible because only one of these fields is required during a particular call to _sigaction()_. (However, this can lead to strange bugs if we naively expect to be able to set the _sa_handler_ and _sa_sigaction_ fields independently of one another, perhaps because we reuse a single _sigaction_ structure in multiple _sigaction()_ calls to establish handlers for different signals.)

Here is an example of the use of SA_SIGINFO to establish a signal handler:

struct sigaction act;  
  
sigemptyset(&act.sa_mask);  
act.sa_sigaction = handler;  
act.sa_flags = SA_SIGINFO;  
  
if (sigaction(SIGINT, &act, NULL) == -1)  
    errExit("sigaction");

For complete examples of the use of the SA_SIGINFO flag, see [Listing 22-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex3) ([page 462](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#page_462)) and [Listing 23-5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch23.xhtml#ch23ex5) ([page 500](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch23.xhtml#page_500)).

##### **The _siginfo_t_ structure**

The _siginfo_t_ structure passed as the second argument to a signal handler that is established with SA_SIGINFO has the following form:

typedef struct {  
    int     si_signo;         /* Signal number */  
    int     si_code;          /* Signal code */  
    int     si_trapno;        /* Trap number for hardware-generated signal  
                                 (unused on most architectures) */  
    union sigval si_value;    /* Accompanying data from sigqueue() */  
    pid_t   si_pid;           /* Process ID of sending process */  
    uid_t   si_uid;           /* Real user ID of sender */  
    int     si_errno;         /* Error number (generally unused) */  
    void   *si_addr;          /* Address that generated signal  
                                 (hardware-generated signals only) */  
    int     si_overrun;       /* Overrun count (Linux 2.6, POSIX timers) */  
    int     si_timerid;       /* (Kernel-internal) Timer ID  
                                 (Linux 2.6, POSIX timers) */  
    long    si_band;          /* Band event (SIGPOLL/SIGIO) */  
    int     si_fd;            /* File descriptor (SIGPOLL/SIGIO) */  
    int     si_status;        /* Exit status or signal (SIGCHLD) */  
    clock_t si_utime;         /* User CPU time (SIGCHLD) */  
    clock_t si_stime;         /* System CPU time (SIGCHLD) */  
} siginfo_t;

The _POSIX_C_SOURCE feature test macro must be defined with a value greater than or equal to 199309 in order to make the declaration of the _siginfo_t_ structure visible from <signal.h>.

On Linux, as on most UNIX implementations, many of the fields in the _siginfo_t_ structure are combined into a union, since not all of the fields are needed for each signal. (See <bits/siginfo.h> for details.)

Upon entry to a signal handler, the fields of the _siginfo_t_ structure are set as follows:

_si_signo_

This field is set for all signals. It contains the number of the signal causing invocation of the handler—that is, the same value as the _sig_ argument to the handler.

_si_code_

This field is set for all signals. It contains a code providing further information about the origin of the signal, as shown in [Table 21-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table2).

_si_value_

This field contains the accompanying data for a signal sent via _sigqueue()_. We describe _sigqueue()_ in [Section 22.8.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev2sec01).

_si_pid_

For signals sent via _kill()_ or _sigqueue()_, this field is set to the process ID of the sending process.

_si_uid_

For signals sent via _kill()_ or _sigqueue()_, this field is set to the real user ID of the sending process. The system provides the real user ID of the sending process because that is more informative than providing the effective user ID. Consider the permission rules for sending signals described in [Section 20.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec05): if the effective user ID grants the sender permission to send the signal, then that user ID must either be 0 (i.e., a privileged process), or be the same as the real user ID or saved set-user-ID of the receiving process. In this case, it could be useful for the receiver to know the sender’s real user ID, which may be different from the effective user ID (e.g., if the sender is a set-user-ID program).

_si_errno_

If this field is set to a nonzero value, then it contains an error number (like _errno_) that identifies the cause of the signal. This field is generally unused on Linux.

_si_addr_

This field is set only for hardware-generated SIGBUS, SIGSEGV, SIGILL, and SIGFPE signals. For the SIGBUS and SIGSEGV signals, this field contains the address that caused the invalid memory reference. For the SIGILL and SIGFPE signals, this field contains the address of the program instruction that caused the signal.

The following fields, which are nonstandard Linux extensions, are set only on the delivery of a signal generated on expiration of a POSIX timer (see [Section 23.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch23.xhtml#ch23lev1sec06)):

_si_timerid_

This field contains an ID that the kernel uses internally to identify the timer.

_si_overrun_

This field is set to the overrun count for the timer.

The following two fields are set only for the delivery of a SIGIO signal ([Section 63.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch63.xhtml#ch63lev1sec03)):

_si_band_

This field contains the “band event” value associated with the I/O event. (In versions of _glibc_ up until 2.3.2, _si_band_ was typed as _int_.)

_si_fd_

This field contains the number of the file descriptor associated with the I/O event. This field is not specified in SUSv3, but it is present on many other implementations.

The following fields are set only for the delivery of a SIGCHLD signal ([Section 26.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch26.xhtml#ch26lev1sec03)):

_si_status_

This field contains either the exit status of the child (if _si_code_ is CLD_EXITED) or the number of the signal sent to the child (i.e., the number of the signal that terminated or stopped the child, as described in [Section 26.1.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch26.xhtml#ch26lev2sec03)).

_si_utime_

This field contains the user CPU time used by the child process. In kernels before 2.6, and since 2.6.27, this is measured in system clock ticks (divide by _sysconf(_SC_CLK_TCK)_). In 2.6 kernels before 2.6.27, a bug meant that this field reported times measured in (user-configurable) jiffies (see [Section 10.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec06)). This field is not specified in SUSv3, but it is present on many other implementations.

_si_stime_

This field contains the system CPU time used by the child process. See the description of the _si_utime_ field. This field is not specified in SUSv3, but it is present on many other implementations.

The _si_code_ field provides further information about the origin of the signal, using the values shown in [Table 21-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table2). Not all of the signal-specific values shown in the second column of this table occur on all UNIX implementations and hardware architectures (especially in the case of the four hardware-generated signals SIGBUS, SIGSEGV, SIGILL, and SIGFPE), although all of these constants are defined on Linux and most appear in SUSv3.

Note the following additional points about the values shown in [Table 21-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table2):

• The values SI_KERNEL and SI_SIGIO are Linux-specific. They are not specified in SUSv3 and do not appear on other UNIX implementations.

• SI_SIGIO is employed only in Linux 2.2. From kernel 2.4 onward, Linux instead employs the POLL_* constants shown in the table.

SUSv4 specifies the _psiginfo()_ function, whose purpose is similar to _psignal()_ ([Section 20.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec08)). The _psiginfo()_ function takes two arguments: a pointer to a _siginfo_t_ structure and a message string. It prints the message string on standard error, followed by information about the signal described in the _siginfo_t_ structure. The _psiginfo()_ function is provided by _glibc_ since version 2.10. The _glibc_ implementation prints the signal description, the origin of the signal (as indicated by the _si_code_ field), and, for some signals, other fields from the _siginfo_t_ structure. The _psiginfo()_ function is new in SUSv4, and it is not available on all systems.

**Table 21-2:** Values returned in the _si_code_ field of the _siginfo_t_ structure

|**Signal**|**_si_code_ value**|**Origin of signal**|
|---|---|---|
|Any|SI_ASYNCIO|Completion of an asynchronous I/O (AIO) operation|
||SI_KERNEL|Sent by the kernel (e.g., a signal from terminal driver)|
||SI_MESGQ|Message arrival on POSIX message queue (since Linux 2.6.6)|
||SI_QUEUE|A realtime signal from a user process via _sigqueue()_|
||SI_SIGIO|SIGIO signal (Linux 2.2 only)|
||SI_TIMER|Expiration of a POSIX (realtime) timer|
||SI_TKILL|A user process via _tkill()_ or _tgkill()_ (since Linux 2.4.19)|
||SI_USER|A user process via _kill()_|
|SIGBUS|BUS_ADRALN|Invalid address alignment|
||BUS_ADRERR|Nonexistent physical address|
||BUS_MCEERR_AO|Hardware memory error; action optional (since Linux 2.6.32)|
||BUS_MCEERR_AR|Hardware memory error; action required (since Linux 2.6.32)|
||BUS_OBJERR|Object-specific hardware error|
|SIGCHLD|CLD_CONTINUED|Child continued by SIGCONT (since Linux 2.6.9)|
||CLD_DUMPED|Child terminated abnormally, with core dump|
||CLD_EXITED|Child exited|
||CLD_KILLED|Child terminated abnormally, without core dump|
||CLD_STOPPED|Child stopped|
||CLD_TRAPPED|Traced child has stopped|
|SIGFPE|FPE_FLTDIV|Floating-point divide-by-zero|
||FPE_FLTINV|Invalid floating-point operation|
||FPE_FLTOVF|Floating-point overflow|
||FPE_FLTRES|Floating-point inexact result|
||FPE_FLTUND|Floating-point underflow|
||FPE_INTDIV|Integer divide-by-zero|
||FPE_INTOVF|Integer overflow|
||FPE_SUB|Subscript out of range|
|SIGILL|ILL_BADSTK|Internal stack error|
||ILL_COPROC|Coprocessor error|
||ILL_ILLADR|Illegal addressing mode|
||ILL_ILLOPC|Illegal opcode|
||ILL_ILLOPN|Illegal operand|
||ILL_ILLTRP|Illegal trap|
||ILL_PRVOPC|Privileged opcode|
||ILL_PRVREG|Privileged register|
|SIGPOLL/SIGIO|POLL_ERR|I/O error|
||POLL_HUP|Device disconnected|
||POLL_IN|Input data available|
||POLL_MSG|Input message available|
||POLL_OUT|Output buffers available|
||POLL_PRI|High-priority input available|
|SIGSEGV|SEGV_ACCERR|Invalid permissions for mapped object|
||SEGV_MAPERR|Address not mapped to object|
|SIGTRAP|TRAP_BRANCH|Process branch trap|
||TRAP_BRKPT|Process breakpoint|
||TRAP_HWBKPT|Hardware breakpoint/watchpoint|
||TRAP_TRACE|Process trace trap|

##### **The _ucontext_ argument**

The final argument passed to a handler established with the SA_SIGINFO flag, _ucontext_, is a pointer to a structure of type _ucontext_t_ (defined in <ucontext.h>). (SUSv3 uses a _void_ pointer for this argument because it doesn’t specify any of the details of the argument.) This structure provides so-called user-context information describing the process state prior to invocation of the signal handler, including the previous process signal mask and saved register values (e.g., program counter and stack pointer). This information is rarely used in signal handlers, so we don’t go into further details.

Another use of _ucontext_t_ structures is with the functions _getcontext()_, _makecontext()_, _setcontext()_, and _swapcontext()_, which allow a process to retrieve, create, change, and swap execution contexts, respectively. (These operations are somewhat like _setjmp()_ and _longjmp()_, but more general.) These functions can be used to implement coroutines, where the thread of execution of a process alternates between two (or more) functions. SUSv3 specifies these functions, but marks them obsolete. SUSv4 removes the specifications, and suggests that applications should be rewritten to use POSIX threads instead. The _glibc_ manual provides further information about these functions.

### **21.5 Interruption and Restarting of System Calls**

Consider the following scenario:

1. We establish a handler for some signal.
    
2. We make a blocking system call, for example, a _read()_ from a terminal device, which blocks until input is supplied.
    
3. While the system call is blocked, the signal for which we established a handler is delivered, and its signal handler is invoked.
    

What happens after the signal handler returns? By default, the system call fails with the error EINTR (“Interrupted function”). This can be a useful feature. In [Section 23.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch23.xhtml#ch23lev1sec03), we’ll see how to use a timer (which results in the delivery of a SIGALRM signal) to set a timeout on a blocking system call such as _read()_.

Often, however, we would prefer to continue the execution of an interrupted system call. To do this, we could use code such as the following to manually restart a system call in the event that it is interrupted by a signal handler:

while ((cnt = read(fd, buf, BUF_SIZE)) == -1 && errno == EINTR)  
    continue;                 /* Do nothing loop body */  
  
if (cnt == -1)                /* read() failed with other than EINTR */  
    errExit("read");

If we frequently write code such as the above, it can be useful to define a macro such as the following:

#define NO_EINTR(stmt) while ((stmt) == -1 && errno == EINTR);

Using this macro, we can rewrite the earlier _read()_ call as follows:

NO_EINTR(cnt = read(fd, buf, BUF_SIZE));  
  
if (cnt == -1)                /* read() failed with other than EINTR */  
    errExit("read");

The GNU C library provides a (nonstandard) macro with the same purpose as our NO_EINTR() macro in <unistd.h>. The macro is called TEMP_FAILURE_RETRY() and is made available if the _GNU_SOURCE feature test macro is defined.

Even if we employ a macro like NO_EINTR(), having signal handlers interrupt system calls can be inconvenient, since we must add code to each blocking system call (assuming that we want to restart the call in each case). Instead, we can specify the SA_RESTART flag when establishing the signal handler with _sigaction()_, so that system calls are automatically restarted by the kernel on the process’s behalf. This means that we don’t need to handle a possible EINTR error return for these system calls.

The SA_RESTART flag is a per-signal setting. In other words, we can allow handlers for some signals to interrupt blocking system calls, while others permit automatic restarting of system calls.

##### **System calls (and library functions) for which** SA_RESTART **is effective**

Unfortunately, not all blocking system calls automatically restart as a result of specifying SA_RESTART. The reasons for this are partly historical:

• Restarting of system calls was introduced in 4.2BSD, and covered interrupted calls to _wait()_ and _waitpid()_, as well as the following I/O system calls: _read()_, _readv()_, _write()_, _writev()_, and blocking _ioctl()_ operations. The I/O system calls are interruptible, and hence automatically restarted by SA_RESTART, only when operating on a “slow” device. Slow devices include terminals, pipes, FIFOs, and sockets. On these file types, various I/O operations may block. (By contrast, disk files don’t fall into the category of slow devices, because disk I/O operations generally can be immediately satisfied via the buffer cache. If a disk I/O is required, the kernel puts the process to sleep until the I/O completes.)

• A number of other blocking system calls are derived from System V, which did not initially provide for restarting of system calls.

On Linux, the following blocking system calls (and library functions layered on top of system calls) are automatically restarted if interrupted by a signal handler established using the SA_RESTART flag:

• The system calls used to wait for a child process ([Section 26.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch26.xhtml#ch26lev1sec01)): _wait()_, _waitpid()_, _wait3()_, _wait4()_, and _waitid()_.

• The I/O system calls _read()_, _readv()_, _write()_, _writev()_, and _ioctl()_ when applied to “slow” devices. In cases where data has already been partially transferred at the time of signal delivery, the input and output system calls will be interrupted, but return a success status: an integer indicating how many bytes were successfully transferred.

• The _open()_ system call, in cases where it can block (e.g., when opening FIFOs, as described in [Section 44.7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch44.xhtml#ch44lev1sec07)).

• Various system calls used with sockets: _accept()_, _accept4()_, _connect()_, _send()_, _sendmsg()_, _sendto()_, _recv()_, _recvfrom()_, and _recvmsg()_. (On Linux, these system calls are not automatically restarted if a timeout has been set on the socket using _setsockopt()_. See the _signal(7)_ manual page for details.)

• The system calls used for I/O on POSIX message queues: _mq_receive()_, _mq_timedreceive()_, _mq_send()_, and _mq_timedsend()_.

• The system calls and library functions used to place file locks: _flock()_, _fcntl()_, and _lockf()_.

• The FUTEX_WAIT operation of the Linux-specific _futex()_ system call.

• The _sem_wait()_ and _sem_timedwait()_ functions used to decrement a POSIX semaphore.

• The functions used to synchronize POSIX threads: _pthread_mutex_lock()_, _pthread_mutex_trylock()_, _pthread_mutex_timedlock()_, _pthread_cond_wait()_, and _pthread_cond_timedwait()_.

In kernels before 2.6.22, _futex()_, _sem_wait()_, and _sem_timedwait()_ always failed with the error EINTR when interrupted, regardless of the setting of the SA_RESTART flag.

The following blocking system calls (and library functions layered on top of system calls) are never automatically restarted (even if SA_RESTART is specified):

• The _poll()_, _ppoll()_, _select()_, and _pselect()_ I/O multiplexing calls. (SUSv3 explicitly states that the behavior of _select()_ and _pselect()_ when interrupted by a signal handler is unspecified, regardless of the setting of SA_RESTART.)

• The Linux-specific _epoll_wait()_ and _epoll_pwait()_ system calls.

• The Linux-specific _io_getevents()_ system call.

• The blocking system calls used with System V message queues and semaphores: _semop()_, _semtimedop()_, _msgrcv()_, and _msgsnd()_. (Although System V did not originally provide automatic restarting of system calls, on some UNIX implementations, these system calls _are_ restarted if the SA_RESTART flag is specified.)

• A _read()_ from an _inotify_ file descriptor.

• The system calls and library functions designed to suspend execution of a program for a specified period: _sleep()_, _nanosleep()_, and _clock_nanosleep()_.

• The system calls designed specifically to wait until a signal is delivered: _pause()_, _sigsuspend()_, _sigtimedwait()_, and _sigwaitinfo()_.

##### **Modifying the** SA_RESTART **flag for a signal**

The _siginterrupt()_ function changes the SA_RESTART setting associated with a signal.

#include <signal.h>  
  
int siginterrupt(int sig, int flag);

Returns 0 on success, or –1 on error

If _flag_ is true (1), then a handler for the signal _sig_ will interrupt blocking system calls. If _flag_ is false (0), then blocking system calls will be restarted after execution of a handler for _sig_.

The _siginterrupt()_ function works by using _sigaction()_ to fetch a copy of the signal’s current disposition, tweaking the SA_RESTART flag in the returned _oldact_ structure, and then calling _sigaction()_ once more to update the signal’s disposition.

SUSv4 marks _siginterrupt()_ obsolete, recommending the use of _sigaction()_ instead for this purpose.

##### **Unhandled stop signals can generate** EINTR **for some Linux system calls**

On Linux, certain blocking system calls can return EINTR even in the absence of a signal handler. This can occur if the system call is blocked and the process is stopped by a signal (SIGSTOP, SIGTSTP, SIGTTIN, or SIGTTOU), and then resumed by delivery of a SIGCONT signal.

The following system calls and functions exhibit this behavior: _epoll_pwait()_, _epoll_wait()_, _read()_ from an _inotify_ file descriptor, _semop()_, _semtimedop()_, _sigtimedwait()_, and _sigwaitinfo()_.

In kernels before 2.6.24, _poll()_ also exhibited this behavior, as did _sem_wait()_, _sem_timedwait()_, and _futex(FUTEX_WAIT)_ in kernels before 2.6.22, _msgrcv()_ and _msgsnd()_ in kernels before 2.6.9, and _nanosleep()_ in Linux 2.4 and earlier.

In Linux 2.4 and earlier, _sleep()_ can also be interrupted in this manner, but, instead of returning an error, it returns the number of remaining unslept seconds.

The upshot of this behavior is that if there is a chance that our program may be stopped and restarted by signals, then we may need to include code to restart these system calls, even in a program that doesn’t install handlers for the stop signals.

### **21.6 Summary**

In this chapter, we considered a range of factors that affect the operation and design of signal handlers.

Because signals are not queued, a signal handler must sometimes be coded to deal with the possibility that multiple events of a particular type have occurred, even though only one signal was delivered. The issue of reentrancy affects how we can update global variables and limits the set of functions that we can safely call from a signal handler.

Instead of returning, a signal handler can terminate in a variety of other ways, including calling __exit()_, terminating the process by sending a signal (_kill()_, _raise()_, or _abort()_), or performing a nonlocal goto. Using _sigsetjmp()_ and _siglongjmp()_ provides a program with explicit control of the treatment of the process signal mask when a nonlocal goto is performed.

We can use _sigaltstack()_ to define an alternate signal stack for a process. This is an area of memory that is used instead of the standard process stack when invoking a signal handler. An alternate signal stack is useful in cases where the standard stack has been exhausted by growing too large (at which point the kernel sends a SIGSEGV signal to the process).

The _sigaction()_ SA_SIGINFO flag allows us to establish a signal handler that receives additional information about a signal. This information is supplied via a _siginfo_t_ structure whose address is passed as an argument to the signal handler.

When a signal handler interrupts a blocked system call, the system call fails with the error EINTR. We can take advantage of this behavior to, for example, set a timer on a blocking system call. Interrupted system calls can be manually restarted if desired. Alternatively, establishing the signal handler with the _sigaction()_ SA_RESTART flag causes many (but not all) system calls to be automatically restarted.

##### **Further information**

See the sources listed in [Section 20.15](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec15).

### **21.7 Exercise**

**21-1.**   Implement _abort()_.