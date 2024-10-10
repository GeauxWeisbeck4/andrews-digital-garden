---
id: 01J9QFJNJ64SK2B2FW9BQ33Q0M
modified: 2024-10-08T21:51:37-04:00
---
# 2 Improving priority queues: d-way heaps

This chapter covers

- Solving the problem of serving tasks based on priority
- Using priority queues to solve our problem
- Implementing a priority queue with a heap
- Introducing and analyzing d-way heaps
- Recognizing use cases where heaps improve performance

In the previous chapter we introduced some basic concepts about data structures and programming techniques, described the structure of this book, and hopefully raised your interest. You should now be aware of why developers need to know about data structures.

In this chapter, we will further develop those ideas and refine our narration. This chapter is meant to be a soft introduction to what is presented in this book; we have chosen a topic that should be familiar to readers with some background in algorithms, while providing a review of heaps along with some new insight on branching factors.

To this end, however, we assume that the reader is familiar with some basic concepts traditionally taught in CS 101 courses: big-O notation, the RAM model, and simple data structures such as arrays, lists, and trees. These building blocks will be leveraged throughout the book to build increasingly complex structures and algorithms, and it’s important that you familiarize yourself with such concepts in order to be able to go through the next chapters. This is why we provided a recap of these fundamental topics in the appendices at the end of the book; feel free to take a look or skim through them in order to make sure you have a good understanding of the material.

Now that we have settled the basics, we will start with the core focus of this book, and in section 2.1 we describe the structure that we will use for each of the remaining chapters.

Section 2.2 introduces the problem we are going to use in this chapter (how to efficiently handle events with priority), while section 2.3 outlines possible solutions, including priority queues, and explains why the latter are better than more basic data structures.

Next, in section 2.4 we describe the priority queue API,[1](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1012948) and we show an example of how to use it as a black box, before delving into its internals in sections 2.5 and 2.6. In the former we analyze in detail how a d-ary heap works, describing the functioning of its methods. In section 2.6 we delve into the implementation of a d-way heap.

Sections 2.7 and 2.8 describe use cases where heaps and priority queues make a difference and allow speeding up applications or other algorithms.

Finally, the focus of section 2.9 is understanding the optimal branching factor for a heap. Although it can be considered optional, I suggest you at least try to read it through to gain a deeper understanding of how a heap works and why you might want to choose a ternary heap instead of a binary heap, or vice versa.

This is a long chapter, and it might look intimidating, but hang in there; we are going to cover a lot of ground and lay the foundations for the whole book.

## 2.1 Structure of this chapter

Starting with the current chapter, we will embrace a schematic way to present our data structures.

Each chapter will be driven by a practical use case for its main topic, a real-world problem showing how the data structure is used in practice, but also a step-by-step explanation of how operations are performed. We also provide code samples showing how to use the algorithms we will focus on next.

So, first we are going to introduce a problem, one that is typically solved using the main topic of the chapter.

Then we present one or more ways to solve it. There might be several possible solutions to the same problem, and if that’s the case, we explain when and why using a particular data structure is a good idea. At this point, usually, we still treat our data structure as a black box: we focus on how to use it and ignore the details of its implementation.

Only in the next section do we start discussing how a data structure works. We focus on describing the mechanism, using figures and pseudo-code examples to clarify how it works.

After the code section, we are going to discuss advanced theory topics such as performance or mathematical proofs for the algorithms.

Usually we also provide a list of additional applications that use the algorithms presented, although for most of these further examples we have to omit coding for the sake of space.

## 2.2 The problem: Handling priority

The first problem we are going to tackle is handling tasks based on priority. This is something all of us are familiar with in some way.

The problem can be described in these terms: given a collection of tasks with different priorities, determine which task should be executed next.

We can find many examples in the real world where we apply, consciously or not, techniques that help us decide what to do next. Our daily lives are full of tasks; usually the order in which we run them is a result of time constraints and the importance we assign to those tasks.

A common example of an environment where tasks are executed by priority is an emergency room, where patients are seen, not according to the order in which they arrived, but instead depending on how urgent their conditions are. If we move closer to our IT domain, there are many tools and systems that have the same behavior. Think, for instance, about your operating system scheduler. Or maybe you are using a mobile app to take a to-do list.

### 2.2.1 Priority in practice: Bug tracking

The example I’d like to use in this chapter, though, is a bug-tracking suite. You are probably already familiar with such a tool. When you work in teams, you need a way to track bugs and tasks so that no two people work on the same issue and duplicate effort, while making sure issues are tackled in the right order (whatever that is, depending on your business model).

To simplify our example, let’s restrict it to the case of a bug-tracking tool where each bug is associated with a priority, expressed as the number of days within which it needs to be solved (lower numbers mean higher priority). Also, let’s assume that bugs are independent, so no bug requires solving another bug as a prerequisite.

For our example, let’s consider the following list of bugs (in sparse order) for a single-page web application.

Each bug will look like a tuple:

<task description, importance of missing the deadline>

So, for instance we could have this.

|   |   |
|---|---|
|`Task description`|`Severity (1-10)`|
|`Page loads take 2+ seconds`|`7`|
|`UI breaks on browser X`|`9`|
|`Optional form field blocked when using browser X on Friday the 13th`|`1`|
|`CSS style causes misalignment`|`8`|
|`CSS style causes 1px misalignment on browser X`|`5`|

Whenever resources (for example, developers) are limited, there comes the need to prioritize bugs. Therefore, some bugs are more urgent than others: that’s why we associate priorities to them.

Now, suppose a developer on our team completes her current task. She asks our suite for the next bug that needs to be solved. If this list were static, our suite’s software could just sort the bugs once, and return them in order.[2](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1012974)

|   |   |
|---|---|
|`Task description`|`Severity (1-10)`|
|`UI breaks on browser X`|`9`|
|`CSS style causes misalignment`|`8`|
|`Page loads take 2+ seconds`|`7`|
|`CSS style causes 1px misalignment on browser X`|`5`|
|`Optional form field blocked when using browser X on Friday the 13th`|`1`|

As you can imagine, though, this is not the case. First, new bugs are discovered all the time, and so new items will be added to the list. Say a nasty encryption bug is found—you’d need to have it solved by yesterday! Moreover, priority for bugs can change over time. For instance, your CEO might decide that you are going after the market share that’s mostly using browser X, and you have a big feature launch next Friday, the 13th, so you really need to solve that bug at the bottom within a couple of days.

|   |   |
|---|---|
|`Task description`|`Severity (1-10)`|
|`Unencrypted password on DB`|`10`|
|`UI breaks on browser X`|`9`|
|`Optional form field blocked when using browser X on Friday the 13th`|`8`|
|`CSS style causes misalignment`|`8`|
|`Page loads take 2+ seconds`|`7`|
|`CSS style causes 1px misalignment on browser X`|`5`|

## 2.3 Solutions at hand: Keeping a sorted list

We could, obviously, update our sorted list every time we have an item inserted, removed, or modified. This can work well if these operations are infrequent and the size of our list is small.

Any of these operations, in fact, would require a linear number of elements changing position, both in worst cases and in the average case.[3](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1012988)

For this use case, it could probably work. But if our list had millions or billions of elements, then we would most likely be in trouble.

### 2.3.1 From sorted lists to priority queues

Luckily for us, there is a better solution. This is the perfect use case for one of the core data structures. A priority queue will keep a partial ordering of the elements, with the guarantee that the next element returned from the queue will hold the highest priority.

By giving up the requirement of a total ordering (which we wouldn’t need in this case, because we only consume tasks one by one), we gain in performance: each of the operations on the queue can now require only logarithmic time.

As a side note, this reminds us how important it is to get our requirements right before implementing any solution. We need to make sure we don’t overcomplicate our work and requirements: for example, keeping a list of elements sorted when all we need is a partial ordering wastes resources and complicates our code, making it harder to maintain and scale.

## 2.4 Describing the data structure API: Priority queues

Before delving into the topic of the chapter, let’s take a step back.

As explained in appendix C, each data structure can be broken down into a few lower-level components:

- _API_—The API is the contract that a data structure _(DS__)_ makes with external clients. It includes method definitions, as well as some guarantees about the methods’ behavior that are provided in the DS’s specification. For example, a priority queue _(PQ__)_ (see table 2.1) provides these methods and guarantees:
    
    - `top()`—Returns and extracts the element with the highest priority.
        
    - `peek()`—Like top it returns the element with the highest priority, but without extracting it from the queue.
        
    - `insert(e, p)`—Adds a new element `e` with priority `p` to the PQ.
        
    - `remove(e)`—Removes element `e` from the queue.
        
    - `update(e, p)`—Changes the priority for element `e` and sets it to `p`.
        
- _Invariants_—(Optional) internal properties that always hold true throughout the life of the data structure. For instance, a sorted list would have one invariant: every element is not greater than its successor. The purpose of invariants is making sure the conditions necessary to live up to the contract with the external clients are always met. They are the internal counterparts of the guarantees in the API.
    
- _Data model_—To host the data. This can be a raw chunk of memory, a list, a tree, etc.
    
- _Algorithms_—The internal logic that is used to update the data structure while making sure that the invariants are not violated.
    

Table 2.1 API and contract for priority queue

|Abstract data structure: Priority queue|   |
|---|---|
|API|class PriorityQueue {<br>  top() → element<br>  peek() → element<br>  insert(element, priority)<br>  remove(element)<br>  update(element, newPriority)<br>  size() → int<br>}|
|Contract with client|The top element returned by the queue is always the element with highest priority currently stored in the queue|

In appendix C we also clarify how there is a difference _between an abstract data structure and concrete data structures_. The former includes the API and invariants, describing at a high level how clients will interact with it and the results and performance of operations. The latter builds on the principles and API expressed by the abstract description, adding a concrete implementation for its structure and algorithms (data model and algorithms).

This is exactly the relationship between _priority queues and heaps_. A priority queue is an abstract data structure that can be implemented in many ways (including as a sorted list). A heap is a concrete implementation of the priority queue using an array to hold elements and specific algorithms to enforce invariants.

### 2.4.1 Priority queue at work

Imagine you are provided with a priority queue. It can come from a third-party library or from a standard library (many languages, such as C++ or Scala, provide an implementation for priority queues in their standard container lib).

You don’t need to know the internals of the library at this point; you just need to follow its public API and use it, confident it’s properly implemented. This is the black box approach (figure 2.1).

For instance, let’s suppose we add our bugs to our PQ in the same order we have seen before.

|   |   |
|---|---|
|`Task description`|`Severity (1-10)`|
|`Page loads take 2+ seconds`|`7`|
|`UI breaks on browser X`|`9`|
|`Optional form field blocked when using browser X on Friday the 13th`|`1`|
|`CSS style causes misalignment`|`8`|
|`CSS style causes 1px misalignment on browser X`|`5`|

If we returned the tasks in the same order as we inserted them, we would just implement as a plain queue (see figure 2.2 for a quick glance at how a queue works, and appendix C for a description of basic containers). Instead, let’s assume that now we have our priority queue containing those five elements; we still don’t know the internals of the PQ, but we can query it through its API.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F1.png)

Figure 2.1 Representation of a priority queue as a black box. If we employ an implementation of a priority queue provided by a third-party library (or from standard libraries), and we trust this implementation to be correct, we can use it as a black box. In other words, we can ignore its internals, and just interact with it through its API.

For instance, we can check how many elements it contains and even take a peek at the one at the top (figure 2.1). Or we can directly ask it to return us the top element (the one with the highest priority) and remove it from the queue.

If, after inserting the five elements in figure 2.1, we call `top`, the element returned will be “UI breaks on browser X” and the size of the queue will become 4. If we call `top` again, the next element will be “CSS style causes misalignment” and the size will become 3.

As long as the priority queue is implemented correctly and given the priorities in our examples, we can be sure those two elements will be the ones returned first, independently of the order in which they are inserted.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F2.png)

Figure 2.2 Operations on a queue: elements are generic integers, but they could be any value here, because in plain queues priority is only given by the order of insertion (see appendix D). Insertion (enqueue) adds an element to the front of the queue. Deletion (dequeue) removes the last element in the queue and returns it. With some caution, both operations can be performed in constant time.

### 2.4.2 Priority matters: Generalize FIFO

Now the question is how we choose the priority of an element. Often, the natural ordering given by how much time an element waits in a line can be considered the fairest. Sometimes, however, there is something special about some elements that might suggest they should be served sooner than others that waited longer. For instance, you don’t always read your emails in the order you received them, but often you skip newsletters or “funny” jokes from friends to read work-related messages first. Likewise, in an emergency room, the next case treated is not necessarily going to be one that has been waiting for the longest time. Rather, every case is evaluated at arrival and assigned a priority, and the highest priority one is going to be called in when a doctor becomes available.

