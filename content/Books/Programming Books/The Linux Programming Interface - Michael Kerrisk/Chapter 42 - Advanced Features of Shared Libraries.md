---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 42 - Advanced Features of Shared Libraries
modified: 2024-11-11T19:43:18-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **42**  
**ADVANCED FEATURES OF SHARED LIBRARIES**

The previous chapter covered the fundamentals of shared libraries. This chapter describes a number of advanced features of shared libraries, including the following:

• dynamically loading shared libraries;

• controlling the visibility of symbols defined by a shared library;

• using linker scripts to create versioned symbols;

• using initialization and finalization functions to automatically execute code when a library is loaded and unloaded;

• shared library preloading; and

• using LD_DEBUG to monitor the operation of the dynamic linker.

### **42.1 Dynamically Loaded Libraries**

When an executable starts, the dynamic linker loads all of the shared libraries in the program’s dynamic dependency list. Sometimes, however, it can be useful to load libraries at a later time. For example, a plug-in is loaded only when it is needed. This functionality is provided by an API to the dynamic linker. This API, usually referred to as the _dlopen_ API, originated on Solaris, and much of it is now specified in SUSv3.

The _dlopen_ API enables a program to open a shared library at run time, search for a function by name in that library, and then call the function. A shared library loaded at run time in this way is commonly referred to as a _dynamically loaded library_, and is created in the same way as any other shared library.

The core _dlopen_ API consists of the following functions (all of which are specified in SUSv3):

• The _dlopen()_ function opens a shared library, returning a handle used by subsequent calls.

• The _dlsym()_ function searches a library for a symbol (a string containing the name of a function or variable) and returns its address.

• The _dlclose()_ function closes a library previously opened by _dlopen()_.

• The _dlerror()_ function returns an error-message string, and is used after a failure return from one of the preceding functions.

The _glibc_ implementation also includes a number of related functions, some of which we describe below.

To build programs that use the _dlopen_ API on Linux, we must specify the _–ldl_ option, in order to link against the _libdl_ library.

#### **42.1.1 Opening a Shared Library: _dlopen()_**

The _dlopen()_ function loads the shared library named in _libfilename_ into the calling process’s virtual address space and increments the count of open references to the library.

#include <dlfcn.h>  
  
void *dlopen(const char *libfilename, int flags);

Returns library handle on success, or NULL on error

