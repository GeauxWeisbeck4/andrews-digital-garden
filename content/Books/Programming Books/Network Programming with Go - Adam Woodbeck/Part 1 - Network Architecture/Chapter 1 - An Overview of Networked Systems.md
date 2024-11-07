---
id: 01JC4BWEXTKKH5VPRWVG54TJZ7
modified: 2024-11-07T17:29:15-05:00
title: Chapter 1 - An Overview of Networked Systems
tags:
  - networking
  - go-lang
  - programming
  - books
  - tcp/ip
---
# 1  
AN OVERVIEW OF NETWORKED SYSTEMS

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

In the digital age, an increasing number of devices communicate over computer networks. A _computer network_ is a connection between two or more devices, or _nodes_, that allows each node to share data. These connections aren’t inherently reliable or secure. Thankfully, Go’s standard library and its rich ecosystem are well suited for writing secure, reliable network applications.

This chapter will give you the foundational knowledge needed for this book’s exercises. You’ll learn about the structure of networks and how networks use protocols to communicate.

## Choosing a Network Topology

The organization of nodes in a network is called its _topology._ A network’s topology can be as simple as a single connection between two nodes or as complex as a layout of nodes that don’t share a direct connection but are nonetheless able to exchange data. That’s generally the case for connections between your computer and nodes on the internet. Topology types fall into six basic categories: point-to-point, daisy chain, bus, ring, star, and mesh.

In the simplest network, _point-to-point_, two nodes share a single connection ([Figure 1-1](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-1)). This type of network connection is uncommon, though it is useful when direct communication is required between two nodes.

![f01001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01001.png)

Figure 1-1: A direct connection between two nodes

A series of point-to-point connections creates a _daisy chain_. In the daisy chain in [Figure 1-2](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-2), traffic from node C, destined for node F, must traverse nodes D and E. Intermediate nodes between an origin node and a destination node are commonly known as _hops_. You are unlikely to encounter this topology in a modern network.

![f01002](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01002.png)

Figure 1-2: Point-to-point segments joined in a daisy chain

_Bus_ topology nodes share a common network link. Wired bus networks aren’t common, but this type of topology drives wireless networks. The nodes on a wired network see all the traffic and selectively ignore or accept it, depending on whether the traffic is intended for them. When node H sends traffic to node L in the bus diagram in [Figure 1-3](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-3), nodes I, J, K, and M receive the traffic but ignore it. Only node L accepts the data because it’s the intended recipient. Although wireless clients can see each other’s traffic, traffic is usually encrypted.

![f01003](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01003.png)

Figure 1-3: Nodes connected in a bus topology

A _ring_ topology, which was used in some fiber-optic network deployments, is a closed loop in which data travels in a single direction. In [Figure 1-4](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-4), for example, node N could send a message destined for node R by way of nodes O, P, and Q. Nodes O, P, and Q retransmit the message until it reaches node R. If node P fails to retransmit the message, it will never reach its destination. Because of this design, the slowest node can limit the speed at which data travels. Assuming traffic travels clockwise and node Q is the slowest, node Q slows traffic sent from node O to node N. However, traffic sent from node N to node O is not limited by node Q’s slow speed since that traffic does not traverse node Q.

![f01004](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01004.png)

Figure 1-4: Nodes arranged in a ring, with traffic traveling in a single direction

In a _star_ topology, a central node has individual point-to-point connections to all other nodes. You will likely encounter this network topology in wired networks. The central node, as shown in [Figure 1-5](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-5), is often a _network switch_, which is a device that accepts data from the origin nodes and retransmits data to the destination nodes, like a postal service. Adding nodes is a simple matter of connecting them to the switch. Data can traverse only a single hop within this topology.

![f01005](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01005.png)

Figure 1-5: Nodes connected to a central node, which handles traffic between nodes

Every node in a fully connected _mesh_ network has a direct connection to every other node ([Figure 1-6](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-6)). This topology eliminates single points of failure because the failure of a single node doesn’t affect traffic between any other nodes on the network. On the other hand, costs and complexity increase as the number of nodes increases, making this topology untenable for large-scale networks. This is another topology you may encounter only in larger wireless networks.

