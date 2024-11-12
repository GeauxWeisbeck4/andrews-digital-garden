---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 28 - Creation and Execution in More Detail
modified: 2024-11-11T19:26:10-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **28**  
**PROCESS CREATION AND PROGRAM EXECUTION IN MORE DETAIL**

This chapter extends the material presented in [Chapters 24](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch24.xhtml#ch24) to [27](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch27.xhtml#ch27) by covering a variety of topics related to process creation and program execution. We describe process accounting, a kernel feature that writes an accounting record for each process on the system as it terminates. We then look at the Linux-specific _clone()_ system call, which is the low-level API that is used to create threads on Linux. We follow this with some comparisons of the performance of _fork()_, _vfork()_, and _clone()_. We conclude with a summary of the effects of _fork()_ and _exec()_ on the attributes of a process.

### **28.1 Process Accounting**

When process accounting is enabled, the kernel writes an accounting record to the system-wide process accounting file as each process terminates. This accounting record contains various information maintained by the kernel about the process, including its termination status and how much CPU time it consumed. The accounting file can be analyzed by standard tools (_sa(8)_ summarizes information from the accounting file, and _lastcomm(1)_ lists information about previously executed commands) or by tailored applications.

In kernels before 2.6.10, a separate process accounting record was written for each thread created using the NPTL threading implementation. Since kernel 2.6.10, a single accounting record is written for the entire process when the last thread terminates. Under the older LinuxThreads threading implementation, a single process accounting record is always written for each thread.

Historically, the primary use of process accounting was to charge users for consumption of system resources on multiuser UNIX systems. However, process accounting can also be useful for obtaining information about a process that was not otherwise monitored and reported on by its parent.

Although available on most UNIX implementations, process accounting is not specified in SUSv3. The format of the accounting records, as well as the location of the accounting file, vary somewhat across implementations. We describe the details for Linux in this section, noting some variations from other UNIX implementations along the way.

On Linux, process accounting is an optional kernel component that is configured via the option CONFIG_BSD_PROCESS_ACCT.

##### **Enabling and disabling process accounting**

The _acct()_ system call is used by a privileged (CAP_SYS_PACCT) process to enable and disable process accounting. This system call is rarely used in application programs. Normally, process accounting is enabled at each system restart by placing appropriate commands in the system boot scripts.

#define _BSD_SOURCE  
#include <unistd.h>  
  
int acct(const char *acctfile);

Returns 0 on success, or –1 on error

To enable process accounting, we supply the pathname of an _existing_ regular file in _acctfile_. A typical pathname for the accounting file is /var/log/pacct or /usr/account/pacct. To disable process accounting, we specify _acctfile_ as NULL.

The program in [Listing 28-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28ex1) uses _acct()_ to switch process accounting on and off. The functionality of this program is similar to the shell _accton(8)_ command.

**Listing 28-1:** Turning process accounting on and off

_______________________________________________________ procexec/acct_on.c  
  
#define _BSD_SOURCE  
#include <unistd.h>  
#include "tlpi_hdr.h"  
  
int  
main(int argc, char *argv[])  
{  
    if (argc > 2 || (argc > 1 && strcmp(argv[1], "--help") == 0))  
        usageErr("%s [file]\n", argv[0]);  
    if (acct(argv[1]) == -1)  
        errExit("acct");  
  
    printf("Process accounting %s\n",  
            (argv[1] == NULL) ? "disabled" : "enabled");  
    exit(EXIT_SUCCESS);  
}  
_______________________________________________________ procexec/acct_on.c

##### **Process accounting records**

Once process accounting is enabled, an _acct_ record is written to the accounting file as each process terminates. The _acct_ structure is defined in <sys/acct.h> as follows:

typedef u_int16_t comp_t;  /* See text */  
  
struct acct {  
    char      ac_flag;     /* Accounting flags (see text) */  
    u_int16_t ac_uid;      /* User ID of process */  
    u_int16_t ac_gid;      /* Group ID of process */  
    u_int16_t ac_tty;      /* Controlling terminal for process (may be  
                              0 if none, e.g., for a daemon) */  
    u_int32_t ac_btime;    /* Start time (time_t; seconds since the Epoch) */  
    comp_t    ac_utime;    /* User CPU time (clock ticks) */  
    comp_t    ac_stime;    /* System CPU time (clock ticks) */  
    comp_t    ac_etime;    /* Elapsed (real) time (clock ticks) */  
    comp_t    ac_mem;      /* Average memory usage (kilobytes) */  
    comp_t    ac_io;       /* Bytes transferred by read(2) and write(2)  
                              (unused) */  
    comp_t    ac_rw;       /* Blocks read/written (unused) */  
    comp_t    ac_minflt;   /* Minor page faults (Linux-specific) */  
    comp_t    ac_majflt;   /* Major page faults (Linux-specific) */  
    comp_t    ac_swaps;    /* Number of swaps (unused; Linux-specific) */  
    u_int32_t ac_exitcode; /* Process termination status */  
#define ACCT_COMM 16  
    char      ac_comm[ACCT_COMM+1];  
                           /* (Null-terminated) command name  
                              (basename of last execed file) */  
    char      ac_pad[10];  /* Padding (reserved for future use) */  
};

Note the following points regarding the _acct_ structure:

• The _u_int16_t_ and _u_int32_t_ data types are 16-bit and 32-bit unsigned integers.

• The _ac_flag_ field is a bit mask recording various events for the process. The bits that can appear in this field are shown in [Table 28-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table1). As indicated in the table, some of these bits are not present on all UNIX implementations. A few other implementations provide additional bits in this field.

• The _ac_comm_ field records the name of the last command (program file) executed by this process. The kernel records this value on each _execve()_. On some other UNIX implementations, this field is limited to 8 characters.

• The _comp_t_ type is a kind of floating-point number. Values of this type are sometimes called _compressed clock ticks_. The floating-point value consists of a 3-bit, base-8 exponent, followed by a 13-bit mantissa; the exponent can represent a factor in the range 80=1 to 87 (2,097,152). For example, a mantissa of 125 and an exponent of 1 represent the value 1000. [Listing 28-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28ex2) defines a function (_comptToLL()_) to convert this type to _long long_. We need to use the type _long long_ because the 32 bits used to represent an _unsigned long_ on x86-32 are insufficient to hold the largest value that can be represented in _comp_t_, which is (213 – 1) * 87.

• The three time fields defined with the type _comp_t_ represent time in system clock ticks. Therefore, we must divide these times by the value returned by _sysconf(_SC_CLK_TCK)_ in order to convert them to seconds.

• The _ac_exitcode_ field holds the termination status of the process (described in [Section 26.1.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch26.xhtml#ch26lev2sec03)). Most other UNIX implementations instead provide a single-byte field named _ac_stat_, which records only the signal that killed the process (if it was killed by a signal) and a bit indicating whether that signal caused the process to dump core. BSD-derived implementations don’t provide either field.

**Table 28-1:** Bit values for the _ac_flag_ field of process accounting records

|**Bit**|**Description**|
|---|---|
|AFORK|Process was created by _fork()_, but did not _exec()_ before terminating|
|ASU|Process made use of superuser privileges|
|AXSIG|Process was terminated by a signal (not present on some implementations)|
|ACORE|Process produced a core dump (not present on some implementations)|

Because accounting records are written only as processes terminate, they are ordered by termination time (a value not recorded in the record), rather than by process start time (_ac_btime_).

If the system crashes, no accounting record is written for any processes that are still executing.

Since writing records to the accounting file can rapidly consume disk space, Linux provides the /proc/sys/kernel/acct virtual file for controlling the operation of process accounting. This file contains three numbers, defining (in order) the parameters _high-water_, _low-water_, and _frequency_. Typical defaults for these three parameters are 4, 2, and 30. If process accounting is enabled and the amount of free disk space falls below _low-water_ percent, accounting is suspended. If the amount of free disk space later rises above _high-water_ percent, then accounting is resumed. The _frequency_ value specifies how often, in seconds, checks should be made on the percentage of free disk space.

##### **Example program**

The program in [Listing 28-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28ex2) displays selected fields from the records in a process accounting file. The following shell session demonstrates the use of this program. We begin by creating a new, empty process accounting file and enabling process accounting:

$ su                           Need privilege to enable process accounting  
Password:  
# touch pacct  
# ./acct_on pacct              This process will be first entry in accounting file  
Process accounting enabled  
# exit                         Cease being superuser

At this point, three processes have already terminated since we enabled process accounting. These processes executed the _acct_on_, _su_, and _bash_ programs. The _bash_ process was started by _su_ to run the privileged shell session.

Now we run a series of commands to add further records to the accounting file:

$ sleep 15 &  
[1] 18063  
$ ulimit -c unlimited           Allow core dumps (shell built-in)  
$ cat                           Create a process  
Type Control-\ (generates SIGQUIT, signal 3) to kill cat process  
Quit (core dumped)  
$  
Press Enter to see shell notification of completion of sleep before next shell prompt  
[1]+  Done          sleep 15  
$ grep xxx badfile              grep fails with status of 2  
grep: badfile: No such file or directory  
$ echo $?                       The shell obtained status of grep (shell built-in)  
2

The next two commands run programs that we presented in previous chapters ([Listing 27-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch27.xhtml#ch27ex1), on [page 566](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch27.xhtml#page_566), and [Listing 24-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch24.xhtml#ch24ex1), on [page 517](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch24.xhtml#page_517)). The first command runs a program that execs the file /bin/echo; this results in an accounting record with the command name _echo_. The second command creates a child process that doesn’t perform an _exec()_.

$ ./t_execve /bin/echo  
hello world goodbye  
$ ./t_fork  
PID=18350 (child) idata=333 istack=666  
PID=18349 (parent) idata=111 istack=222

Finally, we use the program in [Listing 28-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28ex2) to view the contents of the accounting file:

$ ./acct_view pacct  
command  flags   term.  user     start time            CPU   elapsed  
                status                                 time    time  
acct_on   -S--      0   root     2010-07-23 17:19:05   0.00    0.00  
bash      ----      0   root     2010-07-23 17:18:55   0.02   21.10  
su        -S--      0   root     2010-07-23 17:18:51   0.01   24.94  
cat       --XC   0x83   mtk      2010-07-23 17:19:55   0.00    1.72  
sleep     ----      0   mtk      2010-07-23 17:19:42   0.00   15.01  
grep      ----  0x200   mtk      2010-07-23 17:20:12   0.00    0.00  
echo      ----      0   mtk      2010-07-23 17:21:15   0.01    0.01  
t_fork    F---      0   mtk      2010-07-23 17:21:36   0.00    0.00  
t_fork    ----      0   mtk      2010-07-23 17:21:36   0.00    3.01

In the output, we see one line for each process that was created in the shell session. The _ulimit_ and _echo_ commands are shell built-in commands, so they don’t result in the creation of new processes. Note that the entry for _sleep_ appeared in the accounting file after the _cat_ entry because the _sleep_ command terminated after the _cat_ command.

Most of the output is self-explanatory. The _flags_ column shows single letters indicating which of the _ac_flag_ bits is set in each record (see [Table 28-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table1)). [Section 26.1.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch26.xhtml#ch26lev2sec03) describes how to interpret the termination status values shown in the _term. status_ column.

**Listing 28-2:** Displaying data from a process accounting file

______________________________________________________ procexec/acct_view.c  
  
#include <fcntl.h>  
#include <time.h>  
#include <sys/stat.h>  
#include <sys/acct.h>  
#include <limits.h>  
#include "ugid_functions.h"             /* Declaration of userNameFromId() */  
#include "tlpi_hdr.h"  
  
#define TIME_BUF_SIZE 100  
  
static long long                /* Convert comp_t value into long long */  
comptToLL(comp_t ct)  
{  
    const int EXP_SIZE = 3;             /* 3-bit, base-8 exponent */  
    const int MANTISSA_SIZE = 13;       /* Followed by 13-bit mantissa */  
    const int MANTISSA_MASK = (1 << MANTISSA_SIZE) - 1;  
    long long mantissa, exp;  
  
    mantissa = ct & MANTISSA_MASK;  
    exp = (ct >> MANTISSA_SIZE) & ((1 << EXP_SIZE) - 1);  
    return mantissa << (exp * 3);       /* Power of 8 = left shift 3 bits */  
}  
  
int  
main(int argc, char *argv[])  
{  
    int acctFile;  
    struct acct ac;  
    ssize_t numRead;  
    char *s;  
    char timeBuf[TIME_BUF_SIZE];  
    struct tm *loc;  
    time_t t;  
  
    if (argc != 2 || strcmp(argv[1], "--help") == 0)  
        usageErr("%s file\n", argv[0]);  
  
    acctFile = open(argv[1], O_RDONLY);  
    if (acctFile == -1)  
        errExit("open");  
  
    printf("command  flags   term.  user     "  
            "start time            CPU   elapsed\n");  
    printf("                status           "  
            "                      time    time\n");  
  
    while ((numRead = read(acctFile, &ac, sizeof(struct acct))) > 0) {  
        if (numRead != sizeof(struct acct))  
            fatal("partial read");  
  
        printf("%-8.8s ", ac.ac_comm);  
  
        printf("%c", (ac.ac_flag & AFORK) ? 'F' : '-') ;  
        printf("%c", (ac.ac_flag & ASU)   ? 'S' : '-') ;  
        printf("%c", (ac.ac_flag & AXSIG) ? 'X' : '-') ;  
        printf("%c", (ac.ac_flag & ACORE) ? 'C' : '-') ;  
  
#ifdef __linux__  
        printf(" %#6lx   ", (unsigned long) ac.ac_exitcode);  
#else   /* Many other implementations provide ac_stat instead */  
        printf(" %#6lx   ", (unsigned long) ac.ac_stat);  
#endif  
  
        s = userNameFromId(ac.ac_uid);  
        printf("%-8.8s ", (s == NULL) ? "???" : s);  
  
        t = ac.ac_btime;  
        loc = localtime(&t);  
        if (loc == NULL) {  
            printf("???Unknown time??? ");  
        } else {  
            strftime(timeBuf, TIME_BUF_SIZE, "%Y-%m-%d %T ", loc);  
            printf("%s ", timeBuf);  
        }  
  
        printf("%5.2f %7.2f ", (double) (comptToLL(ac.ac_utime) +  
                    comptToLL(ac.ac_stime)) / sysconf(_SC_CLK_TCK),  
                (double) comptToLL(ac.ac_etime) / sysconf(_SC_CLK_TCK));  
        printf("\n");  
    }  
  
    if (numRead == -1)  
        errExit("read");  
  
    exit(EXIT_SUCCESS);  
}  
______________________________________________________ procexec/acct_view.c

##### **Process accounting Version 3 file format**

Starting with kernel 2.6.8, Linux introduced an optional alternative version of the process accounting file that addresses some limitations of the traditional accounting file. To use this alternative version, known as _Version 3_, the CONFIG_BSD_PROCESS_ACCT_V3 kernel configuration option must be enabled before building the kernel.

When using the Version 3 option, the only difference in the operation of process accounting is in the format of records written to the accounting file. The new format is defined as follows:

struct acct_v3 {  
    char      ac_flag;        /* Accounting flags */  
    char      ac_version;     /* Accounting version (3) */  
    u_int16_t ac_tty;         /* Controlling terminal for process */  
    u_int32_t ac_exitcode;    /* Process termination status */  
    u_int32_t ac_uid;         /* 32-bit user ID of process */  
    u_int32_t ac_gid;         /* 32-bit group ID of process */  
    u_int32_t ac_pid;         /* Process ID */  
    u_int32_t ac_ppid;        /* Parent process ID */  
    u_int32_t ac_btime;       /* Start time (time_t) */  
    float     ac_etime;       /* Elapsed (real) time (clock ticks) */  
    comp_t    ac_utime;       /* User CPU time (clock ticks) */  
    comp_t    ac_stime;       /* System CPU time (clock ticks) */  
    comp_t    ac_mem;         /* Average memory usage (kilobytes) */  
    comp_t    ac_io;          /* Bytes read/written (unused) */  
    comp_t    ac_rw;          /* Blocks read/written (unused) */  
    comp_t    ac_minflt;      /* Minor page faults */  
    comp_t    ac_majflt;      /* Major page faults */  
    comp_t    ac_swaps;       /* Number of swaps (unused; Linux-specific) */  
#define ACCT_COMM 16  
    char      ac_comm[ACCT_COMM];   /* Command name */  
};

The following are the main differences between the _acct_v3_ structure and the traditional Linux _acct_ structure:

• The _ac_version_ field is added. This field contains the version number of this type of accounting record. This field is always 3 for an _acct_v3_ record.

• The fields _ac_pid_ and _ac_ppid_, containing the process ID and parent process ID of the terminated process, are added.

• The _ac_uid_ and _ac_gid_ fields are widened from 16 to 32 bits, to accommodate the 32-bit user and group IDs that were introduced in Linux 2.4. (Large user and group IDs can’t be correctly represented in the traditional _acct_ file.)

• The type of the _ac_etime_ field is changed from _comp_t_ to _float_, to allow longer elapsed times to be recorded.

We provide a Version 3 analog of the program in [Listing 28-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28ex2) in the file procexec/acct_v3_view.c in the source code distribution for this book.

### **28.2 The _clone()_ System Call**

Like _fork()_ and _vfork()_, the Linux-specific _clone()_ system call creates a new process. It differs from the other two calls in allowing finer control over the steps that occur during process creation. The main use of _clone()_ is in the implementation of threading libraries. Because _clone()_ is not portable, its direct use in application programs should normally be avoided. We describe it here because it is useful background for the discussion of POSIX threads in [Chapters 29](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch29.xhtml#ch29) to [33](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch33.xhtml#ch33), and also because it further illuminates the operation of _fork()_ and _vfork()_.

#define _GNU_SOURCE  
#include <sched.h>  
  
int clone(int (*func) (void *), void *child_stack, int flags, void *func_arg, ...  
          /* pid_t *ptid, struct user_desc *tls, pid_t *ctid */ );

Returns process ID of child on success, or –1 on error

Like _fork()_, a new process created with _clone()_ is an almost exact duplicate of the parent.

Unlike _fork()_, the cloned child doesn’t continue from the point of the call, but instead commences by calling the function specified in the _func_ argument; we’ll refer to this as the _child function_. When called, the child function is passed the value specified in _func_arg_. Using appropriate casting, the child function can freely interpret this argument; for example, as an _int_ or as a pointer to a structure. (Interpreting it as a pointer is possible because the cloned child either obtains a copy of or shares the calling process’s memory.)

Within the kernel, _fork()_, _vfork()_, and _clone()_ are ultimately implemented by the same function (_do_fork()_ in kernel/fork.c). At this level, cloning is much closer to forking: _sys_clone()_ doesn’t have the _func_ and _func_arg_ arguments, and after the call, _sys_clone()_ returns in the child in the same manner as _fork()_. The main text describes the _clone()_ wrapper function that _glibc_ provides for _sys_clone()_. (This function is defined in architecture-specific _glibc_ assembler sources, such as in sysdeps/unix/sysv/linux/i386/clone.S.) This wrapper function invokes _func_ after _sys_clone()_ returns in the child.

The cloned child process terminates either when _func_ returns (in which case its return value is the exit status of the process) or when the process makes a call to _exit()_ (or __exit()_). The parent process can wait for the cloned child in the usual manner using _wait()_ or similar.

Since a cloned child may (like _vfork()_) share the parent’s memory, it can’t use the parent’s stack. Instead, the caller must allocate a suitably sized block of memory for use as the child’s stack and pass a pointer to that block in the argument _child_stack_. On most hardware architectures, the stack grows downward, so the _child_stack_ argument should point to the high end of the allocated block.

The architecture-dependence on the direction of stack growth is a defect in the design of _clone()_. On the Intel IA-64 architecture, an improved clone API is provided, in the form of _clone2()_. This system call defines the range of the stack of the child in a way that doesn’t depend on the direction of stack growth, by supplying both the start address and size of the stack. See the manual page for details.

The _clone() flags_ argument serves two purposes. First, its lower byte specifies the child’s _termination signal_, which is the signal to be sent to the parent when the child terminates. (If a cloned child is _stopped_ by a signal, the parent still receives SIGCHLD.) This byte may be 0, in which case no signal is generated. (Using the Linux-specific /proc/_PID_/stat file, we can determine the termination signal of any process; see the _proc(5)_ manual page for further details.)

With _fork()_ and _vfork()_, we have no way to select the termination signal; it is always SIGCHLD.

The remaining bytes of the _flags_ argument hold a bit mask that controls the operation of _clone()_. We summarize these bit-mask values in [Table 28-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table2), and describe them in more detail in [Section 28.2.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28lev2sec01).

**Table 28-2:** The _clone() flags_ bit-mask values

|**Flag**|**Effect if present**|
|---|---|
|CLONE_CHILD_CLEARTID|Clear _ctid_ when child calls _exec()_ or __exit()_ (2.6 onward)|
|CLONE_CHILD_SETTID|Write thread ID of child into _ctid_ (2.6 onward)|
|CLONE_FILES|Parent and child share table of open file descriptors|
|CLONE_FS|Parent and child share attributes related to file system|
|CLONE_IO|Child shares parent’s I/O context (2.6.25 onward)|
|CLONE_NEWIPC|Child gets new System V IPC namespace (2.6.19 onward)|
|CLONE_NEWNET|Child gets new network namespace (2.6.24 onward)|
|CLONE_NEWNS|Child gets copy of parent’s mount namespace (2.4.19 onward)|
|CLONE_NEWPID|Child gets new process-ID namespace (2.6.19 onward)|
|CLONE_NEWUSER|Child gets new user-ID namespace (2.6.23 onward)|
|CLONE_NEWUTS|Child gets new UTS (_uname()_) namespace (2.6.19 onward)|
|CLONE_PARENT|Make child’s parent same as caller’s parent (2.4 onward)|
|CLONE_PARENT_SETTID|Write thread ID of child into _ptid_ (2.6 onward)|
|CLONE_PID|Obsolete flag used only by system boot process (up to 2.4)|
|CLONE_PTRACE|If parent is being traced, then trace child also|
|CLONE_SETTLS|_tls_ describes thread-local storage for child (2.6 onward)|
|CLONE_SIGHAND|Parent and child share signal dispositions|
|CLONE_SYSVSEM|Parent and child share semaphore undo values (2.6 onward)|
|CLONE_THREAD|Place child in same thread group as parent (2.4 onward)|
|CLONE_UNTRACED|Can’t force CLONE_PTRACE on child (2.6 onward)|
|CLONE_VFORK|Parent is suspended until child calls _exec()_ or __exit()_|
|CLONE_VM|Parent and child share virtual memory|

The remaining arguments to _clone()_ are _ptid_, _tls_, and _ctid_. These arguments relate to the implementation of threads, in particular the use of thread IDs and thread-local storage. We cover the use of these arguments when describing the _flags_ bit-mask values in [Section 28.2.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28lev2sec01). (In Linux 2.4 and earlier, these three arguments are not provided by _clone()_. They were specifically added in Linux 2.6 to support the NPTL POSIX threads implementation.)

##### **Example program**

[Listing 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28ex3) shows a simple example of the use of _clone()_ to create a child process. The main program does the following:

• Open a file descriptor (for /dev/null) that will be closed by the child ②.

• Set the value for the _clone() flags_ argument to CLONE_FILES ③ if a command-line argument was supplied, so that the parent and child will share a single file descriptor table. If no command-line argument was supplied, _flags_ is set to 0.

• Allocate a stack for use by the child ④.

• If CHILD_SIG is nonzero and is not equal to SIGCHLD, ignore it, in case it is a signal that would terminate the process ⑤. We don’t ignore SIGCHLD, because doing so would prevent waiting on the child to collect its status.

• Call _clone()_ to create the child ⑥. The third (bit-mask) argument includes the termination signal. The fourth argument (_func_arg_) specifies the file descriptor opened earlier (at ②).

• Wait for the child to terminate ⑦.

• Check whether the file descriptor (opened at ②) is still open by trying to _write()_ to it ⑧. The program reports whether the _write()_ succeeds or fails.

Execution of the cloned child begins in _childFunc()_, which receives (in the argument _arg_) the file descriptor opened by the main program (at ②). The child closes this file descriptor and then terminates by performing a return ①.

**Listing 28-3:** Using _clone()_ to create a child process

__________________________________________________ procexec/t_clone.c  
  
   #define _GNU_SOURCE  
   #include <signal.h>  
   #include <sys/wait.h>  
   #include <fcntl.h>  
   #include <sched.h>  
   #include "tlpi_hdr.h"  
  
   #ifndef CHILD_SIG  
   #define CHILD_SIG SIGUSR1       /* Signal to be generated on termination  
                                      of cloned child */  
   #endif  
  
   static int                      /* Startup function for cloned child */  
   childFunc(void *arg)  
   {  
①     if (close(*((int *) arg)) == -1)  
           errExit("close");  
  
       return 0;                           /* Child terminates now */  
   }  
  
   int  
   main(int argc, char *argv[])  
   {  
       const int STACK_SIZE = 65536;       /* Stack size for cloned child */  
       char *stack;                        /* Start of stack buffer */  
       char *stackTop;                     /* End of stack buffer */  
       int s, fd, flags;  
  
       fd = open("/dev/null", O_RDWR);    /* Child will close this fd */  
②     if (fd == -1)  
           errExit("open");  
  
       /* If argc > 1, child shares file descriptor table with parent */  
  
③     flags = (argc > 1) ? CLONE_FILES : 0;  
  
       /* Allocate stack for child */  
  
④     stack = malloc(STACK_SIZE);  
       if (stack == NULL)  
           errExit("malloc");  
       stackTop = stack + STACK_SIZE;      /* Assume stack grows downward */  
  
       /* Ignore CHILD_SIG, in case it is a signal whose default is to  
          terminate the process; but don't ignore SIGCHLD (which is ignored  
          by default), since that would prevent the creation of a zombie. */  
  
⑤     if (CHILD_SIG != 0 && CHILD_SIG != SIGCHLD)  
           if (signal(CHILD_SIG, SIG_IGN) == SIG_ERR)  
               errExit("signal");  
  
       /* Create child; child commences execution in childFunc() */  
  
⑥     if (clone(childFunc, stackTop, flags | CHILD_SIG, (void *) &fd) == -1)  
           errExit("clone");  
  
       /* Parent falls through to here. Wait for child; __WCLONE is  
          needed for child notifying with signal other than SIGCHLD. */  
  
⑦     if (waitpid(-1, NULL, (CHILD_SIG != SIGCHLD) ? __WCLONE : 0) == -1)  
           errExit("waitpid");  
       printf("child has terminated\n");  
  
       /* Did close() of file descriptor in child affect parent? */  
  
⑧     s = write(fd, "x", 1);  
       if (s == -1 && errno == EBADF)  
           printf("file descriptor %d has been closed\n", fd);  
       else if (s == -1)  
           printf("write() on file descriptor %d failed "  
                   "unexpectedly (%s)\n", fd, strerror(errno));  
       else  
           printf("write() on file descriptor %d succeeded\n", fd);  
  
       exit(EXIT_SUCCESS);  
   }  
__________________________________________________ procexec/t_clone.c

When we run the program in [Listing 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28ex3) without a command-line argument, we see the following:

$ ./t_clone                               Doesn't use CLONE_FILES  
child has terminated  
write() on file descriptor 3 succeeded    Child's close() did not affect parent

When we run the program with a command-line argument, we can see that the two processes share the file descriptor table:

$ ./t_clone x                            Uses CLONE_FILES  
child has terminated  
file descriptor 3 has been closed        Child's close() affected parent

We show a more complex example of the use of _clone()_ in the file procexec/demo_clone.c in the source code distribution for this book.

#### **28.2.1 The _clone() flags_ Argument**

The _clone() flags_ argument is a combination (ORing) of the bit-mask values described in the following pages. Rather than presenting these flags in alphabetical order, we present them in an order that eases explanation, and begin with those flags that are used in the implementation of POSIX threads. From the point of view of implementing threads, many uses of the word _process_ below can be replaced by _thread_.

At this point, it is worth remarking that, to some extent, we are playing with words when trying to draw a distinction between the terms _thread_ and _process_. It helps a little to introduce the term _kernel scheduling entity_ (KSE), which is used in some texts to refer to the objects that are dealt with by the kernel scheduler. Really, threads and processes are simply KSEs that provide for greater and lesser degrees of sharing of attributes (virtual memory, open file descriptors, signal dispositions, process ID, and so on) with other KSEs. The POSIX threads specification provides just one out of various possible definitions of which attributes should be shared between threads.

In the course of the following descriptions, we’ll sometimes mention the two main implementations of POSIX threads available on Linux: the older LinuxThreads implementation and the more recent NPTL implementation. Further information about these two implementations can be found in [Section 33.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch33.xhtml#ch33lev1sec05).

Starting in kernel 2.6.16, Linux provides a new system call, _unshare()_, which allows a child created using _clone()_ (or _fork()_ or _vfork()_) to undo some of the attribute sharing (i.e., reverse the effects of some of the _clone() flags_ bits) that was established when the child was created. For details, see the _unshare(2)_ manual page.

##### **Sharing file descriptor tables:** CLONE_FILES

If the CLONE_FILES flag is specified, the parent and the child share the same table of open file descriptors. This means that file descriptor allocation or deallocation (_open()_, _close()_, _dup()_, _pipe()_, _socket()_, and so on) in either process will be visible in the other process. If the CLONE_FILES flag is not set, then the file descriptor table is not shared, and the child gets a copy of the parent’s table at the time of the _clone()_ call. These copied descriptors refer to the same open file descriptions as the corresponding descriptors in the parent (as with _fork()_ and _vfork()_).

The specification of POSIX threads requires that all of the threads in a process share the same open file descriptors.

##### **Sharing file system–related information:** CLONE_FS

If the CLONE_FS flag is specified, then the parent and the child share file system–related information—umask, root directory, and current working directory. This means that calls to _umask()_, _chdir()_, or _chroot()_ in either process will affect the other process. If the CLONE_FS flag is not set, then the parent and child have separate copies of this information (as with _fork()_ and _vfork()_).

The attribute sharing provided by CLONE_FS is required by POSIX threads.

##### **Sharing signal dispositions:** CLONE_SIGHAND

If the CLONE_SIGHAND flag is set, then the parent and child share the same table of signal dispositions. Using _sigaction()_ or _signal()_ to change a signal’s disposition in either process will affect that signal’s disposition in the other process. If the CLONE_SIGHAND flag is not set, then signal dispositions are not shared; instead, the child gets a copy of the parent’s signal disposition table (as with _fork()_ and _vfork()_). The CLONE_SIGHAND flag doesn’t affect the process signal mask and the set of pending signals, which are always distinct for the two processes. From Linux 2.6 onward, CLONE_VM must also be included in _flags_ if CLONE_SIGHAND is specified.

Sharing of signal dispositions is required by POSIX threads.

##### **Sharing the parent’s virtual memory:** CLONE_VM

If the CLONE_VM flag is set, then the parent and child share the same virtual memory pages (as with _vfork()_). Updates to memory or calls to _mmap()_ or _munmap()_ by either process will be visible to the other process. If the CLONE_VM flag is not set, then the child receives a copy of the parent’s virtual memory (as with _fork()_).

Sharing the same virtual memory is one of the defining attributes of threads, and is required by POSIX threads.

##### **Thread groups:** CLONE_THREAD

If the CLONE_THREAD flag is set, then the child is placed in the same thread group as the parent. If this flag is not set, the child is placed in its own new thread group.

_Thread groups_ were introduced in Linux 2.4 to allow threading libraries to support the POSIX threads requirement that all of the threads in a process share a single process ID (i.e., _getpid()_ in each of the threads should return the same value). A thread group is a group of KSEs that share the same _thread group identifier_ (TGID), as shown in [Figure 28-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28fig1). For the remainder of the discussion of CLONE_THREAD, we’ll refer to these KSEs as _threads_.

Since Linux 2.4, _getpid()_ returns the calling thread’s TGID. In other words, a TGID is the same thing as a process ID.

The _clone()_ implementation in Linux 2.2 and earlier did not provide CLONE_THREAD. Instead, LinuxThreads implemented POSIX threads as processes that shared various attributes (e.g., virtual memory) but had distinct process IDs. For compatibility reasons, even on modern Linux kernels, the LinuxThreads implementation doesn’t use the CLONE_THREAD flag, so that threads in that implementation continue to have distinct process IDs.

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f28-01.jpg)

**Figure 28-1:** A thread group containing four threads

Each thread within a thread group is distinguished by a unique _thread identifier_ (TID). Linux 2.4 introduced a new system call, _gettid()_, to allow a thread to obtain its own thread ID (this is the same value as is returned to the thread that calls _clone()_). A thread ID is represented using the same data type that is used for a process ID, _pid_t_. Thread IDs are unique system-wide, and the kernel guarantees that no thread ID will be the same as any process ID on the system, except when a thread is the thread group leader for a process.

The first thread in a new thread group has a thread ID that is the same as its thread group ID. This thread is referred to as the _thread group leader_.

The thread IDs that we are discussing here are not the same as the thread IDs (the _pthread_t_ data type) used by POSIX threads. The latter identifiers are generated and maintained internally (in user space) by a POSIX threads implementation.

All of the threads in a thread group have the same parent process ID—that of the thread group leader. Only after all of the threads in a thread group have terminated is a SIGCHLD signal (or other termination signal) sent to that parent process. These semantics correspond to the requirements of POSIX threads.

When a CLONE_THREAD thread terminates, no signal is sent to the thread that created it using _clone()_. Correspondingly, it is not possible to use _wait()_ (or similar) to wait for a thread created using CLONE_THREAD. This accords with POSIX requirements. A POSIX thread is not the same thing as a process, and can’t be waited for using _wait()_; instead, it must be joined using _pthread_join()_. To detect the termination of a thread created using CLONE_THREAD, a special synchronization primitive, called a _futex_, is used (see the discussion of the CLONE_PARENT_SETTID flag below).

If any of the threads in a thread group performs an _exec()_, then all threads other than the thread group leader are terminated (this behavior corresponds to the semantics required for POSIX threads), and the new program is execed in the thread group leader. In other words, in the new program, _gettid()_ will return the thread ID of the thread group leader. During an _exec()_, the termination signal that this process should send to its parent is reset to SIGCHLD.

If one of the threads in a thread group creates a child using _fork()_ or _vfork()_, then any thread in the group can monitor that child using _wait()_ or similar.

From Linux 2.6 onward, CLONE_SIGHAND must also be included in _flags_ if CLONE_THREAD is specified. This corresponds to further POSIX threads requirements; for details, see the description of how POSIX threads and signals interact in [Section 33.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch33.xhtml#ch33lev1sec02). (The kernel handling of signals for CLONE_THREAD thread groups mirrors the POSIX requirements for how the threads in a process should react to signals.)

##### **Threading library support:** CLONE_PARENT_SETTID, CLONE_CHILD_SETTID**, and** CLONE_CHILD_CLEARTID

The CLONE_PARENT_SETTID, CLONE_CHILD_SETTID, and CLONE_CHILD_CLEARTID flags were added in Linux 2.6 to support the implementation of POSIX threads. These flags affect how _clone()_ treats its _ptid_ and _ctid_ arguments. CLONE_PARENT_SETTID and CLONE_CHILD_CLEARTID are used in the NPTL threading implementation.

If the CLONE_PARENT_SETTID flag is set, then the kernel writes the thread ID of the child thread into the location pointed to by _ptid_. The thread ID is copied into _ptid_ before the memory of the parent is duplicated. This means that, even if the CLONE_VM flag is not specified, both the parent and the child can see the child’s thread ID in this location. (As noted above, the CLONE_VM flag is specified when creating POSIX threads.)

The CLONE_PARENT_SETTID flag exists in order to provide a reliable means for a threading implementation to obtain the ID of the new thread. Note that it isn’t sufficient to obtain the thread ID of the new thread via the return value of _clone()_, like so:

tid = clone(...);

The problem is that this code can lead to various race conditions, because the assignment occurs only after _clone()_ returns. For example, suppose that the new thread terminates, and the handler for its termination signal is invoked before the assignment to _tid_ completes. In this case, the handler can’t usefully access _tid_. (Within a threading library, _tid_ might be an item in a global bookkeeping structure used to track the status of all threads.) Programs that invoke _clone()_ directly often can be designed to work around this race condition. However, a threading library can’t control the actions of the program that calls it. Using CLONE_PARENT_SETTID to ensure that the new thread ID is placed in the location pointed to by _ptid_ before _clone()_ returns allows a threading library to avoid such race conditions.

If the CLONE_CHILD_SETTID flag is set, then _clone()_ writes the thread ID of the child thread into the location pointed to by _ctid_. The setting of _ctid_ is done only in the child’s memory, but this will affect the parent if CLONE_VM is also specified. Although NPTL doesn’t need CLONE_CHILD_SETTID, this flag is provided to allow flexibility for other possible threading library implementations.

If the CLONE_CHILD_CLEARTID flag is set, then _clone()_ zeros the memory location pointed to by _ctid_ when the child terminates.

The _ctid_ argument is the mechanism (described in a moment) by which the NPTL threading implementation obtains notification of the termination of a thread. Such notification is required by the _pthread_join()_ function, which is the POSIX threads mechanism by which one thread can wait for the termination of another thread.

When a thread is created using _pthread_create()_, NPTL makes a _clone()_ call in which _ptid_ and _ctid_ point to the same location. (This is why CLONE_CHILD_SETTID is not required by NPTL.) The CLONE_PARENT_SETTID flag causes that location to be initialized with the new thread’s ID. When the child terminates and _ctid_ is cleared, that change is visible to all threads in the process (since the CLONE_VM flag is also specified).

The kernel treats the location pointed to by _ctid_ as a _futex_, an efficient synchronization mechanism. (See the _futex(2)_ manual page for further details of futexes.) Notification of thread termination can be obtained by performing a _futex()_ system call that blocks waiting for a change in the value at the location pointed to by _ctid_. (Behind the scenes, this is what _pthread_join()_ does.) At the same time that the kernel clears _ctid_, it also wakes up any kernel scheduling entity (i.e., thread) that is blocked performing a futex wait on that address. (At the POSIX threads level, this causes the _pthread_join()_ call to unblock.)

##### **Thread-local storage:** CLONE_SETTLS

If the CLONE_SETTLS flag is set, then the _tls_ argument points to a _user_desc_ structure describing the thread-local storage buffer to be used for this thread. This flag was added in Linux 2.6 to support the NPTL implementation of thread-local storage ([Section 31.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch31.xhtml#ch31lev1sec04)). For details of the _user_desc_ structure, see the definition and use of this structure in the 2.6 kernel sources and the _set_thread_area(2)_ manual page.

##### **Sharing System V semaphore undo values:** CLONE_SYSVSEM

If the CLONE_SYSVSEM flag is set, then the parent and child share a single list of System V semaphore undo values ([Section 47.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch47.xhtml#ch47lev1sec08)). If this flag is not set, then the parent and child have separate undo lists, and the child’s undo list is initially empty.

The CLONE_SYSVSEM flag is available from kernel 2.6 onward, and provides the sharing semantics required by POSIX threads.

##### **Per-process mount namespaces:** CLONE_NEWNS

From kernel 2.4.19 onward, Linux supports the notion of per-process _mount namespaces_. A mount namespace is the set of mount points maintained by calls to _mount()_ and _umount()_. The mount namespace affects how pathnames are resolved to actual files, as well as the operation of system calls such as _chdir()_ and _chroot()_.

By default, the parent and the child share a mount namespace, which means that changes to the namespace by one process using _mount()_ and _umount()_ are visible in the other process (as with _fork()_ and _vfork()_). A privileged (CAP_SYS_ADMIN) process may specify the CLONE_NEWNS flag so that the child obtains a copy of the parent’s mount namespace. Thereafter, changes to the namespace by one process are not visible in the other process. (In earlier 2.4._x_ kernels, as well as in older kernels, we can consider all processes on the system as sharing a single system-wide mount namespace.)

Per-process mount namespaces can be used to create environments that are similar to _chroot()_ jails, but which are more secure and flexible; for example, a jailed process can be provided with a mount point that is not visible to other processes on the system. Mount namespaces are also useful in setting up virtual server environments.

Specifying both CLONE_NEWNS and CLONE_FS in the same call to _clone()_ is nonsensical and is not permitted.

##### **Making the child’s parent the same as the caller’s:** CLONE_PARENT

By default, when we create a new process with _clone()_, the parent of that process (as returned by _getppid()_) is the process that calls _clone()_ (as with _fork()_ and _vfork()_). If the CLONE_PARENT flag is set, then the parent of the child will be the caller’s parent. In other words, CLONE_PARENT is the equivalent of setting _child.PPID = caller.PPID_. (In the default case, without CLONE_PARENT, it would be _child.PPID = caller.PID_.) The parent process (_child.PPID_) is the process that is signaled when the child terminates.

The CLONE_PARENT flag is available in Linux 2.4 and later. Originally, it was designed to be useful for POSIX threads implementations, but the 2.6 kernel pursued an approach to supporting threads (the use of CLONE_THREAD, described above) that removed the need for this flag.

##### **Making the child’s PID the same as the parent’s PID:** CLONE_PID **(obsolete)**

If the CLONE_PID flag is set, then the child has the same process ID as the parent. If this flag is not set, then the parent and child have different process IDs (as with _fork()_ and _vfork()_). Only the system boot process (process ID 0) may specify this flag; it is used when initializing a multiprocessor system.

The CLONE_PID flag is not intended for use in user applications. In Linux 2.6, it has been removed, and is superseded by CLONE_IDLETASK, which causes the process ID of the new process to be set to 0. CLONE_IDLETASK is available only for internal use within the kernel (if specified in the _flags_ argument of _clone()_, it is ignored). It is used to create the invisible per-CPU _idle process_, of which multiple instances may exist on multiprocessor systems.

##### **Process tracing:** CLONE_PTRACE **and** CLONE_UNTRACED

If the CLONE_PTRACE flag is set and the calling process is being traced, then the child is also traced. For details on process tracing (used by debuggers and the _strace_ command), refer to the _ptrace(2)_ manual page.

From kernel 2.6 onward, the CLONE_UNTRACED flag can be set, meaning that a tracing process can’t force CLONE_PTRACE on this child. The CLONE_UNTRACED flag is used internally by the kernel in the creation of kernel threads.

##### **Suspending the parent until the child exits or execs:** CLONE_VFORK

If the CLONE_VFORK flag is set, then the execution of the parent is suspended until the child releases its virtual memory resources via a call to _exec()_ or __exit()_ (as with _vfork()_).

##### **New _clone()_ flags to support containers**

A number of new _clone() flags_ values were added in Linux 2.6.19 and later: CLONE_IO, CLONE_NEWIPC, CLONE_NEWNET, CLONE_NEWPID, CLONE_NEWUSER, and CLONE_NEWUTS. (See the _clone(2)_ manual page for the details of these flags.)

Most of these flags are provided to support the implementation of _containers_ ([[Bhattiprolu et al., 2008](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib05)]). A container is a form of lightweight virtualization, whereby groups of processes running on the same kernel can be isolated from one another in environments that appear to be separate machines. Containers can also be nested, one inside the other. The containers approach contrasts with full virtualization, where each virtualized environment is running a distinct kernel.

To implement containers, the kernel developers had to provide a layer of indirection within the kernel around each of the global system resources—such as process IDs, the networking stack, the identifiers returned by _uname()_, System V IPC objects, and user and group ID namespaces—so that each container can provide its own instance of these resources.

There are various possible uses for containers, including the following:

• controlling allocation of resources on the system, such as network bandwidth or CPU time (e.g., one container might be granted 75% of the CPU time, while the other gets 25%);

• providing multiple lightweight virtual servers on a single host machine;

• freezing a container, so that execution of all processes in the container is suspended, later to be restarted, possibly after migrating to a different machine; and

• allowing an application’s state to be dumped (checkpointed) and then later restored (perhaps after an application crash, or a planned or unplanned system shutdown) to continue computation from the time of the checkpoint.

##### **Use of _clone() flags_**

Roughly, we can say that a _fork()_ corresponds to a _clone()_ call with _flags_ specified as just SIGCHLD, while a _vfork()_ corresponds to a _clone()_ call specifying _flags_ as follows:

CLONE_VM | CLONE_VFORK | SIGCHLD

Since version 2.3.3, the _glibc_ wrapper _fork()_ provided as part of the NPTL threading implementation bypasses the kernel’s _fork()_ system call and invokes _clone()_. This wrapper function invokes any fork handlers that have been established by the caller using _pthread_atfork()_ (see [Section 33.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch33.xhtml#ch33lev1sec03)).

The LinuxThreads threading implementation uses _clone()_ (with just the first four arguments) to create threads by specifying _flags_ as follows:

CLONE_VM | CLONE_FILES | CLONE_FS | CLONE_SIGHAND

The NPTL threading implementation uses _clone()_ (with all seven arguments) to create threads by specifying _flags_ as follows:

CLONE_VM | CLONE_FILES | CLONE_FS | CLONE_SIGHAND | CLONE_THREAD |  
CLONE_SETTLS | CLONE_PARENT_SETTID | CLONE_CHILD_CLEARTID | CLONE_SYSVSEM

#### **28.2.2 Extensions to _waitpid()_ for Cloned Children**

To wait for children produced by _clone()_, the following additional (Linux-specific) values can be included in the _options_ bit-mask argument for _waitpid()_, _wait3()_, and _wait4()_:

__WCLONE

If set, then wait for _clone_ children only. If not set, then wait for _nonclone_ children only. In this context, a _clone_ child is one that delivers a signal other than SIGCHLD to its parent on termination. This bit is ignored if __WALL is also specified.

__WALL (since Linux 2.4)

Wait for all children, regardless of type (_clone_ or _nonclone_).

__WNOTHREAD (since Linux 2.4)

By default, the wait calls wait not only for children of the calling process, but also for children of any other processes in the same thread group as the caller. Specifying the __WNOTHREAD flag limits the wait to children of the calling process.

These flags can’t be used with _waitid()_.

### **28.3 Speed of Process Creation**

[Table 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table3) shows some speed comparisons for different methods of process creation. The results were obtained using a test program that executed a loop that repeatedly created a child process and then waited for it to terminate. The table compares the various methods using three different process memory sizes, as indicated by the _Total virtual memory_ value. The differences in memory size were simulated by having the program _malloc()_ additional memory on the heap prior to performing the timings.

Values for process size (_Total virtual memory_) in [Table 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table3) are taken from the _VSZ_ value displayed by the command _ps –o “pid vsz cmd”_.

**Table 28-3:** Time required to create 100,000 processes using _fork()_, _vfork()_, and _clone()_

|**Method of process creation**|**Total Virtual Memory**|   |   |   |   |   |
|---|---|---|---|---|---|---|
|**1.70 MB**|   |**2.70 MB**|   |**11.70 MB**|   |
|---|---|---|---|---|---|
|**Time (secs)**|**Rate**|**Time (secs)**|**Rate**|**Time (secs)**|**Rate**|
|---|---|---|---|---|---|
|_fork()_|22.27  <br>(7.99)|4544|26.38  <br>(8.98)|4135|126.93  <br>(52.55)|1276|
|_vfork()_|3.52  <br>(2.49)|28955|3.55  <br>(2.50)|28621|3.53  <br>(2.51)|28810|
|_clone()_|2.97  <br>(2.14)|34333|2.98  <br>(2.13)|34217|2.93  <br>(2.10)|34688|
|_fork() + exec()_|135.72  <br>(12.39)|764|146.15  <br>(16.69)|719|260.34  <br>(61.86)|435|
|_vfork() + exec()_|107.36  <br>(6.27)|969|107.81  <br>(6.35)|964|107.97  <br>(6.38)|960|

For each process size, two types of statistics are provided in [Table 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table3):

• The first statistic consists of two time measurements. The main (larger) measurement is the total elapsed (real) time to perform 100,000 process creation operations. The second time, shown in parentheses, is the CPU time consumed by the parent process. Since these tests were run on an otherwise unloaded machine, the difference between the two time values represents the total time consumed by child processes created during the test.

• The second statistic for each test shows the rate at which processes were created per (real) second.

Statistics shown are the average of 20 runs for each case, and were obtained using kernel 2.6.27 running on an x86-32 system.

The first three data rows show times for simple process creation (without execing a new program in the child). In each case, the child processes exit immediately after they are created, and the parent waits for each child to terminate before creating the next.

The first row contains values for the _fork()_ system call. From the data, we can see that as a process gets larger, _fork()_ takes longer. These time differences show the additional time required to duplicate increasingly large page tables for the child and mark all data, heap, and stack segment page entries as read-only. (No _pages_ are copied, since the child doesn’t modify its data or stack segments.)

The second data row provides the same statistics for _vfork()_. We see that as the process size increases, the times remain the same—because no page tables or pages are copied during a _vfork()_, the virtual memory size of the calling process has no effect. The difference between the _fork()_ and _vfork()_ statistics represents the total time required for copying process page tables in each case.

Small variations in the _vfork()_ and _clone()_ values in [Table 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table3) are due to sampling errors and scheduling variations. Even when creating processes up to 300 MB in size, times for these two system calls remained constant.

The third data row shows statistics for process creation using _clone()_ with the following flags:

CLONE_VM | CLONE_VFORK | CLONE_FS | CLONE_SIGHAND | CLONE_FILES

The first two of these flags emulate the behavior of _vfork()_. The remaining flags specify that the parent and child should share their file-system attributes (umask, root directory, and current working directory), table of signal dispositions, and table of open file descriptors. The difference between the _clone()_ and _vfork()_ data represents the small amount of additional work performed in _vfork()_ to copy this information into the child process. The cost of copying file-system attributes and the table of signal dispositions is constant. However, the cost of copying the table of open file descriptors varies according to the number of descriptors. For example, opening 100 file descriptors in the parent process raised the _vfork()_ real time (in the first column of the table) from 3.52 to 5.04 seconds, but left times for _clone()_ unaffected.

The timings for _clone()_ are for the _glibc clone()_ wrapper function, rather than direct calls to _sys_clone()_. Other tests (not summarized here) revealed negligible timing differences between using _sys_clone()_ and calling _clone()_ with a child function that immediately exited.

The differences between _fork()_ and _vfork()_ are quite marked. However, the following points should be kept in mind:

• The final data column, where _vfork()_ is more than 30 times faster than _fork()_, corresponds to a large process. Typical processes would lie somewhere closer to the first two columns of the table.

• Because the times required for process creation are typically much smaller than those required for an _exec()_, the differences are much less marked if a _fork()_ or _vfork()_ is followed by an _exec()_. This is illustrated by the final pair of data rows in [Table 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table3), where each child performs an _exec()_, rather than immediately exiting. The program execed was the _true_ command (/bin/true, chosen because it produces no output). In this case, we see that the relative differences between _fork()_ and _vfork()_ are much lower.

In fact, the data shown in [Table 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table3) doesn’t reveal the full cost of an _exec()_, because the child execs the same program in each loop of the test. As a result, the cost of disk I/O to read the program into memory is essentially eliminated, because the program will be read into the kernel buffer cache on the first _exec()_, and then remain there. If each loop of the test execed a different program (e.g., a differently named copy of the same program), then we would observe a greater cost for an _exec()_.

### **28.4 Effect of _exec()_ and _fork()_ on Process Attributes**

A process has numerous attributes, some of which we have already described in earlier chapters, and others that we explore in later chapters. Regarding these attributes, two questions arise:

• What happens to these attributes when a process performs an _exec()_?

• Which attributes are inherited by a child when a _fork()_ is performed?

[Table 28-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table4) summarizes the answers to these questions. The _exec()_ column indicates which attributes are preserved during an _exec()_. The _fork()_ column indicates which attributes are inherited (or in some cases, shared) by a child after _fork()_. Other than the attributes indicated as being Linux-specific, all listed attributes appear in standard UNIX implementations, and their handling during _exec()_ and _fork()_ conforms to the requirements of SUSv3.

**Table 28-4:** Effect of _exec()_ and _fork()_ on process attributes

|**Process attribute**|**_exec()_**|**_fork()_**|**Interfaces affecting attribute; additional notes**|
|---|---|---|---|
|**Process address space**|   |   |   |
|Text segment|No|Shared|Child process shares text segment with parent.|
|Stack segment|No|Yes|Function entry/exit; _alloca()_, _longjmp()_, _siglongjmp()_.|
|Data and heap segments|No|Yes|_brk()_, _sbrk()_.|
|Environment variables|See notes|Yes|_putenv()_, _setenv()_; direct modification of _environ_. Overwritten by _execle()_ and _execve()_ and preserved by remaining _exec()_ calls.|
|Memory mappings|No|Yes; see notes|_mmap()_, _munmap()_. A mapping’s MAP_NORESERVE flag is inherited across _fork()_. Mappings that have been marked with _madvise(MADV_DONTFORK)_ are not inherited across _fork()_.|
|Memory locks|No|No|_mlock()_, _munlock()_.|
|**Process identifiers and credentials**|   |   |   |
|Process ID|Yes|No||
|Parent process ID|Yes|No||
|Process group ID|Yes|Yes|_setpgid()_.|
|Session ID|Yes|Yes|_setsid()_.|
|Real IDs|Yes|Yes|_setuid()_, _setgid()_, and related calls.|
|Effective and saved set IDs|See notes|Yes|_setuid()_, _setgid()_, and related calls. [Chapter 9](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch09.xhtml#ch09) explains how _exec()_ affects these IDs.|
|Supplementary group IDs|Yes|Yes|_setgroups()_, _initgroups()_.|
|**Files, file I/O, and directories**|   |   |   |
|Open file descriptors|See notes|Yes|_open()_, _close()_, _dup()_, _pipe()_, _socket()_, and so on. File descriptors are preserved across _exec()_ unless marked close-on-exec. Descriptors in child and parent refer to same open file descriptions; see [Section 5.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch05.xhtml#ch05lev1sec04).|
|Close-on-exec flag|Yes (if off)|Yes|_fcntl(F_SETFD)_.|
|File offsets|Yes|Shared|_lseek()_, _read()_, _write()_, _readv()_, _writev()_. Child shares file offsets with parent.|
|Open file status flags|Yes|Shared|_open(), fcntl(F_SETFL)_. Child shares open file status flags with parent.|
|Asynchronous I/O operations|See notes|No|_aio_read()_, _aio_write()_, and related calls. Outstanding operations are canceled during an _exec()_.|
|Directory streams|No|Yes; see notes|_opendir()_, _readdir()_. SUSv3 states that child gets a copy of parent’s directory streams, but these copies may or may not share the directory stream position. On Linux, the directory stream position is not shared.|
|**File system**|   |   |   |
|Current working directory|Yes|Yes|_chdir()_.|
|Root directory|Yes|Yes|_chroot()_.|
|File mode creation mask|Yes|Yes|_umask()_.|
|**Signals**|   |   |   |
|Signal dispositions|See notes|Yes|_signal()_, _sigaction()_. During an _exec()_, signals with dispositions set to default or ignore are unchanged; caught signals revert to their default dispositions. See [Section 27.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch27.xhtml#ch27lev1sec05).|
|Signal mask|Yes|Yes|Signal delivery, _sigprocmask()_, _sigaction()_.|
|Pending signal set|Yes|No|Signal delivery; _raise()_, _kill()_, _sigqueue()_.|
|Alternate signal stack|No|Yes|_sigaltstack()_.|
|**Timers**|   |   |   |
|Interval timers|Yes|No|_setitimer()_.|
|Timers set by _alarm()_|Yes|No|_alarm()_.|
|POSIX timers|No|No|_timer_create()_ and related calls.|
|**POSIX threads**|   |   |   |
|Threads|No|See notes|During _fork()_, only calling thread is replicated in child.|
|Thread cancelability state and type|No|Yes|After an _exec()_, the cancelability type and state are reset to PTHREAD_CANCEL_ENABLE and PTHREAD_CANCEL_DEFERRED, respectively|
|Mutexes and condition variables|No|Yes|See [Section 33.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch33.xhtml#ch33lev1sec03) for details of the treatment of mutexes and other thread resources during _fork()_.|
|**Priority and scheduling**|   |   |   |
|Nice value|Yes|Yes|_nice()_, _setpriority()_.|
|Scheduling policy and priority|Yes|Yes|_sched_setscheduler()_, _sched_setparam()_.|
|**Resources and CPU time**|   |   |   |
|Resource limits|Yes|Yes|_setrlimit()_.|
|Process and child CPU times|Yes|No|As returned by _times()_.|
|Resource usages|Yes|No|As returned by _getrusage()_.|
|**Interprocess communication**|   |   |   |
|System V shared memory segments|No|Yes|_shmat()_, _shmdt()_.|
|POSIX shared memory|No|Yes|_shm_open()_ and related calls.|
|POSIX message queues|No|Yes|_mq_open()_ and related calls. Descriptors in child and parent refer to same open message queue descriptions. A child doesn’t inherit its parent’s message notification registrations.|
|POSIX named semaphores|No|Shared|_sem_open()_ and related calls. Child shares references to same semaphores as parent.|
|POSIX unnamed semaphores|No|See notes|_sem_init()_ and related calls. If semaphores are in a shared memory region, then child shares semaphores with parent; otherwise, child has its own copy of the semaphores.|
|System V semaphore adjustments|Yes|No|_semop()_. See [Section 47.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch47.xhtml#ch47lev1sec08).|
|File locks|Yes|See notes|_flock()_. Child inherits a reference to the same lock as parent.|
|Record locks|See notes|No|_fcntl(F_SETLK)_. Locks are preserved across _exec()_ unless a file descriptor referring to the file is marked close-on-exec; see [Section 55.3.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch55.xhtml#ch55lev2sec07).|
|**Miscellaneous**|   |   |   |
|Locale settings|No|Yes|_setlocale()_. As part of C run-time initialization, the equivalent of _setlocale(LC_ALL, “C”)_ is executed after a new program is execed.|
|Floating-point environment|No|Yes|When a new program is execed, the state of the floating-point environment is reset to the default; see _fenv(3)_.|
|Controlling terminal|Yes|Yes||
|Exit handlers|No|Yes|_atexit()_, _on_exit()_.|
|**Linux-specific**|   |   |   |
|File-system IDs|See notes|Yes|_setfsuid()_, _setfsgid()_. These IDs are also changed any time the corresponding effective IDs are changed.|
|_timerfd_ timers|Yes|See notes|_timerfd_create()_; child inherits file descriptors referring to same timers as parent.|
|Capabilities|See notes|Yes|_capset()_. The handling of capabilities during an _exec()_ is described in [Section 39.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch39.xhtml#ch39lev1sec05).|
|Capability bounding set|Yes|Yes||
|Capabilities _securebits_ flags|See notes|Yes|All _securebits_ flags are preserved during an _exec()_ except SECBIT_KEEP_CAPS, which is always cleared.|
|CPU affinity|Yes|Yes|_sched_setaffinity()_.|
|SCHED_RESET_ON_FORK|Yes|No|See [Section 35.3.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch35.xhtml#ch35lev2sec05).|
|Allowed CPUs|Yes|Yes|See _cpuset(7)_.|
|Allowed memory nodes|Yes|Yes|See _cpuset(7)_.|
|Memory policy|Yes|Yes|See _set_mempolicy(2)_.|
|File leases|Yes|See notes|_fcntl(F_SETLEASE)_. Child inherits a reference to the same lease as parent.|
|Directory change notifications|Yes|No|The _dnotify_ API, available via _fcntl(F_NOTIFY)_.|
|_prctl(PR_SET_DUMPABLE)_|See notes|Yes|During an _exec()_, the PR_SET_DUMPABLE flag is set, unless execing a set-user-ID or set-group-ID program, in which case it is cleared.|
|_prctl(PR_SET_PDEATHSIG)_|Yes|No||
|_prctl(PR_SET_NAME)_|No|Yes||
|oom_adj|Yes|Yes|See [Section 49.9](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch49.xhtml#ch49lev1sec09).|
|coredump_filter|Yes|Yes|See [Section 22.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch22.xhtml#ch22lev1sec01).|

### **28.5 Summary**

When process accounting is enabled, the kernel writes an accounting record to a file for each process that terminates on the system. This record contains statistics on the resources used by the process.

Like _fork()_, the Linux-specific _clone()_ system call creates a new process, but allows finer control over which attributes are shared between the parent and child. This system call is used primarily for implementing threading libraries.

We compared the speed of process creation using _fork()_, _vfork()_, and _clone()_. Although _vfork()_ is faster than _fork()_, the time difference between these system calls is small by comparison with the time required for a child process to do a subsequent _exec()_.

When a child process is created via _fork()_, it inherits copies of (or in some cases shares) certain process attributes from its parent, while other process attributes are not inherited. For example, a child inherits copies of its parent’s file descriptor table and signal dispositions, but doesn’t inherit its parent’s interval timers, record locks, or set of pending signals. Correspondingly, when a process performs an _exec()_, certain process attributes remain unchanged, while others are reset to defaults. For example, the process ID remains the same, file descriptors remain open (unless marked close-on-exec), interval timers are preserved, and pending signals remain pending, but handled signals are reset to their default disposition and shared memory segments are detached.

##### **Further information**

Refer to the sources of further information listed in [Section 24.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch24.xhtml#ch24lev1sec06). [Chapter 17](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch17.xhtml#ch17) of [[Frisch, 2002](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib27)] describes the administration of process accounting, as well as some of the variations across UNIX implementations. [[Bovet & Cesati, 2005](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib09)] describes the implementation of the _clone()_ system call.

### **28.6 Exercise**

**28-1.**   Write a program to see how fast the _fork()_ and _vfork()_ system calls are on your system. Each child process should immediately exit, and the parent should _wait()_ on each child before creating the next. Compare the relative differences for these two system calls with those of [Table 28-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch28.xhtml#ch28table3). The shell built-in command _time_ can be used to measure the execution time of a program.