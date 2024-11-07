---
title: Network Programming with Go
subtitle: Code Secure and Reliable Network Services from Scratch
author: Adam Woodbeck
authors: Adam Woodbeck
category: Computers
categories: Computers
publisher: No Starch Press
publishDate: 2021-03-30
totalPage: 392
coverUrl: http://books.google.com/books/content?id=CqPTDwAAQBAJ&printsec=frontcover&img=1&zoom=1&edge=curl&source=gbs_api
coverSmallUrl: http://books.google.com/books/content?id=CqPTDwAAQBAJ&printsec=frontcover&img=1&zoom=5&edge=curl&source=gbs_api
description: "Network Programming with Go teaches you how to write clean, secure network software with the programming language designed to make it seem easy. Build simple, reliable, network software Combining the best parts of many other programming languages, Go is fast, scalable, and designed for high-performance networking and multiprocessing. In other words, it’s perfect for network programming. Network Programming with Go will help you leverage Go to write secure, readable, production-ready network code. In the early chapters, you’ll learn the basics of networking and traffic routing. Then you’ll put that knowledge to use as the book guides you through writing programs that communicate using TCP, UDP, and Unix sockets to ensure reliable data transmission. As you progress, you’ll explore higher-level network protocols like HTTP and HTTP/2 and build applications that securely interact with servers, clients, and APIs over a network using TLS. You'll also learn: Internet Protocol basics, such as the structure of IPv4 and IPv6, multicasting, DNS, and network address translation Methods of ensuring reliability in socket-level communications Ways to use handlers, middleware, and multiplexers to build capable HTTP applications with minimal code Tools for incorporating authentication and encryption into your applications using TLS Methods to serialize data for storage or transmission in Go-friendly formats like JSON, Gob, XML, and protocol buffers Ways of instrumenting your code to provide metrics about requests, errors, and more Approaches for setting up your application to run in the cloud (and reasons why you might want to) Network Programming with Go is all you’ll need to take advantage of Go’s built-in concurrency, rapid compiling, and rich standard library. Covers Go 1.15 (Backward compatible with Go 1.12 and higher)"
link: https://play.google.com/store/books/details?id=CqPTDwAAQBAJ
previewLink: http://books.google.com/books?id=CqPTDwAAQBAJ&printsec=frontcover&dq=network+programming+with+go&hl=&as_pt=BOOKS&cd=1&source=gbs_api
isbn13: 9781718500891
isbn10: 1718500890
id: 01JC4BH24H8CEVHH2SJT5NCHWV
modified: 2024-11-07T17:22:21-05:00
---
# INTRODUCTION

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098128890/files/image_fi/book_art/chapterart.png)

With the advent of the internet came an ever-increasing demand for network engineers and developers. Today, personal computers, tablets, phones, televisions, watches, gaming systems, vehicles, common household items, and even doorbells communicate over the internet. Network programming makes all this possible. And _secure_ network programming makes it trustworthy, driving increasing numbers of people to adopt these services. This book will teach you how to write contemporary network software using Go’s asynchronous features.

Google created the Go programming language in 2007 to increase the productivity of developers working with large code bases. Since then, Go has earned a reputation as a fast, efficient, and safe language for the development and deployment of software at some of the largest companies in the world. Go is easy to learn and has a rich standard library, well suited for taking advantage of multicore, networked systems.

This book details the basics of network programming with an emphasis on security. You will learn socket-level programming including TCP, UDP, and Unix sockets, interact with application-level protocols like HTTPS and HTTP/2, serialize data with formats like Gob, JSON, XML, and protocol buffers, perform authentication and authorization for your network services, create streams and asynchronous data transfers, write gRPC microservices, perform structured logging and instrumentation, and deploy your applications to the cloud.

At the end of our journey, you should feel comfortable using Go, its standard library, and popular third-party packages to design and implement secure network applications and microservices. Every chapter uses best practices and includes nuggets of wisdom that will help you avoid potential pitfalls.

## Who This Book Is For

If you’d like to learn how to securely share data over a network using standard protocols, all the while writing Go code that is stable, secure, and effective, this book is for you.

The target reader is a security-conscious developer or system administrator who wishes to take a deep dive into network programming and has a working knowledge of Go and Go’s module support. That said, the first few chapters introduce basic networking concepts, so networking newcomers are welcome.