![f01006](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01006.png)

Figure 1-6: Interconnected nodes in a mesh network

You can also create a hybrid network topology by combining two or more basic topologies. Real-world networks are rarely composed of just one network topology. Rather, you are likely to encounter hybrid topologies. [Figure 1-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-7) shows two examples. The _star-ring_ hybrid network is a series of ring networks connected to a central node. The _star-bus_ hybrid network is a hierarchical topology formed by the combination of bus and star network topologies.

![f01007](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01007.png)

Figure 1-7: The star-ring and star-bus hybrid topologies

Hybrid topologies are meant to improve reliability, scalability, and flexibility by taking advantage of each topology’s strengths and by limiting the disadvantages of each topology to individual network segments.

For example, the failure of the central node in the _star-ring_ hybrid in [Figure 1-7](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-7) would affect inter-ring communication only. Each ring network would continue to function normally despite its isolation from the other rings. The failure of a single node in a ring would be much easier to diagnose in a star-ring hybrid network than in a single large ring network. Also, the outage would affect only a subset of the overall network.

## Bandwidth vs. Latency

Network _bandwidth_ is the amount of data we can send over a network connection in an interval of time. If your internet connection is advertised as _100Mbps download_, that means your internet connection should theoretically be able to transfer up to 100 megabits every second from your internet service provider (ISP) to your modem.

ISPs inundate us with advertisements about the amount of bandwidth they offer, so much so that it’s easy for us to fixate on bandwidth and equate it with the speed of the connection. However, faster doesn’t always mean greater performance. It may seem counterintuitive, but a lower-bandwidth network connection may seem to have better performance than a higher-bandwidth network connection because of one characteristic: latency.

Network _latency_ is a measure of the time that passes between sending a network resource request and receiving a response. An example of latency is the delay that occurs between clicking a link on a website and the site’s rendering the resulting page. You’ve probably experienced the frustration of clicking a link that fails to load before your web browser gives up on ever receiving a reply from the server. This happens when the latency is greater than the maximum amount of time your browser will wait for a reply.

High latency can negatively impact the user experience, lead to attacks that make your service inaccessible to its users, and drive users away from your software or service. The importance of managing latency in network software is often underappreciated by software developers. Don’t fall into the trap of thinking that bandwidth is all that matters for optimal network performance.

A website’s latency comes from several sources: the network latency between the client and server, the time it takes to retrieve data from a data store, the time it takes to compile dynamic content on the server side, and the time it takes for the web browser to render the page. If a user clicks a link and the page takes too long to render, the user likely won’t stick around for the results, and the latency will drive traffic away from your application. Keeping latency to a minimum while writing network software, be it web applications or application-programming interfaces, will pay dividends by improving the user experience and your application’s ranking in popular search engines.

You can address the most common sources of latency in several ways. First, you can reduce both the distance and the number of hops between users and your service by using a content delivery network (CDN) or cloud infrastructure to locate your service near your users. Optimizing the request and response sizes will further reduce latency. Incorporating a caching strategy in your network applications can have dramatic effects on performance. Finally, taking advantage of Go’s concurrency to minimize server-side blocking of the response can help. We’ll focus on this in the later chapters of this book.

## The Open Systems Interconnection Reference Model

In the 1970s, as computer networks became increasingly complex, researchers created the _Open Systems Interconnection (OSI) reference model_ to standardize networking. The OSI reference model serves as a framework for the development of and communication about protocols. _Protocols_ are rules and procedures that determine the format and order of data sent over a network. For example, communication using the _Transmission Control Protocol__(TCP)_ requires the recipient of a message to reply with an acknowledgment of receipt. Otherwise, TCP may retransmit the message.

Although OSI is less relevant today than it once was, it’s still important to be familiar with it so you’ll understand common concepts, such as lower-level networking and routing, especially with respect to the involved hardware.

### The Hierarchal Layers of the OSI Reference Model