That’s the idea behind priority queues: they behave like regular, plain queues, except that the front of the queue is dynamically determined based on some kind of priority. The differences caused to the implementation by the introduction of priority are profound, enough to deserve a special kind of data structure.

But that’s not all: we can even define basic containers as _bag_ or _stack_ as special cases of priority queues; appendix D explores how this is possible. This is an interesting topic to help you gain a deeper understanding of how priority queues work, although in practice those containers are usually implemented ad hoc, because we can achieve better performance by leveraging their specific characteristics.

## 2.5 Concrete data structures

Let’s now move from abstract to concrete data structures. Knowing how the API of a priority queue works is good enough to use it, but often it is not enough to use it well. Especially on time-critical components or in data-intensive applications, we often need to understand the internals of the data structures and the details of its implementation to make sure we can integrate it in our solution without introducing a bottleneck.[4](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013002)

Every abstraction needs to be implemented using a concrete data structure. For example, a stack can be implemented using a list, an array, or in theory even a heap (although this would be silly, as will be clear in a few sections). The choice of the underlying data structure used to implement a container will only influence the container’s performance. Choosing the best implementation is usually a tradeoff: some data structures speed up some operations but will make other operations slower.

### 2.5.1 Comparing performance

For the implementation of a priority queue, we will initially consider three naïve alternatives using core data structures discussed in appendix C: an unsorted array, where we just add elements to its end; a sorted array, where we make sure the ordering is reinstated every time we add a new element; and balanced trees, of which heaps are a special case. Let’s compare, in table 2.2, the running times[5](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013016) for basic operations implemented with these data structures.

### 2.5.2 What’s the right concrete data structure?

From table 2.2 we can see that naïve choices would lead to a linear time requirement for at least one of the core operations, while a balanced tree would always guarantee a logarithmic worst case. Although linear time is usually regarded as “feasible,” there is still a tremendous difference between logarithmic and linear: for a billion elements, the difference is between 1 billion operations and a few dozens. If each operation takes 1 millisecond, that means going from 11 days to less than a second.

Table 2.2 Performance for operations provided by PQs, broken bown by underlying data structure

|Operation|Unsorted array|Sorted array|Balanced tree|
|---|---|---|---|
|`Insert`|O(1)|O(n)|O(log n)|
|`Find-Minimum`|O(1) [a](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1022736)|O(1)|O(1) [a](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1022736)|
|`Delete-Minimum`|O(n) [b](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1022783)|O(1) [c](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1022813)|O(log n)|

a By saving an extra value with the minimum and charging the cost of maintaining its value on insert and delete.

b If we use a buffer to speed up find-minimum, then we need to find the next minimum on delete. Unfortunately, nothing comes for free. Alternatively, we could have constant-time delete by giving up the buffer and swapping the deleted element with the last in the array, and linear time find-minimum.

c Storing the array in reverse order, deleting the last element might just be a matter of shrinking the size of the array, or somehow keeping track of what’s the last element in the array.

Moreover, consider that most of the times containers, and priority queues in particular, are used as support structures, meaning that they are part of more complex algorithms/ data structures and each cycle of the main algorithm can call operations on the PQ several times. For example, for a sorting algorithm, this could mean going from `O(n2),` which usually means unfeasible for `n` as large as 1 million, or even less, to `O(n*log(n))`, which is still tractable for inputs of size 1 billion or more.[6](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013030) However, this would come at a cost, because the implementation of balanced binary trees is usually not trivial.

Next, we present a way to efficiently implement a generic priority queue.

### 2.5.3 Heap

A binary heap is the most used version of priority queues. It allows elements to be inserted and retrieved in either ascending or descending order, one at a time.

While in reality an underlying array is used to store heap’s elements, it can conceptually be represented as a particular kind of binary tree, abiding by three invariants:

- Every node has at most two children.
    
- The heap tree is complete and left-adjusted. Complete (see figure 2.3) means that if the heap has height `H`, every leaf node is either at level `H` or `H-1`. All the levels are left-adjusted, which means that no right sub-tree has a height greater than its left sibling. So, if a leaf is at the same height as an internal node,[7](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013046) the leaf can’t be on the left of that node. Invariant numbers 1 and 2 are the structural invariants for heaps.
    
- Every node holds the highest priority in the subtree rooted at that node.
    
    ![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F3.png)
    
    Figure 2.3 Two examples of complete binary trees. All levels in the tree have the maximum possible number of nodes, except (possibly) the last one. All leaves in the last level are left-adjusted; at the previous level, no right subtree has more nodes than its left sibling.
    

Properties (1) and (2) allow for the array representation of the heap. Supposing we need to store `N` elements, the tree structure is directly and compactly represented using an array of `N` elements, without pointers to children or parents. Figure 2.4 shows how the tree and array representation of a heap are equivalent.

In the array representation, if we are starting to count indices from `0`, the children of the `i`-th node are stored in the positions[8](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013060) `(2 * i) + 1` and `2 * (i + 1)`, while the parent of node `i` is at index `(i - 1) / 2` (except for the root, which has no parent). For example, looking at figure 2.4, the node at index `1` (whose priority is `3`) has two children at indices `3` and `4`, and its parent is at index `0`; if we consider a node at position `5`, its parent is the element at index `2` and its children (not shown in the picture) would be at indices `11` and `12`.

It might seem counterintuitive that we use an array to represent a tree. After all, trees were invented to overcome array limitations. This is generally true, and trees have a number of advantages: they are more flexible and, if balanced, allow for better performance, with worst-case logarithmic search, insertion, and delete.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F4.png)

Figure 2.4 A binary heap. Only priorities are shown for the nodes (elements are irrelevant here).The numbers inside the small squares show the indices of the heap elements in the array. Nodes are matched into array elements top to bottom, left to right. Top: the tree representation for the heap. Notice how every parent is smaller than (or at most equal to) its children and, in turn, all elements in its subtree. Bottom: the same heap in its array representation.

But the improvement we get with trees comes with a price, of course. First, as with any data structure that uses pointers (lists, graphs, trees, and so on) we have a memory overhead in comparison to arrays. While with the latter we just need to reserve space for the data (plus maybe, depending on the implementation details, some constant space for pointers and the node structure itself), every tree node requires extra space for the pointers to its children and possibly to its parent.

Without getting into too much detail, arrays also tend to be better at exploiting _memory locality_: all the elements in the array are contiguous in memory, and this means lower latency when reading them.

Table 2.3 shows how a heap matches a priority queue for its abstract part, and what its concrete data model and invariants are.

Table 2.3 Underlying components for a heap

|Concrete data structure: Heap|   |
|---|---|
|API|Heap {<br>  top() → element<br>  peek()→ element<br>  insert(element, priority)<br>  remove(element)<br>  update(element, newPriority)<br>}|
|Contract with client|The top element returned by the queue is always the element with highest priority currently stored in the queue.|
|Data model|An array whose elements are the items stored in the heap.|
|Invariants|Every element has two “children.” For the element at position i, its children are located at indices 2*i+1 and 2*(i+1).<br><br>Every element has higher priority than its children.|

### 2.5.4 Priority, min-heap, and max-heap

When we stated the three heap’s properties, we used the wording “highest priority.” In a heap, we can always say that the highest priority element will be at the top, without raising any ambiguity.

Then when it comes to practice, we will have to define what priority means, but if we implement a general-purpose heap, we can safely parameterize it with a custom priority function, taking an element and returning its priority. As long as our implementation abides by the three laws as stated in section 2.5.3, we will be sure our heap works as expected.

Sometimes, however, it’s better to have specialized implementations that leverage the domain knowledge and avoid the overhead of a custom priority function. For instance, if we store tasks in a heap, we can use tuples (`priority, task`) as elements, and rely on the natural ordering of tuples.[9](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013079)

Either way, we still need to define what highest priority means. If we assume that highest priority means a larger number, that is, if p1 > p2 means p1 is the highest priority, then we call our heap a max-heap.

At other times we need to return smallest numbers first, so we assume instead that p1 > p2 means p2 is the highest priority. In this case, we are using a min-heap. In the rest of the book, and in particular in the coding section, assume we are implementing a _max-heap_.

The implementation of a min-heap differs only slightly from a max-heap, the code is pretty much symmetric, and you just need to exchange all occurrences of < and ≤ to > and ≥ respectively, and swap `min` with `max`.

Or, alternatively, it’s even easier if you have an implementation of one to get the other by simply taking the reciprocal of priorities (either in your priority function or when you pass priority explicitly).

To give you a concrete example, suppose you have a min-heap that you use to store `(age`, `task)` pairs: a min-heap will return the task with smallest ages first. Because we might instead be interested in having the oldest task returned first, we would rather need a max-heap; it turns out we can get the same result without changing any code for out heap! We just need to store elements as tuples `(-age, task)` instead. If `x.age < y.age`, in fact, then `-x.age > -y.age`, and thus the min-heap will return first the tasks whose age have the largest absolute value.

For instance, if we have task `A` with age 2 (days) and task `B` with age 3, then we create the tuples `(-2, A)` and `(-3, B)`; when extracting them from a min-heap, we can check that `(-3, B) < (-2, A)` and so the former will be returned before the latter.

### 2.5.5 Advanced variant: d-ary heap

One could think that heaps needs to be binary trees. After all, binary search trees are the most common kind of trees, intrinsically associated with ordering. It turns out that there is no reason to keep our branching factor[10](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013124) fixed and equal to 2. On the contrary, we can use any value greater than 2, and use the same array representation for the heap.

For a branching factor 3, the children of the i-th node are at indices `3*i + 1`, `3*i + 2` and `3*(i + 1)`, while the parent of node `i` is at position `(i - 1)/3`.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F5.png)

Figure 2.5 A three-way heap. Only priorities are shown for the nodes (values are irrelevant here). The smaller squares next to the elements show the indices of the heap elements in the array. Top: the tree representation for a min-heap. Notice how every parent is smaller than (or at most equal to) its children and, in turn, all elements in its subtree. Bottom: the same heap in its array representation.

Figure 2.5 shows both the tree and array representation of a 3-ary heap. The same idea holds for branching factors 4, 5, and so on.

For a _d-ary_ heap, where the branching factor is the integer `D > 1`, our three heap invariants become

- Every node has at most `D` children.
    
- The heap tree is complete, with all the levels that are left-adjusted. That is, the `i`-th sub-tree has a height at most equal to its siblings on its left (from `0` to `i-1`, `1 < i <= D`).
    
- Every node holds the highest priority in the subtree rooted at that node.
    

Fun fact It’s worth noting that for `D = 1`, the heap becomes a sorted array (or a sorted doubly linked list in its tree representation). The heap construction will be the insertion sort algorithm and requires quadratic time. Every other operation requires linear time.

## 2.6 How to implement a heap

At this point we have a good idea of how a priority queue should be used and its internal representation. It’s about time we delve into the details of the implementation for a heap.

Before we go, let’s once again take a look at our API:

class Heap {
  top()
  peek()
  insert(element, priority)
  remove(element)
  update(element, newPriority)
}

These are just the public methods defining the API of our class.

But first things first. We will assume, hereafter, that we store all the `(element`, `priority)` tuples added to the heap in an array named `pairs`, as shown in listing 2.1.

Listing 2.1 The `DHeap` class properties

**class** DHeap
  **#type** Array[Pair]
  pairs 
  **function** DHeap(pairs=[])

This is the first time we see a code snippet in this book. It might be a good chance for you to review _appendix A_, where we explain the syntax used. For instance, if we have a variable `p` holding such a tuple, we embrace a specific syntax for destructured assignment of its fields into two variables:

(element, priority) ← p

We also assume that the tuple’s fields are named, so that we can access, in turn, `p.element` and `p.priority`, or create a tuple `p` with this syntax:

p ← (element=’x’, priority=1)

In many figures in this section, we will only show an element’s priorities or, to see it in another way, we assume that elements and priorities are the same. This is just for the sake of space and clarity in diagrams, but it also highlights an important characteristic of heaps: in all their methods, we only need to access and move priorities. This can be important when elements are large objects, especially if they are so large that they won’t fit in cache/memory or for any reason they are stored on disk; then the concrete implementation can just store and access a reference to the elements and its priority.

Now, before delving into the API methods, we need to define two helper functions that will be used to reinstate the heap properties whenever a change is performed. The possible changes for a heap are

- Adding a new element to the heap
    
