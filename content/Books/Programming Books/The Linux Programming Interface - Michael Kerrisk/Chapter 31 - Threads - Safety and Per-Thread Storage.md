---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 31 - Threads - Safety and Per-Thread Storage
modified: 2024-11-11T19:28:02-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **31**  
**THREADS: THREAD SAFETY AND PER-THREAD STORAGE**

This chapter extends the discussion of the POSIX threads API, providing a description of thread-safe functions and one-time initialization. We also discuss how to use thread-specific data or thread-local storage to make an existing function thread-safe without changing the function’s interface.

### **31.1 Thread Safety (and Reentrancy Revisited)**

A function is said to be _thread-safe_ if it can safely be invoked by multiple threads at the same time; put conversely, if a function is not thread-safe, then we can’t call it from one thread while it is being executed in another thread. For example, the following function (similar to code that we looked at in [Section 30.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch30.xhtml#ch30lev1sec01)) is not thread-safe:

static int glob = 0;  
  
static void  
incr(int loops)  
{  
    int loc, j;  
    for (j = 0; j < loops; j++) {  
        loc = glob;  
        loc++;  
        glob = loc;  
    }  
}

If multiple threads invoke this function concurrently, the final value in _glob_ is unpredictable. This function illustrates the typical reason that a function is not thread-safe: it employs global or static variables that are shared by all threads.

There are various methods of rendering a function thread-safe. One way is to associate a mutex with the function (or perhaps with all of the functions in a library, if they all share the same global variables), lock that mutex when the function is called, and unlock it when the function returns. This approach has the virtue of simplicity. On the other hand, it means that only one thread at a time can execute the function—we say that access to the function is _serialized_. If the threads spend a significant amount of time executing this function, then this serialization results in a loss of concurrency, because the threads of a program can no longer execute in parallel.

A more sophisticated solution is to associate the mutex with a shared variable. We then determine which parts of the function are critical sections that access the shared variable, and acquire and release the mutex only during the execution of these critical sections. This allows multiple threads to execute the function at the same time and to operate in parallel, except when more than one thread needs to execute a critical section.

##### **Non-thread-safe functions**

To facilitate the development of threaded applications, all of the functions specified in SUSv3 are required to be implemented in a thread-safe manner, except those listed in [Table 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31table1). (Many of these functions are not discussed in this book.)

In addition to the functions listed in [Table 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31table1), SUSv3 specifies the following:

• The _ctermid()_ and _tmpnam()_ functions need not be thread-safe if passed a NULL argument.

• The _wcrtomb()_ and _wcsrtombs()_ functions need not be thread-safe if their final argument (_ps_) is NULL.

SUSv4 modifies the list of functions in [Table 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31table1) as follows:

• The _ecvt()_, _fcvt()_, _gcvt()_, _gethostbyname()_, and _gethostbyaddr()_ functions are removed, since these functions have been removed from the standard.

• The _strsignal()_ and _system()_ functions are added. The _system()_ function is non-reentrant because the manipulations that it must make to signal dispositions have a process-wide effect.

The standards do not prohibit an implementation from making the functions in [Table 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31table1) thread-safe. However, even if some of these functions are thread-safe on some implementations, a portable application can’t rely on this to be the case on all implementations.

**Table 31-1:** Functions that SUSv3 does not require to be thread-safe

|   |
|---|
|_asctime()_  <br>_basename()_  <br>_catgets()_  <br>_crypt()_  <br>_ctime()_  <br>_dbm_clearerr()_  <br>_dbm_close()_  <br>_dbm_delete()_  <br>_dbm_error()_  <br>_dbm_fetch()_  <br>_dbm_firstkey()_  <br>_dbm_nextkey()_  <br>_dbm_open()_  <br>_dbm_store()_  <br>_dirname()_  <br>_dlerror()_  <br>_drand48()_  <br>_ecvt()_  <br>_encrypt()_  <br>_endgrent()_  <br>_endpwent()_  <br>_endutxent()_  <br>_fcvt()_  <br>_ftw()_  <br>_gcvt()_  <br>_getc_unlocked()_  <br>_getchar_unlocked()_  <br>_getdate()_  <br>_getenv()_  <br>_getgrent()_  <br>_getgrgid()_  <br>_getgrnam()_  <br>_gethostbyaddr()_  <br>_gethostbyname()_  <br>_gethostent()_  <br>_getlogin()_  <br>_getnetbyaddr()_  <br>_getnetbyname()_  <br>_getnetent()_  <br>_getopt()_  <br>_getprotobyname()_  <br>_getprotobynumber()_  <br>_getprotoent()_  <br>_getpwent()_  <br>_getpwnam()_  <br>_getpwuid()_  <br>_getservbyname()_  <br>_getservbyport()_  <br>_getservent()_  <br>_getutxent()_  <br>_getutxid()_  <br>_getutxline()_  <br>_gmtime()_  <br>_hcreate()_  <br>_hdestroy()_  <br>_hsearch()_  <br>_inet_ntoa()_  <br>_l64a()_  <br>_lgamma()_  <br>_lgammaf()_  <br>_lgammal()_  <br>_localeconv()_  <br>_localtime()_  <br>_lrand48()_  <br>_mrand48()_  <br>_nftw()_  <br>_nl_langinfo()_  <br>_ptsname()_  <br>_putc_unlocked()_  <br>_putchar_unlocked()_  <br>_putenv()_  <br>_pututxline()_  <br>_rand()_  <br>_readdir()_  <br>_setenv()_  <br>_setgrent()_  <br>_setkey()_  <br>_setpwent()_  <br>_setutxent()_  <br>_strerror()_  <br>_strtok()_  <br>_ttyname()_  <br>_unsetenv()_  <br>_wcstombs()_  <br>_wctomb()_|

##### **Reentrant and nonreentrant functions**

Although the use of critical sections to implement thread safety is a significant improvement over the use of per-function mutexes, it is still somewhat inefficient because there is a cost to locking and unlocking a mutex. A _reentrant function_ achieves thread safety without the use of mutexes. It does this by avoiding the use of global and static variables. Any information that must be returned to the caller, or maintained between calls to the function, is stored in buffers allocated by the caller. (We first encountered reentrancy when discussing the treatment of global variables within signal handlers in [Section 21.1.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev2sec02).) However, not all functions can be made reentrant. The usual reasons are the following:

• By their nature, some functions must access global data structures. The functions in the _malloc_ library provide a good example. These functions maintain a global linked list of free blocks on the heap. The functions of the _malloc_ library are made thread-safe through the use of mutexes.

• Some functions (defined before the invention of threads) have an interface that by definition is nonreentrant, because they return pointers to storage statically allocated by the function, or they employ static storage to maintain information between successive calls to the same (or a related) function. Most of the functions in [Table 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31table1) fall into this category. For example, the _asctime()_ function ([Section 10.2.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev2sec03)) returns a pointer to a statically allocated buffer containing a date-time string.

For several of the functions that have nonreentrant interfaces, SUSv3 specifies reentrant equivalents with names ending with the suffix __r_. These functions require the caller to allocate a buffer whose address is then passed to the function and used to return the result. This allows the calling thread to use a local (stack) variable for the function result buffer. For this purpose, SUSv3 specifies _asctime_r()_, _ctime_r()_, _getgrgid_r()_, _getgrnam_r()_, _getlogin_r()_, _getpwnam_r()_, _getpwuid_r()_, _gmtime_r()_, _localtime_r()_, _rand_r()_, _readdir_r()_, _strerror_r()_, _strtok_r()_, and _ttyname_r()_.

Some implementations also provide additional reentrant equivalents of other traditional nonreentrant functions. For example, _glibc_ provides _crypt_r()_, _gethostbyname_r()_, _getservbyname_r()_, _getutent_r()_, _getutid_r()_, _getutline_r()_, and _ptsname_r()_. However, a portable application can’t rely on these functions being present on other implementations. In some cases, SUSv3 doesn’t specify these reentrant equivalents because alternatives to the traditional functions exist that are both superior and reentrant. For example, _getaddrinfo()_ is the modern, reentrant alternative to _gethostbyname()_ and _getservbyname()_.

### **31.2 One-Time Initialization**

Sometimes, a threaded application needs to ensure that some initialization action occurs just once, regardless of how many threads are created. For example, a mutex may need to be initialized with special attributes using _pthread_mutex_init()_, and that initialization must occur just once. If we are creating the threads from the main program, then this is generally easy to achieve—we perform the initialization before creating any threads that depend on the initialization. However, in a library function, this is not possible, because the calling program may create the threads before the first call to the library function. Therefore, the library function needs a method of performing the initialization the first time that it is called from any thread.

A library function can perform one-time initialization using the _pthread_once()_ function.

#include <pthread.h>  
  
int pthread_once(pthread_once_t *once_control, void (*init)(void));

Returns 0 on success, or a positive error number on error

The _pthread_once()_ function uses the state of the argument _once_control_ to ensure that the caller-defined function pointed to by _init_ is called just once, no matter how many times or from how many different threads the _pthread_once()_ call is made.

The _init_ function is called without any arguments, and thus has the following form:

void  
init(void)  
{  
    /* Function body */  
}

The _once_control_ argument is a pointer to a variable that must be statically initialized with the value PTHREAD_ONCE_INIT:

pthread_once_t once_var = PTHREAD_ONCE_INIT;

The first call to _pthread_once()_ that specifies a pointer to a particular _pthread_once_t_ variable modifies the value of the variable pointed to by _once_control_ so that subsequent calls to _pthread_once()_ don’t invoke _init_.

One common use of _pthread_once()_ is in conjunction with thread-specific data, which we describe next.

The main reason for the existence of _pthread_once()_ is that in early versions of Pthreads, it was not possible to statically initialize a mutex. Instead, the use of _pthread_mutex_init()_ was required ([[Butenhof, 1996](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib10)]). Given the later addition of statically allocated mutexes, it is possible for a library function to perform one-time initialization using a statically allocated mutex and a static Boolean variable. Nevertheless, _pthread_once()_ is retained as a convenience.

### **31.3 Thread-Specific Data**

The most efficient way of making a function thread-safe is to make it reentrant. All new library functions should be implemented in this way. However, for an existing nonreentrant library function (one that was perhaps designed before the use of threads became common), this approach usually requires changing the function’s interface, which means modifying all of the programs that use the function.

Thread-specific data is a technique for making an existing function thread-safe without changing its interface. A function that uses thread-specific data may be slightly less efficient than a reentrant function, but allows us to leave the programs that call the function unchanged.

Thread-specific data allows a function to maintain a separate copy of a variable for each thread that calls the function, as illustrated in [Figure 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31fig1). Thread-specific data is persistent; each thread’s variable continues to exist between the thread’s invocations of the function. This allows the function to maintain per-thread information between calls to the function, and allows the function to pass distinct result buffers (if required) to each calling thread.

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f31-01.jpg)

**Figure 31-1:** Thread-specific data (TSD) provides per-thread storage for a function

#### **31.3.1 Thread-Specific Data from the Library Function’s Perspective**

In order to understand the use of the thread-specific data API, we need to consider things from the point of view of a library function that uses thread-specific data:

• The function must allocate a separate block of storage for each thread that calls the function. This block needs to be allocated once, the first time the thread calls the function.

• On each subsequent call from the same thread, the function needs to be able to obtain the address of the storage block that was allocated the first time this thread called the function. The function can’t maintain a pointer to the block in an automatic variable, since automatic variables disappear when the function returns; nor can it store the pointer in a static variable, since only one instance of each static variable exists in the process. The Pthreads API provides functions to handle this task.

• Different (i.e., independent) functions may each need thread-specific data. Each function needs a method of identifying its thread-specific data (a key), as distinct from the thread-specific data used by other functions.

• The function has no direct control over what happens when the thread terminates. When the thread terminates, it is probably executing code outside the function. Nevertheless, there must be some mechanism (a destructor) to ensure that the storage block allocated for this thread is automatically deallocated when the thread terminates. If this is not done, then a memory leak could occur as threads are continuously created, call the function, and then terminate.

#### **31.3.2 Overview of the Thread-Specific Data API**

The general steps that a library function performs in order to use thread-specific data are as follows:

1. The function creates a _key_, which is the means of differentiating the thread-specific data item used by this function from the thread-specific data items used by other functions. The key is created by calling the _pthread_key_create()_ function. Creating a key needs to be done only once, when the first thread calls the function. For this purpose, _pthread_once()_ is employed. Creating a key doesn’t allocate any blocks of thread-specific data.
    
2. The call to _pthread_key_create()_ serves a second purpose: it allows the caller to specify the address of the programmer-defined destructor function that is used to deallocate each of the storage blocks allocated for this key (see the next step). When a thread that has thread-specific data terminates, the Pthreads API automatically invokes the destructor, passing it a pointer to the data block for this thread.
    
3. The function allocates a thread-specific data block for each thread from which it is called. This is done using _malloc()_ (or a similar function). This allocation is done once for each thread, the first time the thread calls the function.
    
4. In order to save a pointer to the storage allocated in the previous step, the function employs two Pthreads functions: _pthread_setspecific()_ and _pthread_getspecific()_. A call to _pthread_setspecific()_ is a request to the Pthreads implementation that says “save this pointer, recording the fact that it is associated with a particular key (the one for this function) and a particular thread (the calling thread).” Calling _pthread_getspecific()_ performs the complementary task, returning the pointer previously associated with a given key for the calling thread. If no pointer was previously associated with a particular key and thread, then _pthread_getspecific()_ returns NULL. This is how a function can determine that it is being called for the first time by this thread, and thus must allocate the storage block for the thread.
    

#### **31.3.3 Details of the Thread-Specific Data API**

In this section, we provide details of each of the functions mentioned in the previous section, and elucidate the operation of thread-specific data by describing how it is typically implemented. The next section shows how to use thread-specific data to write a thread-safe implementation of the standard C library function _strerror()_.

Calling _pthread_key_create()_ creates a new thread-specific data key that is returned to the caller in the buffer pointed to by _key_.

#include <pthread.h>  
  
int pthread_key_create(pthread_key_t *key, void (*destructor)(void *));

Returns 0 on success, or a positive error number on error

Because the returned key is used by all threads in the process, _key_ should point to a global variable.

The _destructor_ argument points to a programmer-defined function of the following form:

void  
dest(void *value)  
{  
    /* Release storage pointed to by 'value' */  
}

Upon termination of a thread that has a non-NULL value associated with _key_, the destructor function is automatically invoked by the Pthreads API and given that value as its argument. The passed value is normally a pointer to this thread’s thread-specific data block for this key. If a destructor is not required, then _destructor_ can be specified as NULL.

If a thread has multiple thread-specific data blocks, then the order in which the destructors are called is unspecified. Destructor functions should be designed to operate independently of one another.

Looking at the implementation of thread-specific data helps us to understand how it is used. A typical implementation (NPTL is typical), involves the following arrays:

• a single global (i.e., process-wide) array of information about thread-specific data keys; and

• a set of per-thread arrays, each containing pointers to all of the thread-specific data blocks allocated for a particular thread (i.e., this array contains the pointers stored by calls to _pthread_setspecific()_).

In this implementation, the _pthread_key_t_ value returned by _pthread_key_create()_ is simply an index into the global array, which we label _pthread_keys_, whose form is shown in [Figure 31-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31fig2). Each element of this array is a structure containing two fields. The first field indicates whether this array element is in use (i.e., has been allocated by a previous call to _pthread_key_create()_). The second field is used to store the pointer to the destructor function for the thread-specific data blocks for this key (i.e., it is a copy of the _destructor_ argument to _pthread_key_create()_).

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f31-02.jpg)

