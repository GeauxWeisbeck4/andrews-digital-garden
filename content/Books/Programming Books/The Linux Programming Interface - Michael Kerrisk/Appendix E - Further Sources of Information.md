---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Appendix E - Further Sources of Information
modified: 2024-11-11T21:38:28-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **E**  
**FURTHER SOURCES OF INFORMATION**

Aside from the material in this book, many other sources of information about Linux system programming are available. This appendix provides a short introduction to some of them.

### **Manual pages**

Manual pages are accessible via the _man_ command. (The command _man man_ describes how to use _man_ to read manual pages.) The manual pages are divided into numbered sections that categorize information as follows:

1. _Programs and shell commands_: commands executed by users at the shell prompt.
    
2. _System calls_: Linux system calls.
    
3. _Library functions_: standard C library functions (as well as many other library functions).
    
4. _Special files_: special files, such as device files.
    
5. _File formats_: formats of files such as the system password (/etc/passwd) and group (/etc/group) files.
    
6. _Games_: games.
    
7. _Overview, conventions, protocols, and miscellany_: overviews of various topics, and various pages on network protocols and sockets programming.
    
8. _System administration commands_: commands that are for use mainly by the superuser.
    

In some cases, there are manual pages in different sections with the same name. For example, there is a section 1 manual page for the _chmod_ command and a section 2 manual page for the _chmod()_ system call. To distinguish manual pages with the same name, we enclose the section number in parentheses after the name—for example, _chmod(1)_ and _chmod(2)_. To display the manual page from a particular section, we can insert the section number into the _man_ command:

$ man 2 chmod

The manual pages for system calls and library functions are divided into a number of parts, which usually include the following:

• _Name_: the name of the function, accompanied by a one-line description. The following command can be used to obtain a list of all manual pages whose one-line description contains the specified string:

$ man -k string

This is useful if we can’t remember or don’t know exactly which manual page we’re looking for.

• _Synopsis_: the C prototype of the function. This identifies the type and order of the function’s arguments, as well as the type of value returned by the function. In most cases, a list of header files precedes the function prototype. These header files define macros and C types needed for use with this function, as well as the function prototype itself, and should be included in a program using this function.

• _Description_: a description of what the function does.

• _Return value_: a description of the range of values returned by the function, including how the function informs the caller of an error.

• _Errors_: a list of the possible _errno_ values that are returned in the event of an error.

• _Conforming to_: a description of the various UNIX standards to which the function conforms. This gives us an idea of how portable this function is to other UNIX implementations and also identifies Linux-specific aspects of the function.

• _Bugs_: a description of things that are broken or that don’t work as they should.

Although some of the later commercial UNIX implementations have preferred more marketable euphemisms, from early times, the UNIX manual pages called a bug a bug. Linux continues the tradition. Sometimes these “bugs” are philosophical, simply describing ways in which things could be improved, or warning about special or unexpected (but otherwise intended) behaviors.

• _Notes_: miscellaneous additional notes on the function.

• _See also_: a list of manual pages for related functions and commands.