- Removing the top element of the heap
    
- Updating an element’s priority
    

Any of these operations can cause a heap element to have higher priority than its parent or a lower priority than (one of) its children.

### 2.6.1 BubbleUp

When an element has a higher priority than its parent, it is necessary to call the `bubbleUp` method, as shown in listing 2.2. Figure 2.6 shows an example based on our task management tool: the priority of the element at index 7 is higher than its parent (at index 2) and thus these two elements need to be swapped.

Listing 2.2 The `bubbleUp` method

**function** bubbleUp(pairs, index=|pairs|-1)                                ❶
  parentIndex ← index                                                    ❷
  **while** parentIndex > 0 **do**                                               ❸
    currentIndex ← parentIndex
    parentIndex ← getParentIndex(parentIndex)                            ❹
    **if** pairs[parentIndex].priority < pairs[currentIndex].priority **then**   ❺
      swap(pairs, currentIndex, parentIndex)                             ❻
    **else**                                                                 ❼
      **break**

❶ We explicitly pass the array with all pairs and the index (by default the last element) as arguments. In this context, `|A|` means the size of an array `A`.

❷ Start from the element at the index passed as argument (by default the last element of `A`).

❸ Check if the current element is already at the root of the heap. If so, we are finished.

❹ Compute the index of the parent of current element in the heap. The formula can vary with the type of implementation. For a heap with branching factor `D` and array’s indices starting at zero, it’s `(parentIndex-1)/D`.

❺ If the parent has lower priority than the current element `p` . . .

❻ . . . then swaps parent and children.

❼ Otherwise, we are sure that heap’s properties are reinstated, and we can exit the loop and return.

As listing 2.2 shows, we keep swapping elements until either current element gets assigned to the root (line #3), or its priority is lower than its next ancestor (line #6-#9). This means that each call to this method can involve at most `logD(n)` comparisons and exchanges, because it’s upper-bounded by the height of the heap.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F6.png)

Figure 2.6 The bubbleUp method in action on a ternary max-heap. (A) The element at index 7 has a higher priority (9) than its parent (whose priority is 8). (B) We swap element at index 7 with its parent at index 2. The element now at index 7 has certainly found its final position, while the one bubbled up needs to be compared to its new parent. In this case, the root has a higher priority, so bubbling up stops.

Remember we are implementing a max-heap, so higher numbers mean higher priority. Then, as shown in figure 2.6 (A), the element at index [7] is out of place, because its parent, “Optional form field blocked . . . ” at index [2], has priority 8 < 9.

At this point, to fix things we need to call `bubbleUp(pairs, 7)`. We enter the loop at line #3, because `parentIndex` is `7 > 0`, compute the new parent’s index, which is 2, and hence the condition at line #6 is also `true`, so at line #7 the two elements will be swapped. After the update, the heap’s array will look like figure 2.6 (B).

At the next iteration of the loop (`parentIndex` is still `2 > 0`, so the loop will be entered at least once more), the new values for `currentIndex` and `parentIndex` will evaluate, at lines #4-5, to 2 and 0, respectively.

Since elements’ priorities are now, in turn, 9 and 10 for child and parent, the condition at line #6 is not met anymore, and the function will break out of the loop and return.

Notice that this method, if the height of the tree is `H`, requires at most `H` swaps, because each time we swap two elements, we move one level up in the tree, toward its root.

This result is an important result, as we will see in section 2.7. Because heaps are balanced binary trees, their height is logarithmic in the number of elements.

But we can add a further improvement with just a small change. If you notice, we are repeatedly swapping the same element with its current parent; this element, the one on which `bubbleUp` is initially called, keeps moving toward the root. You can see in figure 2.6 that it “bubbles up,” like a soap bubble floating toward the top of the heap. Each swap requires three assignments (and the use of a temporary variable), so the naïve implementation in listing 2.2 would require (at most) `3*H` assignments.

While both indices, the current and the parent node’s, change at each iteration, the value of the current element is always the same.

Figure 2.7 highlights the path that such an element has to travel. In the worst case it travels up to the root, as shown in the picture, but possibly bubbling up could stop at an intermediate node. It turns out that bubbling up an element in the path `P` to the heap’s root is equivalent to inserting that element in the (sub)array obtained by only considering those elements in `P` (see figure 2.7 (B)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F7.png)

Figure 2.7 Bubbling up of an element in a max-heap, using the method that reduces the number of assignments to one-third. The figure shows a call to `bubbleUp(pairs, 7)` displaying the tree representation (A) and the array representation (B) for a max-heap, and filtered elements (C) in the path `P` from the element to the root.

Referring to figure 2.7, the first action needed is saving to a temporary variable the element `X` that will move up (step 1). Then, starting with its parent, we compare all elements in the path to the temporary variable and copy them over in case they have lower priority (steps 1–3). At each time, we copy the parent element over its one child that is on the path `P`. It’s like we filter out all the elements in the heap’s array that are not part of `P: t`His is highlighted in 2.7 (C).

Finally, when we find an element `Y` along `P` that has higher priority than our temporary, we can copy element `X` from the temporary variable to the one child of `Y` that belongs to `P` (step 4).

Now, this can be done efficiently, in the same way each iteration of the insertion sort algorithm works. We initially save in a temporary variable a copy of the element to bubble up (call it `X`), and then check the elements on its left in the array. We “move” the elements to their right (check figure 2.7 (C)) by copying them to the next position on their left, until we find an element with a priority higher than `X`’s. This can be done with just (at most) `H+1` assignments for a path of length `H`, thus saving about 66% of the assignments.

Listing 2.3 shows the improved version of the `bubbleUp` method. Notice how, at some point, there will momentarily be two copies of the element originally at index [3] (and later, of the element at index [1]). This is because rather than actually swapping elements, we can just overwrite the current element with its parent at each iteration (line #6) and write the element bubbling up just once at the end as the last action in the method. This works because bubbling up an element is conceptually equivalent to the inner loop of _insertion sort_, where we are looking for the right position to insert the current element into the path connecting it to heap’s root.

Listing 2.3 An optimization of the bubbleUp method

**function** bubbleUp(pairs, index=|pairs|-1)                     ❶
  current ← pairs[index]                                      ❷
  **while** index > 0 **do**                                          ❸
    parentIndex ← getParentIndex(index)                       ❹
    **if** pairs[parentIndex].priority < current.priority **then**    ❺
      pairs[index] ← pairs[parentIndex]                       ❻
      index ← parentIndex                                     ❼
    **else**                                                      ❽
      **break**
  pairs[index] ← current                                      ❾

❶ We explicitly pass the array with all pairs and the index (by default the last element) as arguments.

❷ Starts from the element at the index passed as argument (by default the last element of `A`)

❸ Checks if the current element is already at the root of the heap. If so, we are finished.

❹ Computes the index of the parent of current element in the heap. The formula can vary with the type of implementation. For a heap with branching factor `D` and array’s indices starting at zero, it’s `(parentIndex-1) / D`.

❺ If the parent has lower priority than the current element . . .

❻ . . . then moves the parent one position down (implicitly the current element goes one up).

❼ Updates the index of the current element for next iteration

❽ Otherwise, we have found the right place for current, and we can exit the loop.

❾ At this point, `index` is the right position for current.

### 2.6.2 PushDown

The `pushDown` method handles the symmetric case where we need to move an element down toward the leaves of the heap, because it might be larger than (at least) one of its children. An implementation is shown in listing 2.4.

There are two differences with the “bubble up” case:

- _The triggering condition_—In this case, the altered node doesn’t violate invariant 3 with respect to its parent but might violate it when compared to its children. This would happen, for instance, when the root is extracted from the heap and replaced with the last leaf in the array, or when a node priority is changed by assigning it a lower priority.
    
- _The algorithm_—For every level the element we are pushing down goes through, we need to find its highest-priority child, in order to find where the element could land without violating any property.
    

Listing 2.4 The `pushDown` method

**function** pushDown(pairs, index=0)
 currentIndex ← index                                         ❶
  **while** currentIndex < firstLeafIndex(pairs) **do**               ❷
    (child, childIndex) ← highestPriorityChild(currentIndex)  ❸
    **if** child.priority > pairs[currentIndex].priority **then**     ❹
      swap(pairs, currentIndex, childIndex)                   ❺
      currentIndex ← childIndex
    **else**                                                      ❻
      **break**

❶ Starts at the index passed as argument (by default at the first element of the array `A`)

❷ Leaves have no children, so can’t be pushed down anymore. `firstLeafIndex` returns the index of the first leaf in the heap. If `D` is the branching factor, and array’s indices starting at zero, then it’s `(|pairs| - 2) / D + 1`.

❸ We need to identify the current node’s children with the highest priority, because that will be the only element that is safe to move up without breaking heap properties.

❹ If the highest priority child has higher priority than the current element `p` . . .

❺ . . . then swaps the current element with the one among its children with highest priority.

❻ Otherwise, the heap properties have been reinstated, and we can break out and exit the function.

This method’s running time, while asymptotically equivalent to `bubbleUp``’`s, requires a slightly larger number of comparisons. In the worst case, `D * logD(n)`, for a heap with branching factor `D` and containing `n` elements. This is also the reason why increasing the branching factor (the max number of children for each node) indefinitely is not a good idea: we’ll see this in section 2.10.

To stick to our tasks example, let’s consider the case where the root’s priority is lower than its children, shown in figure 2.8. Working on a ternary Heap, its first leaf is stored at index 4.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F8.png)

Figure 2.8 An example of applying the `pushDown` method to the root of a ternary heap. (A) The root of the tree violates heap’s properties, being smaller than one of its children. The algorithm finds the child with the highest priority, and then compares it to current node (the root, in this example). (B) We swapped the root with its highest-priority children: the node at index 1. Now, in turn, we compare the updated child with its children: since none of them has a higher priority than current node, we can stop.

We call `pushDown(pairs, 0)` to fix the third heap’s property; helper function `firstLeafIndex` at line #3 will return 3 (because the element at index [7], the last one in the array, is a child of the node at index [2], which is the last internal node), and so we will enter the loop.

The children of element [0] are at positions [1], [2], and [3], and among them we’ll choose the highest priority child of the root, at line #4. It turns out to be the element `<Unencrypted password on DB, 10>`. Its priority is higher than current element’s, so at line #6 we will swap it with current element.

Overall, after one iteration of the `while` loop, we have the situation illustrated by sub-figure 2.8.B, and `currentIndex` is set to 1.

Therefore, when we enter the loop for the second time, at line #3 `currentIndex` evaluates to 1, which is still less than 3, the index of the first leaf. The children of the element at index [1] are at indices [4], [5], and [6]. Among those elements we look for the node with highest priority among the current node’s children, at line #4 in listing 2.4; it turns out it is the element at index [4], `<Page loads take 2+ seconds, 7>`.

Therefore, at line #5, we find out that this time, all children’s priorities are lower than the current element’s, and we break out of the loop and return without any further change.

Similar to `bubbleUp`, `pushDown` can be improved by simply keeping the current element in a temporary variable and avoiding swaps at each iteration.

Listing 2.5 shows the final improved version of the method. We encourage the reader to work our example out line by line, similar to what we have done in the previous section for `bubbleUp`, to have a better grasp of how it works.

Listing 2.5 Optimizing the `pushDown` method

**function** pushDown(pairs, index=0)
  current ← pairs[index]                                   ❶
  **while** index < firstLeafIndex(pairs) **do**                   ❷
       (child, childIndex) ← highestPriorityChild(index)   ❸
    **if** child.priority > current.priority **then**              ❹
      pairs[index] ← pairs[childIndex]                     ❺
      index ← childIndex
    **else**                                                   ❻
      **break**
  pairs[index] ← current                                   ❼

❶ Starts at the index passed as argument (by default at the first element of the array `A`)

❷ Leaves have no children, so they can’t be pushed down anymore. `firstLeafIndex` returns the index of the first leaf in the heap.

❸ We need to identify among the current node’s children the one with the highest priority, because that will be the only element that is safe to move up without breaking heap properties.

❹ If the highest priority child has higher priority than the current element . . .

❺ . . . then moves that child one position up (implicitly current goes one down).

❻ Otherwise, we have found the right place for the current element.

❼ Arrived at this line, `index` is the right position for current.

Now that we have defined all the helper methods we need, we can finally implement the API methods. As a spoiler, you will notice how all the logic about priority and max-heap vs min-heap is encapsulated into `bubbleUp` and `pushDown`, so by defining these two helper methods, we greatly simplify our life whenever it comes to adapting our code to different situations.

### 2.6.3 Insert

Let’s start with insertion. Listing 2.6 describes the pseudocode for inserting a new `(element, priority)` pair into a heap.

As mentioned at the end of the previous section, the `insert` method can be written by leveraging our helper methods, and as such its code is valid independent of whether we use a min-heap or a max-heap and of the definition of priority.

Listing 2.6 The `insert` method

**function** insert(element, priority)
  p ← Pair(element, priority)         ❶
  pairs.append(p)                     ❷
  bubbleUp(pairs, |pairs| – 1)        ❸

❶ Creates a new pair holding our new element and priority

❷ Adds the newly created pair at the end of our heap’s array, incrementing the array’s size

❸ After adding the new element, we need to make sure that the heap’s properties are reinstated.

The first two steps in listing 2.6 are self-explanatory: we are just performing some maintenance on our data model, creating a pair from the two arguments and appending it to the tail of our array. Depending on the programming language and the type of container used for pairs, we might have to manually resize it (statically dimensioned arrays) or just add the element (dynamic arrays or lists).[11](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013138)

The last step is needed because the new pair might violate the heap’s properties we defined in section 2.5.3. In particular, its priority could be higher than its parent’s. To reinstate heap properties, we then need to “bubble it up” toward the root of the heap, until we find the right spot in the tree structure.

Insert operations, just like delete or update, or even heap’s construction, will have to make sure the heaps properties holds on completion.

Because of the compact array implementation there will be no pointers to redirect, and the structure will be guaranteed to be consistent with properties 1 (by construction) and 2 (unless you have bugs in your code, of course).

So, we have to guarantee property 3, or in other words that each node is larger (for a max-heap) than each one of its `D` children. To do so, as we mentioned in previous sections, we might have to push it down (on deletes, and possibly updates), or bubble it up toward the root (on insert, and updates as well), whenever heap invariants are violated.

What’s the running time for insertion?

How many swaps will we need to perform in order to reinstate the heap properties? That depends on the concrete case, but since we move one level up at each swap, it can’t be more than the heap’s height. And since a heap is a balanced complete tree, that means no more than `logD(n)` swaps for a `D`-way heap with `n` elements.

The caveat here is that in any concrete implementation we need to expand the array beyond its current size. If we use a fixed-size static array, we will need to allocate it when the heap is created and set the max number of elements it can hold. In this case, however, the method is logarithmic in the worst case.

If we use a dynamic array, then the logarithmic bound becomes amortized and not worst-case, because we will periodically need to resize the array (see appendix C to check out more on dynamic arrays performance).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F9.png)