Staying abreast of contemporary protocols, standards, and best practices when designing and developing network applications can be difficult. That’s why, as you work through this book, you’ll be given increased responsibility. You’ll also be introduced to tools and tech that will make your workload manageable.

## Installing Go

To follow along with the code in this book, install the latest stable version of Go available at [https://golang.org/](https://golang.org/)_._ For most programs in this book, you’ll need at least Go 1.12. That said, certain programs in this book are compatible with only Go 1.14 or newer. The book calls out the use of this code.

Keep in mind that the Go version available in your operating system’s package manager may be several versions behind the latest stable version.

## Recommended Development Environments

The code samples in this book are mostly compatible with Windows 10, Windows Subsystem for Linux, macOS Catalina, and contemporary Linux distributions, such as Ubuntu 20.04, Fedora 32, and Manjaro 20.1. The book calls out any code samples that are incompatible with any of those operating systems.

Some command line utilities used to test network services, such as `curl` or `nmap`, may not be part of your operating system’s standard installation. You may need to install some of these command line utilities by using a package manager compatible with your operating system, such as Homebrew at [https://brew.sh/](https://brew.sh/) for macOS or Chocolatey at [https://chocolatey.org/](https://chocolatey.org/) for Windows 10. Contemporary Linux operating systems should include newer binaries in their package managers that will allow you to work through the code examples.

## What’s in This Book

This book is divided into four parts. In the first, you’ll learn the foundational networking knowledge you’ll need to understand before you begin writing network software.

1. **Chapter 1: An Overview of Networked Systems** introduces computer network organization models and the concepts of bandwidth, latency, network layers, and data encapsulation.
2. **Chapter 2:****Resource Location and Traffic Routing** teaches you how human-readable names identify network resources, how devices locate network resources using their addresses, and how traffic gets routed between nodes on a network.

Part II of this book will put your new networking knowledge to use and teach you how to write programs that communicate using TCP, UDP, and Unix sockets. These protocols allow different devices to exchange data over a network and are fundamental to most network software you’ll encounter or write.

1. **Chapter 3: Reliable TCP Data Streams** takes a deeper dive into the Transmission Control Protocol’s handshake process, as well as its packet sequence numbering, acknowledgments, retransmissions, and other features that ensure reliable data transmission. You will use Go to establish and communicate over TCP sessions.
2. **Chapter 4: Sending TCP Data** details several programming techniques for transmitting data over a network using TCP, proxying data between network connections, monitoring network traffic, and avoiding common connection-handling bugs.
3. **Chapter 5: Unreliable UDP Communication** introduces you to the User Datagram Protocol, contrasting it with TCP. You’ll learn how the difference between the two translates to your code and when to use UDP in your network applications. You’ll write code that exchanges data with services using UDP.
4. **Chapter 6: Ensuring UDP Reliability** walks you through a practical example of performing reliable data transfers over a network using UDP.
5. **Chapter 7: Unix Domain Sockets** shows you how to efficiently exchange data between network services running on the same node using file-based communication.

The book’s third part teaches you about application-level protocols such as HTTP and HTTP/2. You’ll learn how to build applications that securely interact with servers, clients, and APIs over a network using TLS.

1. **Chapter 8: Writing HTTP Clients** uses Go’s excellent HTTP client to send requests to, and receive resources from, servers over the World Wide Web.
2. **Chapter 9: Building HTTP Services** demonstrates how to use handlers, middleware, and multiplexers to build capable HTTP-based applications with little code.
3. **Chapter 10: Caddy: A Contemporary Web Server** introduces you to a contemporary web server named Caddy that offers security, performance, and extensibility through modules and configuration adapters.
4. **Chapter 11: Securing Communications with TLS** gives you the tools to incorporate authentication and encryption into your applications using TLS, including mutual authentication between a client and a server.

Part IV shows you how to serialize data into formats suitable for exchange over a network; gain insight into your services; and deploy your code to Amazon Web Services, Google Cloud, and Microsoft Azure.

1. **Chapter 12: Data Serialization** discusses how to exchange data between applications that use different platforms and languages. You’ll write programs that serialize and deserialize data using Gob, JSON, and protocol buffers and communicate using gRPC.
2. **Chapter 13: Logging and Metrics** introduces tools that provide insight into how your services are working, allowing you to proactively address potential problems and recover from failures.
3. **Chapter 14: Moving to the Cloud** discusses how to develop and deploy a serverless application on Amazon Web Services, Google Cloud, and Microsoft Azure.