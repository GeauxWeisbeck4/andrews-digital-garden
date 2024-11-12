---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 36 - Process Resources
modified: 2024-11-11T19:31:32-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **36**  
**PROCESS RESOURCES**

Each process consumes system resources such as memory and CPU time. This chapter looks at resource-related system calls. We begin with the _getrusage()_ system call, which allows a process to monitor the resources that it has used or that its children have used. We then look at the _setrlimit()_ and _getrlimit()_ system calls, which can be used to change and retrieve limits on the calling process’s consumption of various resources.

### **36.1 Process Resource Usage**

The _getrusage()_ system call retrieves statistics about various system resources used by the calling process or by all of its children.

#include <sys/resource.h>  
  
int getrusage(int who, struct rusage *res_usage);

Returns 0 on success, or –1 on error

The _who_ argument specifies the process(es) for which resource usage information is to be retrieved. It has one of the following values:

RUSAGE_SELF

Return information about the calling process.

RUSAGE_CHILDREN

Return information about all children of the calling process that have terminated and been waited for.

RUSAGE_THREAD (since Linux 2.6.26)

Return information about the calling thread. This value is Linux-specific.

The _res_usage_ argument is a pointer to a structure of type _rusage_, defined as shown in [Listing 36-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36ex1).

**Listing 36-1:** Definition of the _rusage_ structure

______________________________________________________________________  
  
struct rusage {  
    struct timeval ru_utime;      /* User CPU time used */  
    struct timeval ru_stime;      /* System CPU time used */  
    long           ru_maxrss;     /* Maximum size of resident set (kilobytes)  
                                     [used since Linux 2.6.32] */  
    long           ru_ixrss;      /* Integral (shared) text memory size  
                                     (kilobyte-seconds) [unused] */  
    long           ru_idrss;      /* Integral (unshared) data memory used  
                                     (kilobyte-seconds) [unused] */  
    long           ru_isrss;      /* Integral (unshared) stack memory used  
                                     (kilobyte-seconds) [unused] */  
    long           ru_minflt;     /* Soft page faults (I/O not required) */  
    long           ru_majflt;     /* Hard page faults (I/O required) */  
    long           ru_nswap;      /* Swaps out of physical memory [unused] */  
    long           ru_inblock;    /* Block input operations via file  
                                     system [used since Linux 2.6.22] */  
    long           ru_oublock;    /* Block output operations via file  
                                     system [used since Linux 2.6.22] */  
    long           ru_msgsnd;     /* IPC messages sent [unused] */  
    long           ru_msgrcv;     /* IPC messages received [unused] */  
    long           ru_nsignals;   /* Signals received [unused] */  
    long           ru_nvcsw;      /* Voluntary context switches (process  
                                     relinquished CPU before its time slice  
                                     expired) [used since Linux 2.6] */  
    long           ru_nivcsw;     /* Involuntary context switches (higher  
                                     priority process became runnable or time  
                                     slice ran out) [used since Linux 2.6] */  
};  
______________________________________________________________________

As indicated in the comments in [Listing 36-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36ex1), on Linux, many of the fields in the _rusage_ structure are not filled in by _getrusage()_ (or _wait3()_ and _wait4()_), or they are filled in only by more recent kernel versions. Some of the fields that are unused on Linux are used on other UNIX implementations. These fields are provided on Linux so that, if they are implemented at a future date, the _rusage_ structure does not need to undergo a change that would break existing application binaries.

Although _getrusage()_ appears on most UNIX implementations, it is only weakly specified in SUSv3 (which specifies only the fields _ru_utime_ and _ru_stime_). In part, this is because the meaning of much of the information in the _rusage_ structure is implementation-dependent.