Figure 2.9 Adding a new element to a ternary max-heap. (A) The initial state of the heap; we first add the new element, with priority 9.5 at the end of the heap’s array. (B) Then we bubble it up until we find the right spot for it; that is, the first spot where it does not violate the heap’s properties.

If we go back to our task management example, we can see how insert works on a ternary max-heap. Suppose we have the initial state shown in figure 2.9 (A), describing the heap right after a call to `insert(“Add exception for Superbowl”, 9.5)` to add a particularly urgent task that marketing is pushing to have fixed by the end of the day. (And yes, they do cheat using fractional numbers for priority! But that’s probably not the first time you’ve seen someone changing requirements after you’ve pulled up a lot of work, is it?). After executing line #3 in listing 2.6, in fact, the list would be in this intermediate state (where heap properties are still violated).

Line #4 is then executed to reinstate heap property number 3, as we saw in section 2.6.2, and the final result is shown in figure 2.9 (B). Notice the order in which the children of the root appear in the array: siblings are not ordered, and in fact a heap doesn’t keep a total ordering on its elements like a BST would.

### 2.6.4 Top

Now that we have seen how to insert a new element, let’s define the `top` method that will extract the heap’s root and return it to the caller.

Figure 2.10 shows an example of a max-heap, highlighting the steps that are performed in listing 2.7.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F10.png)

Figure 2.10 Removing the top element from a binary heap. (A) From a valid ternary heap, we remove the root of the heap (storing it in a temporary location, in order to return it at the end). (B) We replace the old root with the last element in the array (which is also removed from its tail). The new root might violate heap’s properties (and it does, in this example), so we need to push it down, comparing it to its children to check that the heap’s properties are not violated. (C) We move the new root down toward the heap’s leaves, until we find a valid spot for it (that is, the first one that does not violate the heap’s properties).

The first thing we do is check that the heap is not empty. If it is, we certainly can’t extract the top element, so we need to issue an error.

The idea is that since we will remove the root of the heap, and by doing so we leave a “hole” in the array, we need to replace it with another element.

We could “promote” one of its children as the new root, as shown in listing 2.7. (One of them will certainly be the next-highest priority element; try to prove it as an exercise.)

Listing 2.7 The `top` method

**function** top()
  **if** pairs.isEmpty() **then error()**   ❶
  p ← pairs.removeLast()            ❷
  **if** pairs.isEmpty() **then**           ❸
    **return** p.element
  **else**  
   (element, priority) ← pairs[0]   ❹
    pairs[0] ← p                    ❺
    pushDown(pairs, 0)              ❻
    **return** element

❶ We need to check that the heap is not empty. Otherwise, we throw an error (or return `null`).

❷ Remove the last element in the `pairs` array and store it in a temporary variable.

❸ Now check again whether there are more elements left; if not, just return `p`.

❹ Otherwise we store the top pair (the first in the pairs array) in temporary variables . . .

❺ . . . and overwrite the first pair in the array with the previously saved `p`.

❻ The last element was a leaf, so it likely had a low priority. Now that it sits at the root, it might violate the heap properties, so we need to move it towards the leaves, until both its children have a lower priority than it.

However, we would then only move the problem to the subtree rooted at this node that we moved to the root, and then to the next level, and so on.

Instead, since we also have to shrink the array, and since it’s easier to add or remove the array’s elements from its tail, we can simply pop the last element in the array and use it to replace the root.

The caveat is that this new root might violate heap’s properties. As a matter of fact, being a leaf, the probability is pretty high that there is a violation. Therefore, we need to reinstate them using a utility method, `pushDown`.

Running time for `top()`

Similar to what happens for insertion, the nature of the heap and the fact that we move one step toward the leaves at every swap guarantee that no more than a logarithmic number of swaps is required, in the worst case.

In this case as well, concrete implementations have to deal with array sizing, and we can only guarantee an amortized logarithmic upper bound (if we need a worst-case logarithmic bound, we need to use static arrays with all the disadvantages they carry).

Back to our task management example. Let’s see what happens when calling `top``()` on the ternary heap shown at the end of figure 2.9 (B).

First, at line #3 we remove the last element of the heap, at index [8], and save it to a temporary variable; at line #4 we check whether the remaining array is empty. If that had been the case, the last element in the array would also have been the top of the heap (its only element!) and we could have just returned it.

But this is clearly not the case in this example, so we move on to line #7, where we store the first element of the heap, `<Unencrypted password on DB, 10>`, to a temporary variable, which will be returned by the method. At this point, we can imagine the heap to be like what’s shown in figure 2.10 (A), three disconnected branches without a root. To sew them together, we add a new root, using the element we had saved at line #3, the one entry that used to be the last in the array, at index `[8]`. Figure 2.10 (B) shows the situation at this point.

This new root, however, is violating heap’s properties, so we need to call `pushDown` at line #8 to reinstate them by swapping it with its second children, whose priority is `9.5`, and producing the heap shown in figure 2.10 (C); the `pushDown` method will handle this phase, stopping when the right position for element `<Memory Leak, 9>` is found.

To conclude this sub-section, it’s worth noting that the `peek` method returns the top element in the heap, but it just returns it without any side effects on the data structure. Its logic is therefore trivial, so we will skip its implementation.

### 2.6.5 Update

This is arguably the most interesting of heap’s public methods, even if it’s not always directly provided in implementations.[12](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013170)

When we change an element, its priority could stay unchanged, and in that case, we don’t need any further action. See listing 2.8. But it could also become lower or higher. If an element’s priority becomes higher, we need to check that it doesn’t violate the third invariant for its parents; if it becomes lower, it could violate the same invariant for its children.

Listing 2.8 The `update` method

**function** update(oldValue, newPriority)
  index ← pairs.find(oldValue)                     ❶
  **if** index ≥ 0 **then**                                ❷
    oldPriority ← pairs[index].priority
    pairs[index] ← Pair(oldValue, newPriority)
    **if** (newPriority < oldPriority) **then**            ❸
      bubbleUp(pairs, index)                       ❹
    **elsif** (newPriority > oldPriority) **then**        
      pushDown(pairs, index)                       ❺

❶ Get the position of the element to update.,

❷ Check whether the element is actually stored in the heap, and update it with the new value/priority.

❸ Check whether the new priority is higher than the old one.

❹ If so, bubble up the element towards the root.

❺ Otherwise, push it down toward the leaves.

In the former situation, we need to bubble up the modified element until we find a higher-priority ancestor or until we reach the root. In the latter, we instead push it down until all its children are lower priority, or we reach a leaf.

Running time for `update`

From a performance point of view, the challenge of this method is in line #2: the search for the old element can take linear time in the worst case, because on a failed search (when the element is not actually in the heap), we will have to search the whole heap.

To improve this worst-case scenario, it is possible to use auxiliary data structures that will help us in performing searches more efficiently. For instance, we can store a `map` associating each element in the heap with its index. If implemented with a hash table, the lookup would only need an amortized `O(1)` time.

We will see this in further detail when describing the `contains` method. For now, let’s just remember that if the `find` operation takes at most logarithmic time, the whole method is also logarithmic.

As we saw, the first case can always be implemented more efficiently[13](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013199) and luckily for us, most algorithms only require us to reduce elements’ priority.

### 2.6.6 Dealing with duplicates

So far, we’ve also assumed that our heap doesn’t hold duplicates. We will have to tackle further challenges if this assumption doesn’t hold true. In particular, we will need to figure out the order we use in case there are duplicates. To show why this is important, we will abandon our example for an easier configuration, illustrated in figure 2.11. For the sake of space we will only show priorities in nodes.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F11.png)

Figure 2.11 Updating an element in a binary max-heap with duplicates.

We could have two duplicates—call them `X` and `Y`—one being the child of the other. Let’s consider the case where `X` is a child of `Y`, and we call update to change their priority to a higher one. Inside `update`, after updating the two elements’ priorities, there would have to be two calls to `bubbleUp`, one for `X`, and one for `Y`. The catch is that if we run them in the wrong order, we will end with an inconsistent heap, which violates the heap’s properties.

Suppose we bubble up `X`, the children, first: then it will immediately find its parent (which is `Y`) and stop (because it has the same value). When we call `bubbleUp(Y),` we will discover that its parent now has lower priority, so we do need to move it toward the root. Unfortunately, we will never check `X` again, so it will never be moved up together with `Y` and heap’s property won’t be correctly reinstated.

Figure 2.11 provides a step-by-step description of how the `update` method could violate heap’s constraints if it’s not implemented in such a way to avoid duplicates:

- In step 1, we can see the initial max-heap.
    
- In step 2, all occurrences are changed from 4 to 8.
    
- In step 3, you see bubbling up the deepest node updated at index `3`. It will stop immediately because its parent has the same priority (it just updated together with current node). The nodes involved in the call to `bubbleUp` are highlighted with a red outline.
    
- In step 4, you see bubbling up the other node that was updated at index `1`. This time some swaps are actually performed as the node’s new priority, `8`, is higher than its parent’s (just `7`). The node that was bubbled up first, at step 3, will never be updated again, and so the heap’s properties will be violated.
    

How can we solve this issue? Following a left-to-right order for the calls to `bubbleUp` will guarantee that the heap properties are properly reinstated.

As an alternative, we could change the conditions in `bubbleUp` and `pushDown`, and only stop when we find a strictly higher-priority parent and strictly lower-priority children, respectively. Yet another alternative could be bubbling up (or pushing down) the nodes as we update them. Both these solutions, however, would be far from ideal for many reasons, not least performance-wise, because they would require a larger bound for the worst-case number of swaps (the proof is easy enough if you count the worst-case possible number of swaps in a path where all elements are the same—details are left to the reader).

### 2.6.7 Heapify

Priority queues can be created empty, but they are also often initialized with a set of elements. If that’s the case, and if we need to initialize the heap with `n` elements, we can still obviously create an empty heap and add those elements one by one.

To do so, we will need at most `O(n)` time to allocate the heap array, and then repeat `n` insertions. Every insertion is logarithmic, so we have an upper bound of `O(n log n)`.