**Figure 31-2:** Implementation of thread-specific data keys

The _pthread_setspecific()_ function requests the Pthreads API to save a copy of _value_ in a data structure that associates it with the calling thread and with _key_, a key returned by a previous call to _pthread_key_create()_. The _pthread_getspecific()_ function performs the converse operation, returning the value that was previously associated with the given _key_ for this thread.

#include <pthread.h>  
  
int pthread_setspecific(pthread_key_t key, const void *value);

Returns 0 on success, or a positive error number on error

void *pthread_getspecific(pthread_key_t key);

Returns pointer, or NULL if no thread-specific data is associated with _key_

The _value_ argument given to _pthread_setspecific()_ is normally a pointer to a block of memory that has previously been allocated by the caller. This pointer will be passed as the argument for the destructor function for this _key_ when the thread terminates.

The _value_ argument doesn’t need to be a pointer to a block of memory. It could be some scalar value that can be assigned (with a cast) to _void *_. In this case, the earlier call to _pthread_key_create()_ would specify _destructor_ as NULL.

[Figure 31-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31fig3) shows a typical implementation of the data structure used to store _value_. In this diagram, we assume that _pthread_keys[1]_ was allocated to a function named _myfunc()_. For each thread, the Pthreads API maintains an array of pointers to thread-specific data blocks. The elements of each of these thread-specific arrays have a one-to-one correspondence with the elements of the global _pthread_keys_ array shown in [Figure 31-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31fig2). The _pthread_setspecific()_ function sets the element corresponding to _key_ in the array for the calling thread.

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f31-03.jpg)

