---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 32 - Threads - Cancellation
modified: 2024-11-11T19:30:20-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **32**  
**THREADS: THREAD CANCELLATION**

Typically, multiple threads execute in parallel, with each thread performing its task until it decides to terminate by calling _pthread_exit()_ or returning from the thread’s start function.

Sometimes, it can be useful to _cancel_ a thread; that is, to send it a request asking it to terminate now. This could be useful, for example, if a group of threads is performing a calculation, and one thread detects an error condition that requires the other threads to terminate. Alternatively, a GUI-driven application may provide a cancel button to allow the user to terminate a task that is being performed by a thread in the background; in this case, the main thread (controlling the GUI) needs to tell the background thread to terminate.

In this chapter, we describe the POSIX threads cancellation mechanism.

### **32.1 Canceling a Thread**

The _pthread_cancel()_ function sends a cancellation request to the specified _thread_.

#include <pthread.h>  
  
int pthread_cancel(pthread_t thread);

Returns 0 on success, or a positive error number on error

Having made the cancellation request, _pthread_cancel()_ returns immediately; that is, it doesn’t wait for the target thread to terminate.

Precisely what happens to the target thread, and when it happens, depend on that thread’s cancellation state and type, as described in the next section.

### **32.2 Cancellation State and Type**

The _pthread_setcancelstate()_ and _pthread_setcanceltype()_ functions set flags that allow a thread to control how it responds to a cancellation request.

#include <pthread.h>  
  
int pthread_setcancelstate(int state, int *oldstate);  
int pthread_setcanceltype(int type, int *oldtype);

Both return 0 on success, or a positive error number on error

The _pthread_setcancelstate()_ function sets the calling thread’s cancelability state to the value given in _state_. This argument has one of the following values:

PTHREAD_CANCEL_DISABLE

The thread is not cancelable. If a cancellation request is received, it remains pending until cancelability is enabled.

PTHREAD_CANCEL_ENABLE

The thread is cancelable. This is the default cancelability state in newly created threads.

The thread’s previous cancelability state is returned in the location pointed to by _oldstate_.

If we are not interested in the previous cancelability state, Linux allows _oldstate_ to be specified as NULL. This is the case on many other implementations as well; however, SUSv3 doesn’t specify this feature, so portable applications can’t rely on it. We should always specify a non-NULL value for _oldstate_.

Temporarily disabling cancellation (PTHREAD_CANCEL_DISABLE) is useful if a thread is executing a section of code where _all_ of the steps must be completed.

If a thread is cancelable (PTHREAD_CANCEL_ENABLE), then the treatment of a cancellation request is determined by the thread’s cancelability type, which is specified by the _type_ argument in a call to _pthread_setcanceltype()_. This argument has one of the following values:

PTHREAD_CANCEL_ASYNCHRONOUS

The thread may be canceled at any time (perhaps, but not necessarily, immediately). Asynchronous cancelability is rarely useful, and we defer discussion of it until [Section 32.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32lev1sec06).

PTHREAD_CANCEL_DEFERRED

The cancellation remains pending until a cancellation point (see the next section) is reached. This is the default cancelability type in newly created threads. We say more about deferred cancelability in the following sections.

The thread’s previous cancelability type is returned in the location pointed to by _oldtype_.

As with the _pthread_setcancelstate() oldstate_ argument, many implementations, including Linux, allow _oldtype_ to be specified as NULL if we are not interested in the previous cancelability type. Again, SUSv3 doesn’t specify this feature, and portable applications can’t rely on it We should always specify a non-NULL value for _oldtype_.

When a thread calls _fork()_, the child inherits the calling thread’s cancelability type and state. When a thread calls _exec()_, the cancelability state and type of the main thread of the new program are reset to PTHREAD_CANCEL_ENABLE and PTHREAD_CANCEL_DEFERRED, respectively.

### **32.3 Cancellation Points**

When cancelability is enabled and deferred, a cancellation request is acted upon only when a thread next reaches a _cancellation point_. A cancellation point is a call to one of a set of functions defined by the implementation.

SUSv3 specifies that the functions shown in [Table 32-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32table1) _must_ be cancellation points if they are provided by an implementation. Most of these are functions that are capable of blocking the thread for an indefinite period of time.