Is this the best we can do? Turns out that we can do better. Suppose that we initialize the heap with the whole sets of `n` elements, in any order.

As we have seen, each position of the array can be seen as the root of a sub-heap. So, leaves, for example, are trivial sub-heaps, containing only one element. Being singletons, they are valid heaps.

How many leaves are there in a heap? That depends on the branching factor. In a binary heap, half the nodes are leaves. In a 4-way heap, the last three-fourths of the array contain leaves.

Let’s stick to the case of a binary heap, for the sake of simplicity. We can see an example of a min-heap in figure 2.12, where we can follow step by step how `heapify` works.

If we start at the last internal node for the heap—let’s call it `X`—it will have at most two children, both leaves, and hence both valid heaps.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F12.png)

Figure 2.12 Heapification of a small array. The boxes with rounded corners surround valid sub-heaps; that is, portions of the tree for which we have verified that they abide by the heap properties. Step 1: Initially, only leaves are valid min-heap; here, smaller numbers mean higher priority. Step 2: The element from which we start our iteration is the first internal node in the heap, at index 2 in our array. In the bottom part of the leftmost figure, it is pointed at by an arrow. We repair the heap properties for the sub-heap rooted at this node by swapping its content with its child’s. Step 3: We move the arrow pointing to the current element one position to the left and repeat the operation for the sub-heap rooted at index 1. Step 4: Finally, the whole array is heapified, by pushing down the temporary root (with priority 5) until we find its right place.

We can’t say yet if `X` is the root of a valid heap, but we can try to push it down and possibly swap it with the smallest of its children. Once we have done that, we are sure that the sub-heap rooted at `X` is a valid heap. This happens because `pushDown`, by construction, reinstates the heap properties on a sub-heap where all children are valid heaps, and only the root might be misplaced.

If we now move one position left in the array, we can repeat the procedure, and get another valid sub-heap. Continuing on, at some point we visit all internal nodes that only have leaves as their children. And after that, we’ll start with the first internal node that’s the root of a sub-heap with height 2. Let’s name it `Y`. If we tackle elements in a right-to-left order in the array, we have the guarantee that `Y`’s children are sub-heaps that

- Have at most height 1 and
    
- Whose properties have already been fixed
    

Once again, we can use `pushDown` and be sure the sub-heap rooted at `Y` will be a valid heap afterward.

If we repeat these steps for all nodes until we reach the root of our heap, we have the guarantee that we will end up with a valid heap, as shown in listing 2.9.

Listing 2.9 The `heapify` method

**function** heapify(pairs)
  **for** index **in** {(|pairs|-1)/D .. 0} **do**     ❶
    pushDown(pairs, index)                 ❷

❶ We start at the first internal node of the heap (`D` is the branching factor) and iterate until the root.

❷ Ensures the sub-heap rooted at index is a valid heap

Running time for `heapify`

In total, in a binary heap, we will call `pushDown n/2` times, with an initial estimate of the upper bound of `O(n*log(n))`.

However, notice that the sub-heaps that only have leaves as children have a height equal to 1, so they only need at most one swap to be fixed. And there is `n/2` of them. Likewise, sub-heaps with height 2 only need at most two swaps, and we have `n/4` of them.

Going up toward the root, the number of swaps increases but the number of calls to `pushDown` decreases at the same rate. Finally, we’ll have only two calls for `pushDown` that requires at most `log2(n)–1` swaps, and only one call, the one on the root, that requires `log2(n)` swaps in the worst case.

Putting all these together, we see that the total number of swaps is given by

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F_EQ1.png)

Since the last summation is limited by geometric series with seed 2, we have

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F_EQ2.png)

and therefore, the total number of swaps is linear in the worst case (at most `2n`).

So, our final analysis is that heapify’s complexity is `O(n)`.

We leave the computation for a d-ary heap as an exercise. It is similar to the previous one, replacing the branching factor.

### 2.6.8 Beyond API methods: Contains

One thing that heaps are definitely not good for is checking whether or not an element is stored in them. We have no other choice than going through all the elements until we find the one we were looking for, or we get to the end of the array. This means a linear time algorithm. Compare it with a hash table’s optimal average constant time, or even `O(log(n))` for binary search trees (average) or balanced binary search trees (worst case).

However, we also would like to support priority increment/decrement. As we saw in section 2.6.5, it’s paramount for these operations to efficiently retrieve the element whose priority needs to be changed. Therefore, when implementing heaps, we might add an auxiliary field, a `HashMap` from elements to positions, that allows us to check whether an element is in the heap (or get its position) in constant time, on average. See a possible implementation of `contains` in listing 2.10.

Listing 2.10 The `contains` method

**function** contains(elem) 
  index ← elementToIndex[elem]
  **return** index >= 0

The function uses an extra field `elementToIndex` we can add to our heap. This is possible under two assumptions:

- That `elementToIndex[elem]` by default returns `-1` if `elem` is not stored in the heap.
    
- That we don’t allow duplicate keys in the heap; otherwise, we will need to store a list of indices for each key.
    

### 2.6.9 Performance recap

With the description of the `contains` method, we have concluded our overview of this data structure.

We have presented several operations on heaps, so it feels like the right time to order and recap their running time (table 2.4), and as importantly, show the amount of extra memory they need.

Table 2.4 Operations provided by heaps, and their cost on a heap with n elements

|Operation|Running time|Extra space|
|---|---|---|
|`Insert`|`O(log(n))`|`O(1)`|
|`Top`|`O(log(n))`|`O(1)`|
|`Remove`|`O(log(n))[a](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1024451)`|`O(n)[a](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1024451)`|
|`Peek`|`O(1)`|`O(1)`|
|`Contains (naïve``)`|`O(n)`|`O(1)`|
|`Contains`|`O(1)[a](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1024451)`|`O(n)[a](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1024451)`|
|`UpdatePriority`|`O(log(n))[a](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1024451)`|`O(n)[a](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1024451)`|
|`Heapify`|`O(n)`|`O(1)`|

a Using the advanced version of contains and maintaining an extra map from elements to indices

As with most things in computer science, time versus space is often a tradeoff. Nevertheless, sometimes there is a tendency to neglect the extra memory factor in informal analysis. With the volumes we’ve operated since the dawn of the era of big data, however, our data structures need to hold and process billions of elements, or even more. Therefore, a fast algorithm that consumes quadratic space could be the best choice for small volumes of data, but it becomes impractical for some real case scenarios where you need to scale out. And so a slower algorithm using constant or logarithmic space could be the choice of election as our dataset grows.

The takeaway is that it’s even more important to take the memory factor into account as early as possible when we design a system that needs to scale.

For heaps, consider that we naturally need constant extra space per element, and linear extra space in total, to host a map from elements to indices.

We have closely examined all operations. Yet, it’s worth spending a few more words on a couple of things.

For `insert` and `top`, the running time guarantee is amortized, not worst case. If a dynamic array is used to provide a flexible size, some calls will require linear time to resize the array. It can be proven that to fill a dynamic array with `n` elements, at most `2n` swaps are needed. However, the worst-case logarithmic guarantee can be offered only if the heap size is set from the beginning. For this reason, and for allocation/garbage collection efficiency, in some languages (for instance, Java), it is advisable to initialize your heaps to their expected size, if you have a reasonable estimate that will remain true for most of the container’s life.

The performance for `remove` and `updatePriority` relies on the efficient implementation of `contains`, in order to provide a logarithmic guarantee. To have efficient search, however, we need to keep a second data structure besides the array for fast indirection. The choice is between a hash table or a bloom filter (see chapter 4).

In case either is used, the running time for contains is assumed to be constant, with a caveat: the hash for each element needs to be computable in constant time. Otherwise, we will need to take that cost into account in our analysis.

### 2.6.10 From pseudo-code to implementation

We have seen how a d-way heap works in a language-agnostic way. Pseudo-code provides a good way to outline and explain a data structure, without worrying about the implementation details so you can focus on its behavior.

At the same time, however, pseudo-code is of little practical use. To be able to move from theory to practice, we need to choose a programming language and implement our d-way heap. Independently of what platform we choose, language-specific concerns will arise and different problematics need to be taken into consideration.

We’ll provide implementations of the algorithms in this book, in an effort to give readers a way to experiment and get their hands dirty with these data structures.

The full code, including tests, can be found in our book’s repo on GitHub.[14](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1025376)

## 2.7 Use case: Find the k largest elements

In this section we are going to describe how we can use a priority queue to keep track of the `k` largest elements of a set.

If we have the full set of `n` elements in advance, we have a few alternatives that don’t need any auxiliary data structure:

- We could sort the input and take the last `k` elements. This naïve approach requires `O(n*log(n))` comparisons and swaps and, depending on the algorithm, might require additional memory.
    
- We could find the largest element from the set and move it to the end of the array, then look at the remaining `n-1` elements and find the second to last and move it to position `n-2`, and so on. Basically, this algorithm runs the inner cycle of `Selection Sort` algorithm `k` times, requiring `O(k)` swaps and `O(n*k)` comparisons. No additional memory would be needed.
    

In this section we will see that by using a heap, we can achieve our goal using `O(n+k*log(k))` comparisons and swaps, and `O(k)` extra memory. This is a game-changing improvement if `k` is much smaller than `n`. In a typical situation, `n` could be on the order of millions or billions of elements, and `k` between a hundred and a few thousand.

Moreover, by using an auxiliary heap, the algorithm can naturally be adapted to work on dynamic streams of data and also to allow consuming elements from the heap.

### 2.7.1 The right data structure . . .

When your problem involves finding a subset of largest/smallest elements, priority queues seem like a natural solution.

In programming, choosing the right data structure can make a difference.[15](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013228) That’s not always enough, because you also need to use it correctly.

Suppose, for instance, that we have a static set of elements available from the beginning. We could use a max-heap, insert all `n` elements, and then extract the largest `k` of them.

We would need `O(n)` extra space for the heap, and then use `heapify` to create it from the full set in linear time, `O(n)`. Then we would call `top k` times, with a cost of `O(log(n))` for each call. The total cost for this solution would be `O(n+k*log(n))` comparisons and swaps.

That’s already better than the naïve solutions, but, if you think about it, if[16](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013244) `n>>k`, we are creating a huge heap just to extract a few elements. That certainly sounds wasteful.

### 2.7.2 . . . and the right use

So our goal should be having a small heap with `k` elements. Using a max-heap doesn’t really work anymore. Let’s see why with an example.

Suppose we want to find the largest three of the following numbers: 2, 4, 1, 3, 7, 6. We add the first three and we have the following max-heap: [4, 2, 1]. Now we proceed to the next number in the list, and it’s 3. It’s larger than two out of three elements currently in the heap, but we have no way of knowing this, because we can only peek at the top of the heap. Then we can insert 3 into the heap, and we obtain [4, 3, 1, 2]. Now, if we want to keep the size of the heap at `k` elements, we need to remove one, which is the minimum. How do we know where it is inside the heap? We only know where the maximum is (at the top of the max-heap), and so the search for the min could take up to linear time (even noting that the minimum will be in one of the leaves, there are unfortunately a linear number of them, `n/D`).

You can verify that even by inserting the elements in a different order, we often find a similar situation.

The catch is that when we want the `k` largest elements, at each step we are interested in understanding if the next number we evaluate is larger than the smallest of the `k` elements we already have. Hence, rather than a max-heap, we can use a min-heap bound to `k` elements where we store the largest elements found so far.

For each new element, we compare it to the top of the heap, and if the new one is smaller, we are sure it’s not one of the `k` largest elements. If our new element is larger than the heap’s top (that is, the _smallest_ of our `k` elements), then we extract the top from the heap and then add our newly arrived. This way, updating the heap at each iteration costs us only constant time,[17](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013263) instead of the linear time bound if we used a max-heap.

### 2.7.3 Coding it up

That’s neat, right? And simple to implement. Since code is worth a thousand words, let’s see our heap in action in listing 2.11.

Listing 2.11 Top `k` elements of a list

**function** topK(A, k)
heap ← DWayHeap()                                   ❶
**for** el **in** A **do**                                      ❷
  **if** (heap.size == k **and** heap.peek() < el) **then**     ❸
    heap.top()                                      ❹
  **if** (heap.size < k) **then**                           ❺
    heap.insert(el)                                 ❻
**return** heap                                         ❼

❶ Creates an empty min-heap

❷ Iterates through the elements in the array `A`

❸ If we have already added at least `k` elements, check if the current element is larger than the top of the heap.

❹ In that case, we can safely remove and discard the heap’s top, because it won’t be among the `k` largest elements. After this, the heap will have `k-1` elements.