**Figure 31-3:** Data structure used to implement thread-specific data (TSD) pointers

When a thread is first created, all of its thread-specific data pointers are initialized to NULL. This means that when our library function is called by a thread for the first time, it must begin by using _pthread_getspecific()_ to check whether the thread already has an associated value for _key_. If it does not, then the function allocates a block of memory and saves a pointer to the block using _pthread_setspecific()_. We show an example of this in the thread-safe _strerror()_ implementation presented in the next section.

#### **31.3.4 Employing the Thread-Specific Data API**

When we first described the standard _strerror()_ function in [Section 3.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch03.xhtml#ch03lev1sec04), we noted that it may return a pointer to a statically allocated string as its function result. This means that _strerror()_ may not be thread-safe. In the next few pages, we look at a non-thread-safe implementation of _strerror()_, and then show how thread-specific data can be used to make this function thread-safe.

On many UNIX implementations, including Linux, the _strerror()_ function provided by the standard C library _is_ thread-safe. However, we use the example of _strerror()_ anyway, because SUSv3 doesn’t require this function to be thread-safe, and its implementation provides a simple example of the use of thread-specific data.

[Listing 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex1) shows a simple non-thread-safe implementation of _strerror()_. This function makes use of a pair of global variables defined by _glibc_: __sys_errlist_ is an array of pointers to strings corresponding to the error numbers in _errno_ (thus, for example, __sys_errlist[EINVAL]_ points to the string _Invalid argument_), and __sys_nerr_ specifies the number of elements in __sys_errlist_.

**Listing 31-1:** An implementation of _strerror()_ that is not thread-safe

________________________________________________________threads/strerror.c  
  
#define _GNU_SOURCE                /* Get '_sys_nerr' and '_sys_errlist'  
                                      declarations from <stdio.h> */  
#include <stdio.h>  
#include <string.h>                /* Get declaration of strerror() */  
  
#define MAX_ERROR_LEN 256          /* Maximum length of string  
                                      returned by strerror() */  
  
static char buf[MAX_ERROR_LEN];    /* Statically allocated return buffer */  
  
char *  
strerror(int err)  
{  
    if (err < 0 || err >= _sys_nerr || _sys_errlist[err] == NULL) {  
        snprintf(buf, MAX_ERROR_LEN, "Unknown error %d", err);  
    } else {  
        strncpy(buf, _sys_errlist[err], MAX_ERROR_LEN - 1);  
        buf[MAX_ERROR_LEN - 1] = '\0';          /* Ensure null termination */  
    }  
  
    return buf;  
}  
________________________________________________________threads/strerror.c

We can use the program in [Listing 31-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex2) to demonstrate the consequences of the fact that the _strerror()_ implementation in [Listing 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex1) is not thread-safe. This program calls _strerror()_ from two different threads, but displays the returned value only after both threads have called _strerror()_. Even though each thread specifies a different value (EINVAL and EPERM) as the argument to _strerror()_, this is what we see when we compile and link this program with the version of _strerror()_ shown in [Listing 31-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex1):

$ ./strerror_test  
Main thread has called strerror()  
Other thread about to call strerror()  
Other thread: str (0x804a7c0) = Operation not permitted  
Main thread:  str (0x804a7c0) = Operation not permitted

Both threads displayed the _errno_ string corresponding to EPERM, because the call to _strerror()_ by the second thread (in _threadFunc_) overwrote the buffer that was written by the call to _strerror()_ in the main thread. Inspection of the output shows that the local variable _str_ in the two threads points to the same memory address.

**Listing 31-2:** Calling _strerror()_ from two different threads

___________________________________________________ threads/strerror_test.c  
  
#include <stdio.h>  
#include <string.h>                 /* Get declaration of strerror() */  
#include <pthread.h>  
#include "tlpi_hdr.h"  
  
static void *  
threadFunc(void *arg)  
{  
    char *str;  
  
    printf("Other thread about to call strerror()\n");  
    str = strerror(EPERM);  
    printf("Other thread: str (%p) = %s\n", str, str);  
  
    return NULL;  
}  
  
int  
main(int argc, char *argv[])  
{  
    pthread_t t;  
    int s;  
    char *str;  
  
    str = strerror(EINVAL);  
    printf("Main thread has called strerror()\n");  
  
    s = pthread_create(&t, NULL, threadFunc, NULL);  
    if (s != 0)  
        errExitEN(s, "pthread_create");  
  
    s = pthread_join(t, NULL);  
    if (s != 0)  
        errExitEN(s, "pthread_join");  
  
    printf("Main thread: str (%p) = %s\n", str, str);  
  
    exit(EXIT_SUCCESS);  
}  
___________________________________________________ threads/strerror_test.c

[Listing 31-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex3) shows a reimplementation of _strerror()_ that uses thread-specific data to ensure thread safety.

The first step performed by the revised _strerror()_ is to call _pthread_once()_ ④ to ensure that the first invocation of this function (from any thread) calls _createKey()_ ②. The _createKey()_ function calls _pthread_key_create()_ to allocate a thread-specific data key that is stored in the global variable _strerrorKey_ ③. The call to _pthread_key_create()_ also records the address of the destructor ① that will be used to free the thread-specific buffers corresponding to this key.

The _strerror()_ function then calls _pthread_getspecific()_ ⑤ to retrieve the address of this thread’s unique buffer corresponding to _strerrorKey_. If _pthread_getspecific()_ returns NULL, then this thread is calling _strerror()_ for the first time, and so the function allocates a new buffer using _malloc()_ ⑥, and saves the address of the buffer using _pthread_setspecific()_ ⑦. If the _pthread_getspecific()_ call returns a non-NULL value, then that pointer refers to an existing buffer that was allocated when this thread previously called _strerror()_.

The remainder of this _strerror()_ implementation is similar to the implementation that we showed earlier, with the difference that _buf_ is the address of a thread-specific data buffer, rather than a static variable.

**Listing 31-3:** A thread-safe implementation of _strerror()_ using thread-specific data

____________________________________________________ threads/strerror_tsd.c  
  
   #define _GNU_SOURCE             /* Get '_sys_nerr' and '_sys_errlist'  
                                      declarations from <stdio.h> */  
   #include <stdio.h>  
   #include <string.h>             /* Get declaration of strerror() */  
   #include <pthread.h>  
   #include "tlpi_hdr.h"  
  
   static pthread_once_t once = PTHREAD_ONCE_INIT;  
   static pthread_key_t strerrorKey;  
  
   #define MAX_ERROR_LEN 256       /* Maximum length of string in per-thread  
                                      buffer returned by strerror() */  
  
   static void                     /* Free thread-specific data buffer */  
① destructor(void *buf)  
   {  
       free(buf);  
   }  
  
   static void                     /* One-time key creation function */  
② createKey(void)  
   {  
       int s;  
  
       /* Allocate a unique thread-specific data key and save the address  
          of the destructor for thread-specific data buffers */  
  
③     s = pthread_key_create(&strerrorKey, destructor);  
       if (s != 0)  
           errExitEN(s, "pthread_key_create");  
   }  
  
   char *  
   strerror(int err)  
   {  
       int s;  
       char *buf;  
  
       /* Make first caller allocate key for thread-specific data */  
  
④     s = pthread_once(&once, createKey);  
       if (s != 0)  
           errExitEN(s, "pthread_once");  
  
⑤     buf = pthread_getspecific(strerrorKey);  
       if (buf == NULL) {          /* If first call from this thread, allocate  
                                      buffer for thread, and save its location */  
⑥         buf = malloc(MAX_ERROR_LEN);  
           if (buf == NULL)  
               errExit("malloc");  
  
⑦         s = pthread_setspecific(strerrorKey, buf);  
           if (s != 0)  
               errExitEN(s, "pthread_setspecific");  
       }  
  
       if (err < 0 || err >= _sys_nerr || _sys_errlist[err] == NULL) {  
           snprintf(buf, MAX_ERROR_LEN, "Unknown error %d", err);  
       } else {  
           strncpy(buf, _sys_errlist[err], MAX_ERROR_LEN - 1);  
           buf[MAX_ERROR_LEN - 1] = '\0';          /* Ensure null termination */  
       }  
  
       return buf;  
   }  
____________________________________________________ threads/strerror_tsd.c

If we compile and link our test program ([Listing 31-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex2)) with the new version of _strerror()_ ([Listing 31-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex3)) to create an executable file, strerror_test_tsd, then we see the following results when running the program:

$ ./strerror_test_tsd  
Main thread has called strerror()  
Other thread about to call strerror()  
Other thread: str (0x804b158) = Operation not permitted  
Main thread:  str (0x804b008) = Invalid argument

From this output, we see that the new version of _strerror()_ is thread-safe. We also see that the address pointed to by the local variable _str_ in the two threads is different.

#### **31.3.5 Thread-Specific Data Implementation Limits**

As implied by our description of how thread-specific data is typically implemented, an implementation may need to impose limits on the number of thread-specific data keys that it supports. SUSv3 requires that an implementation support at least 128 (_POSIX_THREAD_KEYS_MAX) keys. An application can determine how many keys an implementation actually supports either via the definition of PTHREAD_KEYS_MAX (defined in <limits.h>) or by calling _sysconf(_SC_THREAD_KEYS_MAX)_. Linux supports up to 1024 keys.

Even 128 keys should be more than sufficient for most applications. This is because each library function should employ only a small number of keys—often just one. If a function requires multiple thread-specific data values, these can usually be placed in a single structure that has just one associated thread-specific data key.

### **31.4 Thread-Local Storage**

Like thread-specific data, thread-local storage provides persistent per-thread storage. This feature is nonstandard, but it is provided in the same or a similar form on many other UNIX implementations (e.g., Solaris and FreeBSD).

The main advantage of thread-local storage is that it is much simpler to use than thread-specific data. To create a thread-local variable, we simply include the __thread specifier in the declaration of a global or static variable:

static __thread char buf[MAX_ERROR_LEN];

Each thread has its own copy of the variables declared with this specifier. The variables in a thread’s thread-local storage persist until the thread terminates, at which time the storage is automatically deallocated.

Note the following points about the declaration and use of thread-local variables:

• The __thread keyword must immediately follow the static or extern keyword, if either of these is specified in the variable’s declaration.

• The declaration of a thread-local variable can include an initializer, in the same manner as a normal global or static variable declaration.

• The C address (&) operator can be used to obtain the address of a thread-local variable.

Thread-local storage requires support from the kernel (provided in Linux 2.6), the Pthreads implementation (provided in NPTL), and the C compiler (provided on x86-32 with _gcc_ 3.3 and later).

[Listing 31-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex4) shows a thread-safe implementation of _strerror()_ using thread-local storage. If we compile and link our test program ([Listing 31-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31ex2)) with this version of _strerror()_ to create an executable file, strerror_test_tls, then we see the following results when running the program:

$ ./strerror_test_tls  
Main thread has called strerror()  
Other thread about to call strerror()  
Other thread: str (0x40376ab0) = Operation not permitted  
Main thread:  str (0x40175080) = Invalid argument

**Listing 31-4:** A thread-safe implementation of _strerror()_ using thread-local storage

____________________________________________________ threads/strerror_tls.c  
  
#define _GNU_SOURCE                 /* Get '_sys_nerr' and '_sys_errlist'  
                                       declarations from <stdio.h> */  
#include <stdio.h>  
#include <string.h>                 /* Get declaration of strerror() */  
#include <pthread.h>  
  
#define MAX_ERROR_LEN 256           /* Maximum length of string in per-thread  
                                       buffer returned by strerror() */  
  
static __thread char buf[MAX_ERROR_LEN];  
                                    /* Thread-local return buffer */  
  
char *  
strerror(int err)  
{  
    if (err < 0 || err >= _sys_nerr || _sys_errlist[err] == NULL) {  
        snprintf(buf, MAX_ERROR_LEN, "Unknown error %d", err);  
    } else {  
        strncpy(buf, _sys_errlist[err], MAX_ERROR_LEN - 1);  
        buf[MAX_ERROR_LEN - 1] = '\0';          /* Ensure null termination */  
    }  
  
    return buf;  
}  
____________________________________________________ threads/strerror_tls.c

### **31.5 Summary**

A function is said to be thread-safe if it can safely be invoked from multiple threads at the same time. The usual reason a function is not thread-safe is that it makes use of global or static variables. One way to render a non-thread-safe function safe in a multithreaded application is to guard all calls to the function with a mutex lock. This approach suffers the problem that it reduces concurrency, because only one thread can be in the function at any time. An approach that allows greater concurrency is to add mutex locks around just those parts of the function that manipulate shared variables (the critical sections).

Mutexes can be used to render most functions thread-safe, but they carry a performance penalty because there is a cost to locking and unlocking a mutex. By avoiding the use of global and static variables, a reentrant function achieves thread-safety without the use of mutexes.

Most of the functions specified in SUSv3 are required to be thread-safe. SUSv3 also lists a small set of functions that are not required to be thread-safe. Typically, these are functions that employ static storage to return information to the caller or to maintain information between successive calls. By definition, such functions are not reentrant, and mutexes can’t be used to make them thread-safe. We considered two roughly equivalent coding techniques—thread-specific data and thread-local storage—that can be used to render an unsafe function thread-safe without needing to change its interface. Both of these techniques allow a function to allocate persistent, per-thread storage.

##### **Further information**

Refer to the sources of further information listed in [Section 29.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch29.xhtml#ch29lev1sec10).

### **31.6 Exercises**

**31-1.**   Implement a function, _one_time_init(control, init)_, that performs the equivalent of _pthread_once()_. The _control_ argument should be a pointer to a statically allocated structure containing a Boolean variable and a mutex. The Boolean variable indicates whether the function _init_ has already been called, and the mutex controls access to that variable. To keep the implementation simple, you can ignore possibilities such as _init()_ failing or being canceled when first called from a thread (i.e., it is not necessary to devise a scheme whereby, if such an event occurs, the next thread that calls _one_time_init()_ reattempts the call to _init()_).

**31-2.**   Use thread-specific data to write thread-safe versions of _dirname()_ and _basename()_ ([Section 18.14](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch18.xhtml#ch18lev1sec14)).