**Table 32-1:** Functions required to be cancellation points by SUSv3

|   |
|---|
|_accept()_  <br>_aio_suspend()_  <br>_clock_nanosleep()_  <br>_close()_  <br>_connect()_  <br>_creat()_  <br>_fcntl(F_SETLKW)_  <br>_fsync()_  <br>_fdatasync()_  <br>_getmsg()_  <br>_getpmsg()_  <br>_lockf(F_LOCK)_  <br>_mq_receive()_  <br>_mq_send()_  <br>_mq_timedreceive()_  <br>_mq_timedsend()_  <br>_msgrcv()_  <br>_msgsnd()_  <br>_msync()_  <br>_nanosleep()_  <br>_open()_  <br>_pause()_  <br>_poll()_  <br>_pread()_  <br>_pselect()_  <br>_pthread_cond_timedwait()_  <br>_pthread_cond_wait()_  <br>_pthread_join()_  <br>_pthread_testcancel()_  <br>_putmsg()_  <br>_putpmsg()_  <br>_pwrite()_  <br>_read()_  <br>_readv()_  <br>_recv()_  <br>_recvfrom()_  <br>_recvmsg()_  <br>_select()_  <br>_sem_timedwait()_  <br>_sem_wait()_  <br>_send()_  <br>_sendmsg()_  <br>_sendto()_  <br>_sigpause()_  <br>_sigsuspend()_  <br>_sigtimedwait()_  <br>_sigwait()_  <br>_sigwaitinfo()_  <br>_sleep()_  <br>_system()_  <br>_tcdrain()_  <br>_usleep()_  <br>_wait()_  <br>_waitid()_  <br>_waitpid()_  <br>_write()_  <br>_writev()_|

In addition to the functions in [Table 32-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32table1), SUSv3 specifies a larger group of functions that an implementation _may_ define as cancellation points. These include the _stdio_ functions, the _dlopen_ API, the _syslog_ API, _nftw()_, _popen()_, _semop()_, _unlink()_, and various functions that retrieve information from system files such as the _utmp_ file. A portable program must correctly handle the possibility that a thread may be canceled when calling these functions.

SUSv3 specifies that aside from the two lists of functions that must and may be cancellation points, none of the other functions in the standard may act as cancellation points (i.e., a portable program doesn’t need to handle the possibility that calling these other functions could precipitate thread cancellation).

SUSv4 adds _openat()_ to the list of functions that must be cancellation points, and removes _sigpause()_ (it moves to the list of functions that _may_ be cancellation points) and _usleep()_ (which is dropped from the standard).

An implementation is free to mark additional functions that are not specified in the standard as cancellation points. Any function that might block (perhaps because it might access a file) is a likely candidate to be a cancellation point. Within _glibc_, many nonstandard functions are marked as cancellation points for this reason.

Upon receiving a cancellation request, a thread whose cancelability is enabled and deferred terminates when it next reaches a cancellation point. If the thread was not detached, then some other thread in the process must join with it, in order to prevent it from becoming a zombie thread. When a canceled thread is joined, the value returned in the second argument to _pthread_join()_ is a special thread return value: PTHREAD_CANCELED.

##### **Example program**

[Listing 32-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32ex1) shows a simple example of the use of _pthread_cancel()_. The main program creates a thread that executes an infinite loop, sleeping for a second and printing the value of a loop counter. (This thread will terminate only if it is sent a cancellation request or if the process exits.) Meanwhile, the main program sleeps for 3 seconds, and then sends a cancellation request to the thread that it created. When we run this program, we see the following:

$ ./thread_cancel  
New thread started  
Loop 1  
Loop 2  
Loop 3  
Thread was canceled

**Listing 32-1:** Canceling a thread with _pthread_cancel()_

___________________________________________________ threads/thread_cancel.c  
  
#include <pthread.h>  
#include "tlpi_hdr.h"  
  
static void *  
threadFunc(void *arg)  
{  
    int j;  
    printf("New thread started\n");    /* May be a cancellation point */  
    for (j = 1; ; j++) {  
        printf("Loop %d\n", j);        /* May be a cancellation point */  
        sleep(1);                      /* A cancellation point */  
    }  
  
    /* NOTREACHED */  
    return NULL;  
}  
  