❺ If, at this check, the heap size is less than `k` . . .

❻ . . . we must add the current element.

❼ Returns the heap with the largest `k` elements. We might as well use heapsort to return the elements in the right order, at a small additional cost.

## 2.8 More use cases

The heap is one of the most universally used data structures. Together with stack and queue, it is the basis of almost every algorithm that needs to process the input in a specific order.

Replacing a binary heap with a d-ary heap can improve virtually any piece of code that uses a priority queue. Before delving into a few algorithms that can benefit from the use of a heap, make sure you are familiar with graphs, because most of these algorithms will concern this data structure. To that end, chapter 14 provides a quick introduction to graphs.

Let’s now discuss some algorithms that can benefit from the use of a heap.

### 2.8.1 Minimum distance in graphs: Dijkstra

Priority queues are crucial to implementing Dijkstra and A* algorithms, described in detail in chapter 14, sections 14.4 and 14.5. Figure 2.13 shows a minimal example of a graph, illustrating the concept of shortest path between two vertices. As we will discuss in chapter 14, the running time of these fundamental algorithms on graphs (which compute the minimum distance to a target) heavily depends on the implementation chosen for the priority queue, and upgrading from a binary to a d-ary heap can provide a consistent speedup.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F13.png)

Figure 2.13 A directed graph showing the shortest path between vertices `A` and `C`

### 2.8.2 More graphs: Prim's algorithm

Prim’s algorithm computes the minimum spanning tree (MST) of an undirected, connected graph `G`.

Suppose `G` has `n` vertices. The minimum spanning tree of `G` is

1. A tree (a connected, undirected, acyclic graph)
    
2. That is a subgraph of `G` with `n` vertices and
    
3. Whose sum of edges’ weights is the least possible among all of the subgraphs of `G` that are also trees and span over all `n` vertices
    

Considering the graph in the example shown in section 2.8.1, its minimum spanning tree would be the one in figure 2.14.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F14.png)

Figure 2.14 A spanning tree for the graph in figure 2.13

Prim’s algorithm works exactly as Dijkstra’s, except

- Without keeping track of the distance from the source.
    
- Storing the edge that connected the front of the visited vertices to the next closest vertex.
    
- The vertex used as “source” for Prim’s algorithm is going to be the root of the MST.
    

It should be no surprise that its running time is similar to Dijkstra’s:

- `O(V2)` using arrays (sorted or unsorted) for the priority queue
    
- `O(V*log(V) + E*log(V))` using binary or d-ary heap
    
- `O(V*log(V) + E)` using Fibonacci heap
    

### 2.8.3 Data compression: Huffman codes

Huffman’s algorithm is probably the most famous data compression algorithm, and you have likely already heard of it if you took an “introduction to CS” course. It is a simple, brilliant, _greedy_ algorithm that, despite not being the state of the art for compression anymore, was a major breakthrough in the ‘50s.

A Huffman code is a tree, built bottom-up, starting with the list of different characters appearing in a text and their frequency. The algorithm iteratively

1. Selects and removes the two elements in the list with the smallest frequency
    
2. Then creates a new node by combining them (summing the two frequencies)
    
3. And finally adds back the new node to the list
    

While the tree itself is not a heap, a key step of the algorithm is based on efficiently retrieving the smallest elements in the list, as well as efficiently adding new elements to the list. You probably have guessed by now that, once again, that’s where heaps come to the rescue.

Let’s take a look at the algorithm itself in listing 2.12.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F15.png)

Figure 2.15 A Huffman coding tree built from this character frequency table: A=0.6, B=0.2, C=0.07, D=0.06, E=0.05, and F=0.02

We assume the input for the algorithm is a text, stored in a string (of course, the actual text might be stored in a file or stream, but we can always have a way to convert it to a string[18](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013288)), and the output is a map from characters to binary sequences.

The first sub-task that we need to perform is to transform the text: we want to compute some statistics on it to identify the most used and least used characters in it. To that end, we compute the frequency of characters in the text.[19](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013310)

The details of the `ComputeFrequencies` method at line #2 are both out of scope and (at least in its basic version) simple enough, and there is no need to delve into that helper method here.

Once we have computed the frequency map, we create a new priority queue and then at lines #4 and #5 we iterate over the frequency map, creating a new `TreeNode` for each character and then adding it to the priority queue, as in listing 2.12. Obviously, considering the subject of this chapter, for the queue we use a heap, and in particular a min-heap, where the element at the top is the one with the smallest value for the priority field. And in this case the priority field is (not surprisingly) the frequency field of the `TreeNode`.

Listing 2.12 The Huffman coding algorithm

**function** huffman(text)
  charFrequenciesMap ← ComputeFrequencies(text)
  priorityQueue ← MinHeap()
  **for** (char, frequency) **in** charFrequenciesMap **do**
    priorityQueue.insert(TreeNode([char], frequency))
  **while** priorityQueue.size > 1 **do**
    left ← priorityQueue.top()
    right ← priorityQueue.top()
    parent ← TreeNode(left.chars + right.chars,
                      left.frequency + right.frequency)
    parent.left ← left
    parent.right ← right
    priorityQueue.insert(parent)
  **return** buildTable(priorityQueue.top(), [], Map())

Each `TreeNode`, in fact, contains two fields (besides the pointers to its children): a set of characters and the frequency of those characters in the text, computed as the sum of the frequencies of individual characters.

If you look at figure 2.15, you can see that the root of the final tree is the set of all characters in our example text, and hence the total frequency is 1.

This set is split into two groups, each of which is assigned to one of the root’s children, and so each internal node is similarly split until we get to leaves, each of which contains just one character.

Back to our algorithm, you can see how the tree in figure 2.15 is constructed bottom-up, and lines #2 to #5 in listing 2.12 take care of the first step, creating the leaves of the tree and adding them to the priority queue.

Now, from line #6 we enter the core of the algorithm: until there is only one element left in the queue, we extract the top `TreeNode` entries, in lines #7 and #8. As you can see in figure 2.16 (B), those two elements will be the subtrees with the lowest frequencies so far.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F16.png)

Figure 2.16 The first step in the Huffman coding algorithm. As mentioned in the text, the algorithm will use two auxiliary data structures, a priority queue and a binary tree. Each tree node will have a value, a set of characters in the text, and a priority, the sum of the frequencies of those characters in the text. (A) Initially, we create one tree node for each character, associated with its frequency in the text. We also add each node into a priority queue, using the frequency as its priority (smaller frequency means higher priority hence we would use a min-heap). (B) We extract two elements from the top of the priority queue.(C) We create a new tree node to which we add the two nodes extracted at step (B) as its children. By convention, we assume that the smallest node will be added as left child and the second-smallest as right child (but any consistent convention works here). The newly created node will hold the union of the set of characters in its children as value, and the sum of the two priorities as priority. (D) Finally, we can add the new root for this subtree back to the priority queue. Note that the nodes in the heap are showed in sorted order, but for the sake of simplicity the order in which nodes are stored inside a priority queue is an implementation detail, the contract for PQ’s API only guarantees that when we dequeue two elements, those will be the ones with the smallest frequencies.

Let’s call these subtrees `L` and `R` (the reason for these names will be apparent soon).

Figure 2.16 (C) shows the actions performed in lines #9 to #11 of our pseudocode: a new `TreeNode` is created (let’s call it `P`) by merging the two entries’ character sets and setting its frequency as the sum of the old subtrees’ frequencies. Then the new node and two subtrees are combined in a new subtree, where the new node `P` is the root and the subtrees `L` and `R` are its children.

Finally, at line #12 we add this new subtree back into the queue. As it’s shown in figure 2.16 (D), it can sometimes be placed at the top of the queue, but that’s not always the case; the priority queue will take care of this detail for us (notice that here the priority queue is used as a black box, as we discussed in section 2.4).

The Huffman coding algorithm (building a table from the tree)

**function** buildTable(node, sequence, charactersToSequenceMap)
  **if** node.characters.size == 1 **then**
    charactersToSequenceMap[node.characters[0]] ← sequence 
  **else**
    **if** node.left <> **null then**
      buildTable(node.left, 0 + sequence, charactersToSequenceMap) 
    **if** node.right <> **null then**
      buildTable(node.right, 1 + sequence, charactersToSequenceMap) 
  **return** charactersToSequenceMap

These steps are repeated until there is only one element left in the queue (figure 2.17 shows a few more steps), and that last element will be the `TreeNode` that is the root of the final tree.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F17.png)

Figure 2.17 The result of the next couple of steps in the Huffman coding algorithm. (A) We dequeue and merge the top two nodes on the heap, C and D. At the end of this step, EF and CD become the two smallest nodes in the heap. (B) Now we merge those two nodes into CDEF, and we add it back to the heap. Which node between CDEF and B will be kept at the top of the priority queue is an implementation detail, and it’s irrelevant for the Huffman coding algorithm (the code will change slightly depending on which one is extracted first, but its compression ratio will remain unchanged). The next steps are easy to figure, also using figure 2.15 as a reference.

We can then use it in line #13 to create a compression table, which will be the final output of the `huffman` method. In turn, the compression table can be used to perform the compression of the text by translating each one of its characters into a sequence of bits.

While we won’t show this last step,[20](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013324) we provide listing 2.13 with the steps needed to create a compression table from the tree in figure 2.15. And even if this goes beyond the scope of this chapter (because the method doesn’t use a priority queue), providing a brief explanation should help those readers interested in writing an implementation of Huffman coding.

We wrote the `buildTable` method using recursive form. As explained in appendix E, this allows us to provide cleaner and more easily understandable code, but in some languages concrete implementations can be more performant when implemented using explicit iterations.

We pass three arguments to the method: a `TreeNode node` that is the current node in the traversal of the tree, a sequence that is the path from the root to current node (where we add a 0 for a “left turn” and a 1 for a “right turn”), and the `Map` that will hold the associations between characters and bit sequences.

At line #2, we check if the set of characters in the node has only one character. If it does, it means we have reached a leaf, and so the recursion can stop. The bit sequence that is associated with the character in the node is the path from root to current node, stored in the `sequence` variable.

Otherwise, we check whether the node has left and right children (it will have at least one, because it’s not a leaf) and traverse them. The crucial point here is how we build the `sequence` argument in the recursive calls: if we traverse the left child of current node, we add a 0 at the start of the sequence, while if we traverse the right child, we add a 1.

Table 2.5 shows the compression table produced starting from the tree shown in figure 2.15; the last column would not be part of the actual compression table, but it’s useful to understand how the most used characters end up translated into shorter sequences (which is the key to an efficient compression).

Table 2.5 The compression table created from the Huffman tree in figure 2.15

|Character|Bit sequence|Frequency|
|---|---|---|
|`A`|0|0.6|
|`B`|10|0.2|
|`C`|1100|0.07|
|`D`|1101|0.06|
|`E`|1110|0.05|
|`F`|1111|0.02|

Looking at the sequences, the most important property is that they form a prefix code: no sequence is the prefix of another sequence in the code.

This property is the key point for the decoding: iterating on the compressed text, we immediately know how to break it into characters.

For instance, in the compressed text `1001101`, if we start from the first character, we can immediately see that sequence `10` matches `B`, then the next `0` matches `A`, and finally `1101` matches `D`, so the compressed bit sequence is translated into `“BAD”`.

## 2.9 Analysis of branching factor[21](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013338)

Now that we know how d-way heaps work, the next question we need to ask is this: Wouldn’t we be just fine with a regular, binary heap? Is there an advantage in a higher branching factor?

### 2.9.1 Do we need d-ary heaps?

Usually binary heaps are enough for all our programming needs. The main advantage of this data structure is that it guarantees a logarithmic running time for each one of the common operations. In particular, being a binary balanced tree, the main operations are guaranteed to require a number of comparisons proportional, in the worst case, to `log`2`(N)`. As we discuss in appendix B, this guarantees that we can run these methods on much larger containers than if the running time was linear. Consider that even with a billion elements, `log`2`(N)` just evaluates to about 30.

As we have seen in the introduction, constant factors are irrelevant for the running time, that is, `O(c*N) = O(N)`, and we know from algebra that two logarithms with different bases only differ by a constant factor, in particular

logb`(N) = log`2`(N) / log`2`(b)`

So, in conclusion, we have

O(log2(N)) = O(log3(N)) = O(log(N)) 

When we move to the implementation, however, constant factors matter. They matter so much that, in some edge cases, algorithms that would be better according to the running time analysis actually are slower than a simpler algorithm with worse running time, at least for any practical input (for instance, if you compare `2n` and `n*100100000`, then for the constant factor not to matter anymore, the input should be huge).