The manual pages describing the kernel and _glibc_ APIs are available online at _[http://www.kernel.org/doc/man-pages/](http://www.kernel.org/doc/man-pages/)_.

### **GNU _info_ documents**

Rather than using the traditional manual page format, the GNU project documents much of its software using _info_ documents, which are hyperlinked documents that can be browsed using the _info_ command. A tutorial on the use of _info_ can be obtained using the command _info info_.

Although in many cases the information in manual pages and corresponding _info_ documents is the same, sometimes the _info_ documentation for the C library contains additional information not found in the manual pages or vice versa.

The reasons both manual pages and _info_ documents exist, even though both may contain the same information, are somewhat religious. The GNU project prefers the _info_ user interface, and so provides all documentation via _info_. However, users and programmers on UNIX systems have had a long history of using (and in many cases preferring) manual pages, so there is strong momentum in favor of upholding this format. The manual pages also tend to include more historical information (e.g., information about behavior changes across versions) than do the _info_ documents.

### **The GNU C library (_glibc_) manual**

The GNU C library includes a manual that describes the use of many of the functions in the library. The manual is available at _[http://www.gnu.org/](http://www.gnu.org/)_. It is also provided with most distributions in both HTML format and _info_ format (via the command _info libc_).

### **Books**

An extensive bibliography can be found at the end of this book, but a few books deserve special mention.

At the top of the list are the books by the late W. Richard Stevens. _Advanced Programming in the UNIX Environment_ ([[Stevens, 1992](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib88)]) provides detailed coverage of UNIX system programming, focusing on POSIX, System V, and BSD. A recent revision by Stephen Rago, [[Stevens & Rago, 2005](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib94)] updates the text for modern standards and implementations, and adds coverage of threads and a chapter on network programming. This book is a good place to look for an alternative viewpoint on many of the topics covered in this book. The two-volume _UNIX Network Programming_ ([[Stevens et al., 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib93)], [[Stevens, 1999](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib90)]) provides extremely detailed coverage of network programming and interprocess communication on UNIX systems.

[[Stevens et al., 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib93)] is a revision by Bill Fenner and Andrew Rudoff of [[Stevens, 1998](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib89)], the previous edition of Volume 1 of the _UNIX Network Programming_. While the revised edition covers several new areas, in most cases where we make reference to [[Stevens et al., 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib93)], the same material can also be found in [[Stevens, 1998](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib89)], albeit under different chapter and section numbers.

_Advanced UNIX Programming_ ([[Rochkind, 1985](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib79)]) was a good, brief, and sometimes humorous, introduction to UNIX (System V) programming. It is nowadays available in an updated and extended second edition ([[Rochkind, 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib80)]).

The POSIX threading API is thoroughly described in _Programming with POSIX Threads_ ([[_Butenhof_, 1996](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib10)]).

_Linux and the Unix Philosophy_ ([[Gancarz, 2003](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib30)]) is a brief introduction to the philosophy of application design on Linux and UNIX systems.

Various books provide an introduction to reading and modifying the Linux kernel sources, including _Linux Kernel Development_ ([[Love, 2010](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib59)]) and _Understanding the Linux Kernel_ ([[Bovet & Cesati, 2005](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib09)]).

For more general background on UNIX kernels, _The Design of the UNIX Operating System_ ([[Bach, 1986](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib04)]) remains very readable and contains material relevant to Linux. _UNIX Internals: The New Frontiers_ ([[Vahalia, 1996](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib104)]) surveys kernel internals for more modern UNIX implementations.

For writing Linux device drivers, the essential reference is _Linux Device Drivers_ ([[Corbet et al., 2005](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib16)]).

_Operating Systems: Design and Implementation_ ([[Tanenbaum & Woodhull, 2006](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib101)]) describes operating system implementation using the example of Minix. (See also _[http://www.minix3.org/](http://www.minix3.org/)_.)

### **Source code of existing applications**

Looking at the source code of existing applications can often provide good examples of how to use particular system calls and library functions. On Linux distributions employing the RPM Package Manager, we can find the package that contains a particular program (such as _ls_) as follows:

$ which ls                   Find pathname of ls program  
/bin/ls  
$ rpm -qf /bin/ls            Find out which package created the pathname /bin/ls  
coreutils-5.0.75

The corresponding source code package will have a name similar to the above, but with the suffix .src.rpm. This package will be on the installation media for the distribution or be available for download from the distributor’s web site. Once we obtain the package, we can install it using the _rpm_ command, and then examine the source code, which is typically placed in some directory under /usr/src.

On systems using the Debian package manager, the process is similar. We can determine the package that created a pathname (for the _ls_ program, in this example) using the following command:

$ dpkg -S /bin/ls  
coreutils: /bin/ls

### **The Linux Documentation Project**

The Linux Documentation Project (_[http://www.tldp.org/](http://www.tldp.org/)_) produces freely available documentation on Linux, including HOWTO guides and FAQs (frequently asked questions and answers) on various system administration and programming topics. The site also offers more extensive electronic books on a range of topics.

### **The GNU project**

The GNU project (_[http://www.gnu.org/](http://www.gnu.org/)_) provides an enormous quantity of software source code and associated documentation.

### **Newsgroups**

Usenet newsgroups can often be a good source of answers to specific programming questions. The following newsgroups are of particular interest:

• _comp.unix.programmer_ addresses general UNIX programming questions.

• _comp.os.linux.development.apps_ addresses questions relating to application development specifically on Linux.

• _comp.os.linux.development.system_, the Linux system development newsgroup, focuses on questions about modifying the kernel and developing device drivers and loadable modules.

• _comp.programming.threads_ discusses programming with threads, especially POSIX threads.

• _comp.protocols.tcp-ip_ discusses the TCP/IP networking protocol suite.

FAQs for many Usenet news groups can be found at _[http://www.faqs.org/](http://www.faqs.org/)_.

Before posting a question to a newsgroup, check the FAQ for the group (often posted regularly within the group itself) and to try a web search to find a solution to the question. The _[http://groups.google.com/](http://groups.google.com/)_ web site provides a browser-based interface for searching old Usenet postings.

### **Linux kernel mailing list**

The Linux kernel mailing list (LKML) is the principal broadcast communication medium for the Linux kernel developers. It provides an idea of what’s going on in kernel development, and is a forum for submitting kernel bug reports and patches. (LKML is not a forum for system programming questions.) To subscribe to LKML, send an email message to _[majordomo@vger.kernel.org](mailto:majordomo@vger.kernel.org)_ with the following message body as a single line:

subscribe linux-kernel

For information about the workings of the list server, send a message body containing just the word “help” to the same address.

To send a message to LKML, use the address _[linux-kernel@vger.kernel.org](mailto:linux-kernel@vger.kernel.org)_. The FAQ and pointers to some searchable archives for this mailing list are available at _[http://www.kernel.org/](http://www.kernel.org/)_.

### **Web sites**

The following web sites are of particular interest:

• _[http://www.kernel.org/](http://www.kernel.org/)_, _The Linux Kernel Archives_, contains the source code for all versions of the Linux kernel, past and present.

• _[http://www.lwn.net/](http://www.lwn.net/)_, _Linux Weekly News_, provides daily and weekly columns on various Linux-related topics. A weekly kernel-development column summarizes traffic through LKML.

• _[http://www.kernelnewbies.org/](http://www.kernelnewbies.org/)_, _Linux Kernel Newbies_, is a starting point for programmers who want to learn about and modify the Linux kernel.

• _[http://lxr.linux.no/linux/](http://lxr.linux.no/linux/)_, _Linux Cross-reference_, provides browser access to various versions of the Linux kernel source code. Each identifier in a source file is hyperlinked to make it easy to find the definition and uses of that identifier.

### **The kernel source code**

If none of the preceding sources answer our questions, or if we want to confirm that documented information is true, then we can read the kernel source code. Although parts of the source code can be difficult to understand, reading the code of a particular system call in the Linux kernel source (or a library function in the GNU C library source) can often prove to be a surprisingly quick way to find the answer to a question.

If the Linux kernel source code has been installed on the system, it can usually be found in the directory /usr/src/linux. [Table E-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/app05.xhtml#app05table1) provides summary information about some of the subdirectories under this directory.

**Table E-1:** Subdirectories in the Linux kernel source tree

|**Directory**|**Contents**|
|---|---|
|Documentation|Documentation of various aspects of the kernel|
|arch|Architecture-specific code, organized into subdirectories—for example, alpha, arm, ia64, sparc, and x86|
|drivers|Code for device drivers|
|fs|File system–specific code, organized into subdirectories—for example, btrfs, ext4, proc (the /proc file system), and vfat|
|include|Header files needed by kernel code|
|init|Initialization code for the kernel|
|ipc|Code for System V IPC and POSIX message queues|
|kernel|Code related to processes, program execution, kernel modules, signals, time, and timers|
|lib|General-purpose functions used by various parts of the kernel|
|mm|Memory-management code|
|net|Networking code (TCP/IP, UNIX and Internet domain sockets)|
|scripts|Scripts to configure and build the kernel|