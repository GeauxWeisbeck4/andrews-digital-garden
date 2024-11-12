---
id: 01JBM8TMK0BXTWPN7RSKEREABH
title: Chapter 59 - Sockets - Internet Domains
modified: 2024-11-11T20:35:38-05:00
tags:
  - systems-programming
  - linux
  - books
  - programming
---
## **59**  
**SOCKETS: INTERNET DOMAINS**

Having looked at generic sockets concepts and the TCP/IP protocol suite in previous chapters, we are now ready in this chapter to look at programming with sockets in the IPv4 (AF_INET) and IPv6 (AF_INET6) domains.

As noted in [Chapter 58](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch58.xhtml#ch58), Internet domain socket addresses consist of an IP address and a port number. Although computers use binary representations of IP addresses and port numbers, humans are much better at dealing with names than with numbers. Therefore, we describe the techniques used to identify host computers and ports using names. We also examine the use of library functions to obtain the IP address(es) for a particular hostname and the port number that corresponds to a particular service name. Our discussion of hostnames includes a description of the Domain Name System (DNS), which implements a distributed database that maps hostnames to IP addresses and vice versa.

### **59.1 Internet Domain Sockets**

Internet domain stream sockets are implemented on top of TCP. They provide a reliable, bidirectional, byte-stream communication channel.

Internet domain datagram sockets are implemented on top of UDP. UDP sockets are similar to their UNIX domain counterparts, but note the following differences:

• UNIX domain datagram sockets are reliable, but UDP sockets are not—datagrams may be lost, duplicated, or arrive in a different order from that in which they were sent.

• Sending on a UNIX domain datagram socket will block if the queue of data for the receiving socket is full. By contrast, with UDP, if the incoming datagram would overflow the receiver’s queue, then the datagram is silently dropped.

### **59.2 Network Byte Order**

IP addresses and port numbers are integer values. One problem we encounter when passing these values across a network is that different hardware architectures store the bytes of a multibyte integer in different orders. As shown in [Figure 59-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59fig1), architectures that store integers with the most significant byte first (i.e., at the lowest memory address) are termed _big endian_; those that store the least significant byte first are termed _little endian_. (The terms derive from Jonathan Swift’s 1726 satirical novel _Gulliver’s Travels_, in which the terms refer to opposing political factions who open their boiled eggs at opposite ends.) The most notable example of a little-endian architecture is x86. (Digital’s VAX architecture was another historically important example, since BSD was widely used on that machine.) Most other architectures are big endian. A few hardware architectures are switchable between the two formats. The byte ordering used on a particular machine is called the _host byte order_.

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f59-01.jpg)

**Figure 59-1:** Big-endian and little-endian byte order for 2-byte and 4-byte integers

Since port numbers and IP addresses must be transmitted between, and understood by, all hosts on a network, a standard ordering must be used. This ordering is called _network byte order_, and happens to be big endian.