If _libfilename_ contains a slash (/), _dlopen()_ interprets it as an absolute or relative pathname. Otherwise, the dynamic linker searches for the shared library using the rules described in [Section 41.11](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch41.xhtml#ch41lev1sec11).

On success, _dlopen()_ returns a handle that can be used to refer to the library in subsequent calls to functions in the _dlopen_ API. If an error occurred (e.g., the library couldn’t be found), _dlopen()_ returns NULL.

If the shared library specified by _libfilename_ contains dependencies on other shared libraries, _dlopen()_ also automatically loads those libraries. This procedure occurs recursively if necessary. We refer to the set of such loaded libraries as this library’s _dependency tree_.

It is possible to call _dlopen()_ multiple times on the same library file. The library is loaded into memory only once (by the initial call), and all calls return the same handle value. However, the _dlopen_ API maintains a reference count for each library handle. This count is incremented by each call to _dlopen()_ and decremented by each call to _dlclose()_; only when the count reaches 0 does _dlclose()_ unload the library from memory.

The _flags_ argument is a bit mask that must include exactly one of the constants RTLD_LAZY or RTLD_NOW, with the following meanings:

RTLD_LAZY

Undefined function symbols in the library should be resolved only as the code is executed. If a piece of code requiring a particular symbol is not executed, that symbol is never resolved. Lazy resolution is performed only for function references; references to variables are always resolved immediately. Specifying the RTLD_LAZY flag provides behavior that corresponds to the normal operation of the dynamic linker when loading the shared libraries identified in an executable’s dynamic dependency list.

RTLD_NOW

All undefined symbols in the library should be immediately resolved before _dlopen()_ completes, regardless of whether they will ever be required. As a consequence, opening the library is slower, but any potential undefined function symbol errors are detected immediately instead of at some later time. This can be useful when debugging an application, or simply to ensure that an application fails immediately on an unresolved symbol, rather than doing so only after executing for a long time.

By setting the environment variable LD_BIND_NOW to a nonempty string, we can force the dynamic linker to immediately resolve all symbols (i.e., like RTLD_NOW) when loading the shared libraries identified in an executable’s dynamic dependency list. This environment variable is effective in _glibc_ 2.1.1 and later. Setting LD_BIND_NOW overrides the effect of the _dlopen()_ RTLD_LAZY flag.

It is also possible to include further values in _flags_. The following flags are specified in SUSv3:

RTLD_GLOBAL

Symbols in this library and its dependency tree are made available for satisfying subsequent symbol resolution operations in other shared libraries (and in the main program) and also for lookups via _dlsym()_.

RTLD_LOCAL

This is the converse of RTLD_GLOBAL and the default if neither constant is specified. It specifies that symbols in this library and its dependency tree are not available for satisfying subsequent symbol resolution operations in other shared libraries (and in the main program).

SUSv3 doesn’t specify a default if neither RTLD_GLOBAL nor RTLD_LOCAL is specified. Most UNIX implementations assume the same default (RTLD_LOCAL) as Linux, but a few assume a default of RTLD_GLOBAL.

Linux also supports a number of flags that are not specified in SUSv3:

RTLD_NODELETE (since _glibc_ 2.2)

Don’t unload the library during a _dlclose()_, even if the reference count falls to 0. This means that the library’s static variables are not reinitialized if the library is later reloaded by _dlopen()_. (We can achieve a similar effect for libraries loaded automatically by the dynamic linker by specifying the _gcc –Wl,–znodelete_ option when creating the library.)

RTLD_NOLOAD (since _glibc_ 2.2)

Don’t load the library. This serves two purposes. First, we can use this flag to check if a particular library is currently loaded as part of the process’s address space. If it is, _dlopen()_ returns the library’s handle; if it is not, _dlopen()_ returns NULL. Second, we can use this flag to “promote” the _flags_ of an already loaded library. For example, we can specify RTLD_NOLOAD | RTLD_GLOBAL in _flags_ when using _dlopen()_ on a library previously opened with RTLD_LOCAL.

RTLD_DEEPBIND (since _glibc_ 2.3.4)

When resolving symbol references made by this library, search for definitions in the library before searching for definitions in libraries that have already been loaded. This allows a library to be self-contained, using its own symbol definitions in preference to global symbols with the same name defined in other shared libraries that have already been loaded. (This is similar to the effect of the _–Bsymbolic_ linker option described in [Section 41.12](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch41.xhtml#ch41lev1sec12).)

The RTLD_NODELETE and RTLD_NOLOAD flags are also implemented in the Solaris _dlopen_ API, but are available on few other UNIX implementations. The RTLD_DEEPBIND flag is Linux-specific.

As a special case, we can specify _libfilename_ as NULL. This causes _dlopen()_ to return a handle for the main program. (SUSv3 refers to this as a handle for the “global symbol object.”) Specifying this handle in a subsequent call to _dlsym()_ causes the requested symbol to be sought in the main program, followed by all shared libraries loaded at program startup, and then all libraries dynamically loaded with the RTLD_GLOBAL flag.

#### **42.1.2 Diagnosing Errors: _dlerror()_**

If we receive an error return from _dlopen()_ or one of the other functions in the _dlopen_ API, we can use _dlerror()_ to obtain a pointer to a string that indicates the cause of the error.

#include <dlfcn.h>  
  
const char *dlerror(void);

Returns pointer to error-diagnostic string, or NULL if no error has occurred since previous call to _dlerror()_

The _dlerror()_ function returns NULL if no error has occurred since the last call to _dlerror()_. We’ll see how this is useful in the next section.

#### **42.1.3 Obtaining the Address of a Symbol: _dlsym()_**

The _dlsym()_ function searches for the named _symbol_ (a function or variable) in the library referred to by _handle_ and in the libraries in that library’s dependency tree.

#include <dlfcn.h>  
  
void *dlsym(void *handle, char *symbol);

Returns address of _symbol_, or NULL if _symbol_ is not found

If _symbol_ is found, _dlsym()_ returns its address; otherwise, _dlsym()_ returns NULL. The _handle_ argument is normally a library handle returned by a previous call to _dlopen()_. Alternatively, it may be one of the so-called pseudohandles described below.

A related function, _dlvsym(handle, symbol, version)_, is similar to _dlsym()_, but can be used to search a symbol-versioned library for a symbol definition whose version matches the string specified in _version_. (We describe symbol versioning in [Section 42.3.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch42.xhtml#ch42lev2sec08).) The _GNU_SOURCE feature test macro must be defined in order to obtain the declaration of this function from <dlfcn.h>.

The value of a symbol returned by _dlsym()_ may be NULL, which is indistinguishable from the “symbol not found” return. In order to differentiate the two possibilities, we must call _dlerror()_ beforehand (to make sure that any previously held error string is cleared) and then if, after the call to _dlsym()_, _dlerror()_ returns a non-NULL value, we know that an error occurred.

If _symbol_ is the name of a variable, then we can assign the return value of _dlsym()_ to an appropriate pointer type, and obtain the value of the variable by dereferencing the pointer:

int *ip;  
  
ip = (int *) dlsym(handle, symbol);  
if (ip != NULL)  
    printf("Value is %d\n", *ip);

If _symbol_ is the name of a function, then the pointer returned by _dlsym()_ can be used to call the function. We can store the value returned by _dlsym()_ in a pointer of the appropriate type, such as the following:

int (*funcp)(int);         /* Pointer to a function taking an integer  
                              argument and returning an integer */

However, we can’t simply assign the result of _dlsym()_ to such a pointer, as in the following example:

funcp = dlsym(handle, symbol);

The reason is that the ISO C standard leaves the results of casting between function pointers and _void *_ undefined. The solution is to use the following (somewhat clumsy) cast:

*(void **) (&funcp) = dlsym(handle, symbol);

Having used _dlsym()_ to obtain a pointer to the function, we can then call the function using the usual C syntax for dereferencing function pointers:

res = (*funcp)(somearg);

Instead of the _*(void **)_ syntax shown above, one might consider using the following seemingly equivalent code when assigning the return value of _dlsym()_:

(void *) funcp = dlsym(handle, symbol);

However, for this code, _gcc –pedantic_ warns that “ANSI C forbids the use of cast expressions as lvalues.” The _*(void **)_ syntax doesn’t incur this warning because we are not assigning to the cast expression itself, but rather to the result of dereferencing it.

On many UNIX implementations, we can use casts such as the following to eliminate warnings from the C compiler:

funcp = (int (*) (int)) dlsym(handle, symbol);

However, the specification of _dlsym()_ in SUSv3 _Technical Corrigendum Number 1_ notes that the ISO C standard nevertheless requires compilers to generate a warning for such a conversion, and proposes the _*(void **)_ syntax shown above.

SUSv3 TC1 noted that because of the need for the _*(void **)_ syntax, a future version of the standard may define separate _dlsym()_-like APIs for handling data and function pointers. However, SUSv4 contains no changes with respect to this point.

##### **Using library pseudohandles with _dlsym()_**

Instead of specifying a library handle returned by a call to _dlopen()_, either of the following _pseudohandles_ may be specified as the _handle_ argument for _dlsym()_:

RTLD_DEFAULT

Search for _symbol_ starting with the main program, and then proceeding in order through the list of all shared libraries loaded, including those libraries dynamically loaded by _dlopen()_ with the RTLD_GLOBAL flag. This corresponds to the default search model employed by the dynamic linker.

RTLD_NEXT

Search for _symbol_ in shared libraries loaded after the one invoking _dlsym()_. This is useful when creating a wrapper function with the same name as a function defined elsewhere. For example, in our main program, we may define our own version of _malloc()_ (which perhaps does some bookkeeping of memory allocation), and this function can invoke the real _malloc()_ by first obtaining its address via the call _func = dlsym(RTLD_NEXT, “malloc”)_.

The pseudohandle values listed above are not required by SUSv3 (which nevertheless reserves them for future use), and are not available on all UNIX implementations. In order to get the definitions of these constants from <dlfcn.h>, we must define the _GNU_SOURCE feature test macro.

##### **Example program**

[Listing 42-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch42.xhtml#ch42ex1) demonstrates the use of the _dlopen_ API. This program takes two command-line arguments: the name of a shared library to load and the name of a function to execute within that library. The following examples demonstrate the use of this program:

$ ./dynload ./libdemo.so.1 x1  
Called mod1-x1  
$ LD_LIBRARY_PATH=. ./dynload libdemo.so.1 x1  
Called mod1-x1

In the first of the above commands, _dlopen()_ notes that the library path includes a slash and thus interprets it as a relative pathname (in this case, to a library in the current working directory). In the second command, we specify a library search path in LD_LIBRARY_PATH. This search path is interpreted according to the usual rules of the dynamic linker (in this case, likewise to find the library in the current working directory).

**Listing 42-1:** Using the _dlopen_ API

_________________________________________________________ shlibs/dynload.c  
  
#include <dlfcn.h>  
#include "tlpi_hdr.h"  
  
int  
main(int argc, char *argv[])  
{  
    void *libHandle;            /* Handle for shared library */  
    void (*funcp)(void);        /* Pointer to function with no arguments */  
    const char *err;  
  
    if (argc != 3 || strcmp(argv[1], "--help") == 0)  
        usageErr("%s lib-path func-name\n", argv[0]);  
  
    /* Load the shared library and get a handle for later use */  
  
    libHandle = dlopen(argv[1], RTLD_LAZY);  
    if (libHandle == NULL)  
        fatal("dlopen: %s", dlerror());  
  
    /* Search library for symbol named in argv[2] */  
  
    (void) dlerror();                           /* Clear dlerror() */  
    *(void **) (&funcp) = dlsym(libHandle, argv[2]);  
    err = dlerror();  
    if (err != NULL)  
        fatal("dlsym: %s", err);  
  
    /* Try calling the address returned by dlsym() as a function  
       that takes no arguments. */  
  
    (*funcp)();  
  
    dlclose(libHandle);                         /* Close the library */  
  
    exit(EXIT_SUCCESS);  
}  
_________________________________________________________ shlibs/dynload.c

#### **42.1.4 Closing a Shared Library: _dlclose()_**

The _dlclose()_ function closes a library.

#include <dlfcn.h>  
  
int dlclose(void *handle);

Returns 0 on success, or –1 on error

The _dlclose()_ function decrements the system’s counter of open references to the library referred to by _handle_. If this reference count falls to 0, and no symbols in the library are required by other libraries, then the library is unloaded. This procedure is also (recursively) performed for the libraries in this library’s dependency tree. An implicit _dlclose()_ of all libraries is performed on normal process termination via _exit()_.

From _glibc_ 2.2.3 onward, a function within a shared library can use _atexit()_ (or _on_exit()_) to establish a function that is called automatically when the library is unloaded.

#### **42.1.5 Obtaining Information About Loaded Symbols: _dladdr()_**

Given an address in _addr_ (typically, one obtained by an earlier call to _dlsym()_), _dladdr()_ returns a structure containing information about that address.

#define _GNU_SOURCE  
#include <dlfcn.h>  
  
int dladdr(const void *addr, Dl_info *info);

Returns nonzero value if _addr_ was found in a shared library, otherwise 0

The _info_ argument is a pointer to a caller-allocated structure that has the following form:

typedef struct {  
    const char *dli_fname;        /* Pathname of shared library  
                                     containing 'addr' */  
    void       *dli_fbase;        /* Base address at which shared  
                                     library is loaded */  
    const char *dli_sname;        /* Name of symbol whose definition  
                                     overlaps 'addr' */  
    void       *dli_saddr;        /* Actual value of the symbol  
                                     returned in 'dli_sname' */  
} Dl_info;

The first two fields of the _Dl_info_ structure specify the pathname and run-time base address of the shared library containing the address specified in _addr_. The last two fields return information about that address. Assuming that _addr_ points to the exact address of a symbol in the shared library, then _dli_saddr_ returns the same value as was passed in _addr_.

SUSv3 doesn’t specify _dladdr()_, and this function is not available on all UNIX implementations.

#### **42.1.6 Accessing Symbols in the Main Program**

Suppose that we use _dlopen()_ to dynamically load a shared library, use _dlsym()_ to obtain the address of a function _x()_ from that library, and then call _x()_. If _x()_ in turn calls a function _y()_, then _y()_ would normally be sought in one of the shared libraries loaded by the program.

Sometimes, it is desirable instead to have _x()_ invoke an implementation of _y()_ in the main program. (This is similar to a callback mechanism.) In order to do this, we must make the (global-scope) symbols in the main program available to the dynamic linker, by linking the program using the _––export–dynamic_ linker option:

$ gcc -Wl,--export-dynamic main.c    (plus further options and arguments)

Equivalently, we can write the following:

$ gcc -export-dynamic main.c

Using either of these options allows a dynamically loaded library to access global symbols in the main program.

The _gcc –rdynamic_ option and the _gcc –Wl,–E_ option are further synonyms for _–Wl,––export–dynamic_.

### **42.2 Controlling Symbol Visibility**

A well-designed shared library should make visible only those symbols (functions and variables) that form part of its specified application binary interface (ABI). The reasons for this are as follows:

• If the shared library designer accidentally exports unspecified interfaces, then authors of applications that use the library may choose to employ these interfaces. This creates a compatibility problem for future upgrades of the shared library. The library developer expects to be able to change or remove any interfaces other than those in the documented ABI, while the library user expects to continue using the same interfaces (with the same semantics) that they currently employ.

• During run-time symbol resolution, any symbols that are exported by a shared library might interpose definitions that are provided in other shared libraries ([Section 41.12](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch41.xhtml#ch41lev1sec12)).

• Exporting unnecessary symbols increases the size of the dynamic symbol table that must be loaded at run time.

All of these problems can be minimized or avoided altogether if the library designer ensures that only the symbols required by the library’s specified ABI are exported. The following techniques can be used to control the export of symbols:

• In a C program, we can use the static keyword to make a symbol private to a source-code module, thus rendering it unavailable for binding by other object files.

As well as making a symbol private to a source-code module, the static keyword also has a converse effect. If a symbol is marked as static, then all references to the symbol in the same source file will be bound to that definition of the symbol. Consequently, these references won’t be subject to run-time interposition by definitions from other shared libraries (in the manner described in [Section 41.12](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch41.xhtml#ch41lev1sec12)). This effect of the static keyword is similar to the _–Bsymbolic_ linker option described in [Section 41.12](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch41.xhtml#ch41lev1sec12), with the difference that the static keyword affects a single symbol within a single source file.

• The GNU C complier, _gcc_, provides a compiler-specific attribute declaration that performs a similar task to the static keyword:

void  
__attribute__ ((visibility("hidden")))  
func(void) {  
    /* Code */  
}

Whereas the static keyword limits the visibility of a symbol to a single source code file, the hidden attribute makes the symbol available across all source code files that compose the shared library, but prevents it from being visible outside the library.

As with the static keyword, the hidden attribute also has the converse effect of preventing symbol interposition at run time.

• Version scripts ([Section 42.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch42.xhtml#ch42lev1sec03)) can be used to precisely control symbol visibility and to select the version of a symbol to which a reference is bound.

• When dynamically loading a shared library ([Section 42.1.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch42.xhtml#ch42lev2sec01)), the _dlopen()_ RTLD_GLOBAL flag can be used to specify that the symbols defined by the library should be made available for binding by subsequently loaded libraries, and the _––export–dynamic_ linker option ([Section 42.1.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch42.xhtml#ch42lev2sec06)) can be used to make the global symbols of the main program available to dynamically loaded libraries.

For further details on the topic of symbol visibility, see [[Drepper, 2004 (b)](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib21)].

### **42.3 Linker Version Scripts**

A _version script_ is a text file containing instructions for the linker, _ld_. In order to use a version script, we must specify the _––version–script_ linker option:

$ gcc -Wl,--version-script,myscriptfile.map ...

Version scripts are commonly (but not universally) identified using the extension .map.

The following sections describe some uses of version scripts.

#### **42.3.1 Controlling Symbol Visibility with Version Scripts**

One use of version scripts is to control the visibility of symbols that might otherwise accidentally be made global (i.e., visible to applications linking against the library). As a simple example, suppose that we are building a shared library from the three source files vis_comm.c, vis_f1.c, and vis_f2.c, which respectively define the functions _vis_comm()_, _vis_f1()_, and _vis_f2()_. The _vis_comm()_ function is called by _vis_f1()_ and _vis_f2()_, but is not intended for direct use by applications linked against the library. Suppose we build the shared library in the usual way:

$ gcc -g -c -fPIC -Wall vis_comm.c vis_f1.c vis_f2.c  
$ gcc -g -shared -o vis.so vis_comm.o vis_f1.o vis_f2.o

If we use the following _readelf_ command to list the dynamic symbols exported by the library, we see the following:

$ readelf --syms --use-dynamic vis.so | grep vis_  
   30  12: 00000790    59    FUNC GLOBAL DEFAULT  10 vis_f1  
   25  13: 000007d0    73    FUNC GLOBAL DEFAULT  10 vis_f2  
   27  16: 00000770    20    FUNC GLOBAL DEFAULT  10 vis_comm

This shared library exported three symbols: _vis_comm()_, _vis_f1()_, and _vis_f2()_. However, we would like to ensure that only the symbols _vis_f1()_ and _vis_f2()_ are exported by the library. We can achieve this result using the following version script:

$ cat vis.map  
VER_1 {  
    global:  
        vis_f1;  
        vis_f2;  
    local:  
        *;  
};

The identifier _VER_1_ is an example of a _version tag_. As we’ll see in the discussion of symbol versioning in [Section 42.3.2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch42.xhtml#ch42lev2sec08), a version script may contain multiple _version nodes_, each grouped within braces ({}) and prefixed with a unique version tag. If we are using a version script only for the purpose of controlling symbol visibility, then the version tag is redundant; nevertheless, older versions of _ld_ required it. Modern versions of _ld_ allow the version tag to be omitted; in this case, the version node is said to have an anonymous version tag, and no other version nodes may be present in the script.

Within the version node, the global keyword begins a semicolon-separated list of symbols that are made visible outside the library. The local keyword begins a list of symbols that are to be hidden from the outside world. The asterisk (*) here illustrates the fact that we can use wildcard patterns in these symbol specifications. The wildcard characters are the same as those used for shell filename matching—for example, * and ?. (See the _glob(7)_ manual page for further details.) In this example, using an asterisk for the local specification says that everything that wasn’t explicitly declared global is hidden. If we did not say this, then _vis_comm()_ would still be visible, since the default is to make C global symbols visible outside the shared library.

We can then build our shared library using the version script as follows:

$ gcc -g -c -fPIC -Wall vis_comm.c vis_f1.c vis_f2.c  
$ gcc -g -shared -o vis.so vis_comm.o vis_f1.o vis_f2.o \  
        -Wl,--version-script,vis.map

Using _readelf_ once more shows that _vis_comm()_ is no longer externally visible:

$ readelf --syms --use-dynamic vis.so | grep vis_  
   25   0: 00000730    73    FUNC GLOBAL DEFAULT  11 vis_f2  
   29  16: 000006f0    59    FUNC GLOBAL DEFAULT  11 vis_f1

#### **42.3.2 Symbol Versioning**

Symbol versioning allows a single shared library to provide multiple versions of the same function. Each program uses the version of the function that was current when the program was (statically) linked against the shared library. As a result, we can make an incompatible change to a shared library without needing to increase the library’s major version number. Carried to an extreme, symbol versioning can replace the traditional shared library major and minor versioning scheme. Symbol versioning is used in this manner in _glibc_ 2.1 and later, so that all versions of _glibc_ from 2.0 onward are supported within a single major library version (libc.so.6).

We demonstrate the use of symbol versioning with a simple example. We begin by creating the first version of a shared library using a version script:

$ cat sv_lib_v1.c  
#include <stdio.h>  
  
void xyz(void) { printf("v1 xyz\n"); }  
$ cat sv_v1.map  
VER_1 {  
        global: xyz;  
        local:  *;      # Hide all other symbols  
};  
$ gcc -g -c -fPIC -Wall sv_lib_v1.c  
$ gcc -g -shared -o libsv.so sv_lib_v1.o -Wl,--version-script,sv_v1.map

Within a version script, the hash character (#) starts a comment.

(To keep the example simple, we avoid the use of explicit library sonames and library major version numbers.)

At this stage, our version script, sv_v1.map, serves only to control the visibility of the shared library’s symbols; _xyz()_ is exported, but all other symbols (of which there are none in this small example) are hidden. Next, we create a program, _p1_, which makes use of this library:

$ cat sv_prog.c  
#include <stdlib.h>  
  
int  
main(int argc, char *argv[])  
{  
    void xyz(void);  
  
    xyz();  
  
    exit(EXIT_SUCCESS);  
}  
$ gcc -g -o p1 sv_prog.c libsv.so

When we run this program, we see the expected result:

$ LD_LIBRARY_PATH=. ./p1  
v1 xyz

Now, suppose that we want to modify the definition of _xyz()_ within our library, while still ensuring that program _p1_ continues to use the old version of this function. To do this, we must define two versions of _xyz()_ within our library:

$ cat sv_lib_v2.c  
#include <stdio.h>  
  
__asm__(".symver xyz_old,xyz@VER_1");  
__asm__(".symver xyz_new,xyz@@VER_2");  
  
void xyz_old(void) { printf("v1 xyz\n"); }  
  
void xyz_new(void) { printf("v2 xyz\n"); }  
  
void pqr(void) { printf("v2 pqr\n"); }

Our two versions of _xyz()_ are provided by the functions _xyz_old()_ and _xyz_new()_. The _xyz_old()_ function corresponds to our original definition of _xyz()_, which is the one that should continue to be used by program _p1_. The _xyz_new()_ function provides the definition of _xyz()_ to be used by programs linking against the new version of the library.

The two .symver assembler directives are the glue that ties these two functions to different version tags in the modified version script (shown in a moment) that we use to create the new version of the shared library. The first of these directives says that _xyz_old()_ is the implementation of _xyz()_ to be used for applications linked against version tag _VER_1_ (i.e., program _p1_ in our example), and the second says that _xyz_new()_ is the implementation of _xyz()_ to be used by applications linked against version tag _VER_2_.

The use of @@ rather than @ in the second .symver directive indicates that this is the default definition of _xyz()_ to which applications should bind when statically linked against this shared library. Exactly one of the .symver directives for a symbol should be marked using @@.

The corresponding version script for our modified library is as follows:

$ cat sv_v2.map  
VER_1 {  
        global: xyz;  
        local:  *;      # Hide all other symbols  
};  
  
VER_2 {  
        global: pqr;  
} VER_1;

This version script provides a new version tag, _VER_2_, which depends on the tag _VER_1_. This dependency is indicated by the following line:

} VER_1;

Version tag dependencies indicate the relationships between successive library versions. The dependencies thus help human readers understand the relationship between library versions. However, semantically, the dependencies have no effect on the behavior of either the static linker or the dynamic linker.

Dependencies can be chained, so that we could have another version node tagged _VER_3_, which depended on _VER_2_, and so on.

The version tag names have no meanings in themselves. Their relationship with one another is determined only by the specified version dependencies, and we chose the names _VER_1_ and _VER_2_ merely to be suggestive of these relationships. To assist maintenance, recommended practice is to use version tags that include the package name and a version number. For example, _glibc_ uses version tags with names such as _GLIBC_2.0_, _GLIBC_2.1_, and so on.

The _VER_2_ node specifies that the new function _pqr()_ is exported as a public symbol, tagged with the version _VER_2_. When a symbol is marked as global in one version node, it is also implicitly marked as global for all other nodes (except those that explicitly declare it as local). Thus, _xyz()_ with the tag _VER_2_ is also exported as a public symbol.

Note that if we omitted the local specification in the _VER_1_ node, then _xyz_old()_ and _xyz_new()_ would also be exported from the library as public symbols (which is typically not what we want).

We now build the new version of our library in the usual way:

$ gcc -g -c -fPIC -Wall sv_lib_v2.c  
$ gcc -g -shared -o libsv.so sv_lib_v2.o -Wl,--version-script,sv_v2.map

Now we can create a new program, _p2_, which uses the new definition of _xyz()_, while program _p1_ uses the old version of _xyz()_.

$ gcc -g -o p2 sv_prog.c libsv.so  
$ LD_LIBRARY_PATH=. ./p2  
v2 xyz                                     Uses xyz@VER_2  
$ LD_LIBRARY_PATH=. ./p1  
v1 xyz                                     Uses xyz@VER_1

The version tag dependencies of an executable are recorded at static link time. We can use _objdump –t_ to display the symbol tables of each executable, thus showing the different version tag dependencies of each program:

$ objdump -t p1 | grep xyz  
08048380       F *UND*  0000002e             xyz@@VER_1  
$ objdump -t p2 | grep xyz  
080483a0       F *UND*  0000002e             xyz@@VER_2

We can also use _readelf –s_ to obtain similar information.

Further information about symbol versioning can be found using the command _info ld scripts version_ and at _[http://people.redhat.com/drepper/symbol-versioning](http://people.redhat.com/drepper/symbol-versioning)_.

### **42.4 Initialization and Finalization Functions**

It is possible to define one or more functions that are executed automatically when a shared library is loaded and unloaded. This allows us to perform initialization and finalization actions when working with shared libraries. Initialization and finalization functions are executed regardless of whether the library is loaded automatically or loaded explicitly using the _dlopen_ interface ([Section 42.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch42.xhtml#ch42lev1sec01)).

Initialization and finalization functions are defined using the _gcc_ constructor and destructor attributes. Each function that is to be executed when the library is loaded should be defined as follows:

void __attribute__ ((constructor)) some_name_load(void)  
{  
    /* Initialization code */  
}

Unload functions are similarly defined:

void __attribute__ ((destructor)) some_name_unload(void)  
{  
    /* Finalization code */  
}

The function names _some_name_load()_ and _some_name_unload()_ can be replaced by any desired names.

It is also possible to use the _gcc_ constructor and destructor attributes to create initialization and finalization functions in a main program.

##### **The __init()_ and __fini()_ functions**

An older technique for shared library initialization and finalization is to create two functions, __init()_ and __fini()_, as part of the library. The _void _init(void)_ function contains code that is to be executed when the library is first loaded by a process. The _void _fini(void)_ function contains code that is to be executed when the library is unloaded.

If we create __init()_ and __fini()_ functions, then we must specify the _gcc –nostartfiles_ option when building the shared library, in order to prevent the linker from including default versions of these functions. (Using the _–Wl,–init_ and _–Wl,–fini_ linker options, we can choose alternative names for these two functions if desired.)

Use of __init()_ and __fini()_ is now considered obsolete in favor of the _gcc_ constructor and destructor attributes, which, among other advantages, allow us to define multiple initialization and finalization functions.

### **42.5 Preloading Shared Libraries**

For testing purposes, it can sometimes be useful to selectively override functions (and other symbols) that would normally be found by the dynamic linker using the rules described in [Section 41.11](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch41.xhtml#ch41lev1sec11). To do this, we can define the environment variable LD_PRELOAD as a string consisting of space-separated or colon-separated names of shared libraries that should be loaded before any other shared libraries. Since these libraries are loaded first, any functions they define will automatically be used if required by the executable, thus overriding any other functions of the same name that the dynamic linker would otherwise have searched for. For example, suppose that we have a program that calls functions _x1()_ and _x2()_, defined in our _libdemo_ library. When we run this program, we see the following output:

$ ./prog  
Called mod1-x1 DEMO  
Called mod2-x2 DEMO

(In this example, we assume that the shared library is in one of the standard directories, and thus we don’t need to use the LD_LIBRARY_PATH environment variable.)

We could selectively override the function _x1()_ by creating another shared library, libalt.so, which contains a different definition of _x1()_. Preloading this library when running the program would result in the following:

$ LD_PRELOAD=libalt.so ./prog  
Called mod1-x1 ALT  
Called mod2-x2 DEMO

Here, we see that the version of _x1()_ defined in libalt.so is invoked, but that the call to _x2()_, for which no definition is provided in libalt.so, results in the invocation of the _x2()_ function defined in libdemo.so.

The LD_PRELOAD environment variable controls preloading on a per-process basis. Alternatively, the file /etc/ld.so.preload, which lists libraries separated by white space, can be used to perform the same task on a system-wide basis. (Libraries specified by LD_PRELOAD are loaded before those specified in /etc/ld.so.preload.)

For security reasons, set-user-ID and set-group-ID programs ignore LD_PRELOAD.

### **42.6 Monitoring the Dynamic Linker:** LD_DEBUG

Sometimes, it is useful to monitor the operation of the dynamic linker in order to know, for example, where it is searching for libraries. We can use the LD_DEBUG environment variable to do this. By setting this variable to one (or more) of a set of standard keywords, we can obtain various kinds of tracing information from the dynamic linker.

If we assign the value _help_ to LD_DEBUG, the dynamic linker displays help information about LD_DEBUG, and the specified command is _not_ executed:

$ LD_DEBUG=help date  
Valid options for the LD_DEBUG environment variable are:  
  
  libs       display library search paths  
  reloc      display relocation processing  
  files      display progress for input file  
  symbols    display symbol table processing  
  bindings   display information about symbol binding  
  versions   display version dependencies  
  all        all previous options combined  
  statistics display relocation statistics  
  unused     determine unused DSOs  
  help       display this help message and exit  
  
To direct the debugging output into a file instead of standard output  
a filename can be specified using the LD_DEBUG_OUTPUT environment variable.

The following example shows an abridged version of the output provided when we request tracing of information about library searches:

$ LD_DEBUG=libs date  
     10687:     find library=librt.so.1 [0]; searching  
     10687:      search cache=/etc/ld.so.cache  
     10687:       trying file=/lib/librt.so.1  
     10687:     find library=libc.so.6 [0]; searching  
     10687:      search cache=/etc/ld.so.cache  
     10687:       trying file=/lib/libc.so.6  
     10687:     find library=libpthread.so.0 [0]; searching  
     10687:      search cache=/etc/ld.so.cache  
     10687:       trying file=/lib/libpthread.so.0  
     10687:     calling init: /lib/libpthread.so.0  
     10687:     calling init: /lib/libc.so.6  
     10687:     calling init: /lib/librt.so.1  
     10687:     initialize program: date  
     10687:     transferring control: date  
Tue Dec 28 17:26:56 CEST 2010  
     10687:     calling fini: date [0]  
     10687:     calling fini: /lib/librt.so.1 [0]  
     10687:     calling fini: /lib/libpthread.so.0 [0]  
     10687:     calling fini: /lib/libc.so.6 [0]

The value 10687 displayed at the start of each line is the process ID of the process being traced. This is useful if we are monitoring several processes (e.g., parent and child).

By default, LD_DEBUG output is written to standard error, but we can direct it elsewhere by assigning a pathname to the LD_DEBUG_OUTPUT environment variable.

If desired, we can assign multiple options to LD_DEBUG by separating them with commas, colons, or (if the string is quoted) spaces. The output of the _symbols_ option (which traces symbol resolution by the dynamic linker) is particularly voluminous.

LD_DEBUG is effective both for libraries implicitly loaded by the dynamic linker and for libraries dynamically loaded by _dlopen()_.

For security reasons, LD_DEBUG is (since _glibc_ 2.2.5) ignored in set-user-ID and set-group-ID programs.

### **42.7 Summary**

The dynamic linker provides the _dlopen_ API, which allows programs to explicitly load additional shared libraries at run time. This allows programs to implement plug-in functionality.

An important aspect of shared library design is controlling symbol visibility, so that the library exports only those symbols (functions and variables) that should actually be used by programs linked against the library. We looked at a range of techniques that can be used to control symbol visibility. Among these techniques was the use of version scripts, which provide fine-grained control of symbol visibility.

We also showed how version scripts can be used to implement a scheme that allows a single shared library to export multiple definitions of a symbol for use by different applications linked against the library. (Each application uses the definition that was current when the application was statically linked against the library.) This technique provides an alternative to the traditional library versioning approach of using major and minor version numbers in the shared library real name.

Defining initialization and finalization functions within a shared library allows us to automatically execute code when the library is loaded and unloaded.

The LD_PRELOAD environment variable allows us to preload shared libraries. Using this mechanism, we can selectively override functions and other symbols that the dynamic linker would normally find in other shared libraries.

We can assign various values to the LD_DEBUG environment variable in order to monitor the operation of the dynamic linker.

##### **Further information**

Refer to the sources of further information listed in [Section 41.14](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch41.xhtml#ch41lev1sec14).

### **42.8 Exercises**

**42-1.**   Write a program to verify that if a library is closed with _dlclose()_, it is not unloaded if any of its symbols are used by another library.

**42-2.**   Add a _dladdr()_ call to the program in [Listing 42-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch42.xhtml#ch42ex1) (dynload.c) in order to retrieve information about the address returned by _dlsym()_. Print out the values of the fields of the returned _Dl_info_ structure, and verify that they are as expected.