---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 22 - Signals - Advanced Features
modified: 2024-11-11T19:21:57-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **22**  
**SIGNALS: ADVANCED FEATURES**

This chapter completes the discussion of signals that we began in [Chapter 20](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20), covering a number of more advanced topics, including the following:

• core dump files;

• special cases regarding signal delivery, disposition, and handling;

• synchronous and asynchronous generation of signals;

• when and in what order signals are delivered;

• realtime signals;

• the use of _sigsuspend(_) to set the process signal mask and wait for a signal to arrive;

• the use of _sigwaitinfo()_ (and _sigtimedwait()_) to synchronously wait for a signal to arrive;

• the use of _signalfd()_ to receive a signal via a file descriptor; and

• the older BSD and System V signal APIs.

### **22.1 Core Dump Files**

Certain signals cause a process to create a core dump and terminate ([Table 20-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20table1), [page 396](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#page_396)). A core dump is a file containing a memory image of the process at the time it terminated. (The term _core_ derives from an old memory technology.) This memory image can be loaded into a debugger in order to examine the state of a program’s code and data at the moment when the signal arrived.

One way of causing a program to produce a core dump is to type the _quit_ character (usually _Control-\_), which causes the SIGQUIT signal to be generated:

$ ulimit -c unlimited                      Explained in main text  
$ sleep 30  
Type Control-\  
Quit (core dumped)  
$ ls -l core                               Shows core dump file for sleep(1)  
-rw-------   1 mtk   users     57344 Nov 30 13:39 core

In this example, the message _Quit (core dumped)_ is printed by the shell, which detects that its child (the process running _sleep_) was killed by SIGQUIT and did a core dump.

The core dump file was created in the working directory of the process, with the name core. This is the default location and name for a core dump file; shortly, we explain how these defaults can be changed.

Many implementations provide a tool (e.g., _gcore_ on FreeBSD and Solaris) to obtain a core dump of a running process. Similar functionality is available on Linux by attaching to a running process using _gdb_ and then using the _gcore_ command.

##### **Circumstances in which core dump files are not produced**

A core dump is not produced in the following circumstances:

• The process doesn’t have permission to write the core dump file. This could happen because the process doesn’t have write permission for the directory in which the core dump file is to be created, or because a file with the same name already exists and either is not writable or is not a regular file (e.g., it is a directory or a symbolic link).

• A regular file with the same name already exists, and is writable, but there is more than one (hard) link to the file.

• The directory in which the core dump file is to be created doesn’t exist.

• The process resource limit on the size of a core dump file is set to 0. This limit, RLIMIT_CORE, is discussed in more detail in [Section 36.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36lev1sec03). In the example above, we used the _ulimit_ command (_limit_ in the C shell) to ensure that there is no limit on the size of core files.

• The process resource limit on the size of a file that may be produced by the process is set to 0. We describe this limit, RLIMIT_FSIZE, in [Section 36.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36lev1sec03).

• The binary executable file that the process is executing doesn’t have read permission enabled. This prevents users from using a core dump to obtain a copy of the code of a program that they would otherwise be unable to read.

• The file system on which the current working directory resides is mounted read-only, is full, or has run out of i-nodes. Alternatively, the user has reached their quota limit on the file system.

• Set-user-ID (set-group-ID) programs executed by a user other than the file owner (group owner) don’t generate core dumps. This prevents malicious users from dumping the memory of a secure program and examining it for sensitive information such as passwords.

Using the PR_SET_DUMPABLE operation of the Linux-specific _prctl()_ system call, we can set the _dumpable_ flag for a process, so that when a set-user-ID (set-group-ID) program is run by a user other than the owner (group owner), a core dump can be produced. The PR_SET_DUMPABLE operation is available from Linux 2.4 onward. See the _prctl(2)_ manual page for further details. In addition, since kernel 2.6.13, the /proc/sys/fs/suid_dumpable file provides system-wide control over whether or not set-user-ID and set-group-ID processes produce core dumps. For details, see the _proc(5)_ manual page.

Since kernel 2.6.23, the Linux-specific /proc/_PID_/coredump_filter can be used on a per-process basis to determine which types of memory mappings are written to a core dump file. (We explain memory mappings in [Chapter 49](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch49.xhtml#ch49).) The value in this file is a mask of four bits corresponding to the four types of memory mappings: private anonymous mappings, private file mappings, shared anonymous mappings, and shared file mappings. The default value of the file provides traditional Linux behavior: only private anonymous and shared anonymous mappings are dumped. See the _core(5)_ manual page for further details.

##### **Naming the core dump file:** /proc/sys/kernel/core_pattern

Starting with Linux 2.6, the format string contained in the Linux-specific /proc/sys/kernel/core_pattern file controls the naming of all core dump files produced on the system. By default, this file contains the string _core_. A privileged user can define this file to include any of the format specifiers shown in [Table 22-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22table1). These format specifiers are replaced by the value indicated in the right column of the table. Additionally, the string may include slashes (/). In other words, we can control not just the name of the core file, but also the (absolute or relative) directory in which it is created. After all format specifiers have been replaced, the resulting pathname string is truncated to a maximum of 128 characters (64 characters before Linux 2.6.19).

Since kernel 2.6.19, Linux supports an additional syntax in the core_pattern file. If this file contains a string starting with the pipe symbol (|), then the remaining characters in the file are interpreted as a program—with optional arguments that may include the % specifiers shown in [Table 22-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22table1)—that is to be executed when a process dumps core. The core dump is written to the standard input of that program instead of to a file. See the _core(5)_ manual page for further details.

Some other UNIX implementations provide facilities similar to core_pattern. For example, in BSD derivatives, the program name is appended to the filename, thus core._progname_. Solaris provides a tool (_coreadm_) that allows the user to choose the filename and directory where core dump files are placed.

**Table 22-1:** Format specifiers for /proc/sys/kernel/core_pattern

|**Specifier**|**Replaced by**|
|---|---|
|%c|Core file size soft resource limit (bytes; since Linux 2.6.24)|
|%e|Executable filename (without path prefix)|
|%g|Real group ID of dumped process|
|%h|Name of host system|
|%p|Process ID of dumped process|
|%s|Number of signal that terminated process|
|%t|Time of dump, in seconds since the Epoch|
|%u|Real user ID of dumped process|
|%%|A single % character|

### **22.2 Special Cases for Delivery, Disposition, and Handling**

For certain signals, special rules apply regarding delivery, disposition, and handling, as described in this section.

##### SIGKILL **and** SIGSTOP

It is not possible to change the default action for SIGKILL, which always terminates a process, and SIGSTOP, which always stops a process. Both _signal()_ and _sigaction()_ return an error on attempts to change the disposition of these signals. These two signals also can’t be blocked. This is a deliberate design decision. Disallowing changes to the default actions of these signals means that they can always be used to kill or stop a runaway process.

##### SIGCONT **and stop signals**

As noted earlier, the SIGCONT signal is used to continue a process previously stopped by one of the stop signals (SIGSTOP, SIGTSTP, SIGTTIN, and SIGTTOU). Because of their unique purpose, in certain situations the kernel deals with these signals differently from other signals.

If a process is currently stopped, the arrival of a SIGCONT signal always causes the process to resume, even if the process is currently blocking or ignoring SIGCONT. This feature is necessary because it would otherwise be impossible to resume such stopped processes. (If the stopped process was blocking SIGCONT, and had established a handler for SIGCONT, then, after the process is resumed, the handler is invoked only when SIGCONT is later unblocked.)

If any other signal is sent to a stopped process, the signal is not actually delivered to the process until it is resumed via receipt of a SIGCONT signal. The one exception is SIGKILL, which always kills a process—even one that is currently stopped.

Whenever SIGCONT is delivered to a process, any pending stop signals for the process are discarded (i.e., the process never sees them). Conversely, if any of the stop signals is delivered to a process, then any pending SIGCONT signal is automatically discarded. These steps are taken in order to prevent the action of a SIGCONT signal from being subsequently undone by a stop signal that was actually sent beforehand, and vice versa.

##### **Don’t change the disposition of ignored terminal-generated signals**

If, at the time it was execed, a program finds that the disposition of a terminal-generated signal has been set to SIG_IGN (ignore), then generally the program should not attempt to change the disposition of the signal. This is not a rule enforced by the system, but rather a convention that should be followed when writing applications. We explain the reasons for this in [Section 34.7.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch34.xhtml#ch34lev2sec05). The signals for which this convention is relevant are SIGHUP, SIGINT, SIGQUIT, SIGTTIN, SIGTTOU, and SIGTSTP.

### **22.3 Interruptible and Uninterruptible Process Sleep States**

We need to add a proviso to our earlier statement that SIGKILL and SIGSTOP always act immediately on a process. At various times, the kernel may put a process to sleep, and two sleep states are distinguished:

• TASK_INTERRUPTIBLE: The process is waiting for some event. For example, it is waiting for terminal input, for data to be written to a currently empty pipe, or for the value of a System V semaphore to be increased. A process may spend an arbitrary length of time in this state. If a signal is generated for a process in this state, then the operation is interrupted and the process is woken up by the delivery of a signal. When listed by _ps(1)_, processes in the TASK_INTERRUPTIBLE state are marked by the letter _S_ in the STAT (process state) field.

• TASK_UNINTERRUPTIBLE: The process is waiting on certain special classes of event, such as the completion of a disk I/O. If a signal is generated for a process in this state, then the signal is not delivered until the process emerges from this state. Processes in the TASK_UNINTERRUPTIBLE state are listed by _ps(1)_ with a _D_ in the STAT field.

Because a process normally spends only very brief periods in the TASK_UNINTERRUPTIBLE state, the fact that a signal is delivered only when the process leaves this state is invisible. However, in rare circumstances, a process may remain hung in this state, perhaps as the result of a hardware failure, an NFS problem, or a kernel bug. In such cases, SIGKILL won’t terminate the hung process. If the underlying problem can’t otherwise be resolved, then we must restart the system in order to eliminate the process.

The TASK_INTERRUPTIBLE and TASK_UNINTERRUPTIBLE states are present on most UNIX implementations. Starting with kernel 2.6.25, Linux adds a third state to address the hanging process problem just described:

• TASK_KILLABLE: This state is like TASK_UNINTERRUPTIBLE, but wakes the process if a fatal signal (i.e., one that would kill the process) is received. By converting relevant parts of the kernel code to use this state, various scenarios where a hung process requires a system restart can be avoided. Instead, the process can be killed by sending it a fatal signal. The first piece of kernel code to be converted to use TASK_KILLABLE was NFS.

### **22.4 Hardware-Generated Signals**

SIGBUS, SIGFPE, SIGILL, and SIGSEGV can be generated as a consequence of a hardware exception or, less usually, by being sent by _kill()_. In the case of a hardware exception, SUSv3 specifies that the behavior of a process is undefined if it returns from a handler for the signal, or if it ignores or blocks the signal. The reasons for this are as follows:

• _Returning from the signal handler_: Suppose that a machine-language instruction generates one of these signals, and a signal handler is consequently invoked. On normal return from the handler, the program attempts to resume execution at the point where it was interrupted. But this is the very instruction that generated the signal in the first place, so the signal is generated once more. The consequence is usually that the program goes into an infinite loop, repeatedly calling the signal handler.

• _Ignoring the signal_: It makes little sense to ignore a hardware-generated signal, as it is unclear how a program should continue execution after, say, an arithmetic exception. When one of these signals is generated as a consequence of a hardware exception, Linux forces its delivery, even if the program has requested that the signal be ignored.

• _Blocking the signal_: As with the previous case, it makes little sense to block a hardware-generated signal, as it is unclear how a program should then continue execution. On Linux 2.4 and earlier, the kernel simply ignores attempts to block a hardware-generated signal; the signal is delivered to the process anyway, and then either terminates the process or is caught by a signal handler, if one has been established. Starting with Linux 2.6, if the signal is blocked, then the process is always immediately killed by that signal, even if the process has installed a handler for the signal. (The rationale for the Linux 2.6 change in the treatment of blocked hardware-generated signals was that the Linux 2.4 behavior hid bugs and could cause deadlocks in threaded programs.)

The signals/demo_SIGFPE.c program in the source code distribution for this book can be used to demonstrate the results of ignoring or blocking SIGFPE or catching the signal with a handler that performs a normal return.

The correct way to deal with hardware-generated signals is either to accept their default action (process termination) or to write handlers that don’t perform a normal return. Other than returning normally, a handler can complete execution by calling __exit()_ to terminate the process or by calling _siglongjmp()_ ([Section 21.2.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev2sec04)) to ensure that control passes to some point in the program other than the instruction that generated the signal.

### **22.5 Synchronous and Asynchronous Signal Generation**

We have already seen that a process generally can’t predict when it will receive a signal. We now need to qualify this observation by distinguishing between _synchronous_ and _asynchronous_ signal generation.

The model we have implicitly considered so far is _asynchronous_ signal generation, in which the signal is sent either by another process or generated by the kernel for an event that occurs independently of the execution of the process (e.g., the user types the _interrupt_ character or a child of this process terminates). For asynchronously generated signals, the earlier statement that a process can’t predict when the signal will be delivered holds true.

However, in some cases, a signal is generated while the process itself is executing. We have already seen two examples of this:

• The hardware-generated signals (SIGBUS, SIGFPE, SIGILL, SIGSEGV, and SIGEMT) described in [Section 22.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec04) are generated as a consequence of executing a specific machine-language instruction that results in a hardware exception.

• A process can use _raise()_, _kill()_, or _killpg()_ to send a signal to itself.

In these cases, the generation of the signal is _synchronous_—the signal is delivered immediately (unless it is blocked, but see [Section 22.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec04) for a discussion of what happens when blocking hardware-generated signals). In other words, the earlier statement about the unpredictability of the delivery of a signal doesn’t apply. For synchronously generated signals, delivery is predictable and reproducible.

Note that synchronicity is an attribute of how a signal is generated, rather than of the signal itself. All signals may be generated synchronously (e.g., when a process sends itself a signal using _kill()_) or asynchronously (e.g., when the signal is sent by another process using _kill()_).

### **22.6 Timing and Order of Signal Delivery**

As the first topic of this section, we consider exactly when a pending signal is delivered. We then consider what happens if multiple pending blocked signals are simultaneously unblocked.

##### **When is a signal delivered?**

As noted in [Section 22.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec05), synchronously generated signals are delivered immediately. For example, a hardware exception triggers an immediate signal, and when a process sends itself a signal using _raise()_, the signal is delivered before the _raise()_ call returns.

When a signal is generated asynchronously, there may be a (small) delay while the signal is pending between the time when it was generated and the time it is actually delivered, even if we have not blocked the signal. The reason for this is that the kernel delivers a pending signal to a process only at the next switch from kernel mode to user mode while executing that process. In practice, this means the signal is delivered at one of the following times:

• when the process is rescheduled after it earlier timed out (i.e., at the start of a time slice); or

• at completion of a system call (delivery of the signal may cause a blocking system call to complete prematurely).

##### **Order of delivery of multiple unblocked signals**

If a process has multiple pending signals that are unblocked using _sigprocmask()_, then all of these signals are immediately delivered to the process.

As currently implemented, the Linux kernel delivers the signals in ascending order. For example, if pending SIGINT (signal 2) and SIGQUIT (signal 3) signals were both simultaneously unblocked, then the SIGINT signal would be delivered before SIGQUIT, regardless of the order in which the two signals were generated.

We can’t, however, rely on (standard) signals being delivered in any particular order, since SUSv3 says that the delivery order of multiple signals is implementation-defined. (This statement applies only to standard signals. As we’ll see in [Section 22.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec08), the standards governing realtime signals do provide guarantees about the order in which multiple unblocked realtime signals are delivered.)

When multiple unblocked signals are awaiting delivery, if a switch between kernel mode and user mode occurs during the execution of a signal handler, then the execution of that handler will be interrupted by the invocation of a second signal handler (and so on), as shown in [Figure 22-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22fig1).

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f22-01.jpg)

**Figure 22-1:** Delivery of multiple unblocked signals

### **22.7 Implementation and Portability of _signal()_**

In this section, we show how to implement _signal()_ using _sigaction()_. The implementation is straightforward, but needs to account for the fact that, historically and across different UNIX implementations, _signal()_ has had different semantics. In particular, early implementations of signals were unreliable, meaning that:

• On entry to a signal handler, the disposition of the signal was reset to its default. (This corresponds to the SA_RESETHAND flag described in [Section 20.13](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec13).) In order to have the signal handler invoked again for a subsequent delivery of the same signal, the programmer needed to make a call to _signal()_ from within the handler to explicitly reestablish the handler. The problem in this scenario is that there is a small window of time between entering the signal handler and reestablishment of the handler, during which, if the signal arrives a second time, it would be processed according to its default disposition.

• Delivery of further occurrences of a signal was not blocked during execution of a signal handler. (This corresponds to the SA_NODEFER flag described in [Section 20.13](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec13).) This meant that if the signal was delivered again while the handler was still executing, then the handler would be recursively invoked. Given a sufficiently rapid stream of signals, the resulting recursive invocations of the handler could overflow the stack.

As well as being unreliable, early UNIX implementations did not provide automatic restarting of system calls (i.e., the behavior described for the SA_RESTART flag in [Section 21.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev1sec05)).

The 4.2BSD reliable signals implementation rectified these limitations, and several other UNIX implementations followed suit. However, the older semantics live on today in the System V implementation of _signal()_, and even contemporary standards such as SUSv3 and C99 leave these aspects of _signal()_ deliberately unspecified.

Tying the above information together, we implement _signal()_ as shown in [Listing 22-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex1). By default, this implementation provides the modern signal semantics. If compiled with _–DOLD_SIGNAL_, then it provides the earlier unreliable signal semantics and doesn’t enable automatic restarting of system calls.

**Listing 22-1:** An implementation of _signal()_

_________________________________________________________ signals/signal.c  
  
#include <signal.h>  
  
typedef void (*sighandler_t)(int);  
  
sighandler_t  
signal(int sig, sighandler_t handler)  
{  
    struct sigaction newDisp, prevDisp;  
  
    newDisp.sa_handler = handler;  
    sigemptyset(&newDisp.sa_mask);  
#ifdef OLD_SIGNAL  
    newDisp.sa_flags = SA_RESETHAND | SA_NODEFER;  
#else  
    newDisp.sa_flags = SA_RESTART;  
#endif  
  
    if (sigaction(sig, &newDisp, &prevDisp) == -1)  
        return SIG_ERR;  
    else  
        return prevDisp.sa_handler;  
}  
_________________________________________________________ signals/signal.c

##### **Some _glibc_ details**

The _glibc_ implementation of the _signal()_ library function has changed over time. In newer versions of the library (_glibc 2_ and later), the modern semantics are provided by default. In older versions of the library, the earlier unreliable (System V-compatible) semantics are provided.

The Linux kernel contains an implementation of _signal()_ as a system call. This implementation provides the older, unreliable semantics. However, _glibc_ bypasses this system call by providing a _signal()_ library function that calls _sigaction()_.

If we want to obtain unreliable signal semantics with modern versions of _glibc_, we can explicitly replace our calls to _signal()_ with calls to the (nonstandard) _sysv_signal()_ function.

#define _GNU_SOURCE  
#include <signal.h>  
  
void ( *sysv_signal(int sig, void (*handler)(int)) ) (int);

Returns previous signal disposition on success, or SIG_ERR on error

The _sysv_signal()_ function takes the same arguments as _signal()_.

If the _BSD_SOURCE feature test macro is not defined when compiling a program, _glibc_ implicitly redefines all calls to _signal()_ to be calls to _sysv_signal()_, meaning that _signal()_ has unreliable semantics. By default, _BSD_SOURCE _is_ defined, but it is disabled (unless also explicitly defined) if other feature test macros such as _SVID_SOURCE or _XOPEN_SOURCE are defined when compiling a program.

##### **_sigaction()_ is the preferred API for establishing a signal handler**

Because of the System V versus BSD (and old versus recent _glibc_) portability issues described above, it is good practice always to use _sigaction()_, rather than _signal()_, to establish signal handlers. We follow this practice throughout the remainder of this book. (An alternative is to write our own version of _signal()_, probably similar to [Listing 22-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex1), specifying exactly the flags that we require, and employ that version with our applications.) Note, however, that it is portable (and shorter) to use _signal()_ to set the disposition of a signal to SIG_IGN or SIG_DFL, and we’ll often use _signal()_ for that purpose.

### **22.8 Realtime Signals**

Realtime signals were defined in POSIX.1b to remedy a number of limitations of standard signals. They have the following advantages over standard signals:

• Realtime signals provide an increased range of signals that can be used for application-defined purposes. Only two standard signals are freely available for application-defined purposes: SIGUSR1 and SIGUSR2.

• Realtime signals are queued. If multiple instances of a realtime signal are sent to a process, then the signal is delivered multiple times. By contrast, if we send further instances of a standard signal that is already pending for a process, that signal is delivered only once.

• When sending a realtime signal, it is possible to specify data (an integer or pointer value) that accompanies the signal. The signal handler in the receiving process can retrieve this data.

• The order of delivery of different realtime signals is guaranteed. If multiple different realtime signals are pending, then the lowest-numbered signal is delivered first. In other words, signals are prioritized, with lower-numbered signals having higher priority. When multiple signals of the same type are queued, they are delivered—along with their accompanying data—in the order in which they were sent.

SUSv3 requires that an implementation provide a minimum of _POSIX_RTSIG_MAX (defined as 8) different realtime signals. The Linux kernel defines 33 different real-time signals, numbered from 32 to 64. The <limits.h> header file defines the constant RTSIG_MAX to indicate the number of available realtime signals, and the constants SIGRTMIN and SIGRTMAX to indicate the lowest and highest available realtime signal numbers.

On systems employing the LinuxThreads threading implementation, SIGRTMIN is defined as 35 (rather than 32) to allow for the fact that LinuxThreads makes internal use of the first three realtime signals. On systems employing the NPTL threading implementation, SIGRTMIN is defined as 34 to allow for the fact that NPTL makes internal use of the first two realtime signals.

Realtime signals are not individually identified by different constants in the manner of standard signals. However, an application should not hard-code integer values for them, since the range used for realtime signals varies across UNIX implementations. Instead, a realtime signal number can be referred to by adding a value to SIGRTMIN; for example, the expression _(SIGRTMIN + 1)_ refers to the second realtime signal.

Be aware that SUSv3 doesn’t require SIGRTMAX and SIGRTMIN to be simple integer values. They may be defined as functions (as they are on Linux). This means that we can’t write code for the preprocessor such as the following:

#if SIGRTMIN+100 > SIGRTMAX             /* WRONG! */  
#error "Not enough realtime signals"  
#endif

Instead, we must perform equivalent checks at run time.

##### **Limits on the number of queued realtime signals**

Queuing realtime signals (with associated data) requires that the kernel maintain data structures listing the signals queued to each process. Since these data structures consume kernel memory, the kernel places limits on the number of realtime signals that may be queued.

SUSv3 allows an implementation to place an upper limit on the number of real-time signals (of all types) that may be queued by a process, and requires that this limit be at least _POSIX_SIGQUEUE_MAX (defined as 32). An implementation can define the constant SIGQUEUE_MAX to indicate the number of realtime signals it allows to be queued. It can also make this information available through the following call:

lim = sysconf(_SC_SIGQUEUE_MAX);

In systems with _glibc_ versions before 2.4, this call returns –1. Since _glibc_ 2.4, the return value depends on the kernel version. Before Linux 2.6.8, the call returns the value in the Linux-specific /proc/sys/kernel/rtsig-max file. This file defines a systemwide limit on the number of realtime signals that may be queued to all processes. The default value is 1024, but a privileged process can change it. The Linux-specific /proc/sys/kernel/rtsig-nr file shows the number of currently queued realtime signals.

Starting with Linux 2.6.8, these /proc files disappear. In their place, the RLIMIT_SIGPENDING resource limit ([Section 36.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch36.xhtml#ch36lev1sec03)) limits the number of signals that can be queued to all processes owned by a particular real user ID. Since _glibc_ 2.10, the _sysconf()_ call returns the RLIMIT_SIGPENDING limit. (The SigQ field of the Linux-specific /proc/_PID_/status file displays the number of realtime signals pending for a process.)

##### **Using realtime signals**

In order for a pair of processes to send and receive realtime signals, SUSv3 requires the following:

• The sending process sends the signal plus its accompanying data using the _sigqueue()_ system call.

A realtime signal can also be sent using _kill()_, _killpg()_, and _raise()_. However, SUSv3 leaves it as implementation-dependent whether realtime signals sent using these interfaces are queued. On Linux, these interfaces do queue real-time signals, but on many other UNIX implementations, they do not.

• The receiving process establishes a handler for the signal using a call to _sigaction()_ that specifies the SA_SIGINFO flag. This causes the signal handler to be invoked with additional arguments, one of which includes the data accompanying the realtime signal.

On Linux, it is possible to queue realtime signals even if the receiving process doesn’t specify the SA_SIGINFO flag when establishing the signal handler (although it is not then possible to obtain the data associated with the signal in this case). However, SUSv3 doesn’t require implementations to guarantee this behavior, so we can’t portably rely on it.

#### **22.8.1 Sending Realtime Signals**

The _sigqueue()_ system call sends the realtime signal specified by _sig_ to the process specified by _pid_.

#define _POSIX_C_SOURCE 199309  
#include <signal.h>  
  
int sigqueue(pid_t pid, int sig, const union sigval value);

Returns 0 on success, or –1 on error

The same permissions are required to send a signal using _sigqueue()_ as are required with _kill()_ (see [Section 20.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec05)). A null signal (i.e., signal 0) can be sent, with the same meaning as for _kill()_. (Unlike _kill()_, we can’t use _sigqueue()_ to send a signal to an entire process group by specifying a negative value in _pid_.)

**Listing 22-2:** Using _sigqueue()_ to send realtime signals

______________________________________________________ signals/t_sigqueue.c  
  
#define _POSIX_C_SOURCE 199309  
#include <signal.h>  
#include "tlpi_hdr.h"  
  
int  
main(int argc, char *argv[])  
{  
    int sig, numSigs, j, sigData;  
    union sigval sv;  
  
    if (argc < 4 || strcmp(argv[1], "--help") == 0)  
        usageErr("%s pid sig-num data [num-sigs]\n", argv[0]);  
  
    /* Display our PID and UID, so that they can be compared with the  
       corresponding fields of the siginfo_t argument supplied to the  
       handler in the receiving process */  
  
    printf("%s: PID is %ld, UID is %ld\n", argv[0],  
            (long) getpid(), (long) getuid());  
  
    sig = getInt(argv[2], 0, "sig-num");  
    sigData = getInt(argv[3], GN_ANY_BASE, "data");  
    numSigs = (argc > 4) ? getInt(argv[4], GN_GT_0, "num-sigs") : 1;  
  
    for (j = 0; j < numSigs; j++) {  
        sv.sival_int = sigData + j;  
        if (sigqueue(getLong(argv[1], 0, "pid"), sig, sv) == -1)  
            errExit("sigqueue %d", j);  
    }  
  
    exit(EXIT_SUCCESS);  
}  
______________________________________________________ signals/t_sigqueue.c

The _value_ argument specifies the data to accompany the signal. This argument has the following form:

union sigval {  
    int   sival_int;     /* Integer value for accompanying data */  
    void *sival_ptr;     /* Pointer value for accompanying data */  
};

The interpretation of this argument is application-dependent, as is the choice of whether to set the _sival_int_ or the _sival_ptr_ field of the union. The _sival_ptr_ field is seldom useful with _sigqueue()_, since a pointer value that is useful in one process is rarely meaningful in another process. However, this field is useful in other functions that employ _sigval_ unions, as we’ll see when we consider POSIX timers in [Section 23.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch23.xhtml#ch23lev1sec06) and POSIX message queue notification in [Section 52.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch52.xhtml#ch52lev1sec06).

Several UNIX implementations, including Linux, define a _sigval_t_ data type as a synonym for _union sigval_. However, this type is not specified in SUSv3 and is not available on some implementations. Portable applications should avoid using it.

A call to _sigqueue()_ may fail if the limit on the number of queued signals has been reached. In this case, _errno_ is set to EAGAIN, indicating that we need to send the signal again (at some later time when some of the currently queued signals have been delivered).

An example of the use of _sigqueue()_ is provided in [Listing 22-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex2) ([page 459](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#page_459)). This program takes up to four arguments, of which the first three are mandatory: a target process ID, a signal number, and an integer value to accompany the realtime signal. If more than one instance of the specified signal is to be sent, the optional fourth argument specifies the number of instances; in this case, the accompanying integer data value is incremented by one for each successive signal. We demonstrate the use of this program in [Section 22.8.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev2sec02).

#### **22.8.2 Handling Realtime Signals**

We can handle realtime signals just like standard signals, using a normal (single-argument) signal handler. Alternatively, we can handle a realtime signal using a three-argument signal handler established using the SA_SIGINFO flag ([Section 21.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev1sec04)). Here is an example of using SA_SIGINFO to establish a handler for the sixth realtime signal:

struct sigaction act;  
  
sigemptyset(&act.sa_mask);  
act.sa_sigaction = handler;  
act.sa_flags = SA_RESTART | SA_SIGINFO;  
  
if (sigaction(SIGRTMIN + 5, &act, NULL) == -1)  
    errExit("sigaction");

When we employ the SA_SIGINFO flag, the second argument passed to the signal handler is a _siginfo_t_ structure that contains additional information about the realtime signal. We described this structure in detail in [Section 21.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev1sec04). For a realtime signal, the following fields are set in the _siginfo_t_ structure:

• The _si_signo_ field is the same value as is passed in the first argument of the signal handler.

• The _si_code_ field indicates the source of the signal, and contains one of the values shown in [Table 21-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21table2) ([page 441](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#page_441)). For a realtime signal sent via _sigqueue()_, this field always has the value SI_QUEUE.

• The _si_value_ field contains the data specified in the _value_ argument (the _sigval_ union) by the process that sent the signal using _sigqueue()_. As noted already, the interpretation of this data is application-defined. (The _si_value_ field doesn’t contain valid information if the signal was sent using _kill()_.)

• The _si_pid_ and _si_uid_ fields contain, respectively, the process ID and real user ID of the process sending the signal.

[Listing 22-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex3) provides an example of handling realtime signals. This program catches signals and displays various fields from the _siginfo_t_ structure passed to the signal handler. The program takes two optional integer command-line arguments. If the first argument is supplied, the main program blocks all signals, and then sleeps for the number of seconds specified by this argument. During this time, we can queue multiple realtime signals to the process and observe what happens when the signals are unblocked. The second argument specifies the number of seconds that the signal handler should sleep before returning. Specifying a nonzero value (the default is 1 second) is useful for slowing down the program so that we can more easily see what is happening when multiple signals are handled.

We can use the program in [Listing 22-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex3), along with the program in [Listing 22-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex2) (t_sigqueue.c) to explore the behavior of realtime signals, as shown in the following shell session log:

$ ./catch_rtsigs 60 &  
[1] 12842  
$ ./catch_rtsigs: PID is 12842        Shell prompt mixed with program output  
./catch_rtsigs: signals blocked - sleeping 60 seconds  
Press Enter to see next shell prompt  
$ ./t_sigqueue 12842 54 100 3         Send signal three times  
./t_sigqueue: PID is 12843, UID is 1000  
$ ./t_sigqueue 12842 43 200  
./t_sigqueue: PID is 12844, UID is 1000  
$ ./t_sigqueue 12842 40 300  
./t_sigqueue: PID is 12845, UID is 1000

Eventually, the _catch_rtsigs_ program completes sleeping, and displays messages as the signal handler catches various signals. (We see a shell prompt mixed with the next line of the program’s output because the _catch_rtsigs_ program is writing output from the background.) We first observe that realtime signals are delivered lowest-numbered signal first, and that the _siginfo_t_ structure passed to the handler includes the process ID and user ID of the process that sent the signal:

$ ./catch_rtsigs: sleep complete  
caught signal 40  
    si_signo=40, si_code=-1 (SI_QUEUE), si_value=300  
    si_pid=12845, si_uid=1000  
caught signal 43  
    si_signo=43, si_code=-1 (SI_QUEUE), si_value=200  
    si_pid=12844, si_uid=1000

The remaining output is produced by the three instances of the same realtime signal. Looking at the _si_value_ values, we can see that these signals were delivered in the order they were sent:

caught signal 54  
    si_signo=54, si_code=-1 (SI_QUEUE), si_value=100  
    si_pid=12843, si_uid=1000  
caught signal 54  
    si_signo=54, si_code=-1 (SI_QUEUE), si_value=101  
    si_pid=12843, si_uid=1000  
caught signal 54  
    si_signo=54, si_code=-1 (SI_QUEUE), si_value=102  
    si_pid=12843, si_uid=1000

We continue by using the shell _kill_ command to send a signal to the _catch_rtsigs_ program. As before, we see that the _siginfo_t_ structure received by the handler includes the process ID and user ID of the sending process, but in this case, the _si_code_ value is SI_USER:

Press Enter to see next shell prompt  
$ echo $$                             Display PID of shell  
12780  
$ kill -40 12842                      Uses kill(2) to send a signal  
$ caught signal 40  
    si_signo=40, si_code=0 (SI_USER), si_value=0  
    si_pid=12780, si_uid=1000         PID is that of the shell  
Press Enter to see next shell prompt  
$ kill 12842                          Kill catch_rtsigs by sending SIGTERM  
Caught 6 signals  
Press Enter to see notification from shell about terminated background job  
[1]+  Done             ./catch_rtsigs 60

**Listing 22-3:** Handling realtime signals

____________________________________________________ signals/catch_rtsigs.c  
  
#define _GNU_SOURCE  
#include <string.h>  
#include <signal.h>  
#include "tlpi_hdr.h"  
  
static volatile int handlerSleepTime;  
static volatile int sigCnt = 0;         /* Number of signals received */  
static volatile sig_atomic_t allDone = 0;  
  
static void             /* Handler for signals established using SA_SIGINFO */  
siginfoHandler(int sig, siginfo_t *si, void *ucontext)  
{  
    /* UNSAFE: This handler uses non-async-signal-safe functions  
       (printf()); see [Section 21.1.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev2sec02)) */  
  
    /* SIGINT or SIGTERM can be used to terminate program */  
  
    if (sig == SIGINT || sig == SIGTERM) {  
        allDone = 1;  
        return;  
    }  
  
    sigCnt++;  
    printf("caught signal %d\n", sig);  
  
    printf("    si_signo=%d, si_code=%d (%s), ", si->si_signo, si->si_code,  
            (si->si_code == SI_USER) ? "SI_USER" :  
            (si->si_code == SI_QUEUE) ? "SI_QUEUE" : "other");  
    printf("si_value=%d\n", si->si_value.sival_int);  
    printf("    si_pid=%ld, si_uid=%ld\n", (long) si->si_pid, (long) si->si_uid);  
  
    sleep(handlerSleepTime);  
}  
  
int  
main(int argc, char *argv[])  
{  
    struct sigaction sa;  
    int sig;  
    sigset_t prevMask, blockMask;  
  
    if (argc > 1 && strcmp(argv[1], "--help") == 0)  
        usageErr("%s [block-time [handler-sleep-time]]\n", argv[0]);  
  
    printf("%s: PID is %ld\n", argv[0], (long) getpid());  
  
    handlerSleepTime = (argc > 2) ?  
                getInt(argv[2], GN_NONNEG, "handler-sleep-time") : 1;  
  
    /* Establish handler for most signals. During execution of the handler,  
       mask all other signals to prevent handlers recursively interrupting  
       each other (which would make the output hard to read). */  
  
    sa.sa_sigaction = siginfoHandler;  
    sa.sa_flags = SA_SIGINFO;  
    sigfillset(&sa.sa_mask);  
  
    for (sig = 1; sig < NSIG; sig++)  
        if (sig != SIGTSTP && sig != SIGQUIT)  
            sigaction(sig, &sa, NULL);  
  
    /* Optionally block signals and sleep, allowing signals to be  
       sent to us before they are unblocked and handled */  
  
    if (argc > 1) {  
        sigfillset(&blockMask);  
        sigdelset(&blockMask, SIGINT);  
        sigdelset(&blockMask, SIGTERM);  
  
        if (sigprocmask(SIG_SETMASK, &blockMask, &prevMask) == -1)  
            errExit("sigprocmask");  
  
        printf("%s: signals blocked - sleeping %s seconds\n", argv[0], argv[1]);  
        sleep(getInt(argv[1], GN_GT_0, "block-time"));  
        printf("%s: sleep complete\n", argv[0]);  
  
        if (sigprocmask(SIG_SETMASK, &prevMask, NULL) == -1)  
            errExit("sigprocmask");  
    }  
  
    while (!allDone)                      /* Wait for incoming signals */  
        pause();  
  
    printf("Caught %d signals\n", sigCnt);  
    exit(EXIT_SUCCESS);  
}  
____________________________________________________ signals/catch_rtsigs.c

### **22.9 Waiting for a Signal Using a Mask: _sigsuspend()_**

Before we explain what _sigsuspend()_ does, we first describe a situation where we need to use it. Consider the following scenario that is sometimes encountered when programming with signals:

1. We temporarily block a signal so that the handler for the signal doesn’t interrupt the execution of some critical section of code.
    
2. We unblock the signal, and then suspend execution until the signal is delivered.
    

In order to do this, we might try using code such as that shown in [Listing 22-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex4).

**Listing 22-4:** Incorrectly unblocking and waiting for a signal

______________________________________________________________________  
  
    sigset_t prevMask, intMask;  
    struct sigaction sa;  
  
    sigemptyset(&intMask);  
    sigaddset(&intMask, SIGINT);  
  
    sigemptyset(&sa.sa_mask);  
    sa.sa_flags = 0;  
    sa.sa_handler = handler;  
  
    if (sigaction(SIGINT, &sa, NULL) == -1)  
        errExit("sigaction");  
  
    /* Block SIGINT prior to executing critical section. (At this  
       point we assume that SIGINT is not already blocked.) */  
  
    if (sigprocmask(SIG_BLOCK, &intMask, &prevMask) == -1)  
        errExit("sigprocmask - SIG_BLOCK");  
  
    /* Critical section: do some work here that must not be  
       interrupted by the SIGINT handler */  
  
    /* End of critical section - restore old mask to unblock SIGINT */  
  
    if (sigprocmask(SIG_SETMASK, &prevMask, NULL) == -1)  
        errExit("sigprocmask - SIG_SETMASK");  
  
    /* BUG: what if SIGINT arrives now... */  
  
    pause();                            /* Wait for SIGINT */  
______________________________________________________________________

There is a problem with the code in [Listing 22-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex4). Suppose that the SIGINT signal is delivered after execution of the second _sigprocmask()_, but before the _pause()_ call. (The signal might actually have been generated at any time during the execution of the critical section, and then be delivered only when it is unblocked.) Delivery of the SIGINT signal will cause the handler to be invoked, and after the handler returns and the main program resumes, the _pause()_ call will block until a _second_ instance of SIGINT is delivered. This defeats the purpose of the code, which was to unblock SIGINT and then wait for its _first_ occurrence.

Even if the likelihood of SIGINT being generated between the start of the critical section (i.e., the first _sigprocmask()_ call) and the _pause()_ call is small, this nevertheless constitutes a bug in the above code. This time-dependent bug is an example of a race condition ([Section 5.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch05.xhtml#ch05lev1sec01)). Normally, race conditions occur where two processes or threads share common resources. However, in this case, the main program is racing against its own signal handler.

To avoid this problem, we require a means of _atomically_ unblocking a signal and suspending the process. That is the purpose of the _sigsuspend()_ system call.

#include <signal.h>  
  
int sigsuspend(const sigset_t *mask);

(Normally) returns –1 with _errno_ set to EINTR

The _sigsuspend()_ system call replaces the process signal mask by the signal set pointed to by _mask_, and then suspends execution of the process until a signal is caught and its handler returns. Once the handler returns, _sigsuspend()_ restores the process signal mask to the value it had prior to the call.

Calling _sigsuspend()_ is equivalent to atomically performing these operations:

sigprocmask(SIG_SETMASK, &mask, &prevMask);     /* Assign new mask */  
pause();  
sigprocmask(SIG_SETMASK, &prevMask, NULL);      /* Restore old mask */

Although restoring the old signal mask (i.e., the last step in the above sequence) may at first appear inconvenient, it is essential to avoid race conditions in situations where we need to repeatedly wait for signals. In such situations, the signals must remain blocked except during the _sigsuspend()_ calls. If we later need to unblock the signals that were blocked prior to the _sigsuspend()_ call, we can employ a further call to _sigprocmask()_.

When _sigsuspend()_ is interrupted by delivery of a signal, it returns –1, with _errno_ set to EINTR. If _mask_ doesn’t point to a valid address, _sigsuspend()_ fails with the error EFAULT.

##### **Example program**

[Listing 22-5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex5) demonstrates the use of _sigsuspend()_. This program performs the following steps:

• Display the initial value of the process signal mask using the _printSigMask()_ function ([Listing 20-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20ex4), on [page 408](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#page_408)) ①.

• Block SIGINT and SIGQUIT, and save the original process signal mask ②.

• Establish the same handler for both SIGINT and SIGQUIT ③. This handler displays a message, and, if it was invoked via delivery of SIGQUIT, sets the global variable _gotSigquit_.

• Loop until _gotSigquit_ is set ④. Each loop iteration performs the following steps:

– Display the current value of the signal mask using our _printSigMask()_ function.

– Simulate a critical section by executing a CPU busy loop for a few seconds.

– Display the mask of pending signals using our _printPendingSigs()_ function ([Listing 20-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20ex4)).

– Use _sigsuspend()_ to unblock SIGINT and SIGQUIT and wait for a signal (if one is not already pending).

• Use _sigprocmask()_ to restore the process signal mask to its original state ⑤, and then display the signal mask using _printSigMask()_ ⑥.

**Listing 22-5:** Using _sigsuspend()_

____________________________________________________ signals/t_sigsuspend.c  
  
   #define _GNU_SOURCE     /* Get strsignal() declaration from <string.h> */  
   #include <string.h>  
   #include <signal.h>  
   #include <time.h>  
   #include "signal_functions.h"           /* Declarations of printSigMask()  
                                              and printPendingSigs() */  
   #include "tlpi_hdr.h"  
  
   static volatile sig_atomic_t gotSigquit = 0;  
  
   static void  
   handler(int sig)  
   {  
       printf("Caught signal %d (%s)\n", sig, strsignal(sig));  
                                           /* UNSAFE (see [Section 21.1.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev2sec02)) */  
       if (sig == SIGQUIT)  
           gotSigquit = 1;  
   }  
  
   int  
   main(int argc, char *argv[])  
   {  
       int loopNum;  
       time_t startTime;  
       sigset_t origMask, blockMask;  
       struct sigaction sa;  
  
①     printSigMask(stdout, "Initial signal mask is:\n");  
  
       sigemptyset(&blockMask);  
       sigaddset(&blockMask, SIGINT);  
       sigaddset(&blockMask, SIGQUIT);  
②     if (sigprocmask(SIG_BLOCK, &blockMask, &origMask) == -1)  
           errExit("sigprocmask - SIG_BLOCK");  
  
       sigemptyset(&sa.sa_mask);  
       sa.sa_flags = 0;  
       sa.sa_handler = handler;  
③     if (sigaction(SIGINT, &sa, NULL) == -1)  
           errExit("sigaction");  
       if (sigaction(SIGQUIT, &sa, NULL) == -1)  
           errExit("sigaction");  
  
④     for (loopNum = 1; !gotSigquit; loopNum++) {  
           printf("=== LOOP %d\n", loopNum);  
  
           /* Simulate a critical section by delaying a few seconds */  
  
           printSigMask(stdout, "Starting critical section, signal mask is:\n");  
           for (startTime = time(NULL); time(NULL) < startTime + 4; )  
               continue;                   /* Run for a few seconds elapsed time */  
  
           printPendingSigs(stdout,  
                   "Before sigsuspend() - pending signals:\n");  
           if (sigsuspend(&origMask) == -1 && errno != EINTR)  
               errExit("sigsuspend");  
       }  
  
⑤     if (sigprocmask(SIG_SETMASK, &origMask, NULL) == -1)  
           errExit("sigprocmask - SIG_SETMASK");  
  
⑥     printSigMask(stdout, "=== Exited loop\nRestored signal mask to:\n");  
  
       /* Do other processing... */  
  
       exit(EXIT_SUCCESS);  
   }  
____________________________________________________ signals/t_sigsuspend.c

The following shell session log shows an example of what we see when running the program in [Listing 22-5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex5):

$ ./t_sigsuspend  
Initial signal mask is:  
                <empty signal set>  
=== LOOP 1  
Starting critical section, signal mask is:  
                2 (Interrupt)  
                3 (Quit)  
Type Control-C; SIGINT is generated, but remains pending because it is blocked  
Before sigsuspend() - pending signals:  
                2 (Interrupt)  
Caught signal 2 (Interrupt)         sigsuspend() is called, signals are unblocked

The last line of output appeared when the program called _sigsuspend()_, which caused SIGINT to be unblocked. At that point, the signal handler was called and displayed that line of output.

The main program continues its loop:

=== LOOP 2  
Starting critical section, signal mask is:  
                2 (Interrupt)  
                3 (Quit)  
Type Control-\ to generate SIGQUIT  
Before sigsuspend() - pending signals:  
                3 (Quit)  
Caught signal 3 (Quit)              sigsuspend() is called, signals are unblocked  
=== Exited loop                     Signal handler set gotSigquit  
Restored signal mask to:  
                <empty signal set>

This time, we typed _Control-\_, which caused the signal handler to set the _gotSigquit_ flag, which in turn caused the main program to terminate its loop.

### **22.10 Synchronously Waiting for a Signal**

In [Section 22.9](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec09), we saw how to use a signal handler plus _sigsuspend()_ to suspend execution of a process until a signal is delivered. However, the need to write a signal handler and to handle the complexities of asynchronous delivery makes this approach cumbersome for some applications. Instead, we can use the _sigwaitinfo()_ system call to synchronously _accept_ a signal.

#define _POSIX_C_SOURCE 199309  
#include <signal.h>  
  
int sigwaitinfo(const sigset_t *set, siginfo_t *info);

Returns signal number on success, or –1 on error

The _sigwaitinfo()_ system call suspends execution of the process until one of the signals in the signal set pointed to by _set_ becomes pending. If one of the signals in _set_ is already pending at the time of the call, _sigwaitinfo()_ returns immediately. One of the signals is removed from the process’s list of pending signals, and the signal number is returned as the function result. If the _info_ argument is not NULL, then it points to a _siginfo_t_ structure that is initialized to contain the same information provided to a signal handler taking a _siginfo_t_ argument ([Section 21.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev1sec04)).

The delivery order and queuing characteristics of signals accepted by _sigwaitinfo()_ are the same as for signals caught by a signal handler; that is, standard signals are not queued, and realtime signals are queued and delivered lowest signal number first.

As well as saving us the extra baggage of writing a signal handler, waiting for signals using _sigwaitinfo()_ is somewhat faster than the combination of a signal handler plus _sigsuspend()_ (see [Exercise 22-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22exe3)).

It usually makes sense to use _sigwaitinfo()_ only in conjunction with blocking the set of signals for which we were interested in waiting. (We can fetch a pending signal with _sigwaitinfo()_ even while that signal is blocked.) If we fail to do this and a signal arrives before the first, or between successive calls to _sigwaitinfo()_, then the signal will be handled according to its current disposition.

According to SUSv3, calling _sigwaitinfo()_ without blocking the signals in _set_ results in undefined behavior.

An example of the use of _sigwaitinfo()_ is shown in [Listing 22-6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex6). This program first blocks all signals, then delays for the number of seconds specified in its optional command-line argument. This allows signals to be sent to the program before _sigwaitinfo()_. The program then loops continuously using _sigwaitinfo()_ to accept incoming signals, until SIGINT or SIGTERM is received.

The following shell session log demonstrates the use of the program in [Listing 22-6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex6). We run the program in the background, specifying that it should delay 60 seconds before calling _sigwaitinfo()_, and then send it two signals:

$ ./t_sigwaitinfo 60 &  
./t_sigwaitinfo: PID is 3837  
./t_sigwaitinfo: signals blocked  
./t_sigwaitinfo: about to delay 60 seconds  
[1] 3837  
$ ./t_sigqueue 3837 43 100                 Send signal 43  
./t_sigqueue: PID is 3839, UID is 1000  
$ ./t_sigqueue 3837 42 200                 Send signal 42  
./t_sigqueue: PID is 3840, UID is 1000

Eventually, the program completes its sleep interval, and the _sigwaitinfo()_ loop accepts the queued signals. (We see a shell prompt mixed with the next line of the program’s output because the _t_sigwaitinfo_ program is writing output from the background.) As with realtime signals caught with a handler, we see that signals are delivered lowest number first, and that the _siginfo_t_ structure passed to the signal handler allows us to obtain the process ID and user ID of the sending process:

$ ./t_sigwaitinfo: finished delay  
got signal: 42  
    si_signo=42, si_code=-1 (SI_QUEUE), si_value=200  
    si_pid=3840, si_uid=1000  
got signal: 43  
    si_signo=43, si_code=-1 (SI_QUEUE), si_value=100  
    si_pid=3839, si_uid=1000

We continue, using the shell _kill_ command to send a signal to the process. This time, we see that the _si_code_ field is set to SI_USER (instead of SI_QUEUE):

Press Enter to see next shell prompt  
$ echo $$                                   Display PID of shell  
3744  
$ kill -USR1 3837                           Shell sends SIGUSR1 using kill()  
$ got signal: 10                            Delivery of SIGUSR1  
    si_signo=10, si_code=0 (SI_USER), si_value=100  
    si_pid=3744, si_uid=1000                3744 is PID of shell  
Press Enter to see next shell prompt  
$ kill %1                                   Terminate program with SIGTERM  
$  
Press Enter to see notification of background job termination  
[1]+  Done             ./t_sigwaitinfo 60

In the output for the accepted SIGUSR1 signal, we see that the _si_value_ field has the value 100. This is the value to which the field was initialized by the preceding signal that was sent using _sigqueue()_. We noted earlier that the _si_value_ field contains valid information only for signals sent using _sigqueue()_.

**Listing 22-6:** Synchronously waiting for a signal with _sigwaitinfo()_

___________________________________________________ signals/t_sigwaitinfo.c  
  
#define _GNU_SOURCE  
#include <string.h>  
#include <signal.h>  
#include <time.h>  
#include "tlpi_hdr.h"  
  
int  
main(int argc, char *argv[])  
{  
    int sig;  
    siginfo_t si;  
    sigset_t allSigs;  
  
    if (argc > 1 && strcmp(argv[1], "--help") == 0)  
        usageErr("%s [delay-secs]\n", argv[0]);  
  
    printf("%s: PID is %ld\n", argv[0], (long) getpid());  
  
    /* Block all signals (except SIGKILL and SIGSTOP) */  
  
    sigfillset(&allSigs);  
    if (sigprocmask(SIG_SETMASK, &allSigs, NULL) == -1)  
        errExit("sigprocmask");  
    printf("%s: signals blocked\n", argv[0]);  
  
    if (argc > 1) {             /* Delay so that signals can be sent to us */  
        printf("%s: about to delay %s seconds\n", argv[0], argv[1]);  
        sleep(getInt(argv[1], GN_GT_0, "delay-secs"));  
        printf("%s: finished delay\n", argv[0]);  
    }  
  
    for (;;) {                  /* Fetch signals until SIGINT (^C) or SIGTERM */  
        sig = sigwaitinfo(&allSigs, &si);  
        if (sig == -1)  
            errExit("sigwaitinfo");  
  
        if (sig == SIGINT || sig == SIGTERM)  
            exit(EXIT_SUCCESS);  
  
        printf("got signal: %d (%s)\n", sig, strsignal(sig));  
        printf("    si_signo=%d, si_code=%d (%s), si_value=%d\n",  
                si.si_signo, si.si_code,  
                (si.si_code == SI_USER) ? "SI_USER" :  
                    (si.si_code == SI_QUEUE) ? "SI_QUEUE" : "other",  
                si.si_value.sival_int);  
        printf("    si_pid=%ld, si_uid=%ld\n",  
                (long) si.si_pid, (long) si.si_uid);  
    }  
}  
___________________________________________________ signals/t_sigwaitinfo.c

The _sigtimedwait()_ system call is a variation on _sigwaitinfo()_. The only difference is that _sigtimedwait()_ allows us to specify a time limit for waiting.

#define _POSIX_C_SOURCE 199309  
#include <signal.h>  
  
int sigtimedwait(const sigset_t *set, siginfo_t *info,  
                 const struct timespec *timeout);

Returns signal number on success, or –1 on error or timeout (EAGAIN)

The _timeout_ argument specifies the maximum time that _sigtimedwait()_ should wait for a signal. It is a pointer to a structure of the following type:

struct timespec {  
    time_t tv_sec;      /* Seconds ('time_t' is an integer type) */  
    long   tv_nsec;     /* Nanoseconds */  
};

The fields of the _timespec_ structure are filled in to specify the maximum number of seconds and nanoseconds that _sigtimedwait()_ should wait. Specifying both fields of the structure as 0 causes an immediate timeout—that is, a poll to check if any of the specified set of signals is pending. If the call times out without a signal being delivered, _sigtimedwait()_ fails with the error EAGAIN.

If the _timeout_ argument is specified as NULL, then _sigtimedwait()_ is exactly equivalent to _sigwaitinfo()_. SUSv3 leaves the meaning of a NULL _timeout_ unspecified, and some UNIX implementations instead interpret this as a poll request that returns immediately.

### **22.11 Fetching Signals via a File Descriptor**

Starting with kernel 2.6.22, Linux provides the (nonstandard) _signalfd()_ system call, which creates a special file descriptor from which signals directed to the caller can be read. The _signalfd_ mechanism provides an alternative to the use of _sigwaitinfo()_ for synchronously accepting signals.

#include <sys/signalfd.h>  
  
int signalfd(int fd, const sigset_t *mask, int flags);

Returns file descriptor on success, or –1 on error

The _mask_ argument is a signal set that specifies the signals that we want to be able to read via the _signalfd_ file descriptor. As with _sigwaitinfo()_, we should normally also block all of the signals in _mask_ using _sigprocmask()_, so that the signals don’t get handled according to their default dispositions before we have a chance to read them.

If _fd_ is specified as –1, then _signalfd()_ creates a new file descriptor that can be used to read the signals in _mask_; otherwise, it modifies the mask associated with _fd_, which must be a file descriptor created by a previous call to _signalfd()_.

In the initial implementation, the _flags_ argument was reserved for future use and had to be specified as 0. However, since Linux 2.6.27, two flags are supported:

SFD_CLOEXEC

Set the close-on-exec flag (FD_CLOEXEC) for the new file descriptor. This flag is useful for the same reasons as the _open()_ O_CLOEXEC flag described in [Section 4.3.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch04.xhtml#ch04lev2sec01).

SFD_NONBLOCK

Set the O_NONBLOCK flag on the underlying open file description, so that future reads will be nonblocking. This saves additional calls to _fcntl()_ to achieve the same result.

Having created the file descriptor, we can then read signals from it using _read()_. The buffer given to _read()_ must be large enough to hold at least one _signalfd_siginfo_ structure, defined as follows in <sys/signalfd.h>:

struct signalfd_siginfo {  
    uint32_t  ssi_signo;    /* Signal number */  
    int32_t   ssi_errno;    /* Error number (generally unused) */  
    int32_t   ssi_code;     /* Signal code */  
    uint32_t  ssi_pid;      /* Process ID of sending process */  
    uint32_t  ssi_uid;      /* Real user ID of sender */  
    int32_t   ssi_fd;       /* File descriptor (SIGPOLL/SIGIO) */  
    uint32_t  ssi_tid;      /* (Kernel-internal) timer ID (POSIX timers) */  
    uint32_t  ssi_band;     /* Band event (SIGPOLL/SIGIO) */  
    uint32_t  ssi_overrun;  /* Overrun count (POSIX timers) */  
    uint32_t  ssi_trapno;   /* Trap number */  
    int32_t   ssi_status;   /* Exit status or signal (SIGCHLD) */  
    int32_t   ssi_int;      /* Integer sent by sigqueue() */  
    uint64_t  ssi_ptr;      /* Pointer sent by sigqueue() */  
    uint64_t  ssi_utime;    /* User CPU time (SIGCHLD) */  
    uint64_t  ssi_stime;    /* System CPU time (SIGCHLD) */  
    uint64_t  ssi_addr;     /* Address that generated signal  
                               (hardware-generated signals only) */  
};

The fields in this structure return the same information as the similarly named fields in the traditional _siginfo_t_ structure ([Section 21.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch21.xhtml#ch21lev1sec04)).

Each call to _read()_ returns as many _signalfd_siginfo_ structures as there are signals pending and will fit in the supplied buffer. If no signals are pending at the time of the call, then _read()_ blocks until a signal arrives. We can also use the _fcntl()_ F_SETFL operation ([Section 5.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch05.xhtml#ch05lev1sec03)) to set the O_NONBLOCK flag for the file descriptor, so that reads are nonblocking and will fail with the error EAGAIN if no signals are pending.

When a signal is read from a _signalfd_ file descriptor, it is consumed and ceases to be pending for the process.

**Listing 22-7:** Using _signalfd()_ to read signals

__________________________________________________signals/signalfd_sigval.c  
  
#include <sys/signalfd.h>  
#include <signal.h>  
#include "tlpi_hdr.h"  
  
int  
main(int argc, char *argv[])  
{  
    sigset_t mask;  
    int sfd, j;  
    struct signalfd_siginfo fdsi;  
    ssize_t s;  
  
    if (argc < 2 || strcmp(argv[1], "--help") == 0)  
        usageErr("%s sig-num...\n", argv[0]);  
  
    printf("%s: PID = %ld\n", argv[0], (long) getpid());  
  
    sigemptyset(&mask);  
    for (j = 1; j < argc; j++)  
        sigaddset(&mask, atoi(argv[j]));  
  
    if (sigprocmask(SIG_BLOCK, &mask, NULL) == -1)  
        errExit("sigprocmask");  
  
    sfd = signalfd(-1, &mask, 0);  
    if (sfd == -1)  
        errExit("signalfd");  
  
    for (;;) {  
        s = read(sfd, &fdsi, sizeof(struct signalfd_siginfo));  
        if (s != sizeof(struct signalfd_siginfo))  
            errExit("read");  
  
        printf("%s: got signal %d", argv[0], fdsi.ssi_signo);  
        if (fdsi.ssi_code == SI_QUEUE) {  
            printf("; ssi_pid = %d; ", fdsi.ssi_pid);  
            printf("ssi_int = %d", fdsi.ssi_int);  
        }  
        printf("\n");  
    }  
}  
__________________________________________________signals/signalfd_sigval.c

A _signalfd_ file descriptor can be monitored along with other descriptors using _select()_, _poll()_, and _epoll_ (described in [Chapter 63](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch63.xhtml#ch63)). Among other uses, this feature provides an alternative to the self-pipe trick described in [Section 63.5.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch63.xhtml#ch63lev2sec17). If signals are pending, then these techniques indicate the file descriptor as being readable.

When we no longer require a _signalfd_ file descriptor, we should close it, in order to release the associated kernel resources.

[Listing 22-7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex7) (on [page 473](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#page_473)) demonstrates the use of _signalfd()_. This program creates a mask of the signal numbers specified in its command-line arguments, blocks those signals, and then creates a _signalfd_ file descriptor to read those signals. It then loops, reading signals from the file descriptor and displaying some of the information from the returned _signalfd_siginfo_ structure. In the following shell session, we run the program in [Listing 22-7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex7) in the background and send it a realtime signal with accompanying data using the program in [Listing 22-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22ex2) (t_sigqueue.c):

$ ./signalfd_sigval 44 &  
./signalfd_sigval: PID = 6267  
[1] 6267  
$ ./t_sigqueue 6267 44 123          Send signal 44 with data 123 to PID 6267  
./t_sigqueue: PID is 6269, UID is 1000  
./signalfd_sigval: got signal 44; ssi_pid=6269; ssi_int=123  
$ kill %1                           Kill program running in background

### **22.12 Interprocess Communication with Signals**

From one viewpoint, we can consider signals as a form of interprocess communication (IPC). However, signals suffer a number of limitations as an IPC mechanism. First, by comparison with other methods of IPC that we examine in later chapters, programming with signals is cumbersome and difficult. The reasons for this are as follows:

• The asynchronous nature of signals means that we face various problems, including reentrancy requirements, race conditions, and the correct handling of global variables from signal handlers. (Most of these problems do not occur if we are using _sigwaitinfo()_ or _signalfd()_ to synchronously fetch signals.)

• Standard signals are not queued. Even for realtime signals, there are upper limits on the number of signals that may be queued. This means that in order to avoid loss of information, the process receiving the signals must have a method of informing the sender that it is ready to receive another signal. The most obvious method of doing this is for the receiver to send a signal to the sender.

A further problem is that signals carry only a limited amount of information: the signal number, and in the case of realtime signals, a word (an integer or a pointer) of additional data. This low bandwidth makes signals slow by comparison with other methods of IPC such as pipes.

As a consequence of the above limitations, signals are rarely used for IPC.

### **22.13 Earlier Signal APIs (System V and BSD)**

Our discussion of signals has focused on the POSIX signal API. We now briefly look at the historical APIs provided by System V and BSD. Although all new applications should use the POSIX API, we may encounter these obsolete APIs when porting (usually older) applications from other UNIX implementations. Because Linux (like many other UNIX implementations) provides System V and BSD compatibility APIs, often all that is required to port programs using these older APIs is to recompile them on Linux.

##### **The System V signal API**

As noted earlier, one important difference in the System V signal API is that when a handler is established with _signal()_, we get the older, unreliable signal semantics. This means that the signal is not added to the process signal mask, the disposition of the signal is reset to the default when the handler is called, and system calls are not automatically restarted.

Below, we briefly describe the functions in the System V signal API. The manual pages provide full details. SUSv3 specifies all of these functions, but notes that the modern POSIX equivalents are preferred. SUSv4 marks these functions obsolete.

#define _XOPEN_SOURCE 500  
#include <signal.h>  
  
void (*sigset(int sig, void (*handler)(int)))(int);

On success: returns the previous disposition of _sig_, or SIG_HOLD if _sig_ was previously blocked; on error –1 is returned

To establish a signal handler with reliable semantics, System V provided the _sigset()_ call (with a prototype similar to that of _signal()_). As with _signal()_, the _handler_ argument for _sigset()_ can be specified as SIG_IGN, SIG_DFL, or the address of a signal handler. Alternatively, it can be specified as SIG_HOLD, to add the signal to the process signal mask while leaving the disposition of the signal unchanged.

If _handler_ is specified as anything other than SIG_HOLD, _sig_ is removed from the process signal mask (i.e., if _sig_ was blocked, it is unblocked).

#define _XOPEN_SOURCE 500  
#include <signal.h>  
  
int sighold(int sig);  
int sigrelse(int sig);  
int sigignore(int sig);

All return 0 on success, or –1 on error

int sigpause(int sig);

Always returns –1 with _errno_ set to EINTR

The _sighold()_ function adds a signal to the process signal mask. The _sigrelse()_ function removes a signal from the signal mask. The _sigignore()_ function sets a signal’s disposition to _ignore_. The _sigpause()_ function is similar to _sigsuspend()_, but removes just one signal from the process signal mask before suspending the process until the arrival of a signal.

##### **The BSD signal API**

The POSIX signal API drew heavily on the 4.2BSD API, so the BSD functions are mainly direct analogs of those in POSIX.

As with the functions in the System V signal API described above, we present the prototypes of the functions in the BSD signal API, and briefly explain the operation of each function. Once again, the manual pages provide full details.

#define _BSD_SOURCE  
#include <signal.h>  
  
int sigvec(int sig, const struct sigvec *vec, struct sigvec *ovec);

Returns 0 on success, or –1 on error

The _sigvec()_ function is analogous to _sigaction()_. The _vec_ and _ovec_ arguments are pointers to structures of the following type:

struct sigvec {  
    void (*sv_handler)(int);  
    int  sv_mask;  
    int  sv_flags;  
};

The fields of the _sigvec_ structure correspond closely with those of the _sigaction_ structure. The first notable difference is that the _sv_mask_ field (the analog of _sa_mask_) was an integer rather than a _sigset_t_, which meant that, on 32-bit architectures, there was a maximum of 31 different signals. The other difference is the use of the SV_INTERRUPT flag in the _sv_flags_ field (the analog of _sa_flags_). Since system call restarting was the default on 4.2BSD, this flag was used to specify that slow system calls should be interrupted by signal handlers. (This contrasts with the POSIX API, where we must explicitly specify SA_RESTART in order to enable restarting of system calls when establishing a signal handler with _sigaction()_.)

#define _BSD_SOURCE  
#include <signal.h>  
  
int sigblock(int mask);  
int sigsetmask(int mask);

Both return previous signal mask

int sigpause(int sigmask);

Always returns –1 with _errno_ set to EINTR

int sigmask(int sig);

Returns signal mask value with bit _sig_ set

The _sigblock()_ function adds a set of signals to the process signal mask. It is analogous to the _sigprocmask()_ SIG_BLOCK operation. The _sigsetmask()_ call specifies an absolute value for the signal mask. It is analogous to the _sigprocmask()_ SIG_SETMASK operation.

The _sigpause()_ function is analogous to _sigsuspend()_. Note that although a function with this name exists in both the System V and BSD APIs, the _int_ argument is interpreted differently in the two APIs. The GNU C library provides the System V version by default, unless we specify the _BSD_SOURCE feature test macro when compiling a program.

The _sigmask()_ macro turns a signal number into the corresponding 32-bit mask value. Such bit masks can then be ORed together to create a set of signals, as in the following:

sigblock(sigmask(SIGINT) | sigmask(SIGQUIT));

### **22.14 Summary**

Certain signals cause a process to create a core dump and terminate. Core dumps contain information that can be used by a debugger to inspect the state of a process at the time that it terminated. By default, a core dump file is named core, but Linux provides the /proc/sys/kernel/core_pattern file to control the naming of core dump files.

A signal may be generated asynchronously or synchronously. Asynchronous generation occurs when a signal is sent to a process by the kernel or by another process. A process can’t predict precisely when an asynchronously generated signal will be delivered. (We noted that asynchronous signals are normally delivered the next time the receiving process switches from kernel mode to user mode.) Synchronous generation occurs when the process itself executes code that directly generates the signal—for example, by executing an instruction that causes a hardware exception or by calling _raise()_. The delivery of a synchronously generated signal is precisely predictable (it occurs immediately).

Realtime signals are a POSIX addition to the original signal model, and differ from standard signals in that they are queued, have a specified delivery order, and can be sent with an accompanying piece of data. Realtime signals are designed to be used for application-defined purposes. A realtime signal is sent using the _sigqueue()_ system call, and an additional argument (the _siginfo_t_ structure) is supplied to the signal handler so that it can obtain the data accompanying the signal, as well as the process ID and real user ID of the sending process.

The _sigsuspend()_ system call allows a program to atomically modify the process signal mask and suspend execution until a signal arrives. The atomicity of _sigsuspend()_ is essential to avoid race conditions when unblocking a signal and then suspending execution until that signal arrives.

We can use _sigwaitinfo()_ and _sigtimedwait()_ to synchronously wait for a signal. This saves us the work of designing and writing a signal handler, which may be unnecessary if our only aim is to wait for the delivery of a signal.

Like _sigwaitinfo()_ and _sigtimedwait()_, the Linux-specific _signalfd()_ system call can be used to synchronously wait for a signal. The distinctive feature of this interface is that signals can be read via a file descriptor. This file descriptor can also be monitored using _select()_, _poll()_, and _epoll_.

Although signals can be viewed as a method of IPC, many factors make them generally unsuitable for this purpose, including their asynchronous nature, the fact that they are not queued, and their low bandwidth. More usually, signals are used as a method of process synchronization and for a variety of other purposes (e.g., event notification, job control, and timer expiration).

Although the fundamental signal concepts are straightforward, our discussion has stretched over three chapters, since there were many details to cover. Signals play an important role in various parts of the system call API, and we’ll revisit their use in several later chapters. In addition, various signal-related functions are specific to threads (e.g., _pthread_kill()_ and _pthread_sigmask()_), and we defer discussion of these functions until [Section 33.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch33.xhtml#ch33lev1sec02).

##### **Further information**

See the sources listed in [Section 20.15](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch20.xhtml#ch20lev1sec15).

### **22.15 Exercises**

**22-1.**   [Section 22.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec02) noted that if a stopped process that has established a handler for and blocked SIGCONT is later resumed as a consequence of receiving a SIGCONT, then the handler is invoked only when SIGCONT is unblocked. Write a program to verify this. Recall that a process can be stopped by typing the terminal _suspend_ character (usually _Control-Z_) and can be sent a SIGCONT signal using the command _kill –CONT_ (or implicitly, using the shell _fg_ command).

**22-2.**   If both a realtime and a standard signal are pending for a process, SUSv3 leaves it unspecified which is delivered first. Write a program that shows what Linux does in this case. (Have the program set up a handler for all signals, block signals for a period of time so that you can send various signals to it, and then unblock all signals.)

**22-3.**   [Section 22.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec10) stated that accepting signals using _sigwaitinfo()_ is faster than the use of a signal handler plus _sigsuspend()_. The program signals/sig_speed_sigsuspend.c, supplied in the source code distribution for this book, uses _sigsuspend()_ to alternately send signals back and forward between a parent and a child process. Time the operation of this program to exchange one million signals between the two processes. (The number of signals to exchange is provided as a command-line argument to the program.) Create a modified version of the program that instead uses _sigwaitinfo()_, and time that version. What is the speed difference between the two programs?

**22-4.**   Implement the System V functions _sigset()_, _sighold()_, _sigrelse()_, _sigignore()_, and _sigpause()_ using the POSIX signal API.