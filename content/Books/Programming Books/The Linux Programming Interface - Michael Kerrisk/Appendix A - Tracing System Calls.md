---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Appendix A - Tracing System Calls
modified: 2024-11-11T21:36:15-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **A**  
**TRACING SYSTEM CALLS**

The _strace_ command allows us to trace the system calls made by a program. This is useful for debugging, or simply to find out what a program is doing. In its simplest form, we use _strace_ as follows:

$ strace command arg...

This runs _command_, with the given command-line arguments, producing a trace of the system calls it makes. By default, _strace_ writes its output to _stderr_, but we can change this using the _–o filename_ option.

Examples of the type of output produced by _strace_ include the following (taken from the output of the command _strace date_):

execve("/bin/date", ["date"], [/* 114 vars */]) = 0  
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)  
open("/etc/ld.so.cache", O_RDONLY)      = 3  
fstat64(3, {st_mode=S_IFREG|0644, st_size=111059, ...}) = 0  
mmap2(NULL, 111059, PROT_READ, MAP_PRIVATE, 3, 0) = 0xb7f38000  
close(3)                                = 0  
open("/lib/libc.so.6", O_RDONLY)        = 3  
fstat64(3, {st_mode=S_IFREG|0755, st_size=1491141, ...}) = 0  
close(3)                                = 0  
write(1, "Mon Jan 17 12:14:24 CET 2011\n", 29) = 29  
exit_group(0)                           = ?

Each system call is displayed in the form of a function call, with both input and output arguments shown in parentheses. As can be seen from the above examples, arguments are printed in symbolic form:

• Bit masks are represented using the corresponding symbolic constants.

• Strings are printed in text form (up to a limit of 32 characters, but the _–s strsize_ option can be used to change this limit).

• Structure fields are individually displayed (by default, only an abbreviated subset of large structures is displayed, but the _–v_ option can be used to display the whole structure).

After the closing parenthesis of the traced call, _strace_ prints an equal sign (=), followed by the return value of the system call. If the system call failed, the symbolic _errno_ value is also displayed. Thus, we see ENOENT displayed for the failure of the _access()_ call above.

Even for a simple program, the output produced by _strace_ is made voluminous by the system calls executed by the C run-time startup code and the loading of shared libraries. For a complex program, the _strace_ output can be extremely long. For these reasons, it is sometimes useful to selectively filter the output of _strace_. One way to do this is to use _grep_, like so:

$ strace date 2>&1 | grep open

Another method is to use the _–e_ option to select the events to be traced. For example, we can use the following command to trace _open()_ and _close()_ system calls:

$ strace -e trace=open,close date

When using either of the above techniques, we need to be aware that, in a few cases, the true name of a system call differs from the name of its _glibc_ wrapper. For example, though we refer to all of the _wait()_-type functions as system calls in [Chapter 26](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch26.xhtml#ch26), most of them (_wait()_, _waitpid()_, and _wait3()_) are wrappers that invoke the kernel’s _wait4()_ system call service routine. This latter name is displayed by _strace_, and we must specify that name in the _–e trace=_ option. Similarly, all of the _exec_ library functions ([Section 27.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch27.xhtml#ch27lev1sec02)) invoke the _execve()_ system call. Often, we can make a good guess about such transformations by looking at the _strace_ output (or looking at the output produced by _strace –c_, described below), but, failing that, we may need to check the _glibc_ source code to see what transformations may be occurring inside wrapper functions.

The _strace(1)_ manual page documents a host of further options to _strace_, including the following:

• The _–p pid_ option is used to trace an existing process, by specifying its process ID. Unprivileged users are restricted to tracing only processes that they own and that are not executing set-user-ID or set-group-ID programs ([Section 9.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch09.xhtml#ch09lev1sec03)).

• The _–c_ option causes _strace_ to print a summary of all system calls made by the program. For each system call, the summary information includes the total number of calls, the number of calls that failed, and the total time spent executing the calls.

• The _–f_ option causes children of this process also to be traced. If we are sending trace output to a file (_–o filename_), then the alternative _–ff_ option causes each process to write its trace output to a file named _filename.PID_.

The _strace_ command is Linux-specific, but most UNIX implementations provide their own equivalents (e.g., _truss_ on Solaris and _ktrace_ on the BSDs).

The _ltrace_ command performs an analogous task to _strace_, but for library functions. See the _ltrace(1)_ manual page for details.