int  
main(int argc, char *argv[])  
{  
    pthread_t thr;  
    int s;  
    void *res;  
  
    s = pthread_create(&thr, NULL, threadFunc, NULL);  
    if (s != 0)  
        errExitEN(s, "pthread_create");  
  
    sleep(3);                           /* Allow new thread to run a while */  
  
    s = pthread_cancel(thr);  
    if (s != 0)  
        errExitEN(s, "pthread_cancel");  
  
    s = pthread_join(thr, &res);  
    if (s != 0)  
        errExitEN(s, "pthread_join");  
  
    if (res == PTHREAD_CANCELED)  
        printf("Thread was canceled\n");  
    else  
        printf("Thread was not canceled (should not happen!)\n");  
  
    exit(EXIT_SUCCESS);  
}  
___________________________________________________ threads/thread_cancel.c

### **32.4 Testing for Thread Cancellation**

In [Listing 32-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32ex1), the thread created by _main()_ accepted the cancellation request because it executed a function that was a cancellation point (_sleep()_ is a cancellation point; _printf()_ may be one). However, suppose a thread executes a loop that contains no cancellation points (e.g., a compute-bound loop). In this case, the thread would never honor the cancellation request.

The purpose of _pthread_testcancel()_ is simply to be a cancellation point. If a cancellation is pending when this function is called, then the calling thread is terminated.

#include <pthread.h>  
  
void pthread_testcancel(void);

A thread that is executing code that does not otherwise include cancellation points can periodically call _pthread_testcancel()_ to ensure that it responds in a timely fashion to a cancellation request sent by another thread.

### **32.5 Cleanup Handlers**

If a thread with a pending cancellation were simply terminated when it reached a cancellation point, then shared variables and Pthreads objects (e.g., mutexes) might be left in an inconsistent state, perhaps causing the remaining threads in the process to produce incorrect results, deadlock, or crash. To get around this problem, a thread can establish one or more _cleanup handlers_—functions that are automatically executed if the thread is canceled. A cleanup handler can perform tasks such as modifying the values of global variables and unlocking mutexes before the thread is terminated.

Each thread can have a stack of cleanup handlers. When a thread is canceled, the cleanup handlers are executed working down from the top of the stack; that is, the most recently established handler is called first, then the next most recently established, and so on. When all of the cleanup handlers have been executed, the thread terminates.

The _pthread_cleanup_push()_ and _pthread_cleanup_pop()_ functions respectively add and remove handlers on the calling thread’s stack of cleanup handlers.

#include <pthread.h>  
  
void pthread_cleanup_push(void (*routine)(void*), void *arg);  
void pthread_cleanup_pop(int execute);

The _pthread_cleanup_push()_ function adds the function whose address is specified in _routine_ to the top of the calling thread’s stack of cleanup handlers. The _routine_ argument is a pointer to a function that has the following form:

void  
routine(void *arg)  
{  
    /* Code to perform cleanup */  
}

The _arg_ value given to _pthread_cleanup_push()_ is passed as the argument of the cleanup handler when it is invoked. This argument is typed as _void *_, but, using judicious casting, other data types can be passed in this argument.

Typically, a cleanup action is needed only if a thread is canceled during the execution of a particular section of code. If the thread reaches the end of that section without being canceled, then the cleanup action is no longer required. Thus, each call to _pthread_cleanup_push()_ has an accompanying call to _pthread_cleanup_pop()_. This function removes the topmost function from the stack of cleanup handlers. If the _execute_ argument is nonzero, the handler is also executed. This is convenient if we want to perform the cleanup action even if the thread was not canceled.

Although we have described _pthread_cleanup_push()_ and _pthread_cleanup_pop()_ as functions, SUSv3 permits them to be implemented as macros that expand to statement sequences that include an opening ({) and closing (}) brace, respectively. Not all UNIX implementations do things this way, but Linux and many others do. This means that each use of _pthread_cleanup_push()_ must be paired with exactly one corresponding _pthread_cleanup_pop()_ in the same lexical block. (On implementations that do things this way, variables declared between the _pthread_cleanup_push()_ and _pthread_cleanup_pop()_ will be limited to that lexical scope.) For example, it is not correct to write code such as the following:

pthread_cleanup_push(func, arg);  
...  
if (cond) {  
    pthread_cleanup_pop(0);  
}

As a coding convenience, any cleanup handlers that have not been popped are also executed automatically if a thread terminates by calling _pthread_exit()_ (but not if it does a simple return).

##### **Example program**

The program in [Listing 32-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32ex2) provides a simple example of the use of a cleanup handler. The main program creates a thread ⑧ whose first actions are to allocate a block of memory ③ whose location is stored in _buf_, and then lock the mutex _mtx_ ④. Since the thread may be canceled, it uses _pthread_cleanup_push()_ ⑤ to install a cleanup handler that is called with the address stored in _buf_. If it is invoked, the cleanup handler deallocates the freed memory ① and unlocks the mutex ②.

The thread then enters a loop waiting for the condition variable _cond_ to be signaled ⑥. This loop will terminate in one of two ways, depending on whether the program is supplied with a command-line argument:

• If no command-line argument is supplied, the thread is canceled by _main()_ ⑨. In this case, cancellation will occur at the call to _pthread_cond_wait()_ ⑥, which is one of the cancellation points shown in [Table 32-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32table1). As part of cancellation, the cleanup handler established using _pthread_cleanup_push()_ is invoked automatically. (When a _pthread_cond_wait()_ call is canceled, the mutex is automatically relocked before the cleanup handler is invoked. This means the mutex can be safely unlocked in the cleanup handler.)

• If a command-line argument is supplied, the condition variable is signaled ⑩ after the associated global variable, _glob_, is first set to a nonzero value. In this case, the thread falls through to execute _pthread_cleanup_pop()_ ⑦, which, given a nonzero argument, also causes the cleanup handler to be invoked.

The main program joins with the terminated thread ⑪, and reports whether the thread was canceled or terminated normally.

**Listing 32-2:** Using cleanup handlers

__________________________________________________ threads/thread_cleanup.c  
  
   #include <pthread.h>  
   #include "tlpi_hdr.h"  
  
   static pthread_cond_t cond = PTHREAD_COND_INITIALIZER;  
   static pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;  
   static int glob = 0;                    /* Predicate variable */  
  
   static void     /* Free memory pointed to by 'arg' and unlock mutex */  
   cleanupHandler(void *arg)  
   {  
       int s;  
  
       printf("cleanup: freeing block at %p\n", arg);  
①     free(arg);  
  
       printf("cleanup: unlocking mutex\n");  
②     s = pthread_mutex_unlock(&mtx);  
       if (s != 0)  
           errExitEN(s, "pthread_mutex_unlock");  
   }  
  
   static void *  
   threadFunc(void *arg)  
   {  
       int s;  
       void *buf = NULL                   /* Buffer allocated by thread */  
  
③     buf = malloc(0x10000);             /* Not a cancellation point */  
       printf("thread: allocated memory at %p\n", buf);  
  
④     s = pthread_mutex_lock(&mtx);      /* Not a cancellation point */  
       if (s != 0)  
           errExitEN(s, "pthread_mutex_lock");  
  
⑤     pthread_cleanup_push(cleanupHandler, buf);  
  
       while (glob == 0) {  
⑥         s = pthread_cond_wait(&cond, &mtx);    /* A cancellation point */  
           if (s != 0)  
               errExitEN(s, "pthread_cond_wait");  
       }  
  
       printf("thread: condition wait loop completed\n");  
⑦     pthread_cleanup_pop(1);            /* Executes cleanup handler */  
       return NULL;  
   }  
  
   int  
   main(int argc, char *argv[])  
   {  
       pthread_t thr;  
       void *res;  
       int s;  
  
⑧     s = pthread_create(&thr, NULL, threadFunc, NULL);  
       if (s != 0)  
           errExitEN(s, "pthread_create");  
  
       sleep(2);                   /* Give thread a chance to get started */  
  
       if (argc == 1) {            /* Cancel thread */  
           printf("main:    about to cancel thread\n");  
⑨         s = pthread_cancel(thr);  
           if (s != 0)  
               errExitEN(s, "pthread_cancel");  
  
       } else {                    /* Signal condition variable */  
           printf("main:    about to signal condition variable\n");  
           glob = 1;  
⑩         s = pthread_cond_signal(&cond);  
           if (s != 0)  
               errExitEN(s, "pthread_cond_signal");  
       }  
  
⑪     s = pthread_join(thr, &res);  
       if (s != 0)  
           errExitEN(s, "pthread_join");  
       if (res == PTHREAD_CANCELED)  
           printf("main:    thread was canceled\n");  
       else  
           printf("main:    thread terminated normally\n");  
  
       exit(EXIT_SUCCESS);  
   }  