Later in this chapter, we look at various functions that convert hostnames (e.g., [www.kernel.org](http://www.kernel.org/)) and service names (e.g., _http_) into the corresponding numeric forms. These functions generally return integers in network byte order, and these integers can be copied directly into the relevant fields of a socket address structure.

However, we sometimes make direct use of integer constants for IP addresses and port numbers. For example, we may choose to hard-code a port number into our program, specify a port number as a command-line argument to a program, or use constants such as INADDR_ANY and INADDR_LOOPBACK when specifying an IPv4 address. These values are represented in C according to the conventions of the host machine, so they are in host byte order. We must convert these values to network byte order before storing them in socket address structures.

The _htons()_, _htonl()_, _ntohs()_, and _ntohl()_ functions are defined (typically as macros) for converting integers in either direction between host and network byte order.

#include <arpa/inet.h>  
  
uint16_t htons(uint16_t host_uint16);

Returns _host_uint16_ converted to network byte order

uint32_t htonl(uint32_t host_uint32);

Returns _host_uint32_ converted to network byte order

uint16_t ntohs(uint16_t net_uint16);

Returns _net_uint16_ converted to host byte order

uint32_t ntohl(uint32_t net_uint32);

Returns _net_uint32_ converted to host byte order

In earlier times, these functions had prototypes such as the following:

unsigned long htonl(unsigned long hostlong);

This reveals the origin of the function names—in this case, _host to network long_. On most early systems on which sockets were implemented, short integers were 16 bits, and long integers were 32 bits. This no longer holds true on modern systems (at least for long integers), so the prototypes given above provide a more exact definition of the types dealt with by these functions, although the names remain unchanged. The _uint16_t_ and _uint32_t_ data types are 16-bit and 32-bit unsigned integers.

Strictly speaking, the use of these four functions is necessary only on systems where the host byte order differs from network byte order. However, these functions should always be used, so that programs are portable to different hardware architectures. On systems where the host byte order is the same as network byte order, these functions simply return their arguments unchanged.

### **59.3 Data Representation**

When writing network programs, we need to be aware of the fact that different computer architectures use different conventions for representing various data types. We have already noted that integer types can be stored in big-endian or little-endian form. There are also other possible differences. For example, the C _long_ data type may be 32 bits on some systems and 64 bits on others. When we consider structures, the issue is further complicated by the fact that different implementations employ different rules for aligning the fields of a structure to address boundaries on the host system, leaving different numbers of padding bytes between the fields.

Because of these differences in data representation, applications that exchange data between heterogeneous systems over a network must adopt some common convention for encoding that data. The sender must encode data according to this convention, while the receiver decodes following the same convention. The process of putting data into a standard format for transmission across a network is referred to as _marshalling_. Various marshalling standards exist, such as XDR (External Data Representation, described in RFC 1014), ASN.1-BER (Abstract Syntax Notation 1, _[http://www.asn1.org/](http://www.asn1.org/)_), CORBA, and XML. Typically, these standards define a fixed format for each data type (defining, for example, byte order and number of bits used). As well as being encoded in the required format, each data item is tagged with extra field(s) identifying its type (and, possibly, length).

However, a simpler approach than marshalling is often employed: encode all transmitted data in text form, with separate data items delimited by a designated character, typically a newline character. One advantage of this approach is that we can use _telnet_ to debug an application. To do this, we use the following command:

$ telnet host port

We can then type lines of text to be transmitted to the application, and view the responses sent by the application. We demonstrate this technique in [Section 59.11](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec11).

The problems associated with differences in representation across heterogeneous systems apply not only to data transfer across a network, but also to any mechanism of data exchange between such systems. For example, we face the same problems when transferring files on disk or tape between heterogeneous systems. Network programming is simply the most common programming context in which we are nowadays likely to encounter this issue.

If we encode data transmitted on a stream socket as newline-delimited text, then it is convenient to define a function such as _readLine()_, shown in [Listing 59-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex1).

#include "read_line.h"  
  
ssize_t readLine(int fd, void *buffer, size_t n);

Returns number of bytes copied into _buffer_ (excluding terminating null byte), or 0 on end-of-file, or –1 on error

The _readLine()_ function reads bytes from the file referred to by the file descriptor argument _fd_ until a newline is encountered. The input byte sequence is returned in the location pointed to by _buffer_, which must point to a region of at least _n_ bytes of memory. The returned string is always null-terminated; thus, at most _(n – 1)_ bytes of actual data will be returned. On success, _readLine()_ returns the number of bytes of data placed in _buffer_; the terminating null byte is not included in this count.

**Listing 59-1:** Reading data a line at a time

______________________________________________________ sockets/read_line.c  
  
#include <unistd.h>  
#include <errno.h>  
#include "read_line.h"                  /* Declaration of readLine() */  
  
ssize_t  
readLine(int fd, void *buffer, size_t n)  
{  
    ssize_t numRead;                    /* # of bytes fetched by last read() */  
    size_t totRead;                     /* Total bytes read so far */  
    char *buf;  
    char ch;  
  
    if (n <= 0 || buffer == NULL) {  
        errno = EINVAL;  
        return -1;  
    }  
  
    buf = buffer;                       /* No pointer arithmetic on "void *" */  
  
    totRead = 0;  
    for (;;) {  
        numRead = read(fd, &ch, 1);  
  
        if (numRead == -1) {  
            if (errno == EINTR)         /* Interrupted --> restart read() */  
                continue;  
            else  
                return -1;              /* Some other error */  
  
        } else if (numRead == 0) {      /* EOF */  
            if (totRead == 0)           /* No bytes read; return 0 */  
                return 0;  
            else                        /* Some bytes read; add '\0' */  
                break;  
  
        } else {                        /* 'numRead' must be 1 if we get here */  
            if (totRead < n - 1) {      /* Discard > (n - 1) bytes */  
                totRead++;  
                *buf++ = ch;  
            }  
  
            if (ch == '\n')  
                break;  
        }  
    }  
  
    *buf = '\0';  
    return totRead;  
}  
______________________________________________________ sockets/read_line.c

If the number of bytes read before a newline is encountered is greater than or equal to _(n – 1)_, then the _readLine()_ function discards the excess bytes (including the newline). If a newline was read within the first _(n – 1)_ bytes, then it is included in the returned string. (Thus, we can determine if bytes were discarded by checking if a newline precedes the terminating null byte in the returned _buffer_.) We take this approach so that application protocols that rely on handling input in units of lines don’t end up processing a long line as though it were multiple lines. This would likely break the protocol, as the applications on either end would become desynchronized. An alternative approach would be to have _readLine()_ read only sufficient bytes to fill the supplied buffer, leaving any remaining bytes up to the next newline for the next call to _readLine()_. In this case, the caller of _readLine()_ would need to handle the possibility of a partial line being read.

We employ the _readLine()_ function in the example programs presented in [Section 59.11](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec11).

### **59.4 Internet Socket Addresses**

There are two types of Internet domain socket addresses: IPv4 and IPv6.

##### **IPv4 socket addresses: _struct sockaddr_in_**

An IPv4 socket address is stored in a _sockaddr_in_ structure, defined in <netinet/in.h> as follows:

struct in_addr {                  /* IPv4 4-byte address */  
    in_addr_t s_addr;             /* Unsigned 32-bit integer */  
};  
  
struct sockaddr_in {              /* IPv4 socket address */  
    sa_family_t sin_family;       /* Address family (AF_INET) */  
    in_port_t sin_port;           /* Port number */  
    struct in_addr sin_addr;      /* IPv4 address */  
    unsigned char __pad[X];       /* Pad to size of 'sockaddr'  
                                     structure (16 bytes) */  
};

In [Section 56.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch56.xhtml#ch56lev1sec04), we saw that the generic _sockaddr_ structure commences with a field identifying the socket domain. This corresponds to the _sin_family_ field in the _sockaddr_in_ structure, which is always set to AF_INET. The _sin_port_ and _sin_addr_ fields are the port number and the IP address, both in network byte order. The _in_port_t_ and _in_addr_t_ data types are unsigned integer types, 16 and 32 bits in length, respectively.

##### **IPv6 socket addresses: _struct sockaddr_in6_**

Like an IPv4 address, an IPv6 socket address includes an IP address plus a port number. The difference is that an IPv6 address is 128 bits instead of 32 bits. An IPv6 socket address is stored in a _sockaddr_in6_ structure, defined in <netinet/in.h> as follows:

struct in6_addr {                   /* IPv6 address structure */  
    uint8_t s6_addr[16];            /* 16 bytes == 128 bits */  
};  
  
struct sockaddr_in6 {               /* IPv6 socket address */  
    sa_family_t sin6_family;        /* Address family (AF_INET6) */  
    in_port_t   sin6_port;          /* Port number */  
    uint32_t    sin6_flowinfo;      /* IPv6 flow information */  
    struct in6_addr sin6_addr;      /* IPv6 address */  
    uint32_t    sin6_scope_id;      /* Scope ID (new in kernel 2.4) */  
};

The _sin6_family_ field is set to AF_INET6. The _sin6_port_ and _sin6_addr_ fields are the port number and the IP address. (The _uint8_t_ data type, used to type the bytes of the _in6_addr_ structure, is an 8-bit unsigned integer.) The remaining fields, _sin6_flowinfo_ and _sin6_scope_id_, are beyond the scope of this book; for our purposes, they are always set to 0. All of the fields in the _sockaddr_in6_ structure are in network byte order.

IPv6 addresses are described in RFC 4291. Information about IPv6 flow control (_sin6_flowinfo_) can be found in [Appendix A](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/app01.xhtml#app01) of [[Stevens et al., 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib93)] and in RFCs 2460 and 3697. RFCs 3493 and 4007 provide information about _sin6_scope_id_.

IPv6 has equivalents of the IPv4 wildcard and loopback addresses. However, their use is complicated by the fact that an IPv6 address is stored in an array (rather than using a scalar type). We use the IPv6 wildcard address (0::0) to illustrate this point. The constant IN6ADDR_ANY_INIT is defined for this address as follows:

#define IN6ADDR_ANY_INIT { { 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 } }

On Linux, some details in the header files differ from our description in this section. In particular, the _in6_addr_ structure contains a union definition that divides the 128-bit IPv6 address into 16 bytes, eight 2-byte integers, or four 4-byte integers. Because of the presence of this definition, the _glibc_ definition of the IN6ADDR_ANY_INIT constant actually includes one more set of nested braces than is shown in the main text.

We can use the IN6ADDR_ANY_INIT constant in the initializer that accompanies a variable declaration, but can’t use it on the right-hand side of an assignment statement, since C syntax doesn’t permit structured constants to be used in assignments. Instead, we must use a predefined variable, _in6addr_any_, which is initialized as follows by the C library:

const struct in6_addr in6addr_any = IN6ADDR_ANY_INIT;

Thus, we can initialize an IPv6 socket address structure using the wildcard address as follows:

struct sockaddr_in6 addr;  
  
memset(&addr, 0, sizeof(struct sockaddr_in6));  
addr.sin6_family = AF_INET6;  
addr.sin6_addr = in6addr_any;  
addr.sin6_port = htons(SOME_PORT_NUM);

The corresponding constant and variable for the IPv6 loopback address (::1) are IN6ADDR_LOOPBACK_INIT and _in6addr_loopback_.

Unlike their IPv4 counterparts, the IPv6 constant and variable initializers are in network byte order. But, as shown in the above code, we still must ensure that the port number is in network byte order.

If IPv4 and IPv6 coexist on a host, they share the same port-number space. This means that if, for example, an application binds an IPv6 socket to TCP port 2000 (using the IPv6 wildcard address), then an IPv4 TCP socket can’t be bound to the same port. (The TCP/IP implementation ensures that sockets on other hosts are able to communicate with this socket, regardless of whether those hosts are running IPv4 or IPv6.)

##### **The _sockaddr_storage_ structure**

With the IPv6 sockets API, the new generic _sockaddr_storage_ structure was introduced. This structure is defined to be large enough to hold any type of socket address (i.e., any type of socket address structure can be cast and stored in it). In particular, this structure allows us to transparently store either an IPv4 or an IPv6 socket address, thus removing IP version dependencies from our code. The _sockaddr_storage_ structure is defined on Linux as follows:

#define __ss_aligntype uint32_t         /* On 32-bit architectures */  
struct sockaddr_storage {  
    sa_family_t ss_family;  
    __ss_aligntype __ss_align;          /* Force alignment */  
    char __ss_padding[SS_PADSIZE];      /* Pad to 128 bytes */  
};

### **59.5 Overview of Host and Service Conversion Functions**

Computers represent IP addresses and port numbers in binary. However, humans find names easier to remember than numbers. Employing symbolic names also provides a useful level of indirection; users and programs can continue to use the same name even if the underlying numeric value changes.

A _hostname_ is the symbolic identifier for a system that is connected to a network (possibly with multiple IP addresses). A _service name_ is the symbolic representation of a port number.

The following methods are available for representing host addresses and ports:

• A host address can be represented as a binary value, as a symbolic hostname, or in presentation format (dotted-decimal for IPv4 or hex-string for IPv6).

• A port can be represented as a binary value or as a symbolic service name.

Various library functions are provided for converting between these formats. This section briefly summarizes these functions. The following sections describe the modern APIs (_inet_ntop()_, _inet_pton()_, _getaddrinfo()_, _getnameinfo()_, and so on) in detail. In [Section 59.13](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec13), we briefly discuss the obsolete APIs (_inet_aton()_, _inet_ntoa()_, _gethostbyname()_, _getservbyname()_, and so on).

##### **Converting IPv4 addresses between binary and human-readable forms**

The _inet_aton()_ and _inet_ntoa()_ functions convert an IPv4 address in dotted-decimal notation to binary and vice versa. We describe these functions primarily because they appear in historical code. Nowadays, they are obsolete. Modern programs that need to do such conversions should use the functions that we describe next.

##### **Converting IPv4 and IPv6 addresses between binary and human-readable forms**

The _inet_pton()_ and _inet_ntop()_ functions are like _inet_aton()_ and _inet_ntoa()_, but differ in that they also handle IPv6 addresses. They convert binary IPv4 and IPv6 addresses to and from _presentation_ format—that is, either dotted-decimal or hex-string notation.

Since humans deal better with names than with numbers, we normally use these functions only occasionally in programs. One use of _inet_ntop()_ is to produce a printable representation of an IP address for logging purposes. Sometimes, it is preferable to use this function instead of converting (“resolving”) an IP address to a hostname, for the following reasons:

• Resolving an IP address to a hostname involves a possibly time-consuming request to a DNS server.

• In some circumstances, there may not be a DNS (PTR) record that maps the IP address to a corresponding hostname.

We describe these functions (in [Section 59.6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec06)) before _getaddrinfo()_ and _getnameinfo()_, which perform conversions between binary representations and the corresponding symbolic names, principally because they present a much simpler API. This allows us to quickly show some working examples of the use of Internet domain sockets.

##### **Converting host and service names to and from binary form (obsolete)**

The _gethostbyname()_ function returns the binary IP address(es) corresponding to a hostname and the _getservbyname()_ function returns the port number corresponding to a service name. The reverse conversions are performed by _gethostbyaddr()_ and _getservbyport()_. We describe these functions because they are widely used in existing code. However, they are now obsolete. (SUSv3 marks these functions obsolete, and SUSv4 removes their specifications.) New code should use the _getaddrinfo()_ and _getnameinfo()_ functions (described next) for such conversions.

##### **Converting host and service names to and from binary form (modern)**

The _getaddrinfo()_ function is the modern successor to both _gethostbyname()_ and _getservbyname()_. Given a hostname and a service name, _getaddrinfo()_ returns a set of structures containing the corresponding binary IP address(es) and port number. Unlike _gethostbyname()_, _getaddrinfo()_ transparently handles both IPv4 and IPv6 addresses. Thus, we can use it to write programs that don’t contain dependencies on the IP version being employed. All new code should use _getaddrinfo()_ for converting hostnames and service names to binary representation.

The _getnameinfo()_ function performs the reverse translation, converting an IP address and port number into the corresponding hostname and service name.

We can also use _getaddrinfo()_ and _getnameinfo()_ to convert binary IP addresses to and from presentation format.

The discussion of _getaddrinfo()_ and _getnameinfo()_, in [Section 59.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec10), requires an accompanying description of DNS ([Section 59.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec08)) and the /etc/services file ([Section 59.9](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec09)). DNS allows cooperating servers to maintain a distributed database that maps binary IP addresses to hostnames and vice versa. The existence of a system such as DNS is essential to the operation of the Internet, since centralized management of the enormous set of Internet hostnames would be impossible. The /etc/services file maps port numbers to symbolic service names.

### **59.6 The _inet_pton()_ and _inet_ntop()_ Functions**

The _inet_pton()_ and _inet_ntop()_ functions allow conversion of both IPv4 and IPv6 addresses between binary form and dotted-decimal or hex-string notation.

#include <arpa/inet.h>  
  
int inet_pton(int domain, const char *src_str, void *addrptr);

Returns 1 on successful conversion, 0 if _src_str_ is not in presentation format, or –1 on error

const char *inet_ntop(int domain, const void *addrptr, char *dst_str, size_t len);

Returns pointer to _dst_str_ on success, or NULL on error

The _p_ in the names of these functions stands for “presentation,” and the _n_ stands for “network.” The presentation form is a human-readable string, such as the following:

• 204.152.189.116 (IPv4 dotted-decimal address);

• ::1 (an IPv6 colon-separated hexadecimal address); or

• ::FFFF:204.152.189.116 (an IPv4-mapped IPv6 address).

The _inet_pton()_ function converts the presentation string contained in _src_str_ into a binary IP address in network byte order. The _domain_ argument should be specified as either AF_INET or AF_INET6. The converted address is placed in the structure pointed to by _addrptr_, which should point to either an _in_addr_ or an _in6_addr_ structure, according to the value specified in _domain_.

The _inet_ntop()_ function performs the reverse conversion. Again, _domain_ should be specified as either AF_INET or AF_INET6, and _addrptr_ should point to an _in_addr_ or _in6_addr_ structure that we wish to convert. The resulting null-terminated string is placed in the buffer pointed to by _dst_str_. The _len_ argument must specify the size of this buffer. On success, _inet_ntop()_ returns _dst_str_. If _len_ is too small, then _inet_ntop()_ returns NULL, with _errno_ set to ENOSPC.

To correctly size the buffer pointed to by _dst_str_, we can employ two constants defined in <netinet/in.h>. These constants indicate the maximum lengths (including the terminating null byte) of the presentation strings for IPv4 and IPv6 addresses:

#define INET_ADDRSTRLEN  16     /* Maximum IPv4 dotted-decimal string */  
#define INET6_ADDRSTRLEN 46     /* Maximum IPv6 hexadecimal string */

We provide examples of the use of _inet_pton()_ and _inet_ntop()_ in the next section.

### **59.7 Client-Server Example (Datagram Sockets)**

In this section, we take the case-conversion server and client programs shown in [Section 57.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch57.xhtml#ch57lev1sec03) and modify them to use datagram sockets in the AF_INET6 domain. We present these programs with a minimum of commentary, since their structure is similar to the earlier programs. The main differences in the new programs lie in the declaration and initialization of the IPv6 socket address structure, which we described in [Section 59.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec04).

The client and server both employ the header file shown in [Listing 59-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex2). This header file defines the server’s port number and the maximum size of messages that the client and server can exchange.

**Listing 59-2:** Header file used by i6d_ucase_sv.c and i6d_ucase_cl.c

______________________________________________________ sockets/i6d_ucase.h  
  
#include <netinet/in.h>  
#include <arpa/inet.h>  
#include <sys/socket.h>  
#include <ctype.h>  
#include "tlpi_hdr.h"  
  
#define BUF_SIZE 10                   /* Maximum size of messages exchanged  
                                         between client and server */  
  
#define PORT_NUM 50002                /* Server port number */  
______________________________________________________ sockets/i6d_ucase.h

[Listing 59-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex3) shows the server program. The server uses the _inet_ntop()_ function to convert the host address of the client (obtained via the _recvfrom()_ call) to printable form.

The client program shown in [Listing 59-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex4) contains two notable modifications from the earlier UNIX domain version ([Listing 57-7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch57.xhtml#ch57ex7), on [page 1173](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch57.xhtml#page_1173)). The first difference is that the client interprets its initial command-line argument as the IPv6 address of the server. (The remaining command-line arguments are passed as separate datagrams to the server.) The client converts the server address to binary form using _inet_pton()_. The other difference is that the client doesn’t bind its socket to an address. As noted in [Section 58.6.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch58.xhtml#ch58lev2sec01), if an Internet domain socket is not bound to an address, the kernel binds the socket to an ephemeral port on the host system. We can observe this in the following shell session log, where we run the server and the client on the same host:

$ ./i6d_ucase_sv &  
[1] 31047  
$ ./i6d_ucase_cl ::1 ciao                     Send to server on local host  
Server received 4 bytes from (::1, 32770)  
Response 1: CIAO

From the above output, we see that the server’s _recvfrom()_ call was able to obtain the address of the client’s socket, including the ephemeral port number, despite the fact that the client did not do a _bind()_.

**Listing 59-3:** IPv6 case-conversion server using datagram sockets

____________________________________________________ sockets/i6d_ucase_sv.c  
  
#include "i6d_ucase.h"  
  
int  
main(int argc, char *argv[])  
{  
    struct sockaddr_in6 svaddr, claddr;  
    int sfd, j;  
    ssize_t numBytes;  
    socklen_t len;  
    char buf[BUF_SIZE];  
    char claddrStr[INET6_ADDRSTRLEN];  
  
    sfd = socket(AF_INET6, SOCK_DGRAM, 0);  
    if (sfd == -1)  
        errExit("socket");  
  
    memset(&svaddr, 0, sizeof(struct sockaddr_in6));  
    svaddr.sin6_family = AF_INET6;  
    svaddr.sin6_addr = in6addr_any;                    /* Wildcard address */  
    svaddr.sin6_port = htons(PORT_NUM);  
  
    if (bind(sfd, (struct sockaddr *) &svaddr,  
                sizeof(struct sockaddr_in6)) == -1)  
        errExit("bind");  
  
    /* Receive messages, convert to uppercase, and return to client */  
  
    for (;;) {  
        len = sizeof(struct sockaddr_in6);  
        numBytes = recvfrom(sfd, buf, BUF_SIZE, 0,  
                            (struct sockaddr *) &claddr, &len);  
        if (numBytes == -1)  
            errExit("recvfrom");  
  
        if (inet_ntop(AF_INET6, &claddr.sin6_addr, claddrStr,  
                    INET6_ADDRSTRLEN) == NULL)  
            printf("Couldn't convert client address to string\n");  
        else  
            printf("Server received %ld bytes from (%s, %u)\n",  
                    (long) numBytes, claddrStr, ntohs(claddr.sin6_port));  
  
        for (j = 0; j < numBytes; j++)  
            buf[j] = toupper((unsigned char) buf[j]);  
  
        if (sendto(sfd, buf, numBytes, 0, (struct sockaddr *) &claddr, len) !=  
                numBytes)  
            fatal("sendto");  
    }  
}  
____________________________________________________ sockets/i6d_ucase_sv.c

**Listing 59-4:** IPv6 case-conversion client using datagram sockets

____________________________________________________ sockets/i6d_ucase_cl.c  
  
#include "i6d_ucase.h"  
  
int  
main(int argc, char *argv[])  
{  
    struct sockaddr_in6 svaddr;  
    int sfd, j;  
    size_t msgLen;  
    ssize_t numBytes;  
    char resp[BUF_SIZE];  
  
    if (argc < 3 || strcmp(argv[1], "--help") == 0)  
        usageErr("%s host-address msg...\n", argv[0]);  
  
    sfd = socket(AF_INET6, SOCK_DGRAM, 0);      /* Create client socket */  
    if (sfd == -1)  
        errExit("socket");  
  
    memset(&svaddr, 0, sizeof(struct sockaddr_in6));  
    svaddr.sin6_family = AF_INET6;  
    svaddr.sin6_port = htons(PORT_NUM);  
    if (inet_pton(AF_INET6, argv[1], &svaddr.sin6_addr) <= 0)  
        fatal("inet_pton failed for address '%s'", argv[1]);  
  
    /* Send messages to server; echo responses on stdout */  
  
    for (j = 2; j < argc; j++) {  
        msgLen = strlen(argv[j]);  
        if (sendto(sfd, argv[j], msgLen, 0, (struct sockaddr *) &svaddr,  
                    sizeof(struct sockaddr_in6)) != msgLen)  
            fatal("sendto");  
  
        numBytes = recvfrom(sfd, resp, BUF_SIZE, 0, NULL, NULL);  
        if (numBytes == -1)  
            errExit("recvfrom");  
  
        printf("Response %d: %.*s\n", j - 1, (int) numBytes, resp);  
    }  
  
    exit(EXIT_SUCCESS);  
}  
____________________________________________________ sockets/i6d_ucase_cl.c

### **59.8 Domain Name System (DNS)**

In [Section 59.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec10), we describe _getaddrinfo()_, which obtains the IP address(es) corresponding to a hostname, and _getnameinfo()_, which performs the converse task. However, before looking at these functions, we explain how DNS is used to maintain the mappings between hostnames and IP addresses.

Before the advent of DNS, mappings between hostnames and IP addresses were defined in a manually maintained local file, /etc/hosts, containing records of the following form:

# IP-address    canonical hostname      [aliases]  
127.0.0.1       localhost

The _gethostbyname()_ function (the predecessor to _getaddrinfo()_) obtained an IP address by searching this file, looking for a match on either the canonical hostname (i.e., the official or primary name of the host) or one of the (optional, space-delimited) aliases.

However, the /etc/hosts scheme scales poorly, and then becomes impossible, as the number of hosts in the network increases (e.g., the Internet, with millions of hosts).

DNS was devised to address this problem. The key ideas of DNS are the following:

• Hostnames are organized into a hierarchical namespace ([Figure 59-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59fig2)). Each _node_ in the DNS hierarchy has a _label_ (name), which may be up to 63 characters. At the root of the hierarchy is an unnamed node, the “anonymous root.”

• A node’s _domain name_ consists of all of the names from that node up to the root concatenated together, with each name separated by a period (.). For example, google.com is the domain name for the node google.

• A _fully qualified domain name_ (FQDN), such as [www.kernel.org](http://www.kernel.org/)., identifies a host within the hierarchy. A fully qualified domain name is distinguished by being terminated by a period, although in many contexts the period may be omitted.

• No single organization or system manages the entire hierarchy. Instead, there is a hierarchy of DNS servers, each of which manages a branch (a _zone_) of the tree. Normally, each zone has a _primary master name server_, and one or more _slave name servers_ (sometimes also known as _secondary master name servers_), which provide backup in the event that the primary master name server crashes. Zones may themselves be divided into separately managed smaller zones. When a host is added within a zone, or the mapping of a hostname to an IP address is changed, the administrator responsible for the corresponding local name server updates the name database on that server. (No manual changes are required on any other name-server databases in the hierarchy.)

The DNS server implementation employed on Linux is the widely used Berkeley Internet Name Domain (BIND) implementation, _named(8)_, maintained by the _Internet Systems Consortium_ (_[http://www.isc.org/](http://www.isc.org/)_). The operation of this daemon is controlled by the file /etc/named.conf (see the _named.conf(5)_ manual page). The key reference on DNS and BIND is [[Albitz & Liu, 2006](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib02)]. Information about DNS can also be found in [Chapter 14](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch14.xhtml#ch14) of [[Stevens, 1994](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib91)], [Chapter 11](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch11.xhtml#ch11) of [[Stevens et al., 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib93)], and [Chapter 24](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch24.xhtml#ch24) of [[Comer, 2000](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib12)].

• When a program calls _getaddrinfo()_ to _resolve_ (i.e., obtain the IP address for) a domain name, _getaddrinfo()_ employs a suite of library functions (the _resolver library_) that communicate with the local DNS server. If this server can’t supply the required information, then it communicates with other DNS servers within the hierarchy in order to obtain the information. Occasionally, this resolution process may take a noticeable amount of time, and DNS servers employ caching techniques to avoid unnecessary communication for frequently queried domain names.

Using the above approach allows DNS to cope with large namespaces, and does not require centralized management of names.

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f59-02.jpg)

**Figure 59-2:** A subset of the DNS hierarchy

##### **Recursive and iterative resolution requests**

DNS resolution requests fall into two categories: _recursive_ and _iterative_. In a recursive request, the requester asks the server to handle the entire task of resolution, including the task of communicating with any other DNS servers, if necessary. When an application on the local host calls _getaddrinfo()_, that function makes a recursive request to the local DNS server. If the local DNS server does not itself have the information to perform the resolution, it resolves the domain name iteratively.

We explain iterative resolution via an example. Suppose that the local DNS server is asked to resolve the name [www.otago.ac.nz](http://www.otago.ac.nz/). To do this, it first communicates with one of a small set of _root name servers_ that every DNS server is required to know about. (We can obtain a list of these servers using the command _dig . NS_ or from the web page at _[http://www.root-servers.org/](http://www.root-servers.org/)_.) Given the name [www.otago.ac.nz](http://www.otago.ac.nz/), the root name server refers the local DNS server to one of the nz DNS servers. The local DNS server then queries the nz server with the name [www.otago.ac.nz](http://www.otago.ac.nz/), and receives a response referring it to the ac.nz server. The local DNS server then queries the ac.nz server with the name [www.otago.ac.nz](http://www.otago.ac.nz/), and is referred to the otago.ac.nz server. Finally, the local DNS server queries the otago.ac.nz server with the name [www.otago.ac.nz](http://www.otago.ac.nz/), and obtains the required IP address.

If we supply an incomplete domain name to _gethostbyname()_, the resolver will attempt to complete it before resolving it. The rules on how a domain name is completed are defined in /etc/resolv.conf (see the _resolv.conf(5)_ manual page). By default, the resolver will at least try completion using the domain name of the local host. For example, if we are logged in on the machine oghma.otago.ac.nz and we type the command _ssh octavo_, the resulting DNS query will be for the name octavo.otago.ac.nz.

##### **Top-level domains**

The nodes immediately below the anonymous root form the so-called _top-level domains_ (TLDs). (Below these are the _second-level domains_, and so on.) TLDs fall into two categories: _generic_ and _country_.

Historically, there were seven _generic_ TLDs, most of which can be considered international. We have shown four of the original generic TLDs in [Figure 59-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59fig2). The other three are int, mil, and gov; the latter two are reserved for the United States. In more recent times, a number of new generic TLDs have been added (e.g., info, name, and museum).

Each nation has a corresponding _country_ (or _geographical_) TLD (standardized as ISO 3166-1), with a 2-character name. In [Figure 59-2](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59fig2), we have shown a few of these: de (Germany, _Deutschland_), eu (a supra-national geographical TLD for the European Union), nz (New Zealand), and us (United States of America). Several countries divide their TLD into a set of second-level domains in a manner similar to the generic domains. For example, New Zealand has ac.nz (academic institutions), co.nz (commercial), and govt.nz (government).

### **59.9 The** /etc/services **File**

As noted in [Section 58.6.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch58.xhtml#ch58lev2sec01), well-known port numbers are centrally registered by IANA. Each of these ports has a corresponding _service name_. Because service numbers are centrally managed and are less volatile than IP addresses, an equivalent of the DNS server is usually not necessary. Instead, the port numbers and service names are recorded in the file /etc/services. The _getaddrinfo()_ and _getnameinfo()_ functions use the information in this file to convert service names to port numbers and vice versa.

The /etc/services file consists of lines containing three columns, as shown in the following examples:

# Service name  port/protocol  [aliases]  
echo            7/tcp          Echo     # echo service  
echo            7/udp          Echo  
ssh             22/tcp                  # Secure Shell  
ssh             22/udp  
telnet          23/tcp                  # Telnet  
telnet          23/udp  
smtp            25/tcp                  # Simple Mail Transfer Protocol  
smtp            25/udp  
domain          53/tcp                  # Domain Name Server  
domain          53/udp  
http            80/tcp                  # Hypertext Transfer Protocol  
http            80/udp  
ntp             123/tcp                 # Network Time Protocol  
ntp             123/udp  
login           513/tcp                 # rlogin(1)  
who             513/udp                 # rwho(1)  
shell           514/tcp                 # rsh(1)  
syslog          514/udp                 # syslog

The _protocol_ is typically either tcp or udp. The optional (space-delimited) _aliases_ specify alternative names for the service. In addition to the above, lines may include comments starting with the # character.

As noted previously, a given port number refers to distinct entities for UDP and TCP, but IANA policy assigns both port numbers to a service, even if that service uses only one protocol. For example, _telnet_, _ssh_, HTTP, and SMTP all use TCP, but the corresponding UDP port is also assigned to these services. Conversely, NTP uses only UDP, but the TCP port 123 is also assigned to this service. In some cases, a service uses both UDP and TCP; DNS and _echo_ are examples of such services. Finally, there are a very few cases where the UDP and TCP ports with the same number are assigned to different services; for example, _rsh_ uses TCP port 514, while the _syslog_ daemon ([Section 37.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch37.xhtml#ch37lev1sec05)) uses UDP port 514. This is because these port numbers were assigned before the adoption of the present IANA policy.

The /etc/services file is merely a record of name-to-number mappings. It is not a reservation mechanism: the appearance of a port number in /etc/services doesn’t guarantee that it will actually be available for binding by a particular service.

### **59.10 Protocol-Independent Host and Service Conversion**

The _getaddrinfo()_ function converts host and service names to IP addresses and port numbers. It was defined in POSIX.1g as the (reentrant) successor to the obsolete _gethostbyname()_ and _getservbyname()_ functions. (Replacing the use of _gethostbyname()_ with _getaddrinfo()_ allows us to eliminate IPv4-versus-IPv6 dependencies from our programs.)

The _getnameinfo()_ function is the converse of _getaddrinfo()_. It translates a socket address structure (either IPv4 or IPv6) to strings containing the corresponding host and service name. This function is the (reentrant) equivalent of the obsolete _gethostbyaddr()_ and _getservbyport()_ functions.

[Chapter 11](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch11.xhtml#ch11) of [[Stevens et al., 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib93)] describes _getaddrinfo()_ and _getnameinfo()_ in detail, and provides implementations of these functions. These functions are also described in RFC 3493.

#### **59.10.1 The _getaddrinfo()_ Function**

Given a host name and a service name, _getaddrinfo()_ returns a list of socket address structures, each of which contains an IP address and port number.

#include <sys/socket.h>  
#include <netdb.h>  
  
int getaddrinfo(const char *host, const char *service,  
                const struct addrinfo *hints, struct addrinfo **result);

Returns 0 on success, or nonzero on error

As input, _getaddrinfo()_ takes the arguments _host_, _service_, and _hints_. The _host_ argument contains either a hostname or a numeric address string, expressed in IPv4 dotted-decimal notation or IPv6 hex-string notation. (To be precise, _getaddrinfo()_ accepts IPv4 numeric strings in the more general numbers-and-dots notation described in [Section 59.13.1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev2sec05).) The _service_ argument contains either a service name or a decimal port number. The _hints_ argument points to an _addrinfo_ structure that specifies further criteria for selecting the socket address structures returned via _result_. We describe the _hints_ argument in more detail below.

As output, _getaddrinfo()_ dynamically allocates a linked list of _addrinfo_ structures and sets _result_ pointing to the beginning of this list. Each of these _addrinfo_ structures includes a pointer to a socket address structure corresponding to _host_ and _service_ ([Figure 59-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59fig3)). The _addrinfo_ structure has the following form:

struct addrinfo {  
    int       ai_flags;         /* Input flags (AI_* constants) */  
    int       ai_family;        /* Address family */  
    int       ai_socktype;      /* Type: SOCK_STREAM, SOCK_DGRAM */  
    int       ai_protocol;      /* Socket protocol */  
    socklen_t ai_addrlen;       /* Size of structure pointed to by ai_addr */  
    char     *ai_canonname;     /* Canonical name of host */  
    struct sockaddr *ai_addr;   /* Pointer to socket address structure */  
    struct addrinfo *ai_next;   /* Next structure in linked list */  
};

The _result_ argument returns a list of structures, rather than a single structure, because there may be multiple combinations of host and service corresponding to the criteria specified in _host_, _service_, and _hints_. For example, multiple address structures could be returned for a host with more than one network interface. Furthermore, if _hints.ai_socktype_ was specified as 0, then two structures could be returned—one for a SOCK_DGRAM socket, the other for a SOCK_STREAM socket—if the given _service_ was available for both UDP and TCP.

The fields of each _addrinfo_ structure returned via _result_ describe properties of the associated socket address structure. The _ai_family_ field is set to either AF_INET or AF_INET6, informing us of the type of the socket address structure. The _ai_socktype_ field is set to either SOCK_STREAM or SOCK_DGRAM, indicating whether this address structure is for a TCP or a UDP service. The _ai_protocol_ field returns a protocol value appropriate for the address family and socket type. (The three fields _ai_family_, _ai_socktype_, and _ai_protocol_ supply the values required for the arguments used when calling _socket()_ to create a socket for this address.) The _ai_addrlen_ field gives the size (in bytes) of the socket address structure pointed to by _ai_addr_. The _ai_addr_ field points to the socket address structure (a _sockaddr_in_ structure for IPv4 or a _sockaddr_in6_ structure for IPv6). The _ai_flags_ field is unused (it is used for the _hints_ argument). The _ai_canonname_ field is used only in the first _addrinfo_ structure, and only if the AI_CANONNAME flag is employed in _hints.ai_flags_, as described below.

As with _gethostbyname()_, _getaddrinfo()_ may need to send a request to a DNS server, and this request may take some time to complete. The same applies for _getnameinfo()_, which we describe in [Section 59.10.4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev2sec04).

We demonstrate the use of _getaddrinfo()_ in [Section 59.11](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec11).

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781593272203/files/images/f59-03.jpg)

**Figure 59-3:** Structures allocated and returned by _getaddrinfo()_

##### **The _hints_ argument**

The _hints_ argument specifies further criteria for selecting the socket address structures returned by _getaddrinfo()_. When used as the _hints_ argument, only the _ai_flags_, _ai_family_, _ai_socktype_, and _ai_protocol_ fields of the _addrinfo_ structure can be set. The other fields are unused, and should be initialized to 0 or NULL, as appropriate.

The _hints.ai_family_ field selects the domain for the returned socket address structures. It may be specified as AF_INET or AF_INET6 (or some other AF_* constant, if the implementation supports it). If we are interested in getting back all types of socket address structures, we can specify the value AF_UNSPEC for this field.

The _hints.ai_socktype_ field specifies the type of socket for which the returned address structure is to be used. If we specify this field as SOCK_DGRAM, then a lookup is performed for the UDP service, and a corresponding socket address structure is returned via _result_. If we specify SOCK_STREAM, a lookup for the TCP service is performed. If _hints.ai_socktype_ is specified as 0, any socket type is acceptable.

The _hints.ai_protocol_ field selects the socket protocol for the returned address structures. For our purposes, this field is always specified as 0, meaning that the caller will accept any protocol.

The _hints.ai_flags_ field is a bit mask that modifies the behavior of _getaddrinfo()_. This field is formed by ORing together zero or more of the following values:

AI_ADDRCONFIG

Return IPv4 addresses only if there is at least one IPv4 address configured for the local system (other than the IPv4 loopback address), and return IPv6 addresses only if there is at least one IPv6 address configured for the local system (other than the IPv6 loopback address).

AI_ALL

See the description of AI_V4MAPPED below.

AI_CANONNAME

If _host_ is not NULL, return a pointer to a null-terminated string containing the canonical name of the host. This pointer is returned in a buffer pointed to by the _ai_canonname_ field of the first of the _addrinfo_ structures returned via _result_.

AI_NUMERICHOST

Force interpretation of _host_ as a numeric address string. This is used to prevent name resolution in cases where it is unnecessary, since name resolution can be time-consuming.

AI_NUMERICSERV

Interpret _service_ as a numeric port number. This flag prevents the invocation of any name-resolution service, which is not required if _service_ is a numeric string.

AI_PASSIVE

Return socket address structures suitable for a passive open (i.e., a listening socket). In this case, _host_ should be NULL, and the IP address component of the socket address structure(s) returned by _result_ will contain a wildcard IP address (i.e., INADDR_ANY or IN6ADDR_ANY_INIT). If this flag is not set, then the address structure(s) returned via _result_ will be suitable for use with _connect()_ and _sendto()_; if _host_ is NULL, then the IP address in the returned socket address structures will be set to the loopback IP address (either INADDR_LOOPBACK or IN6ADDR_LOOPBACK_INIT, according to the domain).

AI_V4MAPPED

If AF_INET6 was specified in the _ai_family_ field of _hints_, then IPv4-mapped IPv6 address structures should be returned in _result_ if no matching IPv6 address could be found. If AI_ALL is specified in conjunction with AI_V4MAPPED, then both IPv6 and IPv4 address structures are returned in _result_, with IPv4 addresses being returned as IPv4-mapped IPv6 address structures.

As noted above for AI_PASSIVE, _host_ can be specified as NULL. It is also possible to specify _service_ as NULL, in which case the port number in the returned address structures is set to 0 (i.e., we are just interested in resolving hostnames to addresses). It is not permitted, however, to specify both _host_ and _service_ as NULL.

If we don’t need to specify any of the above selection criteria in hints, then _hints_ may be specified as NULL, in which case _ai_socktype_ and _ai_protocol_ are assumed as 0, _ai_flags_ is assumed as (AI_V4MAPPED | AI_ADDRCONFIG), and _ai_family_ is assumed as AF_UNSPEC. (The _glibc_ implementation deliberately deviates from SUSv3, which states that if _hints_ is NULL, _ai_flags_ is assumed as 0.)

#### **59.10.2 Freeing _addrinfo_ Lists: _freeaddrinfo()_**

The _getaddrinfo()_ function dynamically allocates memory for all of the structures referred to by _result_ ([Figure 59-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59fig3)). Consequently, the caller must deallocate these structures when they are no longer needed. The _freeaddrinfo()_ function is provided to conveniently perform this deallocation in a single step.

#include <sys/socket.h>  
#include <netdb.h>  
  
void freeaddrinfo(struct addrinfo *result);

If we want to preserve a copy of one of the _addrinfo_ structures or its associated socket address structure, then we must duplicate the structure(s) before calling _freeaddrinfo()_.

#### **59.10.3 Diagnosing Errors: _gai_strerror()_**

On error, _getaddrinfo()_ returns one of the nonzero error codes shown in [Table 59-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59table1).

**Table 59-1:** Error returns for _getaddrinfo()_ and _getnameinfo()_

|**Error constant**|**Description**|
|---|---|
|EAI_ADDRFAMILY|No addresses for _host_ exist in _hints.ai_family_ (not in SUSv3, but defined on most implementations; _getaddrinfo()_ only)|
|EAI_AGAIN|Temporary failure in name resolution (try again later)|
|EAI_BADFLAGS|An invalid flag was specified in _hints.ai_flags_|
|EAI_FAIL|Unrecoverable failure while accessing name server|
|EAI_FAMILY|Address family specified in _hints.ai_family_ is not supported|
|EAI_MEMORY|Memory allocation failure|
|EAI_NODATA|No address associated with _host_ (not in SUSv3, but defined on most implementations; _getaddrinfo()_ only)|
|EAI_NONAME|Unknown _host_ or _service_, or both _host_ and _service_ were NULL, or AI_NUMERICSERV specified and _service_ didn’t point to numeric string|
|EAI_OVERFLOW|Argument buffer overflow|
|EAI_SERVICE|Specified _service_ not supported for _hints.ai_socktype_ (_getaddrinfo()_ only)|
|EAI_SOCKTYPE|Specified _hints.ai_socktype_ is not supported (_getaddrinfo()_ only)|
|EAI_SYSTEM|System error returned in _errno_|

Given one of the error codes in [Table 59-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59table1), the _gai_strerror()_ function returns a string describing the error. (This string is typically briefer than the description shown in [Table 59-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59table1).)

#include <netdb.h>  
  
const char *gai_strerror(int errcode);

Returns pointer to string containing error message

We can use the string returned by _gai_strerror()_ as part of an error message displayed by an application.

#### **59.10.4 The _getnameinfo()_ Function**

The _getnameinfo()_ function is the converse of _getaddrinfo()_. Given a socket address structure (either IPv4 or IPv6), it returns strings containing the corresponding host and service name, or numeric equivalents if the names can’t be resolved.

#include <sys/socket.h>  
#include <netdb.h>  
  
int getnameinfo(const struct sockaddr *addr, socklen_t addrlen, char *host,  
                socklen_t hostlen, char *service, socklen_t servlen, int flags);

Returns 0 on success, or nonzero on error

The _addr_ argument is a pointer to the socket address structure that is to be converted. The length of that structure is given in _addrlen_. Typically, the values for _addr_ and _addrlen_ are obtained from a call to _accept()_, _recvfrom()_, _getsockname()_, or _getpeername()_.

The resulting host and service names are returned as null-terminated strings in the buffers pointed to by _host_ and _service_. These buffers must be allocated by the caller, and their sizes must be passed in _hostlen_ and _servlen_. The <netdb.h> header file defines two constants to assist in sizing these buffers. NI_MAXHOST indicates the maximum size, in bytes, for a returned hostname string. It is defined as 1025. NI_MAXSERV indicates the maximum size, in bytes, for a returned service name string. It is defined as 32. These two constants are not specified in SUSv3, but they are defined on all UNIX implementations that provide _getnameinfo()_. (Since _glibc_ 2.8, we must define one of the feature test macros _BSD_SOURCE, _SVID_SOURCE, or _GNU_SOURCE to obtain the definitions of NI_MAXHOST and NI_MAXSERV.)

If we are not interested in obtaining the hostname, we can specify _host_ as NULL and _hostlen_ as 0. Similarly, if we don’t need the service name, we can specify _service_ as NULL and _servlen_ as 0. However, at least one of _host_ and _service_ must be non-NULL (and the corresponding length argument must be nonzero).

The final argument, _flags_, is a bit mask that controls the behavior of _getnameinfo()_. The following constants may be ORed together to form this bit mask:

NI_DGRAM

By default, _getnameinfo()_ returns the name corresponding to a _stream_ socket (i.e., TCP) service. Normally, this doesn’t matter, because, as noted in [Section 59.9](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec09), the service names are usually the same for corresponding TCP and UDP ports. However, in the few instances where the names differ, the NI_DGRAM flag forces the name of the datagram socket (i.e., UDP) service to be returned.

NI_NAMEREQD

By default, if the hostname can’t be resolved, a numeric address string is returned in _host_. If the NI_NAMEREQD flag is specified, an error (EAI_NONAME) is returned instead.

NI_NOFQDN

By default, the fully qualified domain name for the host is returned. Specifying the NI_NOFQDN flag causes just the first (i.e., the hostname) part of the name to be returned, if this is a host on the local network.

NI_NUMERICHOST

Force a numeric address string to be returned in _host_. This is useful if we want to avoid a possibly time-consuming call to the DNS server.

NI_NUMERICSERV

Force a decimal port number string to be returned in _service_. This is useful in cases where we know that the port number doesn’t correspond to a service name—for example, if it is an ephemeral port number assigned to the socket by the kernel—and we want to avoid the inefficiency of unnecessarily searching /etc/services.

On success, _getnameinfo()_ returns 0. On error, it returns one of the nonzero error codes shown in [Table 59-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59table1).

### **59.11 Client-Server Example (Stream Sockets)**

We now have enough information to look at a simple client-server application using TCP sockets. The task performed by this application is the same as that performed by the FIFO client-server application presented in [Section 44.8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch44.xhtml#ch44lev1sec08): allocating unique sequence numbers (or ranges of sequence numbers) to clients.

In order to handle the possibility that integers may be represented in different formats on the server and client hosts, we encode all transmitted integers as strings terminated by a newline, and use our _readLine()_ function ([Listing 59-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex1)) to read these strings.

##### **Common header file**

Both the server and the client include the header file shown in [Listing 59-5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex5). This file includes various other header files, and defines the TCP port number to be used by the application.

##### **Server program**

The server program shown in [Listing 59-6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex6) performs the following steps:

• Initialize the server’s sequence number either to 0 or to the value supplied in the optional command-line argument ①.

• Ignore the SIGPIPE signal ②. This prevents the server from receiving the SIGPIPE signal if it tries to write to a socket whose peer has been closed; instead, the _write()_ fails with the error EPIPE.

• Call _getaddrinfo()_ ④ to obtain a set of socket address structures for a TCP socket that uses the port number PORT_NUM. (Instead of using a hard-coded port number, we would more typically use a service name.) We specify the AI_PASSIVE flag ③ so that the resulting socket will be bound to the wildcard address ([Section 58.5](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch58.xhtml#ch58lev1sec05)). As a result, if the server is run on a multihomed host, it can accept connection requests sent to any of the host’s network addresses.

• Enter a loop that iterates through the socket address structures returned by the previous step ⑤. The loop terminates when the program finds an address structure that can be used to successfully create and bind a socket ⑦.

• Set the SO_REUSEADDR option for the socket created in the previous step ⑥. We defer discussion of this option until [Section 61.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch61.xhtml#ch61lev1sec10), where we note that a TCP server should usually set this option on its listening socket.

• Mark the socket as a listening socket ⑧.

• Commence an infinite for loop ⑨ that services clients iteratively ([Chapter 60](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch60.xhtml#ch60)). Each client’s request is serviced before the next client’s request is accepted. For each client, the server performs the following steps:

– Accept a new connection ⑩. The server passes non-NULL pointers for the second and third arguments to _accept()_, in order to obtain the address of the client. The server displays the client’s address (IP address plus port number) on standard output ⑪.

– Read the client’s message ⑫, which consists of a newline-terminated string specifying how many sequence numbers the client wants. The server converts this string to an integer and stores it in the variable _reqLen_ ⑬.

– Send the current value of the sequence number (_seqNum_) back to the client, encoding it as a newline-terminated string ⑭. The client can assume that it has been allocated all of the sequence numbers in the range _seqNum_ to _(seqNum + reqLen – 1)_.

– Update the value of the server’s sequence number by adding _reqLen_ to _seqNum_ ⑮.

**Listing 59-5:** Header file used by is_seqnum_sv.c and is_seqnum_cl.c

______________________________________________________ sockets/is_seqnum.h  
  
#include <netinet/in.h>  
#include <sys/socket.h>  
#include <signal.h>  
#include "read_line.h"         /* Declaration of readLine() */  
#include "tlpi_hdr.h"  
  
#define PORT_NUM "50000"       /* Port number for server */  
  
#define INT_LEN 30             /* Size of string able to hold largest  
                                  integer (including terminating '\n') */  
______________________________________________________ sockets/is_seqnum.h

**Listing 59-6:** An iterative server that uses a stream socket to communicate with clients

____________________________________________________ sockets/is_seqnum_sv.c  
  
   #define _BSD_SOURCE             /* To get definitions of NI_MAXHOST and  
                                      NI_MAXSERV from <netdb.h> */  
   #include <netdb.h>  
   #include "is_seqnum.h"  
  
   #define BACKLOG 50  
  
   int  
   main(int argc, char *argv[])  
   {  
       uint32_t seqNum;  
       char reqLenStr[INT_LEN];            /* Length of requested sequence */  
       char seqNumStr[INT_LEN];            /* Start of granted sequence */  
       struct sockaddr_storage claddr;  
       int lfd, cfd, optval, reqLen;  
       socklen_t addrlen;  
       struct addrinfo hints;  
       struct addrinfo *result, *rp;  
   #define ADDRSTRLEN (NI_MAXHOST + NI_MAXSERV + 10)  
       char addrStr[ADDRSTRLEN];  
       char host[NI_MAXHOST];  
       char service[NI_MAXSERV];  
  
       if (argc > 1 && strcmp(argv[1], "--help") == 0)  
           usageErr("%s [init-seq-num]\n", argv[0]);  
  
①     seqNum = (argc > 1) ? getInt(argv[1], 0, "init-seq-num") : 0;  
  
②     if (signal(SIGPIPE, SIG_IGN) == SIG_ERR)  
           errExit("signal");  
  
       /* Call getaddrinfo() to obtain a list of addresses that  
          we can try binding to */  
  
       memset(&hints, 0, sizeof(struct addrinfo));  
       hints.ai_canonname = NULL;  
       hints.ai_addr = NULL;  
       hints.ai_next = NULL;  
       hints.ai_socktype = SOCK_STREAM;  
       hints.ai_family = AF_UNSPEC;        /* Allows IPv4 or IPv6 */  
③     hints.ai_flags = AI_PASSIVE | AI_NUMERICSERV;  
                           /* Wildcard IP address; service name is numeric */  
④     if (getaddrinfo(NULL, PORT_NUM, &hints, &result) != 0)  
           errExit("getaddrinfo");  
  
       /* Walk through returned list until we find an address structure  
          that can be used to successfully create and bind a socket */  
  
       optval = 1;  
⑤     for (rp = result; rp != NULL; rp = rp->ai_next) {  
           lfd = socket(rp->ai_family, rp->ai_socktype, rp->ai_protocol);  
           if (lfd == -1)  
               continue;                   /* On error, try next address */  
  
⑥         if (setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(optval))  
                   == -1)  
               errExit("setsockopt");  
  
⑦         if (bind(lfd, rp->ai_addr, rp->ai_addrlen) == 0)  
               break;                      /* Success */  
  
           /* bind() failed: close this socket and try next address */  
  
           close(lfd);  
       }  
  
       if (rp == NULL)  
           fatal("Could not bind socket to any address");  
  
⑧     if (listen(lfd, BACKLOG) == -1)  
           errExit("listen");  
  
       freeaddrinfo(result);  
  
⑨     for (;;) {                  /* Handle clients iteratively */  
  
           /* Accept a client connection, obtaining client's address */  
  
           addrlen = sizeof(struct sockaddr_storage);  
⑩         cfd = accept(lfd, (struct sockaddr *) &claddr, &addrlen);  
           if (cfd == -1) {  
               errMsg("accept");  
               continue;  
           }  
  
⑪         if (getnameinfo((struct sockaddr *) &claddr, addrlen,  
                       host, NI_MAXHOST, service, NI_MAXSERV, 0) == 0)  
               snprintf(addrStr, ADDRSTRLEN, "(%s, %s)", host, service);  
           else  
               snprintf(addrStr, ADDRSTRLEN, "(?UNKNOWN?)");  
           printf("Connection from %s\n", addrStr);  
  
           /* Read client request, send sequence number back */  
  
⑫         if (readLine(cfd, reqLenStr, INT_LEN) <= 0) {  
               close(cfd);  
               continue;                  /* Failed read; skip request */  
           }  
  
⑬         reqLen = atoi(reqLenStr);  
           if (reqLen <= 0) {             /* Watch for misbehaving clients */  
               close(cfd);  
               continue;                  /* Bad request; skip it */  
           }  
  
⑭         snprintf(seqNumStr, INT_LEN, "%d\n", seqNum);  
           if (write(cfd, seqNumStr, strlen(seqNumStr)) != strlen(seqNumStr))  
               fprintf(stderr, "Error on write");  
  
⑮         seqNum += reqLen;              /* Update sequence number */  
  
           if (close(cfd) == -1)          /* Close connection */  
               errMsg("close");  
       }  
   }  
____________________________________________________ sockets/is_seqnum_sv.c

##### **Client program**

The client program is shown in [Listing 59-7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex7). This program accepts two arguments. The first argument, which is the name of the host on which the server is running, is mandatory. The optional second argument is the length of the sequence desired by the client. The default length is 1. The client performs the following steps:

• Call _getaddrinfo()_ to obtain a set of socket address structures suitable for connecting to a TCP server bound to the specified host ①. For the port number, the client specifies PORT_NUM.

• Enter a loop ② that iterates through the socket address structures returned by the previous step, until the client finds one that can be used to successfully create ③ and connect ④ a socket to the server. Since the client has not bound its socket, the _connect()_ call causes the kernel to assign an ephemeral port to the socket.

• Send an integer specifying the length of the client’s desired sequence ⑤. This integer is sent as a newline-terminated string.

• Read the sequence number sent back by the server (which is likewise a newline-terminated string) ⑥ and print it on standard output ⑦.

When we run the server and the client on the same host, we see the following:

$ ./is_seqnum_sv &  
[1] 4075  
$ ./is_seqnum_cl localhost             Client 1: requests 1 sequence number  
Connection from (localhost, 33273)     Server displays client address + port  
Sequence number: 0                     Client displays returned sequence number  
$ ./is_seqnum_cl localhost 10          Client 2: requests 10 sequence numbers  
Connection from (localhost, 33274)  
Sequence number: 1  
$ ./is_seqnum_cl localhost             Client 3: requests 1 sequence number  
Connection from (localhost, 33275)  
Sequence number: 11

Next, we demonstrate the use of _telnet_ for debugging this application:

$ telnet localhost 50000               Our server uses this port number  
                                       Empty line printed by telnet  
Trying 127.0.0.1...  
Connection from (localhost, 33276)  
Connected to localhost.  
Escape character is '^]'.  
1                                      Enter length of requested sequence  
12                                     telnet displays sequence number and  
Connection closed by foreign host.     detects that server closed connection

In the shell session log, we see that the kernel cycles sequentially through the ephemeral port numbers. (Other implementations exhibit similar behavior.) On Linux, this behavior is the result of an optimization to minimize hash lookups in the kernel’s table of local socket bindings. When the upper limit for these numbers is reached, the kernel recommences allocating an available number starting at the low end of the range (defined by the Linux-specific /proc/sys/net/ipv4/ip_local_port_range file).

**Listing 59-7:** A client that uses stream sockets

____________________________________________________ sockets/is_seqnum_cl.c  
  
   #include <netdb.h>  
   #include "is_seqnum.h"  
  
   int  
   main(int argc, char *argv[])  
   {  
       char *reqLenStr;                    /* Requested length of sequence */  
       char seqNumStr[INT_LEN];            /* Start of granted sequence */  
       int cfd;  
       ssize_t numRead;  
       struct addrinfo hints;  
       struct addrinfo *result, *rp;  
  
       if (argc < 2 || strcmp(argv[1], "--help") == 0)  
           usageErr("%s server-host [sequence-len]\n", argv[0]);  
  
       /* Call getaddrinfo() to obtain a list of addresses that  
           we can try connecting to */  
  
       memset(&hints, 0, sizeof(struct addrinfo));  
       hints.ai_canonname = NULL;  
       hints.ai_addr = NULL;  
       hints.ai_next = NULL;  
       hints.ai_family = AF_UNSPEC;                /* Allows IPv4 or IPv6 */  
       hints.ai_socktype = SOCK_STREAM;  
       hints.ai_flags = AI_NUMERICSERV;  
  
①     if (getaddrinfo(argv[1], PORT_NUM, &hints, &result) != 0)  
           errExit("getaddrinfo");  
  
       /* Walk through returned list until we find an address structure  
          that can be used to successfully connect a socket */  
  
②     for (rp = result; rp != NULL; rp = rp->ai_next) {  
③         cfd = socket(rp->ai_family, rp->ai_socktype, rp->ai_protocol);  
           if (cfd == -1)  
               continue;                           /* On error, try next address */  
  
④         if (connect(cfd, rp->ai_addr, rp->ai_addrlen) != -1)  
               break;                              /* Success */  
  
           /* Connect failed: close this socket and try next address */  
  
           close(cfd);  
       }  
  
       if (rp == NULL)  
           fatal("Could not connect socket to any address");  
  
       freeaddrinfo(result);  
  
       /* Send requested sequence length, with terminating newline */  
  
⑤     reqLenStr = (argc > 2) ? argv[2] : "1";  
       if (write(cfd, reqLenStr, strlen(reqLenStr)) !=  strlen(reqLenStr))  
           fatal("Partial/failed write (reqLenStr)");  
       if (write(cfd, "\n", 1) != 1)  
           fatal("Partial/failed write (newline)");  
  
       /* Read and display sequence number returned by server */  
  
⑥     numRead = readLine(cfd, seqNumStr, INT_LEN);  
       if (numRead == -1)  
           errExit("readLine");  
       if (numRead == 0)  
           fatal("Unexpected EOF from server");  
  
⑦     printf("Sequence number: %s", seqNumStr);           /* Includes '\n' */  
  
       exit(EXIT_SUCCESS);                                 /* Closes 'cfd' */  
  
   }  
____________________________________________________ sockets/is_seqnum_cl.c

### **59.12 An Internet Domain Sockets Library**

In this section, we use the functions presented in [Section 59.10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec10) to implement a library of functions to perform tasks commonly required for Internet domain sockets. (This library abstracts many of the steps shown in the example programs presented in [Section 59.11](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec11).) Since these functions employ the protocol-independent _getaddrinfo()_ and _getnameinfo()_ functions, they can be used with both IPv4 and IPv6. [Listing 59-8](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex8) shows the header file that declares these functions.

Many of the functions in this library have similar arguments:

• The _host_ argument is a string containing either a hostname or a numeric address (in IPv4 dotted-decimal, or IPv6 hex-string notation). Alternatively, _host_ can be specified as a NULL pointer to indicate that the loopback IP address is to be used.

• The _service_ argument is either a service name or a port number specified as a decimal string.

• The _type_ argument is a socket type, specified as either SOCK_STREAM or SOCK_DGRAM.

**Listing 59-8:** Header file for inet_sockets.c

____________________________________________________ sockets/inet_sockets.h  
  
#ifndef INET_SOCKETS_H  
#define INET_SOCKETS_H         /* Prevent accidental double inclusion */  
  
#include <sys/socket.h>  
#include <netdb.h>  
  
int inetConnect(const char *host, const char *service, int type);  
  
int inetListen(const char *service, int backlog, socklen_t *addrlen);  
  
int inetBind(const char *service, int type, socklen_t *addrlen);  
  
char *inetAddressStr(const struct sockaddr *addr, socklen_t addrlen,  
                char *addrStr, int addrStrLen);  
  
#define IS_ADDR_STR_LEN 4096  
                        /* Suggested length for string buffer that caller  
                           should pass to inetAddressStr(). Must be greater  
                           than (NI_MAXHOST + NI_MAXSERV + 4) */  
#endif  
____________________________________________________ sockets/inet_sockets.h

The _inetConnect()_ function creates a socket with the given socket _type_, and connects it to the address specified by _host_ and _service_. This function is designed for TCP or UDP clients that need to connect their socket to a server socket.

#include "inet_sockets.h"  
  
int inetConnect(const char *host, const char *service, int type);

Returns a file descriptor on success, or –1 on error

The file descriptor for the new socket is returned as the function result.

The _inetListen()_ function creates a listening stream (SOCK_STREAM) socket bound to the wildcard IP address on the TCP port specified by _service_. This function is designed for use by TCP servers.

#include "inet_sockets.h"  
  
int inetListen(const char *service, int backlog, socklen_t *addrlen);

Returns a file descriptor on success, or –1 on error

The file descriptor for the new socket is returned as the function result.

The _backlog_ argument specifies the permitted backlog of pending connections (as for _listen()_).

If _addrlen_ is specified as a non-NULL pointer, then the location it points to is used to return the size of the socket address structure corresponding to the returned file descriptor. This value allows us to allocate a socket address buffer of the appropriate size to be passed to a later _accept()_ call if we want to obtain the address of a connecting client.

The _inetBind()_ function creates a socket of the given _type_, bound to the wildcard IP address on the port specified by _service_ and _type_. (The socket _type_ indicates whether this is a TCP or UDP service.) This function is designed (primarily) for UDP servers and clients to create a socket bound to a specific address.

#include "inet_sockets.h"  
  
int inetBind(const char *service, int type, socklen_t *addrlen);

Returns a file descriptor on success, or –1 on error

The file descriptor for the new socket is returned as the function result.

As with _inetListen()_, _inetBind()_ returns the length of the associated socket address structure for this socket in the location pointed to by _addrlen_. This is useful if we want to allocate a buffer to pass to _recvfrom()_ in order to obtain the address of the socket sending a datagram. (Many of the steps required for _inetListen()_ and _inetBind()_ are the same, and these steps are implemented within the library by a single function, _inetPassiveSocket()_.)

The _inetAddressStr()_ function converts an Internet socket address to printable form.

#include "inet_sockets.h"  
  
char *inetAddressStr(const struct sockaddr *addr, socklen_t addrlen,  
                     char *addrStr, int addrStrLen);

Returns pointer to _addrStr_, a string containing host and service name

Given a socket address structure in _addr_, whose length is specified in _addrlen_, _inetAddressStr()_ returns a null-terminated string containing the corresponding hostname and port number in the following form:

(hostname, port-number)

The string is returned in the buffer pointed to by _addrStr_. The caller must specify the size of this buffer in _addrStrLen_. If the returned string would exceed _(addrStrLen – 1)_ bytes, it is truncated. The constant IS_ADDR_STR_LEN defines a suggested size for the _addrStr_ buffer that should be large enough to handle all possible return strings. As its function result, _inetAddressStr()_ returns _addrStr_.

The implementation of the functions described in this section is shown in [Listing 59-9](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex9).

**Listing 59-9:** An Internet domain sockets library

____________________________________________________ sockets/inet_sockets.c  
  
#define _BSD_SOURCE             /* To get NI_MAXHOST and NI_MAXSERV  
                                   definitions from <netdb.h> */  
#include <sys/socket.h>  
#include <netinet/in.h>  
#include <arpa/inet.h>  
#include <netdb.h>  
#include "inet_sockets.h"       /* Declares functions defined here */  
#include "tlpi_hdr.h"  
  
int  
inetConnect(const char *host, const char *service, int type)  
{  
    struct addrinfo hints;  
    struct addrinfo *result, *rp;  
    int sfd, s;  
  
    memset(&hints, 0, sizeof(struct addrinfo));  
    hints.ai_canonname = NULL;  
    hints.ai_addr = NULL;  
    hints.ai_next = NULL;  
    hints.ai_family = AF_UNSPEC;        /* Allows IPv4 or IPv6 */  
    hints.ai_socktype = type;  
  
    s = getaddrinfo(host, service, &hints, &result);  
    if (s != 0) {  
        errno = ENOSYS;  
        return -1;  
    }  
  
    /* Walk through returned list until we find an address structure  
       that can be used to successfully connect a socket */  
  
    for (rp = result; rp != NULL; rp = rp->ai_next) {  
        sfd = socket(rp->ai_family, rp->ai_socktype, rp->ai_protocol);  
        if (sfd == -1)  
            continue;                   /* On error, try next address */  
  
        if (connect(sfd, rp->ai_addr, rp->ai_addrlen) != -1)  
            break;                      /* Success */  
  
        /* Connect failed: close this socket and try next address */  
  
        close(sfd);  
    }  
  
    freeaddrinfo(result);  
  
    return (rp == NULL) ? -1 : sfd;  
}  
  
static int              /* Public interfaces: inetBind() and inetListen() */  
inetPassiveSocket(const char *service, int type, socklen_t *addrlen,  
                  Boolean doListen, int backlog)  
{  
    struct addrinfo hints;  
    struct addrinfo *result, *rp;  
    int sfd, optval, s;  
  
    memset(&hints, 0, sizeof(struct addrinfo));  
    hints.ai_canonname = NULL;  
    hints.ai_addr = NULL;  
    hints.ai_next = NULL;  
    hints.ai_socktype = type;  
    hints.ai_family = AF_UNSPEC;        /* Allows IPv4 or IPv6 */  
    hints.ai_flags = AI_PASSIVE;        /* Use wildcard IP address */  
  
    s = getaddrinfo(NULL, service, &hints, &result);  
    if (s != 0)  
        return -1;  
  
    /* Walk through returned list until we find an address structure  
       that can be used to successfully create and bind a socket */  
  
    optval = 1;  
    for (rp = result; rp != NULL; rp = rp->ai_next) {  
        sfd = socket(rp->ai_family, rp->ai_socktype, rp->ai_protocol);  
        if (sfd == -1)  
            continue;                   /* On error, try next address */  
  
        if (doListen) {  
            if (setsockopt(sfd, SOL_SOCKET, SO_REUSEADDR, &optval,  
                    sizeof(optval)) == -1) {  
                close(sfd);  
                freeaddrinfo(result);  
                return -1;  
            }  
        }  
  
        if (bind(sfd, rp->ai_addr, rp->ai_addrlen) == 0)  
            break;                      /* Success */  
  
        /* bind() failed: close this socket and try next address */  
  
        close(sfd);  
    }  
  
    if (rp != NULL && doListen) {  
        if (listen(sfd, backlog) == -1) {  
            freeaddrinfo(result);  
            return -1;  
        }  
    }  
  
    if (rp != NULL && addrlen != NULL)  
        *addrlen = rp->ai_addrlen;      /* Return address structure size */  
  
    freeaddrinfo(result);  
  
    return (rp == NULL) ? -1 : sfd;  
}  
  
int  
inetListen(const char *service, int backlog, socklen_t *addrlen)  
{  
    return inetPassiveSocket(service, SOCK_STREAM, addrlen, TRUE, backlog);  
}  
  
int  
inetBind(const char *service, int type, socklen_t *addrlen)  
{  
    return inetPassiveSocket(service, type, addrlen, FALSE, 0);  
}  
  
char *  
inetAddressStr(const struct sockaddr *addr, socklen_t addrlen,  
               char *addrStr, int addrStrLen)  
{  
    char host[NI_MAXHOST], service[NI_MAXSERV];  
  
    if (getnameinfo(addr, addrlen, host, NI_MAXHOST,  
                    service, NI_MAXSERV, NI_NUMERICSERV) == 0)  
        snprintf(addrStr, addrStrLen, "(%s, %s)", host, service);  
    else  
        snprintf(addrStr, addrStrLen, "(?UNKNOWN?)");  
  
    return addrStr;  
}  
____________________________________________________ sockets/inet_sockets.c

### **59.13 Obsolete APIs for Host and Service Conversions**

In the following sections, we describe the older, now obsolete functions for converting host names and service names to and from binary and presentation formats. Although new programs should perform these conversions using the modern functions described earlier in this chapter, a knowledge of the obsolete functions is useful because we may encounter them in older code.

#### **59.13.1 The _inet_aton()_ and _inet_ntoa()_ Functions**

The _inet_aton()_ and _inet_ntoa()_ functions convert IPv4 addresses between dotted-decimal notation and binary form (in network byte order). These functions are nowadays made obsolete by _inet_pton()_ and _inet_ntop()_.

The _inet_aton()_ (“ASCII to network”) function converts the dotted-decimal string pointed to by _str_ into an IPv4 address in network byte order, which is returned in the _in_addr_ structure pointed to by _addr_.

#include <arpa/inet.h>  
  
int inet_aton(const char *str, struct in_addr *addr);

Returns 1 (true) if _str_ is a valid dotted-decimal address, or 0 (false) on error

The _inet_aton()_ function returns 1 if the conversion was successful, or 0 if _str_ was invalid.

The numeric components of the string given to _inet_aton()_ need not be decimal. They can be octal (specified by a leading 0) or hexadecimal (specified by a leading 0x or 0X). Furthermore, _inet_aton()_ supports shorthand forms that allow an address to be specified using fewer than four numeric components. (See the _inet(3)_ manual page for details.) The term _numbers-and-dots notation_ is used for the more general address strings that employ these features.

SUSv3 doesn’t specify _inet_aton()_. Nevertheless, this function is available on most implementations. On Linux, we must define one of the feature test macros _BSD_SOURCE, _SVID_SOURCE, or _GNU_SOURCE in order to obtain the declaration of _inet_aton()_ from <arpa/inet.h>.

The _inet_ntoa()_ (“network to ASCII”) function performs the converse of _inet_aton()_.

#include <arpa/inet.h>  
  
char *inet_ntoa(struct in_addr addr);

Returns pointer to (statically allocated) dotted-decimal string version of _addr_

Given an _in_addr_ structure (a 32-bit IPv4 address in network byte order), _inet_ntoa()_ returns a pointer to a (statically allocated) string containing the address in dotted-decimal notation.

Because the string returned by _inet_ntoa()_ is statically allocated, it is overwritten by successive calls.

#### **59.13.2 The _gethostbyname()_ and _gethostbyaddr()_ Functions**

The _gethostbyname()_ and _gethostbyaddr()_ functions allow conversion between hostnames and IP addresses. These functions are nowadays made obsolete by _getaddrinfo()_ and _getnameinfo()_.

#include <netdb.h>  
  
extern int h_errno;  
  
struct hostent *gethostbyname(const char *name);  
struct hostent *gethostbyaddr(const void *addr, socklen_t len, int type);

Both return pointer to (statically allocated) _hostent_ structure on success, or NULL on error

The _gethostbyname()_ function resolves the hostname given in _name_, returning a pointer to a statically allocated _hostent_ structure containing information about that hostname. This structure has the following form:

struct hostent {  
    char *h_name;               /* Official (canonical) name of host */  
    char **h_aliases;           /* NULL-terminated array of pointers  
                                   to alias strings */  
    int    h_addrtype;          /* Address type (AF_INET or AF_INET6) */  
    int    h_length;            /* Length (in bytes) of addresses pointed  
                                   to by h_addr_list (4 bytes for AF_INET,  
                                   16 bytes for AF_INET6) */  
    char **h_addr_list;         /* NULL-terminated array of pointers to  
                                   host IP addresses (in_addr or in6_addr  
                                   structures) in network byte order */  
};  
  
#define h_addr h_addr_list[0]

The _h_name_ field returns the official name of the host, as a null-terminated string. The _h_aliases_ fields points to an array of pointers to null-terminated strings containing aliases (alternative names) for this hostname.

The _h_addr_list_ field is an array of pointers to IP address structures for this host. (A multihomed host has more than one address.) This list consists of either _in_addr_ or _in6_addr_ structures. We can determine the type of these structures from the _h_addrtype_ field, which contains either AF_INET or AF_INET6, and their length from the _h_length_ field. The _h_addr_ definition is provided for backward compatibility with earlier implementations (e.g., 4.2BSD) that returned just one address in the _hostent_ structure. Some existing code relies on this name (and thus is not multihomed-host aware).

With modern versions of _gethostbyname()_, _name_ can also be specified as a numeric IP address string; that is, numbers-and-dots notation for IPv4 or hex-string notation for IPv6. In this case, no lookup is performed; instead, _name_ is copied into the _h_name_ field of the _hostent_ structure, and _h_addr_list_ is set to the binary equivalent of _name_.

The _gethostbyaddr()_ function performs the converse of _gethostbyname()_. Given a binary IP address, it returns a _hostent_ structure containing information about the host with that address.

On error (e.g., a name could not be resolved), both _gethostbyname()_ and _gethostbyaddr()_ return a NULL pointer and set the global variable _h_errno_. As the name suggests, this variable is analogous to _errno_ (possible values placed in this variable are described in the _gethostbyname(3)_ manual page), and the _herror()_ and _hstrerror()_ functions are analogous to _perror()_ and _strerror()_.

The _herror()_ function displays (on standard error) the string given in _str_, followed by a colon (:), and then a message for the current error in _h_errno_. Alternatively, we can use _hstrerror()_ to obtain a pointer to a string corresponding to the error value specified in _err_.

#define _BSD_SOURCE           /* Or _SVID_SOURCE or _GNU_SOURCE */  
#include <netdb.h>  
  
void herror(const char *str);  
  
const char *hstrerror(int err);

Returns pointer to _h_errno_ error string corresponding to _err_

[Listing 59-10](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex10) demonstrates the use of _gethostbyname()_. This program displays _hostent_ information for each of the hosts named on its command line. The following shell session demonstrates the use of this program:

$ ./t_gethostbyname www.jambit.com  
Canonical name: jamjam1.jambit.com  
        alias(es):      www.jambit.com  
        address type:   AF_INET  
        address(es):    62.245.207.90

**Listing 59-10:** Using _gethostbyname()_ to retrieve host information

_________________________________________________ sockets/t_gethostbyname.c  
  
#define _BSD_SOURCE     /* To get hstrerror() declaration from <netdb.h> */  
#include <netdb.h>  
#include <netinet/in.h>  
#include <arpa/inet.h>  
#include "tlpi_hdr.h"  
  
int  
main(int argc, char *argv[])  
{  
    struct hostent *h;  
    char **pp;  
    char str[INET6_ADDRSTRLEN];  
  
    for (argv++; *argv != NULL; argv++) {  
        h = gethostbyname(*argv);  
        if (h == NULL) {  
            fprintf(stderr, "gethostbyname() failed for '%s': %s\n",  
                    *argv, hstrerror(h_errno));  
            continue;  
        }  
  
        printf("Canonical name: %s\n", h->h_name);  
  
        printf("        alias(es):     ");  
        for (pp = h->h_aliases; *pp != NULL; pp++)  
            printf(" %s", *pp);  
        printf("\n");  
  
        printf("        address type:   %s\n",  
                (h->h_addrtype == AF_INET) ? "AF_INET" :  
                (h->h_addrtype == AF_INET6) ? "AF_INET6" : "???");  
  
        if (h->h_addrtype == AF_INET || h->h_addrtype == AF_INET6) {  
            printf("        address(es):   ");  
            for (pp = h->h_addr_list; *pp != NULL; pp++)  
                printf(" %s", inet_ntop(h->h_addrtype, *pp,  
                                        str, INET6_ADDRSTRLEN));  
            printf("\n");  
        }  
    }  
  
    exit(EXIT_SUCCESS);  
}  
_________________________________________________ sockets/t_gethostbyname.c

#### **59.13.3 The _getservbyname()_ and _getservbyport()_ Functions**

The _getservbyname()_ and _getservbyport()_ functions retrieve records from the /etc/services file ([Section 59.9](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec09)). These functions are nowadays made obsolete by _getaddrinfo()_ and _getnameinfo()_.

#include <netdb.h>  
  
struct servent *getservbyname(const char *name, const char *proto);  
struct servent *getservbyport(int port, const char *proto);

Both return pointer to a (statically allocated) _servent_ structure on success, or NULL on not found or error

The _getservbyname()_ function looks up the record whose service name (or one of its aliases) matches _name_ and whose protocol matches _proto_. The _proto_ argument is a string such as _tcp_ or _udp_, or it can be NULL. If _proto_ is specified as NULL, any record whose service name matches _name_ is returned. (This is usually sufficient since, where both UDP and TCP records with the same name exist in the /etc/services file, they normally have the same port number.) If a matching record is found, then _getservbyname()_ returns a pointer to a statically allocated structure of the following type:

struct servent {  
    char  *s_name;         /* Official service name */  
    char **s_aliases;      /* Pointers to aliases (NULL-terminated) */  
    int    s_port;         /* Port number (in network byte order) */  
    char  *s_proto;        /* Protocol */  
};

Typically, we call _getservbyname()_ only in order to obtain the port number, which is returned in the _s_port_ field.

The _getservbyport()_ function performs the converse of _getservbyname()_. It returns a _servent_ record containing information from the /etc/services record whose port number matches _port_ and whose protocol matches _proto_. Again, we can specify _proto_ as NULL, in which case the call will return any record whose port number matches the one specified in _port_. (This may not return the desired result in the few cases mentioned above where the same port number maps to different service names in UDP and TCP.)

An example of the use of the _getservbyname()_ function is provided in the file sockets/t_getservbyname.c in the source code distribution for this book.

### **59.14 UNIX Versus Internet Domain Sockets**

When writing applications that communicate over a network, we must necessarily use Internet domain sockets. However, when using sockets to communicate between applications on the same system, we have the choice of using either Internet or UNIX domain sockets. In this case, which domain should we use and why?

Writing an application using just Internet domain sockets is often the simplest approach, since it will work on both a single host and across a network. However, there are some reasons why we may choose to use UNIX domain sockets:

• On some implementations, UNIX domain sockets are faster than Internet domain sockets.

• We can use directory (and, on Linux, file) permissions to control access to UNIX domain sockets, so that only applications with a specified user or group ID can connect to a listening stream socket or send a datagram to a datagram socket. This provides a simple method of authenticating clients. With Internet domain sockets, we need to do rather more work if we wish to authenticate clients.

• Using UNIX domain sockets, we can pass open file descriptors and sender credentials, as summarized in [Section 61.13.3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch61.xhtml#ch61lev2sec10).

### **59.15 Further Information**

There is a wealth of printed and online resources on TCP/IP and the sockets API:

• The key book on network programming with the sockets API is [[Stevens at al., 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib93)]. [[Snader, 2000](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib87)] adds some useful guidelines on sockets programming.

• [[Stevens, 1994](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib91)] and [[Wright & Stevens, 1995](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib111)] describe TCP/IP in detail. [[Comer, 2000](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib12)], [[Comer & Stevens, 1999](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib13)], [[Comer & Stevens, 2000](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib12)], [[Kozierok, 2005](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib49)], and [[Goralksi, 2009](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib36)] also provide good coverage of the same material.

• [[Tanenbaum, 2002](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib100)] provides general background on computer networks.

• [[Herbert, 2004](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib42)] describes the details of the Linux 2.6 TCP/IP stack.

• The GNU C library manual (online at _[http://www.gnu.org/](http://www.gnu.org/)_) has an extensive discussion of the sockets API.

• The IBM Redbook, _TCP/IP Tutorial and Technical Overview_, provides lengthy coverage of networking concepts, TCP/IP internals, the sockets API, and a host of related topics. It is freely downloadable from _[http://www.redbooks.ibm.com/](http://www.redbooks.ibm.com/)_.

• [[Gont, 2008](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib32)] and [[Gont, 2009b](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib34)] provide security assessments of IPv4 and TCP.

• The Usenet newsgroup _comp.protocols.tcp-ip_ is dedicated to questions related to the TCP/IP networking protocols.

• [[Sarolahti & Kuznetsov, 2002](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/bib.xhtml#bib85)] describes congestion control and other details of the Linux TCP implementation.

• Linux-specific information can be found in the following manual pages: _socket(7)_, _ip(7)_, _raw(7)_, _tcp(7)_, _udp(7)_, and _packet(7)_.

• See also the RFC list in [Section 58.7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch58.xhtml#ch58lev1sec07).

### **59.16 Summary**

Internet domain sockets allow applications on different hosts to communicate via a TCP/IP network. An Internet domain socket address consists of an IP address and a port number. In IPv4, an IP address is a 32-bit number; in IPv6, it is a 128-bit number. Internet domain datagram sockets operate over UDP, providing connectionless, unreliable, message-oriented communication. Internet domain stream sockets operate over TCP, and provide a reliable, bidirectional, byte-stream communication channel between two connected applications.

Different computer architectures use different conventions for representing data types. For example, integers may be stored in little-endian or big-endian form, and different computers may use different numbers of bytes to represent numeric types such as _int_ or _long_. These differences mean that we need to employ some architecture-independent representation when transferring data between heterogeneous machines connected via a network. We noted that various marshalling standards exist to deal with this problem, and also described a simple solution used by many applications: encoding all transmitted data in text form, with fields delimited by a designated character (usually a newline).

We looked at a range of functions that can be used to convert between (numeric) string representations of IP addresses (dotted-decimal for IPv4 and hex-string for IPv6) and their binary equivalents. However, it is generally preferable to use host and service names rather than numbers, since names are easier to remember and continue to be usable, even if the corresponding number is changed. We looked at various functions that convert host and service names to their numeric equivalents and vice versa. The modern function for translating host and service names into socket addresses is _getaddrinfo()_, but it is common to see the historical functions _gethostbyname()_ and _getservbyname()_ in existing code.

Consideration of hostname conversions led us into a discussion of DNS, which implements a distributed database for a hierarchical directory service. The advantage of DNS is that the management of the database is not centralized. Instead, local zone administrators update changes for the hierarchical component of the database for which they are responsible, and DNS servers communicate with one another in order to resolve a hostname.

### **59.17 Exercises**

**59-1.**   When reading large quantities of data, the _readLine()_ function shown in [Listing 59-1](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex1) is inefficient, since a system call is required to read each character. A more efficient interface would read a block of characters into a buffer and extract a line at a time from this buffer. Such an interface might consist of two functions. The first of these functions, which might be called _readLineBufInit(fd, &rlbuf)_, initializes the bookkeeping data structure pointed to by _rlbuf_. This structure includes space for a data buffer, the size of that buffer, and a pointer to the next “unread” character in that buffer. It also includes a copy of the file descriptor given in the argument _fd_. The second function, _readLineBuf(&rlbuf)_, returns the next line from the buffer associated with _rlbuf_. If required, this function reads a further block of data from the file descriptor saved in _rlbuf_. Implement these two functions. Modify the programs in [Listing 59-6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex6) (is_seqnum_sv.c) and [Listing 59-7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex7) (is_seqnum_cl.c) to use these functions.

**59-2.**   Modify the programs in [Listing 59-6](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex6) (is_seqnum_sv.c) and [Listing 59-7](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex7) (is_seqnum_cl.c) to use the _inetListen()_ and _inetConnect()_ functions provided in [Listing 59-9](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59ex9) (inet_sockets.c).

**59-3.**   Write a UNIX domain sockets library with an API similar to the Internet domain sockets library shown in [Section 59.12](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch59.xhtml#ch59lev1sec12). Rewrite the programs in [Listing 57-3](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch57.xhtml#ch57ex3) (us_xfr_sv.c, on [page 1168](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch57.xhtml#page_1168)) and [Listing 57-4](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch57.xhtml#ch57ex4) (us_xfr_cl.c, on [page 1169](https://learning.oreilly.com/library/view/the-linux-programming/9781593272203/xhtml/ch57.xhtml#page_1169)) to use this library.

**59-4.**   Write a network server that stores name-value pairs. The server should allow names to be added, deleted, modified, and retrieved by clients. Write one or more client programs to test the server. Optionally, implement some kind of security mechanism that allows only the client that created the name to delete it or to modify the value associated with it.

**59-5.**   Suppose that we create two Internet domain datagram sockets, bound to specific addresses, and connect the first socket to the second. What happens if we create a third datagram socket and try to send (_sendto()_) a datagram via that socket to the first socket? Write a program to determine the answer.