The OSI reference model divides all network activities into a strict hierarchy composed of seven layers. Visual representations of the OSI reference model, like the one in [Figure 1-8](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-8), arrange the layers into a stack, with Layer 7 at the top and Layer 1 at the bottom.

![f01008](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01008.png)

Figure 1-8: Seven layers of the OSI reference model

It’s easy to interpret these layer designations as independent units of code. Rather, they describe abstractions we ascribe to parts of our software. For example, there is no _Layer 7_ library you can incorporate into your software. But you can say that the software you wrote implements a service at Layer 7. The seven layers of the OSI model are as follows:

1. Layer 7—application layer Your network applications and libraries most often interact with the application layer, which is responsible for identifying hosts and retrieving resources. Web browsers, Skype, and bit torrent clients are examples of Layer 7 applications.
2. Layer 6—presentation layer The presentation layer prepares data for the network layer when that data is moving down the stack, and it presents data to the application layer when that data moves up the stack. Encryption, decryption, and data encoding are examples of Layer 6 functions.
3. Layer 5—session layer The session layer manages the connection life cycle between nodes on a network. It’s responsible for establishing the connection, managing connection time-outs, coordinating the mode of operation, and terminating the connection. Some Layer 7 protocols rely on services provided by Layer 5.
4. Layer 4—transport layer The transport layer controls and coordinates the transfer of data between two nodes while maintaining the reliability of the transfer. Maintaining the reliability of the transfer includes correcting errors, controlling the speed of data transfer, chunking or segmenting the data, retransmitting missing data, and acknowledging received data. Often protocols in this layer might retransmit data if the recipient doesn’t acknowledge receipt of the data.
5. Layer 3—network layer The network layer is responsible for transmitting data between nodes. It allows you to send data to a network address without having a direct point-to-point connection to the remote node. OSI does not require protocols in this layer to provide reliable transport or report transmission errors to the sender. The network layer is home to network management protocols involved in routing, addressing, multicasting, and traffic control. The next chapter covers these topics.
6. Layer 2—data link layer The data link layer handles data transfers between two directly connected nodes. For example, the data link layer facilitates data transfer from a computer to a switch and from the switch to another computer. Protocols in this layer identify and attempt to correct errors on the physical layer.
7. The data link layer’s retransmission and flow control functions are dependent on the underlying physical medium. For example, Ethernet does not retransmit incorrect data, whereas wireless does. This is because bit errors on Ethernet networks are infrequent, whereas they’re common over wireless. Protocols further up the network protocol stack can ensure that the data transmission is reliable if this layer doesn’t do so, though generally with less efficiency.
8. Layer 1—physical layer The physical layer converts bits from the network stack to electrical, optic, or radio signals suitable for the underlying physical medium and from the physical medium back into bits. This layer controls the bit rate. The bit rate is the data speed limit. A gigabit per second bit rate means data can travel at a maximum of 1 billion bits per second between the origin and destination.

A common confusion when discussing network transmission rates is using bytes per second instead of bits per second. We count the number of zeros and ones, or _bits_, we can transfer per second. Therefore, network transmission rates are measured in bits per second. We use bytes per second when discussing the amount of data transferred.

If your ISP advertises a 100Mbps download rate, that doesn’t mean you can download a 100MB file in one second. Rather, it may take closer to eight seconds under ideal network conditions. It’s appropriate to say we can transfer a maximum of 12.5MB per second over the 100Mbps connection.

### Sending Traffic by Using Data Encapsulation

_Encapsulation_ is a method of hiding implementation details or making only relevant details available to the recipient. Think of encapsulation as being like a package you send through the postal service. We could say that the envelope encapsulates its contents. In doing so, it may include the destination address or other crucial details used by the next leg of its journey. The actual contents of your package are irrelevant; only the details on the package are important for transport.

As data travels down the stack, it’s encapsulated by the layer below. We typically call the data traveling down the stack a _payload_, although you might see it referred to as a _message body_. The literature uses the term _service data unit__(SDU)_. For example, the transport layer encapsulates payloads from the session layer, which in turn encapsulates payloads from the presentation layer. When the payload moves up the stack, each layer strips the header information from the previous stack.