A prominent example of this behavior is given by Fibonacci heap[22](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013354): in theory they provide amortized constant time for some crucial operations such as insert or priority update, but in practice they are both complicated to implement and slow for any viable input size.

The constant factors, in general, are due to several different reasons that include

- Lag for reading/writing to memory (scattered vs localized readings)
    
- The cost of maintaining counters or to iterate loops
    
- The cost of recursion
    
- Nitty/gritty coding details that in the asymptotic analysis are abstracted away (for instance, as we have seen, static vs dynamic arrays)
    

So, at this point it should be clear that in any implementation we should strive to keep this constant multiplicators as small as possible.

Consider this formula:

`logb(N) = log`2`(N) / log2(b)`

If `b > 2`, it’s apparent that `logb(N) < log2(N)`, and therefore if we have a logarithmic factor in our algorithm’s running time, and we manage to provide an implementation that instead of `log2(N)` steps will require `logb(N)`, while all other factors stay unchanged, then we will have provided a (constant-time) speed-up.

In section 2.10 `we` will further investigate how this applies to d-ary heaps.

### 2.9.2 Running time

So the answer is yes, there is an advantage in tuning the heap’s branching factor, but compromise is the key.

The insertion will always be quicker with larger branching factors, as we at most need to bubble up the new element to the root, with `O(logD(n))` comparisons and swaps at most.

Instead, the branching factor will affect deletion and priority update. If you recall the algorithms for popping elements, for each node we need to first find the highest priority among all its children, then compare it to the element we are pushing down.

The larger the branch factor, the smaller the height of the tree (it shrinks logarithmically with the branching factor). But, on the other hand, the number of children to compare at each level also grows linearly with the branch factor. As you can imagine, a branching factor of 1000 wouldn’t work very well (and it would translate into a linear search for less than 1001 elements!).

In practice, through profiling and performance tests, the conclusion has been reached that in most situations, `D=4` is the best compromise.

### 2.9.3 Finding the optimal branching factor

If you are looking for an optimal value for `D` that works in every situation, then you are going to be disappointed. To a certain extent, theory comes to the rescue by showing us a range of optimal values. It can be shown that the optimal value can’t be greater than `5`. Or, to put it another way, it can be mathematically proven that

- The tradeoff between insert and delete is best balanced with `2 <= D <= 5.`
    
- A 3-way heap is in theory faster than a 2-way heap.
    
- 4-way heaps and 3-way heaps have similar performance.
    
- 5-way heaps are a bit slower.
    

In practice, the best value for `D` depends on the details of the implementation and of the data you are going to store in the heap. The optimal branching factor for a heap can only be determined empirically, case by case. There is no overall optimal branching factor, and it depends on the actual data and on the ratio of insertions/deletions, or, for instance, how expensive it is to compute priority versus copying elements, among other things.

In common experience, binary heaps are never the fastest, 5-way heaps are seldom faster (for narrow domains), and the best choice usually falls between 3 and 4, depending on nuances.

So while I feel safe suggesting starting with a branching factor of 4, if this data structure is used in a key section of your application and small performance improvement can make a relevant difference, then you need to tune the branching factor as a parameter.

### 2.9.4 Branching factor vs memory

Notice that I suggested the larger branching factor among the two best performing ones for a reason. It turns out there is, in fact, another consideration to be made when looking for the optimal branching factor for heaps: locality of reference.

When the size of the heap is larger than available cache or than the available memory, or in any case where caching and multiple levels of storage are involved, then on average a binary heap requires more cache misses or page faults than a d-ary heap. Intuitively, this is due to the fact that children are stored in clusters, and that on update or deletion, for every node reached, all its children will have to be examined. The larger the branching factor becomes, the more the heap becomes short and wide, and the more the principle of locality applies.

D-way heap appears to be the best traditional data structure for reducing page faults.[23](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013386) New alternatives focused on reducing cache misses and page swaps have been proposed over the years; for instance, you can check out splay trees.

While these alternatives aren’t in general able to have the same balance between practical and theoretical performance as heaps, when the cost of page faults or disk access dominates, it might be better to resort to a linearithmic[24](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013403) algorithm with higher locality rather than sticking with a linear algorithm with poor locality.

## 2.10 Performance analysis: Finding the best branching factor

We’ve discussed the theory. Now let’s try to apply it to a real case and describe how to profile the implementation of a data structure and an application.

We have seen that priority queues are a key component in the Huffman compression pipeline. If we measure the performance of our heap’s methods as the number of swaps performed, we have shown that we have at most `h` swaps per method call, if `h` is the height of the heap. In section 2.5 we also showed that because the heap is a complete balanced tree, the height of a d-ary heap is exactly `logD(n)`.[25](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013425)

Therefore, for both methods `insert` and `top`, it would seem that the larger the branching factor, the smaller the height, and in turn the better heap performance should be.

But just limiting to swaps doesn’t tell us the whole story. In section 2.9 we delve into the performance analysis of these two methods and take into consideration also the number of array accesses, or equivalently the number of comparisons on heap elements that are needed for each method. While `insert` accesses only one element per heap’s level, method `top` traverses the tree from root to leaves, and at each level it needs to go through the list of children of a node. Therefore, it needs approximately up to `D*logD(n)` accesses on a heap with branching factor `D` and containing `n` elements.

Tables 2.6 and 2.7 summarize the performance analysis for the three main methods in heaps’ API.

Table 2.6 Main operations provided by a d-ary heap, and the number of swaps (assuming n elements)

|Operation|Number of swaps|Extra space|
|---|---|---|
|`Insert`|`~logD(n)`|`O(1)`|
|`Top`|`~logD(n)`|`O(1)`|
|`Heapify`|`~n`|`O(1)`|

For method `top`, a larger branching factor doesn’t always improve performance, because while `logD(n)` becomes smaller, `D` becomes larger. Bringing it to an extreme, if we choose `D > n-1`, then the heap becomes a root with a list of `n-1` children, and so while `insert` will require just `1` comparison and `1` swap, `top` method will need `n` comparisons and `1` swap (in practice, as bad as keeping an unsorted list of elements).

There is no easy[26](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013446) way to find a value for `D` that minimizes function `f(n) = D*logD(n)` for all values of `n`, and besides, this formula gives us just an estimate of the maximum number of accesses/swaps performed. The exact number of comparisons and swaps actually performed depends on the sequence of operations and on the order in which elements are added.

Table 2.7 Cost of main operations provided by heaps as a number of comparisons (assuming n elements)

|Operation|Number of comparisons|Extra space|
|---|---|---|
|`Insert`|`~logD(n)`|`O(1)`|
|`Top`|`~D*logD(n)`|`O(1)`|
|`Heapify`|`~n`|`O(1)`|

Then the question arises: How do we choose the best value for the branching factor?

The best we can do here is profile our applications to choose the best value for this parameter. In theory, applications with more calls to `insert` than to `top` will perform better with larger branching factors, while when the ratio of calls between the two methods approaches 1.0, a more balanced choice will be best.

### 2.10.1 Please welcome profiling

And so we are stuck with _profiling_. If you are asking yourself “What’s profiling?” or “Where do I start?”, here are a few tips:

- Profiling means measuring the running time and possibly the memory consumption of different parts of your code.
    
- It can be done at a high level (measuring the calls to high level functions) or a lower level, for each instruction, and although you can set it up manually (measuring the time before and after a call), there are great tools to aid you—usually guaranteeing an error-free measurement.
    
- Profiling can’t give you general-purpose answers: it can measure the performance of _your_ code on the input you provide.
    
- In turn, this means that the result of profiling is as good as the input you use, meaning that if you only use a very specific input, you might tune your code for an edge case, and it could perform poorly on different inputs. Also, another key factor is the data volume: to be statistically significant, it can be nice to collect profiling results on many runs over (pseudo)random inputs.
    
- It also means that results are not generalizable to different programming languages, or even different implementations in the same language.
    
- Profiling requires time. The more in depth you go with tuning, the more time it requires. Remember Donald Knuth’s advice: “premature optimization is the root of all evil.” Meaning you should only get into profiling to optimize critical code paths. Spending two days to shave 5 milliseconds on an application that takes 1 minute is frankly a waste of time (and possibly, if you end up making your code more complicated to tune your app, it will also make your code worse).
    

If, in spite all of these disclaimers, you realize that your application actually needs some tuning, then brace yourself and choose the best profiling tool available for your framework.

To perform profiling, obviously we will have to abandon pseudocode and choose an actual implementation; in our example, we will profile a `Python` implementation of Huffman encoding and d-ary heap. You can check out the implementation on our repo[27](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013463) on GitHub.

Code and tests are written in Python 3, specifically using version 3.7.4, the latest stable version at the time of writing. We are going to use a few libraries and tools to make sense of the profiling stats we collect:

- Pandas
    
- Matplotlib
    
- Jupyter Notebook
    

To make your life easier, if you’d like to try out the code I suggest you install the Anaconda distribution, which already includes the latest Python distribution and all the packages listed.

To do the actual profiling, we use `cProfile` package, which is already included in the basic `Python` distro.

We won’t explain in detail how to use `cProfile` (lots of free material online covers this, starting from the Python docs linked previously), but to sum it up, `cProfile` allows running a method or function and records the per-call, total, and cumulative time taken by every method involved.

Using `pStats`.`Stats`, we can retrieve and print (or process) those stats; the output of profiling looks something like what’s shown in figure 2.18.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F18.png)

Figure 2.18 Printing Stats after profiling Huffman encoding function.

Now, to reduce the noise, we are only interested in a few methods, specifically the functions that use a heap: in particular `_frequency_table_to_heap`, which takes the dictionary with the frequency (or number of occurrences) of each character in the input text and creates a heap with one entry per character, and `_heap_to_tree`, which in turn takes the heap created by the former function and uses it to create the Huffman encoding tree.

We also want to track down the calls to the heap methods used: `_heapify`, `top`, and `insert`. Instead of just printing those stats, we can read them as a dictionary, and filter the entries corresponding to those five functions.

To have meaningful, reliable results, we also need to profile several calls to the method `huffman.create_encoding`, and so processing the stats and saving the result to a CSV file seems the best choice anyway.

To see the profiling in action, check out our example[28](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1032128) on GitHub. The example profiles several calls to the method creating a Huffman encoding over an ensemble of large text files[29](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013479) and a bitmap image. The bitmap needs to be duly pre-processed in order to make it processable as text. The details of this pre-processing are not particularly interesting, but for the curious reader we encode image bytes in base64 to have valid characters.

### 2.10.2 Interpreting results

Now that we’ve stored the results of profiling in a CSV file, we _just_ need to interpret them to understand what’s the best choice of branching factor for our Huffman encoding app. At this point it should look like a piece of cake, right?

We can do it in many ways, but my personal favorite is displaying some plots in a Jupyter Notebook: take a look here at an example.[30](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013497) Isn’t it great that GitHub lets you already display the Notebook without having to run it locally?

Before delving into the results, though, we need to take a step back: because we will use a boxplot[31](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1032218) to display our data, let’s make sure we know how to interpret these plots first. If you feel you could use a hint, figure 2.19 comes to the rescue!

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F19.png)

Figure 2.19 Explaining how to read an example boxplot. Boxplots show the distribution of samples using a nice and effective chart. The main idea is that the most relevant samples are those whose values lie between the first and third quartile. That is, we find three values, Q1, Q2 (aka the median), and Q3 such that 25% of the samples are smaller than Q1; 50% of the samples are smaller than Q2 (which is the very definition of median, by the way); and 75% of the samples are smaller than Q3. The box in the boxplot is meant to clearly visualize the values for Q1 and Q3. Whiskers, instead, shows how far from the median the data extends. To that end, we could also display outliers, that is, samples that are outside the whiskers’ span. Sometimes outliers, though, end up being more confusing than useful, so in this example we will not use them.

To make sense of the data our example notebook uses _Pandas_ library, with which we read the CSV file with the stats and transform it into a `DataFrame`, an internal Pandas representation. You can think about it as a SQL table on steroids. In fact, using this `DataFrame` we can partition data by test case (_image_ versus _text_), then group it by method, and finally process each method separately and display results, for each method, grouped by branching factor. Figure 2.20 shows these results for encoding a large text, comparing the two main functions in Huffman encoding that use a heap.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F20.png)

Figure 2.20 The distribution of running times for the two high-level functions in Huffman.py using a heap. (Top) Mean running time by branching factor. (Bottom) Boxplots for the distribution of running times for each branching factor. All charts are created using data about the running time per single call for the compression of an ensemble of text files.