__________________________________________________ threads/thread_cleanup.c

If we invoke the program in [Listing 32-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32ex2) without any command-line arguments, then _main()_ calls _pthread_cancel()_, the cleanup handler is invoked automatically, and we see the following:

$ ./thread_cleanup  
thread:  allocated memory at 0x804b050  
main:    about to cancel thread  
cleanup: freeing block at 0x804b050  
cleanup: unlocking mutex  
main:    thread was canceled

If we invoke the program with a command-line argument, then _main()_ sets _glob_ to 1 and signals the condition variable, the cleanup handler is invoked by _pthread_cleanup_pop()_, and we see the following:

$ ./thread_cleanup s  
thread:  allocated memory at 0x804b050  
main:    about to signal condition variable  
thread:  condition wait loop completed  
cleanup: freeing block at 0x804b050  
cleanup: unlocking mutex  
main:    thread terminated normally

### **32.6 Asynchronous Cancelability**

When a thread is made asynchronously cancelable (cancelability type PTHREAD_CANCEL_ASYNCHRONOUS), it may be canceled at any time (i.e., at any machine-language instruction); delivery of a cancellation is not held off until the thread next reaches a cancellation point.

The problem with asynchronous cancellation is that, although cleanup handlers are still invoked, the handlers have no way of determining the state of a thread. In the program in [Listing 32-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch32.xhtml#ch32ex2), which employs the deferred cancelability type, the thread can be canceled only when it executes the call to _pthread_cond_wait()_, which is the only cancellation point. By this time, we know that _buf_ has been initialized to point to a block of allocated memory and that the mutex _mtx_ has been locked. However, with asynchronous cancelability, the thread could be canceled at any point; for example, before the _malloc()_ call, between the _malloc()_ call and locking the mutex, or after locking the mutex. The cleanup handler has no way of knowing where cancellation has occurred, or precisely which cleanup steps are required. Furthermore, the thread might even be canceled _during_ the _malloc()_ call, after which chaos is likely to result ([Section 7.1.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch07.xhtml#ch07lev2sec03)).

As a general principle, an asynchronously cancelable thread can’t allocate any resources or acquire any mutexes, semaphores, or locks. This precludes the use of a wide range of library functions, including most of the Pthreads functions. (SUSv3 makes exceptions for _pthread_cancel()_, _pthread_setcancelstate()_, and _pthread_setcanceltype()_, which are explicitly required to be _async-cancel-safe_; that is, an implementation must make them safe to call from a thread that is asynchronously cancelable.) In other words, there are few circumstances where asynchronous cancellation is useful. One such circumstance is canceling a thread that is in a compute-bound loop.

### **32.7 Summary**

The _pthread_cancel()_ function allows one thread to send another thread a cancellation request, which is a request that the target thread should terminate.

How the target thread reacts to this request is determined by its cancelability state and type. If the cancelability state is currently set to disabled, the request will remain pending until the cancelability state is set to enabled. If cancelability is enabled, the cancelability type determines when the target thread reacts to the request. If the type is deferred, the cancellation occurs when the thread next calls one of a number of functions specified as cancellation points by SUSv3. If the type is asynchronous, cancellation may occur at any time (this is rarely useful).

A thread can establish a stack of cleanup handlers, which are programmer-defined functions that are invoked automatically to perform cleanups (e.g., restoring the states of shared variables, or unlocking mutexes) if the thread is canceled.

##### **Further information**

Refer to the sources of further information listed in [Section 29.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch29.xhtml#ch29lev1sec10).