Even protocols that operate in a single OSI layer use data encapsulation. Take version 1 of the _HyperText Transfer Protocol__(HTTP/1)_, for example, a Layer 7 protocol that both the client and the server use to exchange web content. HTTP defines a complete message, including header information, that the client sends from its Layer 7 to the server’s Layer 7; the network stack delivers the client’s request to the HTTP server application. The HTTP server application initiates a response to its network stack, which creates a Layer 7 payload and sends it back to the client’s Layer 7 application ([Figure 1-9](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-9)).

Communication between the client and the server on the same layer is called _horizontal communication,_ a term that makes it sound like a single-layer protocol on the client directly communicates with its counterpart on the server. In fact, in horizontal communication, data must travel all the way down the client’s stack, then back up the server’s stack.

For example, [Figure 1-10](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-10) shows how an HTTP request traverses the stack.

Generally, a payload travels down the client’s network stack, over physical media to the server, and up the server’s network stack to its corresponding layer. The result is that data sent from one layer at the origin node arrives at the same layer on the destination node. The server’s response takes the same path in the opposite direction. On the client’s side, Layer 6 receives Layer 7’s payload, then encapsulates the payload with a header to create Layer 6’s payload. Layer 5 receives Layer 6’s payload, adds its own header, and sends its payload on to Layer 4, where we’re introduced to our first transmission protocol: TCP.

![f01009](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01009.png)

Figure 1-9: Horizontal communication from the client to the server and back

![f01010](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01010.png)

Figure 1-10: An HTTP request traveling from Layer 7 on the client to Layer 7 on the server

TCP is a Layer 4 protocol whose payloads are also known as _segments_ or _datagrams_. TCP accepts Layer 5’s payload and adds its header before sending the segment on to Layer 3. The _Internet Protocol__(IP)_ at Layer 3 receives the TCP segment and encapsulates it with a header to create Layer 3’s payload, which is known as a _packet_. Layer 2 accepts the packet and adds a header and a footer, creating its payload, called a _frame_. Layer 2’s header translates the recipient’s IP address into a _media access control__(MAC)_ address, which is a unique identifier assigned to the node’s network interface. Its footer contains a _frame check sequence__(FCS)_, which is a checksum to facilitate error detection. Layer 1 receives Layer 2’s payload in the form of bits and sends the bits to the server.

The server’s Layer 1 receives the bits, converts them to a frame, and sends the frame up to Layer 2. Layer 2 strips its header and footer from the frame and passes the packet up to Layer 3. The process of reversing each layer’s encapsulation continues up the stack until the payload reaches Layer 7. Finally, the HTTP server receives the client’s request from the network stack.

## The TCP/IP Model

Around the same time as researchers were developing the OSI reference model, the Defense Advanced Research Projects Agency (DARPA), an agency of the US Department of Defense, spearheaded a parallel effort to develop protocols. This effort resulted in a set of protocols we now call the _TCP/IP model_. The project’s impact on the US military, and subsequently the world’s communications, was profound. The TCP/IP model reached critical mass when Microsoft incorporated it into Windows 95 in the early 1990s. Today, TCP/IP is ubiquitous in computer networking, and it’s the protocol stack we’ll use in this book.

TCP/IP—named for the Transmission Control Protocol and the Internet Protocol—facilitated networks designed using the _end-to-end principle_, whereby each network segment includes only enough functionality to properly transmit and route bits; all other functionality belongs to the endpoints, or the sender and receiver’s network stacks. Contrast this with modern cellular networks, where more of the network functionality must be provided by the network between cell phones to allow for a cell phone connection to jump from tower to tower without disconnecting its phone call. The TCP/IP specification recommends that implementations be robust; they should send well-formed packets yet accept any packet whose intention is clear, regardless of whether the packet adheres to technical specifications.