The _ru_utime_ and _ru_stime_ fields are structures of type _timeval_ ([Section 10.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec01)), which return the number of seconds and microseconds of CPU time consumed by a process in user mode and kernel mode, respectively. (Similar information is retrieved by the _times()_ system call described in [Section 10.7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch10.xhtml#ch10lev1sec07).)

The Linux-specific /proc/_PID_/stat files expose some resource usage information (CPU time and page faults) about all processes on the system. See the _proc(5)_ manual page for further details.

The _rusage_ structure returned by the _getrusage()_ RUSAGE_CHILDREN operation includes the resource usage statistics of all of the descendants of the calling process. For example, if we have three processes related as parent, child, and grandchild, then, when the child does a _wait()_ on the grandchild, the resource usage values of the grandchild are added to the child’s RUSAGE_CHILDREN values; when the parent performs a _wait()_ for the child, the resource usage values of both the child and the grandchild are added to the parent’s RUSAGE_CHILDREN values. Conversely, if the child does not _wait()_ on the grandchild, then the grandchild’s resource usages are not recorded in the RUSAGE_CHILDREN values of the parent.

For the RUSAGE_CHILDREN operation, the _ru_maxrss_ field returns the maximum resident set size among all of the descendants of the calling process (rather than a sum for all descendants).

SUSv3 specifies that if SIGCHLD is being ignored (so that children are not turned into zombies that can be waited on), then the child statistics should not be added to the values returned by RUSAGE_CHILDREN. However, as noted in [Section 26.3.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch26.xhtml#ch26lev2sec09), in kernels before 2.6.9, Linux deviates from this requirement—if SIGCHLD is ignored, then the resource usage values for dead children _are_ included in the values returned for RUSAGE_CHILDREN.

### **36.2 Process Resource Limits**

Each process has a set of resource limits that can be used to restrict the amounts of various system resources that the process may consume. For example, we may want to set resource limits on a process before execing an arbitrary program, if we are concerned that it may consume excessive resources. We can set the resource limits of the shell using the _ulimit_ built-in command (_limit_ in the C shell). These limits are inherited by the processes that the shell creates to execute user commands.

Since kernel 2.6.24, the Linux-specific /proc/_PID_/limits file can be used to view all of the resource limits of any process. This file is owned by the real user ID of the corresponding process and its permissions allow reading only by that user ID (or by a privileged process).

The _getrlimit()_ and _setrlimit()_ system calls allow a process to fetch and modify its resource limits.

#include <sys/resource.h>  
  
int getrlimit(int resource, struct rlimit *rlim);  
int setrlimit(int resource, const struct rlimit *rlim);

Both return 0 on success, or –1 on error

The _resource_ argument identifies the resource limit to be retrieved or changed. The _rlim_ argument is used to return resource limit values (_getrlimit()_) or to specify new resource limit values (_setrlimit()_), and is a pointer to a structure containing two fields:

struct rlimit {  
    rlim_t rlim_cur;      /* Soft limit (actual process limit) */  
    rlim_t rlim_max;      /* Hard limit (ceiling for rlim_cur) */  
};

These fields correspond to the two associated limits for a resource: the _soft_ (_rlim_cur_) and _hard_ (_rlim_max_) limits. (The _rlim_t_ data type is an integer type.) The soft limit governs the amount of the resource that may be consumed by the process. A process can adjust the soft limit to any value from 0 up to the hard limit. For most resources, the sole purpose of the hard limit is to provide this ceiling for the soft limit. A privileged (CAP_SYS_RESOURCE) process can adjust the hard limit in either direction (as long as its value remains greater than the soft limit), but an unprivileged process can adjust the hard limit only to a lower value (irreversibly). The value RLIM_INFINITY in _rlim_cur_ or _rlim_max_ means infinity (no limit on the resource), both when retrieved via _getrlimit()_ and when set via _setrlimit()_.

In most cases, resource limits are enforced for both privileged and unprivileged processes. They are inherited by child processes created via _fork()_ and are preserved across an _exec()_.

The values that can be specified for the _resource_ argument of _getrlimit()_ and _setrlimit()_ are summarized in [Table 36-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36table1) and detailed in [Section 36.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36lev1sec03).

Although a resource limit is a per-process attribute, in some cases, the limit is measured against not just that process’s consumption of the corresponding resource, but also against the sum of resources consumed by all processes with the same real user ID. The RLIMIT_NPROC limit, which places a limit on the number of processes that can be created, is a good example of the rationale for this approach. Applying this limit against only the number of children that the process itself created would be ineffective, since each child that the process created would also be able to create further children, which could create more children, and so on. Instead, the limit is measured against the count of all processes that have the same real user ID. Note, however, that the resource limit is checked only in the processes where it has been set (i.e., the process itself and its descendants, which inherit the limit). If another process owned by the same real user ID has not set the limit (i.e., the limit is infinite) or has set a different limit, then that process’s capacity to create children will be checked according to the limit that it has set.

As we describe each resource limit below, we note those limits that are measured against the resources consumed by all processes with the same real user ID. If not otherwise specified, then a resource limit is measured only against the process’s own consumption of the resource.

Be aware that, in many cases, the shell commands for getting and setting resource limits (_ulimit_ in _bash_ and the Korn shell, and _limit_ in the C shell) use different units from those used in _getrlimit()_ and _setrlimit()_. For example, the shell commands typically express the limits on the size of various memory segments in kilobytes.

**Table 36-1:** Resource values for _getrlimit()_ and _setrlimit()_

|**_resource_**|**Limit on**|**SUSv3**|
|---|---|---|
|RLIMIT_AS|Process virtual memory size (bytes)|•|
|RLIMIT_CORE|Core file size (bytes)|•|
|RLIMIT_CPU|CPU time (seconds)|•|
|RLIMIT_DATA|Process data segment (bytes)|•|
|RLIMIT_FSIZE|File size (bytes)|•|
|RLIMIT_MEMLOCK|Locked memory (bytes)||
|RLIMIT_MSGQUEUE|Bytes allocated for POSIX message queues for real user ID (since Linux 2.6.8)||
|RLIMIT_NICE|Nice value (since Linux 2.6.12)||
|RLIMIT_NOFILE|Maximum file descriptor number plus one|•|
|RLIMIT_NPROC|Number of processes for real user ID||
|RLIMIT_RSS|Resident set size (bytes; not implemented)||
|RLIMIT_RTPRIO|Realtime scheduling priority (since Linux 2.6.12)||
|RLIMIT_RTTIME|Realtime CPU time (microseconds; since Linux 2.6.25)||
|RLIMIT_SIGPENDING|Number of queued signals for real user ID (since Linux 2.6.8)||
|RLIMIT_STACK|Size of stack segment (bytes)|•|

##### **Example program**

Before going into the specifics of each resource limit, we look at a simple example of the use of resource limits. [Listing 36-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36ex2) defines the function _printRlimit()_, which displays a message, along with the soft and hard limits for a specified resource.

The _rlim_t_ data type is typically represented in the same way as _off_t_, to handle the representation of RLIMIT_FSIZE, the file size resource limit. For this reason, when printing _rlim_t_ values (as in [Listing 36-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36ex2)), we cast them to _long long_ and use the %lld _printf()_ specifier, as explained in [Section 5.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch05.xhtml#ch05lev1sec10).

The program in [Listing 36-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36ex3) calls _setrlimit()_ to set the soft and hard limits on the number of processes that a user may create (RLIMIT_NPROC), uses the _printRlimit()_ function of [Listing 36-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36ex2) to display the limits before and after the change, and then creates as many processes as possible. When we run this program, setting the soft limit to 30 and the hard limit to 100, we see the following:

$ ./rlimit_nproc 30 100  
Initial maximum process limits:  soft=1024; hard=1024  
New maximum process limits:      soft=30; hard=100  
Child 1 (PID=15674) started  
Child 2 (PID=15675) started  
Child 3 (PID=15676) started  
Child 4 (PID=15677) started  
ERROR [EAGAIN Resource temporarily unavailable] fork

In this example, the program managed to create only 4 new processes, because 26 processes were already running for this user.

**Listing 36-2:** Displaying process resource limits

____________________________________________________ procres/print_rlimit.c  
  
#include <sys/resource.h>  
#include "print_rlimit.h"           /* Declares function defined here */  
#include "tlpi_hdr.h"  
  
int                     /* Print 'msg' followed by limits for 'resource' */  
printRlimit(const char *msg, int resource)  
{  
    struct rlimit rlim;  
  
    if (getrlimit(resource, &rlim) == -1)  
        return -1;  
  
    printf("%s soft=", msg);  
    if (rlim.rlim_cur == RLIM_INFINITY)  
        printf("infinite");  
#ifdef RLIM_SAVED_CUR               /* Not defined on some implementations */  
    else if (rlim.rlim_cur == RLIM_SAVED_CUR)  
        printf("unrepresentable");  
#endif  
    else  
        printf("%lld", (long long) rlim.rlim_cur);  
  
    printf("; hard=");  
    if (rlim.rlim_max == RLIM_INFINITY)  
        printf("infinite\n");  
#ifdef RLIM_SAVED_MAX               /* Not defined on some implementations */  
    else if (rlim.rlim_max == RLIM_SAVED_MAX)  
        printf("unrepresentable");  
#endif  
    else  
        printf("%lld\n", (long long) rlim.rlim_max);  
  
    return 0;  
}  
____________________________________________________ procres/print_rlimit.c

**Listing 36-3:** Setting the RLIMIT_NPROC resource limit

____________________________________________________ procres/rlimit_nproc.c  
  
#include <sys/resource.h>  
#include "print_rlimit.h"             /* Declaration of printRlimit() */  
#include "tlpi_hdr.h"  
  
int  
main(int argc, char *argv[])  
{  
    struct rlimit rl;  
    int j;  
    pid_t childPid;  
  
    if (argc < 2 || argc > 3 || strcmp(argv[1], "--help") == 0)  
        usageErr("%s soft-limit [hard-limit]\n", argv[0]);  
  
    printRlimit("Initial maximum process limits: ", RLIMIT_NPROC);  
  
    /* Set new process limits (hard == soft if not specified) */  
  
    rl.rlim_cur = (argv[1][0] == 'i') ? RLIM_INFINITY :  
                                getInt(argv[1], 0, "soft-limit");  
    rl.rlim_max = (argc == 2) ? rl.rlim_cur :  
                (argv[2][0] == 'i') ? RLIM_INFINITY :  
                                getInt(argv[2], 0, "hard-limit");  
    if (setrlimit(RLIMIT_NPROC, &rl) == -1)  
        errExit("setrlimit");  
  
    printRlimit("New maximum process limits:     ", RLIMIT_NPROC);  
  
    /* Create as many children as possible */  
  
    for (j = 1; ; j++) {  
        switch (childPid = fork()) {  
        case -1: errExit("fork");  
  
        case 0: _exit(EXIT_SUCCESS);            /* Child */  
  
        default:        /* Parent: display message about each new child  
                           and let the resulting zombies accumulate */  
            printf("Child %d (PID=%ld) started\n", j, (long) childPid);  
            break;  
        }  
    }  
}  
____________________________________________________ procres/rlimit_nproc.c

##### **Unrepresentable limit values**

In some programming environments, the _rlim_t_ data type may not be able to represent the full range of values that could be maintained for a particular resource limit. This may be the case on a system that offers multiple programming environments in which the size of the _rlim_t_ data type differs. Such systems can arise if a large-file compilation environment with a 64-bit _off_t_ is added to a system on which _off_t_ was traditionally 32 bits. (In each environment, _rlim_t_ would be the same size as _off_t_.) This leads to the situation where a program with a small _rlim_t_ can, after being execed by a program with a 64-bit _off_t_, inherit a resource limit (e.g., the file size limit) that is greater than the maximum _rlim_t_ value.

To assist portable applications in handling the possibility that a resource limit may be unrepresentable, SUSv3 specifies two constants to indicate unrepresentable limit values: RLIM_SAVED_CUR and RLIM_SAVED_MAX. If a soft resource limit can’t be represented in _rlim_t_, then _getrlimit()_ will return RLIM_SAVED_CUR in the _rlim_cur_ field. RLIM_SAVED_MAX performs an analogous function for an unrepresentable hard limit returned in the _rlim_max_ field.

If all possible resource limit values can be represented in _rlim_t_, then SUSv3 permits an implementation to define RLIM_SAVED_CUR and RLIM_SAVED_MAX to be the same as RLIM_INFINITY. This is how these constants are defined on Linux, implying that all possible resource limit values can be represented in _rlim_t_. However, this is not the case on 32-bit architectures such as x86-32. On those architectures, in a large-file compilation environment (i.e., setting the _FILE_OFFSET_BITS feature test macro to 64 as described in [Section 5.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch05.xhtml#ch05lev1sec10)), the _glibc_ definition of _rlim_t_ is 64 bits wide, but the kernel data type for representing a resource limit is _unsigned long_, which is only 32 bits wide. Current versions of _glibc_ deal with this situation as follows: if a program compiled with _FILE_OFFSET_BITS=64 tries to set a resource limit to a value larger than can be represented in a 32-bit _unsigned long_, then the _glibc_ wrapper for _setrlimit()_ silently converts the value to RLIM_INFINITY. In other words, the requested setting of the resource limit is not honored.

Because utilities that handle files are normally compiled with _FILE_OFFSET_BITS=64 in many x86-32 distributions, the failure to honor resource limits larger than the value that can be represented in 32 bits is a problem that can affect not only application programmers, but also end users.

One could argue that it might be better for the _glibc setrlimit()_ wrapper to give an error if the requested resource limit exceeds the capacity of a 32-bit _unsigned long_. However, the fundamental problem is a kernel limitation, and the behavior described in the main text is the approach that the _glibc_ developers have taken to dealing with it.

### **36.3 Details of Specific Resource Limits**

In this section, we provide details on each of the resource limits available on Linux, noting those that are Linux-specific.

##### RLIMIT_AS

The RLIMIT_AS limit specifies the maximum size for the process’s virtual memory (address space), in bytes. Attempts (_brk()_, _sbrk()_, _mmap()_, _mremap()_, and _shmat()_) to exceed this limit fail with the error ENOMEM. In practice, the most common place where a program may hit this limit is in calls to functions in the _malloc_ package, which make use of _sbrk()_ and _mmap()_. Upon encountering this limit, stack growth can also fail with the consequences listed below for RLIMIT_STACK.

##### RLIMIT_CORE

The RLIMIT_CORE limit specifies the maximum size, in bytes, for core dump files produced when a process is terminated by certain signals ([Section 22.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec01)). Production of a core dump file will stop at this limit. Specifying a limit of 0 prevents creation of core dump files, which is sometimes useful because core dump files can be very large, and end users usually don’t know what to do with them. Another reason for disabling core dumps is security—to prevent the contents of a program’s memory from being dumped to disk. If the RLIMIT_FSIZE limit is lower than this limit, core dump files are limited to RLIMIT_FSIZE bytes.

##### RLIMIT_CPU

The RLIMIT_CPU limit specifies the maximum number of seconds of CPU time (in both system and user mode) that can be used by the process. SUSv3 requires that the SIGXCPU signal be sent to the process when the soft limit is reached, but leaves other details unspecified. (The default action for SIGXCPU is to terminate a process with a core dump.) It is possible to establish a handler for SIGXCPU that does whatever processing is desired and then returns control to the main program. Thereafter, (on Linux) SIGXCPU is sent once per second of consumed CPU time. If the process continues executing until the hard CPU limit is reached, then the kernel sends it a SIGKILL signal, which always terminates the process.

UNIX implementations vary in the details of how they deal with processes that continue consuming CPU time after handling a SIGXCPU signal. Most continue to deliver SIGXCPU at regular intervals. If aiming for portable use of this signal, we should code an application so that, on first receipt of this signal, it does whatever cleanup is required and terminates. (Alternatively, the program could change the resource limit after receiving the signal.)

##### RLIMIT_DATA

The RLIMIT_DATA limit specifies the maximum size, in bytes, of the process’s data segment (the sum of the initialized data, uninitialized data, and heap segments described in [Section 6.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch06.xhtml#ch06lev1sec03)). Attempts (_sbrk()_ and _brk()_) to extend the data segment (program break) beyond this limit fail with the error ENOMEM. As with RLIMIT_AS, the most common place where a program may hit this limit is in calls to functions in the _malloc_ package.

##### RLIMIT_FSIZE

The RLIMIT_FSIZE limit specifies the maximum size of files that the process may create, in bytes. If a process attempts to extend a file beyond the soft limit, it is sent a SIGXFSZ signal, and the system call (e.g., _write()_ or _truncate()_) fails with the error EFBIG. The default action for SIGXFSZ is to terminate a process and produce a core dump. It is possible to instead catch this signal and return control to the main program. However, any further attempt to extend the file will yield the same signal and error.

##### RLIMIT_MEMLOCK

The RLIMIT_MEMLOCK limit (BSD-derived; absent from SUSv3 and available only on Linux and the BSDs) specifies the maximum number of bytes of virtual memory that a process may lock into physical memory, to prevent the memory from being swapped out. This limit affects the _mlock()_ and _mlockall()_ system calls, and the locking options for the _mmap()_ and _shmctl()_ system calls. We describe the details in [Section 50.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch50.xhtml#ch50lev1sec02).

If the MCL_FUTURE flag is specified when calling _mlockall()_, then the RLIMIT_MEMLOCK limit may also cause later calls to _brk()_, _sbrk()_, _mmap()_, or _mremap()_ to fail.

##### RLIMIT_MSGQUEUE

The RLIMIT_MSGQUEUE limit (Linux-specific; since Linux 2.6.8) specifies the maximum number of bytes that can be allocated for POSIX message queues for the real user ID of the calling process. When a POSIX message queue is created using _mq_open()_, bytes are deducted against this limit according to the following formula:

bytes = attr.mq_maxmsg * sizeof(struct msg_msg *) +  
        attr.mq_maxmsg * attr.mq_msgsize;

In this formula, _attr_ is the _mq_attr_ structure that is passed as the fourth argument to _mq_open()_. The addend that includes _sizeof(struct msg_msg *)_ ensures that the user can’t queue an unlimited number of zero-length messages. (The _msg_msg_ structure is a data type used internally by the kernel.) This is necessary because, although zero-length messages contain no data, they do consume some system memory for bookkeeping overhead.

The RLIMIT_MSGQUEUE limit affects only the calling process. Other processes belonging to this user are not affected unless they also set this limit or inherit it.

##### RLIMIT_NICE

The RLIMIT_NICE limit (Linux-specific; since Linux 2.6.12) specifies a floor on the nice value that may be set for this process using _setpriority()_ and _nice()_. The floor is calculated as _20 – rlim_cur_, where _rlim_cur_ is the current RLIMIT_NICE soft resource limit. Refer to [Section 35.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch35.xhtml#ch35lev1sec01) for further details.

##### RLIMIT_NOFILE

The RLIMIT_NOFILE limit specifies a number one greater than the maximum file descriptor number that a process may allocate. Attempts (e.g., _open()_, _pipe()_, _socket()_, _accept()_, _shm_open()_, _dup()_, _dup2()_, _fcntl(F_DUPFD)_, and _epoll_create()_) to allocate descriptors beyond this limit fail. In most cases, the error is EMFILE, but for _dup2(fd, newfd)_ it is EBADF and for _fcntl(fd, F_DUPFD, newfd)_ it is EINVAL if _newfd_ is, in either case, greater than or equal to the limit.

Changes to the RLIMIT_NOFILE limit are reflected in the value returned by _sysconf(_SC_OPEN_MAX)_. SUSv3 permits, but doesn’t require, an implementation to return different values for a call to _sysconf(_SC_OPEN_MAX)_ before and after changing the RLIMIT_NOFILE limit; other implementations may not behave the same as Linux on this point.

SUSv3 states that if an application sets the soft or hard RLIMIT_NOFILE limit to a value less than or equal to the number of the highest file descriptor that the process currently has open, unexpected behavior may occur.

On Linux, we can check which file descriptors a process currently has open by using _readdir()_ to scan the contents of the /proc/_PID_/fd directory, which contains symbolic links for each of the file descriptors currently opened by the process.

The kernel imposes a ceiling on the value to which the RLIMIT_NOFILE limit may be raised. In kernels before 2.6.25, this ceiling is a hard-coded value defined by the kernel constant NR_OPEN, whose value is 1,048,576. (A kernel rebuild is required to raise this ceiling.) Since kernel 2.6.25, the limit is defined by the value in the Linux-specific /proc/sys/fs/nr_open file. The default value in this file is 1,048,576; this can be modified by the superuser. Attempts to set the soft or hard RLIMIT_NOFILE limit higher than the ceiling value yield the error EPERM.

There is also a system-wide limit on the total number of files that may be opened by all processes. This limit can be retrieved and modified via the Linux-specific /proc/sys/fs/file-max file. (Referring to [Section 5.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch05.xhtml#ch05lev1sec04), we can define file-max more precisely as a system-wide limit on the number of open file descriptions.) Only privileged (CAP_SYS_ADMIN) processes can exceed the file-max limit. In an unprivileged process, a system call that encounters the file-max limit fails with the error ENFILE.

##### RLIMIT_NPROC

The RLIMIT_NPROC limit (BSD-derived; absent from SUSv3 and available only on Linux and the BSDs) specifies the maximum number of processes that may be created for the real user ID of the calling process. Attempts (_fork()_, _vfork()_, and _clone()_) to exceed this limit fail with the error EAGAIN.

The RLIMIT_NPROC limit affects only the calling process. Other processes belonging to this user are not affected unless they also set or inherit this limit. This limit is not enforced for privileged (CAP_SYS_ADMIN or CAP_SYS_RESOURCE) processes.

Linux also imposes a system-wide limit on the number of processes that can be created by all users. On Linux 2.4 and later, the Linux-specific /proc/sys/kernel/threads-max file can be used to retrieve and modify this limit.

To be precise, the RLIMIT_NPROC resource limit and the threads-max file are actually limits on the numbers of threads that can be created, rather than the number of processes.

The manner in which the default value for the RLIMIT_NPROC resource limit is set has varied across kernel versions. In Linux 2.2, it was calculated according to a fixed formula. In Linux 2.4 and later, it is calculated using a formula based on the amount of available physical memory.

SUSv3 doesn’t specify the RLIMIT_NPROC resource limit. The SUSv3-mandated method for retrieving (but not changing) the maximum number of processes permitted to a user ID is via the call _sysconf(_SC_CHILD_MAX)_. This _sysconf()_ call is supported on Linux, but in kernel versions before 2.6.23, the call does not return accurate information—it always returns the value 999. Since Linux 2.6.23 (and with _glibc_ 2.4 and later), this call correctly reports the limit (by checking the value of the RLIMIT_NPROC resource limit).

There is no portable way of discovering how many processes have already been created for a specific user ID. On Linux, we can try scanning all of the /proc/_PID_/status files on the system and examining the information under the Uid entry (which lists the four process user IDs in the order: real, effective, saved set, and file system) in order to estimate the number of processes currently owned by a user. Be aware, however, that by the time we have completed such a scan, this information may already have changed.

##### RLIMIT_RSS

The RLIMIT_RSS limit (BSD-derived; absent from SUSv3, but widely available) specifies the maximum number of pages in the process’s resident set; that is, the total number of virtual memory pages currently in physical memory. This limit is provided on Linux, but it currently has no effect.

In older Linux 2.4 kernels (up to and including 2.4.29), RLIMIT_RSS did have an effect on the behavior of the _madvise()_ MADV_WILLNEED operation ([Section 50.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch50.xhtml#ch50lev1sec04)). If this operation could not be performed as a result of encountering the RLIMIT_RSS limit, the error EIO was returned in _errno_.

##### RLIMIT_RTPRIO

The RLIMIT_RTPRIO limit (Linux-specific; since Linux 2.6.12) specifies a ceiling on the realtime priority that may be set for this process using _sched_setscheduler()_ and _sched_setparam()_. Refer to [Section 35.3.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch35.xhtml#ch35lev2sec05) for further details.

##### RLIMIT_RTTIME

The RLIMIT_RTTIME limit (Linux-specific; since Linux 2.6.25) specifies the maximum amount of CPU time in microseconds that a process running under a realtime scheduling policy may consume without sleeping (i.e., performing a blocking system call). The behavior if this limit is reached is the same as for RLIMIT_CPU: if the process reaches the soft limit, then a SIGXCPU signal is sent to the process, and further SIGXCPU signals are sent for each additional second of CPU time consumed. On reaching the hard limit, a SIGKILL signal is sent. Refer to [Section 35.3.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch35.xhtml#ch35lev2sec05) for further details.

##### RLIMIT_SIGPENDING

The RLIMIT_SIGPENDING limit (Linux-specific; since Linux 2.6.8) specifies the maximum number of signals that may be queued for the real user ID of the calling process. Attempts (_sigqueue()_) to exceed this limit fail with the error EAGAIN.

The RLIMIT_SIGPENDING limit affects only the calling process. Other processes belonging to this user are not affected unless they also set or inherit this limit.

As initially implemented, the default value for the RLIMIT_SIGPENDING limit was 1024. Since kernel 2.6.12, the default value has been changed to be the same as the default value for RLIMIT_NPROC.

For the purposes of checking the RLIMIT_SIGPENDING limit, the count of queued signals includes both realtime and standard signals. (Standard signals can be queued only once to a process.) However, this limit is enforced only for _sigqueue()_. Even if the number of signals specified by this limit has already been queued to processes belonging to this real user ID, it is still possible to use _kill()_ to queue one instance of each of the signals (including realtime signals) that are not already queued to a process.

From kernel 2.6.12 onward, the SigQ field of the Linux-specific /proc/_PID_/status file displays the current and maximum number of queued signals for the real user ID of the process.

##### RLIMIT_STACK

The RLIMIT_STACK limit specifies the maximum size of the process stack, in bytes. Attempts to grow the stack beyond this limit result in the generation of a SIGSEGV signal for the process. Since the stack is exhausted, the only way to catch this signal is by establishing an alternate signal stack, as described in [Section 21.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev1sec03).

Since Linux 2.6.23, the RLIMIT_STACK limit also determines the amount of space available for holding the process’s command-line arguments and environment variables. See the _execve(2)_ manual page for details.

### **36.4 Summary**

Processes consume various system resources. The _getrusage()_ system call allows a process to monitor certain of the resources consumed by itself and by its children.

The _setrlimit()_ and _getrlimit()_ system calls allow a process to set and retrieve limits on its consumption of various resources. Each resource limit has two components: a soft limit, which is what the kernel enforces when checking a process’s resource consumption, and a hard limit, which acts as a ceiling on the value of the soft limit. An unprivileged process can set the soft limit for a resource to any value in the range from 0 up to the hard limit, but can only lower the hard limit. A privileged process can make any changes to either limit value, as long as the soft limit is less than or equal to the hard limit. If a process encounters a soft limit, it is typically informed of the fact either by receiving a signal or via failure of the system call that attempts to exceed the limit.

### **36.5 Exercises**

**36-1.**   Write a program that shows that the _getrusage()_ RUSAGE_CHILDREN flag retrieves information about only the children for which a _wait()_ call has been performed. (Have the program create a child process that consumes some CPU time, and then have the parent call _getrusage()_ before and after calling _wait()_.)

**36-2.**   Write a program that executes a command and then displays its resource usage. This is analogous to what the _time(1)_ command does. Thus, we would use this program as follows:

$ ./rusage command arg...

**36-3.**   Write programs to determine what happens if a process’s consumption of various resources already exceeds the soft limit specified in a call to _setrlimit()_.