---
id: 01JADKK16BDR284GVWPG15P9S2
title: Basic Data Structures
tags:
  - data-structures
  - dsa
  - roadmaps
  - programming
  - arrays
  - linked-lists
  - stacks
  - queues
  - hash-tables
  - 8-
modified: 2024-10-17T12:06:57-04:00
---
# Basic Data Structures

The five main types of basic data structures are: **Arrays**, **Linked Lists**, **Stacks**, **Queues**, and **Hash Tables**.

- **Arrays** are static data structures that store elements of the same type in contiguous memory locations.
- **Linked Lists** are dynamic data structures that store elements in individual nodes, with each node pointing to the next.
- **Stacks** follow the Last-In-First-Out principle (LIFO) and primarily assist in function calls in most programming languages.
- **Queues** operate on the First-In-First-Out principle (FIFO) and are commonly used in task scheduling.
- Lastly, **Hash Tables** store key-value pairs allowing for fast insertion, deletion, and search operations

# Array

An array is a linear data structure that can hold elements and arrange them. It uses contiguous memory space to store elements. In an array, we can directly access any element based on its index which makes it an efficient data structure. Arrays have two types: one-dimensional and multi-dimensional. In a one-dimensional array, data is stored in a linear form while a multi-dimensional array can store data in the form of a matrix or in 3-D format.

Free Resources

---

- [videoArrays in Python](https://www.youtube.com/watch?v=gDqQf4Ekr2A&ab_channel=codebasics)
- [videoArrays in Java](https://www.youtube.com/watch?v=ei_4Nt7XWOw&ab_channel=BroCode)
- [videoArrays in Javascript](https://www.youtube.com/watch?v=yQ1fz8LY354)
- [videoArrays in GoLang](https://www.youtube.com/watch?v=e-oBn806Pzc&pp=ygUIYXJyYXkgZ28%3D)
- [videoArrays in C#](https://www.youtube.com/watch?v=YiE0oetGMAg&pp=ygUIYXJyYXkgYyM%3D)
- [videoArrays in C++](https://www.youtube.com/watch?v=G38hQKXa_RU&pp=ygUJYXJyYXkgYysr)
- [videoArrays in Rust](https://www.youtube.com/watch?v=cH6Qv47MPwk&pp=ygUKYXJyYXkgcnVzdA%3D%3D)
- [videoArrays in Ruby](https://www.youtube.com/watch?v=SP3Vf2KcYeU&pp=ygUKYXJyYXkgcnVieQ%3D%3D)

# Linked Lists

Linked Lists are a type of data structure used for storing collections of data. The data is stored in nodes, each of which contains a data field and a reference (link) to the next node in the sequence. Structurally, a linked list is organized into a sequence or chain of nodes, hence the name. Two types of linked lists are commonly used: singly linked lists, where each node points to the next node and the last node points to null, and doubly linked lists, where each node has two links, one to the previous node and another one to the next. Linked Lists are used in other types of data structures like stacks and queues.

Learn more from the following links:

Free Resources

---

- [videoIntroduction To Linked List](https://youtu.be/Nq7ok-OyEpg?si=xttaGoYKcoJ09Ln2)
- [videoPython Linked List](https://www.youtube.com/watch?v=qp8u-frRAnU&list=PLeo1K3hjS3uu_n_a__MI_KktGTLYopZ12&index=4&ab_channel=codebasics)

# Stacks

A **stack** is a linear data structure that follows a particular order in which the operations are performed. The order may be LIFO (Last In First Out) or FILO (First In Last Out). Mainly three basic operations are performed in the stack:

1. **Push**: adds an element to the collection.
    
2. **Pop**: removes an element from the collection. A pop can result in stack underflow if the stack is empty.
    
3. **Peek** or **Top**: returns the top item without removing it from the stack.
    

The basic principle of stack operation is that in a stack, the element that is added last is the first one to come off, thus the name “Last in First Out”.

Learn more from the following links:

Free Resources

---

- [articleLeetcode](https://leetcode.com/problems/valid-parentheses/)
- [videoStacks](https://www.youtube.com/watch?v=GYptUgnIM_I&list=PLgUwDviBIf0p4ozDR_kJJkONnb1wdx2Ma&index=69&ab_channel=takeUforward)
- [videoStack Data Structure Tutorial](https://www.youtube.com/watch?v=O1KeXo8lE8A)
- [videoPython Stacks](https://www.youtube.com/watch?v=zwb3GmNAtFk)

# Queues

Queues are a type of data structure in which elements are held in a sequence and access is restricted to one end. Elements are added (“enqueued”) at the rear end and removed (“dequeued”) from the front. This makes queues a First-In, First-Out (FIFO) data structure. This type of organization is particularly useful for specific situations such as printing jobs, handling requests in a web server, scheduling tasks in a system, etc. Due to its FIFO property, once a new element is inserted into the queue, all elements that were inserted before the new element must be removed before the new element can be invoked. The fundamental operations associated with queues include Enqueue (insert), Dequeue (remove) and Peek (get the top element).

Learn more from the following links:

Free Resources

---

- [videoQueue](https://www.youtube.com/watch?v=M6GnoUDpqEE)
- [videoPython Queue](https://www.youtube.com/watch?v=rUUrmGKYwHw)

# Hash Tables

`Hash Tables` are specialized data structures that allow fast access to data based on a key. Essentially, a hash table works by taking a key input, and then computes an index into an array in which the desired value can be found. It uses a hash function to calculate this index. Suppose the elements are integers and the hash function returns the value at the unit’s place. If the given key is 22, it will check the value at index 2. Collisions occur when the hash function returns the same output for two different inputs. There are different methods to handle these collisions such as chaining and open addressing.

Learn more from the following links:

Free Resources

---

- [videoHash Table](https://www.youtube.com/watch?v=KEs5UyBJ39g&ab_channel=takeUforward)
- [videoPython Hash Table Part 1](https://www.youtube.com/watch?v=ea8BRGxGmlA)
- [videoPython Hash Table Part 2](https://www.youtube.com/watch?v=54iv1si4YCM)