If we look at those results, we can confirm a few points we mentioned in section 2.9:

- 2 is never the best choice for a branching factor.
    
- A branching factor of 4 or 5 seems a good compromise.
    
- There is a consistent difference (even -50% for `_frequency_table_to_heap`) between a binary heap and the d-ary heap with best branching factor.
    

There is, however, also something that might look surprising: `_frequency_table_ to_heap` seems to improve with larger branching factors, while `_heap_to_tree` has a minimum for branching factors around 9 and 10.

As explained in section 2.6, which shows the pseudocode implementation of heap methods (and as you can see from the code on GitHub), the former method only calls the`_push_down` helper method, while the latter uses both `top` (which in turn calls `_push_down`) and `insert` (relying on `_bubble_up` instead), so we would expect to see the opposite result. Anyway, it is true that even `_heap_to_tree` has a ratio of two calls to `top` per `insert`.

Let’s now delve into the internals of these high-level methods, and see the running time per call of heap’s internal method `_heapify` (figure 2.22), and of API’s methods `top` and `insert` and their helper methods `_push_down` and `_bubble_up` (figures 2.21 and 2.23).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F21.png)

Figure 2.21 The distribution of recorded running times per call for `insert` and `_bubble_up`

Before digging into the most interesting part, let’s quickly check out the `insert` method. Figure 2.21 shows there are no surprises here; the method tends to improve with larger branching factors, as does `_bubble_up`, its main helper.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F22.png)

Figure 2.22 The distribution of recorded running times per call for `heapify`.

The `_heapify` method, as expected, shows a trend similar to `_frequency_table_to_heap``,` because all this method does is create a heap from the frequency table. Still, is it a bit surprising that `_heapify`‘s running time doesn’t degrade with larger branching factors?

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F23.png)

Figure 2.23 The distribution of recorded running times per call for top and _push_down

Now to the juicy part. Let’s take a look at `top` in figure 2.23. If we look at the running time per call, the median and distribution clearly show a local minimum (around `D==9`), as we would have expected, considering that the method’s running time is `O(D*logD(n))`, confirming what we have previously discussed and summarized in table 2.7. Not surprisingly, the `_push_down` helper method has an identical distribution.

If we look at listings 2.7 and 2.4 and sections 2.6.2 and 2.6.4, it’s clear how `pushDown` is the heart of the `top` method, and in turn the largest chunk of work for `pushDown` is the method that at each heap’s level retrieves the child of current node. We called it `HighestPriorityChild`.`[32](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch02.htm#pgfId-1013514)`

And if then we look at the running time _per call_ for `_highest_priority_child` (figure 2.24 (A)), we have a confirmation that our profiling does make sense, because the running time per call increases with the branching factor. The larger `D` is, the longer the list of children for each node, which this method needs to traverse entirely in order to find which tree branch should be traversed next.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F24.png)

Figure 2.24 (A) The distribution of recorded running times per-call for `_highest_priority_child`. (B) The distribution of cumulative running times for `_highest_priority_child`.

You might then wonder why `_push_down` doesn’t have the same trend. Remember that while the running time for `_highest_priority_child` is `O(D)`, in particular with `D` comparisons, `_push_down` performs (at most) `logD(n)` swap sand `D*logD(n)` comparisons, because it calls `_highest_priority_child` at most `logD(n)` times.

The larger the branching factor `D`, the fewer calls are made to `_highest_priority_ child`. This becomes apparent if, instead of plotting the running time per-call for `_highest_priority_child`, we use the total cumulative time (the sum of all calls to each method), as shown in figure 2.24 (B). There we can see again how this composite function, `f(D) = D*logD(n)`, has a minimum at `D==9`.

### 2.10.3 The mystery with heapify

Summing up, `_heapify` keeps improving even at larger branching factors, although we can also say it practically plateaus after `D==13`, but this behavior is not caused by the methods `top` and `_push_down`, which do behave as expected.

There could be a few explanations for how the `_heapify` running time grows:

1. It’s possible that, checking larger branching factors, we discover that there is a minimum (we just haven’t found it yet).
    
2. The performance of this method heavily depends on the order of insertions: with sorted data it performs way better than with random data.
    
3. Implementation-specific details make the contribution of calls to `_push_down` less relevant over the total running time.
    

But . . . are we sure that this is in contrast with theory? You should know by now that I like rhetorical questions; this means it is time to get into some math.

And indeed, if we take a closer look at section 2.6.7, we can find out that the number of swaps performed by `_heapify` is limited by:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F_EQ3.png)

As you can imagine, analyzing this function is not straightforward. But, and here is the fun part, now you can certainly plot the function using your (possibly newly acquired) Jupyter Notebook skills. It turns out that, indeed, plotting this function of `D` for a few values of `n`, we get the charts in figure 2.25, showing that, despite the fact that the single calls to `_push_down` will be slower with a higher branching factor, the total number of swaps is expected to decrease.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F25.png)

Figure 2.25 The upper bound for the number of swaps needed by the heapify method, as a function of the branching factor D. Each line is a different value of n. Notice that the y axis uses a logarithmic scale.

So, mystery solved; `_heapify` does behave as expected. And good news, too: it gets faster as we increase the branching factor.

### 2.10.4 Choosing the best branching factor

For a mystery solved, there is still a big question standing: What is the best branching factor to speed up our Huffman encoding method? We digressed, delving into the details of the analysis of heaps and somehow left the most important question behind.

To answer this question, there is only one way: looking at figure 2.26, showing the running time for the main method of our Huffman encoding library.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch02_F26.png)

Figure 2.26 Distribution of per-call running time for the method huffman.create_encoding

It shows three interesting facts:

- The best choice seems to be `D==9.`
    
- Choosing any value larger than `7` will likely be as good.
    
- While the max gain for `_frequency_table_to_heap` is 50%, and for `_heapify` even up to 80%, we merely get to a 5% here.
    

Notice how the chart for `create_encoding` looks more similar to the method `_heap_ to_tree` than to `_frequency_table_to_heap`. Also, considering the third point, the explanation is that the operation on the heap only contributes to a fraction of the running time, while the most demanding methods needs to be searched elsewhere. (Hint: for `create_frequency_table`, the running time depends on the length of the input file, while for the other methods it only depends on the size of the alphabet.)

You can check out the full results, including the other examples, on the notebook on GitHub. Keep in mind that this analysis uses a limited input, and as such is likely to be biased. This is just a starting point, and I highly encourage you to try running a more thorough profiling using more runs on different inputs.

You can also pull the full profiling and visualization code and delve even deeper into the analysis.

To wrap up, I’d like to highlight a few takeaways:

- D-ary heaps are faster than binary ones.
    
- If you have parametric code, profiling is the way to tune your parameter. The best choice depends on your implementation as well as on input.
    
- Be sure to run your profiling on a representative sample of your domain. If you use a limited subset, you will likely optimize for an edge case, and your results will be biased.
    

As a general rule, make sure to optimize the right thing: only profile critical code, and perform high-level profiling first to spot the critical section of code whose optimization would give you the largest improvement, and verify that this improvement is worth the time you’ll spend on it.

## Summary

- The theory behind the functioning and analysis of d-way heaps makes heavy use of the basic structures and tools we describe in appendices A to F.
    
- There is a difference between concrete and abstract data structures, where the latter are better served to be used as black boxes, during the design of an application or algorithm using them.
    
- When we move from abstract data structures to their concrete implementations in a programming language, we need to pay attention to those nitty-gritty details that are specific to the language we chose, and make sure we don’t slow down our methods by choosing the wrong implementation.
    
- A heap is conceptually a tree, but it’s implemented using an array for the sake of efficiency.
    
- Changing the branching factor of a heap won’t affect asymptotic running time for its methods, but will still provide a constant factor improvement that matters when we move from pure theory to applications with a massive amount of data to manage.
    
- For several advanced algorithms we make use of priority queues to improve their performance: BFS, Dijkstra, and Huffman codes are just some examples.
    
- When high performance is paramount, and the data structures we use allow parameters (such as _branching factor_ for heaps), the only way to find the best parameters for our implementation is profiling our code.
    

---

1. Application Programming Interface (API).

2. Often bug-tracking suites associate lower numbers with higher priority. To keep things simple in our discussion, we will instead assume higher numbers mean higher priority.

3. With an array implementation, we could find the right position for our item in logarithmic time, using binary search. But then we would need to move all the elements on the right of the insertion point to make room for it, and this requires linear time on average.

4. Besides performance, there are other aspects we might need to check, depending on our context. For instance, in a distributed environment, we must make sure our implementation is thread-safe, or we will incur race conditions, the nastiest possible bugs that could afflict our applications.

5. You can find an introduction to algorithm analysis and big-O notation in appendix B.

6. Whether a problem is really tractable or not for a certain size of the input also depends on the kind of operations performed and on the time they take. But even if each operation takes as little as 1 nanosecond, if the input has 1 million elements, a quadratic algorithm will take more than 16 minutes, while a linearithmic algorithm would require less than 10 milliseconds.

7. A leaf is a tree node that doesn’t have any children. An internal node is a node that has at least one child or, equivalently, a node that is not a leaf.

8. We use explicit parenthesis for the following expressions. In the rest of the book, we will generally omit redundant parentheses, so that, for instance, we will write `2*i + 1.`

9. (a1, b1, c1,...) < (a2, b2, c2,...) if and only if a1 < a2 or (a1 == a2 and (b1, c1,...) < (b2, c2,...)).

10. The branching factor of a tree is the maximum number of children a node can have. For example, a binary heap, which in turn is a binary tree, has a branching factor of 2. See appendix C for more details.

11. As we explain in appendix C, this doesn’t change the asymptotic analysis, because it can be proven that inserting `n` elements in a dynamic array requires at most `2n` assignments, so each insertion requires an amortized constant time.

12. For instance, in Java’s `PriorityQueue`, a class provided in standard library, you need to remove an element and then insert the new one to perform the same operation. This is far from ideal, especially because the removal of a random element is implemented inefficiently, requiring linear time.

13. Using Fibonacci heaps, it could be implemented even in amortized constant time, at least in theory.

14. See [https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction#d-ary-heap](https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction#d-ary-heap).

15. Usually, though, the greatest concern is to avoid choosing the wrong data structure.

16. `n>>k` is normally interpreted as “n is much larger than k.”

17. `O(log(k))` to be precise, but since `k` in this context is a constant (much smaller than, and not depending on `n`), then `O(log(k))==O(1)`.

18. Additional challenges can occur if the text is so large that either it can’t be contained in memory, or a map-reduce approach is advisable: we encapsulate all that complexity in the `ComputeFrequencies` method.

19. Instead of the actual frequency it might be easier, and equivalent for the algorithm, just counting the number of occurrences of each character.

20. It’s simply going to be a 1:1 mapping over the characters in the text, with extra attention needed to efficiently construct the bits encoding in output.

21. This section includes advanced concepts.

22. The Fibonacci heap is an advanced version of priority queue that is implemented with a set of heaps. For a Fibonacci heap, find-minimum, insert, and update priority operations take constant amortized time, `O(1)`, while deleting an element (including the minimum) requires `O(log n)` amortized time, where `n` is the size of the heap. So, in theory, Fibonacci heaps are faster than any other heap, although in practice, being overly complicated, their implementations end up being slower than simple binary heaps.

23. See [http://comjnl.oxfordjournals.org/content/34/5/428](http://comjnl.oxfordjournals.org/content/34/5/428).

24. `O(n*log(n))`, for an input of size n.

25. Where D is the branching factor for the d-ary heap under analysis.

26. It is, obviously, possible to find a formula for `f(D)`’s minima using calculus, and in particular computing the first and second order derivatives of the function.

27. See [https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction#huffman-compression](https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction#huffman-compression).

28. [https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction/blob/master/Python/mlarocca/tests/huffman_profile.py](https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction/blob/master/Python/mlarocca/tests/huffman_profile.py).

29. We used copyright-free novels downloaded from Project Gutenberg, [http://www.gutenberg.org](http://www.gutenberg.org/) (worth checking out!).

30. Disclaimer: There are many possible ways to do this, and possibly better ways too. This is just one of the possible ways. [https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction/blob/master/Python/mlarocca/notebooks/Huffman_profiling.ipynb](https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction/blob/master/Python/mlarocca/notebooks/Huffman_profiling.ipynb).

31. [https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.pyplot.boxplot.html](https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.pyplot.boxplot.html).

32. In our Python implementation, the names become respectively _push_down and `_highest_priority_ child`, to follow Python naming conventions.