Like the OSI reference model, TCP/IP relies on layer encapsulation to abstract functionality. It consists of four named layers: the application, transport, internet, and link layers. TCP/IP’s application and link layers generalize their OSI counterparts, as shown in [Figure 1-11](https://learning.oreilly.com/library/view/network-programming-with/9781098128890/c01.xhtml#figure1-11).

![f01011](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/500884c01/f01011.png)

Figure 1-11: The four-layer TCP/IP model compared to the seven-layer OSI reference model

The TCP/IP model simplifies OSI’s application, presentation, and session layers into a single application layer, primarily because TCP/IP’s protocols frequently cross boundaries of OSI Layers 5 through 7. Likewise, OSI’s data link and physical layers correspond to TCP/IP’s link layer. TCP/IP’s and OSI’s transport and network layers share a one-to-one relationship.

This simplification exists because researchers developed prototype implementations first and then formally standardized their final implementation, resulting in a model geared toward practical use. On the other hand, committees spent considerable time devising the OSI reference model to address a wide range of requirements before anyone created an implementation, leading to the model’s increased complexity.

### The Application Layer

Like OSI’s application layer, the TCP/IP model’s _application layer_ interacts directly with software applications. Most of the software we write uses protocols in this layer, and when your web browser retrieves a web page, it reads from this layer of the stack.

You’ll notice that the TCP/IP application layer encompasses three OSI layers. This is because TCP/IP doesn’t define specific presentation or session functions. Instead, the specific application protocol implementations concern themselves with those details. As you’ll see, some TCP/IP application layer protocols would be hard-pressed to fit neatly into a single upper layer of the OSI model, because they have functionality that spans more than one OSI layer.

Common TCP/IP application layer protocols include HTTP, the _File Transfer Protocol__(FTP)_ for file transfers between nodes, and the _Simple Mail Transfer Protocol__(SMTP)_ for sending email to mail servers. The _Dynamic Host Configuration Protocol__(DHCP)_ and the _Domain Name System__(DNS)_ also function in the application layer. DHCP and DNS provide the addressing and name resolution services, respectively, that allow other application layer protocols to operate. HTTP, FTP, and SMTP are examples of protocol implementations that provide the presentation or session functionality in TCP/IP’s application layer. We’ll discuss these protocols in later chapters.

### The Transport Layer

_Transport layer_ protocols handle the transfer of data between two nodes, like OSI’s Layer 4. These protocols can help ensure _data integrity_ by making sure that all data sent from the origin completely and correctly makes its way to the destination. Keep in mind that data integrity doesn’t mean the destination will receive all segments we send through the transport layer. There are just too many causes of packet loss over a network. It does mean that TCP specifically will make sure the data received by the destination is in the correct order, without duplicate data or missing data.

The primary transport layer protocols you’ll use in this book are TCP and the _User Datagram Protocol (UDP)_. As mentioned in “Sending Traffic by Using Data Encapsulation” on page 10, this layer handles segments, or datagrams.

## NOTE

TCP also overlaps a bit with OSI’s Layer 5, because TCP includes session-handling capabilities that would otherwise fall under the scope of OSI’s session layer. But, for our purposes, it’s okay to generalize the transport layer as representing OSI’s Layer 4.

Most of our network applications rely on the transport layer protocols to handle the error correction, flow control, retransmission, and transport acknowledgment of each segment. However, the TCP/IP model doesn’t require every transport layer protocol to fulfill each of those elements. UDP is one such example. If your application requires the use of UDP for maximal throughput, the onus is on you to implement some sort of error checking or session management, since UDP provides neither.

### The Internet Layer

The _internet layer_ is responsible for routing packets of data from the upper layers between the origin node and the destination node, often over multiple networks with heterogeneous physical media. It has the same functions as OSI’s Layer 3 network layer. (Some sources may refer to TCP/IP’s internet layer as the _network layer_.)

_Internet Protocol version 4 (IPv4)_, _Internet Protocol version 6 (IPv6)_, _Border Gateway Protocol__(BGP)_, _Internet Control Message Protocol__(ICMP)_, _Internet Group Management Protocol (IGMP)_, and the _Internet Protocol Security__(IPsec)_ suite, among others, provide host identification and routing to TCP/IP’s internet layer. We will cover these protocols in the next chapter, when we discuss host addressing and routing. For now, know that this layer plays an integral role in ensuring that the data we send reaches its destination, no matter the complexity between the origin and destination.

### The Link Layer

The _link layer_, which corresponds to Layers 1 and 2 of the OSI reference model, is the interface between the core TCP/IP protocols and the physical media.

The link layer’s _Address Resolution Protocol__(ARP)_ translates a node’s IP address to the MAC address of its network interface. The link layer embeds the MAC address in each frame’s header before passing the frame on to the physical network. We’ll discuss MAC addresses and their routing significance in the next chapter.

Not all TCP/IP implementations include link layer protocols. Older readers may remember the joys of connecting to the internet over phone lines using analog modems. Analog modems made serial connections to ISPs. These serial connections didn’t include link layer support via the serial driver or modem. Instead, they required the use of link layer protocols, such as the _Serial Line Internet Protocol__(SLIP)_ or the _Point-to-Point Protocol__(PPP)_, to fill the void. Those that do not implement a link layer typically rely on the underlying network hardware and device drivers to pick up the slack. The TCP/IP implementations over Ethernet, wireless, and fiber-optic networks we’ll use in this book rely on device drivers or network hardware to fulfill the link layer portion of their TCP/IP stack.

## What You’ve Learned

In this chapter, you learned about common network topologies and how to combine those topologies to maximize their advantages and minimize their disadvantages. You also learned about the OSI and TCP/IP reference models, their layering, and data encapsulation. You should feel comfortable with the order of each layer and how data moves from one layer to the next. Finally, you learned about each layer’s function and the role it plays in sending and receiving data between nodes on a network.

This chapter’s goal was to give you enough networking knowledge to make sense of the next chapter. However, it’s important that you explore these topics in greater depth, because comprehensive knowledge of networking principles and architectures can help you devise better algorithms. I’ll suggest additional reading for each of the major topics covered in this chapter to get you started. I also recommend you revisit this chapter after working through some of the examples in this book.

The OSI reference model is available for reading online at [https://www.itu.int/rec/T-REC-X.200-199407-I/en/](https://www.itu.int/rec/T-REC-X.200-199407-I/en/). Two Requests for Comments (RFCs)—detailed publications meant to describe internet technologies—outline the TCP/IP reference model: RFC 1122 and RFC 1123 ([https://tools.ietf.org/html/rfc1122/](https://tools.ietf.org/html/rfc1122/) and [https://tools.ietf.org/html/rfc1123/](https://tools.ietf.org/html/rfc1123/)). RFC 1122 covers the lower three layers of the TCP/IP model, whereas RFC 1123 describes the application layer and support protocols, such as DNS. If you’d like a more comprehensive reference for the TCP/IP model, you’d be hard-pressed to do better than _The TCP/IP Guide_ by Charles M. Kozierok (No Starch Press, 2005).

Network latency has plagued countless network applications and spawned an industry. Some CDN providers write prolifically on the topic of latency and interesting issues they’ve encountered while improving their offerings. CDN blogs that provide insightful posts include the Cloudflare Blog ([https://blog.cloudflare.com/](https://blog.cloudflare.com/)), the KeyCDN Blog ([https://www.keycdn.com/blog/](https://www.keycdn.com/blog/)), and the Fastly Blog ([https://www.fastly.com/blog/](https://www.fastly.com/blog/)). If you’re purely interested in learning more about latency and its sources, consider “Latency (engineering)” on Wikipedia ([https://en.wikipedia.org/wiki/Latency_(engineering)](https://en.wikipedia.org/wiki/Latency_(engineering))) and Cloudflare’s glossary ([https://www.cloudflare.com/learning/performance/glossary/what-is-latency/](https://www.cloudflare.com/learning/performance/glossary/what-is-latency/)) as starting points.