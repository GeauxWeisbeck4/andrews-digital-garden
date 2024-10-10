---
id: 01J9QFPKYBE43YM2HBPXGCH0YW
modified: 2024-10-08T21:53:46-04:00
---
# 5 Disjoint sets: Sub-linear time processing

This chapter covers

- Solving the problem of keeping a set partitioned into disjoint sets and merging partitions dynamically
- Describing an API for a data structure for disjoint sets
- Providing a simple linear-time solution for all methods
- Improving the running time by using the right underlying data structure
- Adding easy-to-implement heuristics to get quasi-constant running time
- Recognizing use cases where the best solution is needed for performance

In this chapter we are going to introduce a problem that seems quite trivial—so trivial that many developers wouldn’t even consider it worth a performance analysis, so they’d just implement the obvious solution to it. Nevertheless, if the expression “wolf in sheep’s clothing” was applied to data structures, this would be the best heading for this chapter.

We will use a disjoint set every time that we would like to account for the partitioning of an initial set of objects into disjoint groups (that is, subsets without any element in common between them). For instance, we might start with a list of wines, which would be the initial set, and partition those wines depending on their flavor, creating a disjoint set where wines with a similar flavor are grouped together, and groups have no intersections with each other. Or we could group foods based on their nature and properties, grouping vegetables, fruit, processed food, and so on. This would be a trivial example of a disjoint set, shown in figure 5.1.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F1.png)

Figure 5.1 An example of a disjoint set. The whole set, the so-called Universe, is the set of “Food.” Three partitions are shown: Fruit, Vegetables, and Sweets. The key point is that these subsets have no intersections with each other.

We will tackle this problem starting with its definition, then cover the most basic (and naïve) algorithms, to give readers an understanding of what an actual solution looks like. Once that’s clear, we will be ready to delve into making our solutions more efficient, and show how to use them as part of more complex algorithms. By the end of the chapter you will be able to code the best possible solution for disjoint sets and employ it to improve the performance of higher-level applications.

## 5.1 The distinct subsets problem

For instance, imagine this: you are running a new, recently created, e-commerce website, and for your launch, you’d like to provide non-personalized recommendations to your users. If it helps you feel like this is a real situation, you might imagine owning a time-machine and being back in 1999. Or more realistically, you can think about opening a geographically localized website with stronger ties to your country’s retailers; or maybe you open a specialized retail website focusing on niche products. Either way, this can be a very interesting exercise.

So, non-personalized recommendations, we were saying. You might ask what that’s about. Allow me to take a short detour to explain this: personalized recommendations are targeted on individual customers, based on the data we have about them (past purchases or metadata that shows similarity with other users). But sometimes we don’t have this data at all: for instance, when we start a new website, or even when we get a new customer about which we know nothing at all. That’s why many websites, such as Twitter, Pinterest, Netflix, and MovieLens, ask you questions on sign-up to understand your tastes and be able to provide some rough personalized recommendations based on users with similar profiles.

Non-personalized recommendations, on the other hand, are not targeted to you, the customer, and are the same for all customers. They might be hardcoded associations, if you have no data at all, or based on purchases made by all other customers, for example.

And that’s exactly what we are going to do: whenever customers add something to their cart, we would like to provide them with a recommendation about something else they might want to buy along with it. Our goal is finding products that are frequently bought together; sometimes we are going to find reasonable associations, such as milk and bread bought together. Other times, the result might be more surprising: you probably have already heard about the research performed at Walmart linking the purchase of diapers and beer, since it’s one of the most quoted data science anecdotes.

Figure 5.2 illustrates what we would like to achieve. Initially, since we have no data at all, we consider every product as a group of its own, or if you prefer, no two products will have an association.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F2.png)

Figure 5.2 An example of an application of a disjoint set. (A) In this scenario, an e-commerce website is trying to understand what products sell well together in order to provide better recommendations to its customers. (B) Initially, each item on sale is in a different category. (This would also work if we started from already predefined categories, such as SSD disks or blenders, and grouped categories of objects selling together. But for the sake of simplicity, bear with me and imagine there is only one product of each category on sale.) (C) Items frequently bought together, such as laptops and external disks, or tennis rackets and tennis balls, are grouped together. (D) After a while, things tend to stabilize and steady groups are formed. Now, the next time a customer adds a football to her cart, we can suggest a pair of skis as a follow-up purchase.

When customers frequently buy two products together, we then establish a link between them, considering both to be part of the same group. To keep things simple, imagine that the rule set by the data science team is that if item `X` and item `Y` are bought together more than a fixed threshold during the last hour, then their two categories are merged.

For instance, it might be that if, during the last hour, phones and tablets are bought together more than 500 times (or for more than 1% of total purchases), then they should be in the same group, so we merge their groups.

Then if a customer buys a product `X`, we can suggest to them a random item from the same group.

This process described is pretty common in data science. Some of you might have recognized that it’s nothing less than hierarchical clustering. If that doesn’t ring a bell, do not worry; we will expand on clustering in section 5.7.3.

This is, obviously, an extreme simplification. In a real non-personalized recommender system, we would track associations between products, measuring the strength of the link as the confidence that when `X` is bought, `Y` is also. For that, we may compute the number of purchases where both appear, divided by the total number of purchases where at least `Y` appears. That would give us better insight about what goes with what, we could define a threshold on the confidence for merging groups, and instead of picking a random item in the same group, show the top five strongest associations.

Nevertheless, clustering items in groups will probably be a smart move, because it will help performance, allowing us to run some algorithms on each group of items separately rather than on the whole catalog.

If you are interested in knowing more about non-personalized (and personalized) recommender systems, we suggest you look at Practical Recommender Systems (by Kim Falk, Manning Publications, 2019), a fine and thorough guide to the topic.

Back to our example: the gist is that we would need to start from this huge set of items and partition them into disjoint groups. And of course, new items are added to the catalog all the time, and relations are dynamic, so we would need to be able to update both the list of items and the groups.

## 5.2 Reasoning on solutions

In this and the following sections, we will mainly use the term _partition_ to refer to disjoint groups. However, _group_ and _set_ can also appear as synonyms.

We will also restrict to the aggregative case; in other words, two partitions can be merged in a single, bigger set; the opposite, however, is not allowed. That is, a partition can’t be split into two subsets.

Now imagine that there was a design discussion between your data science team and your support engineering team, and one engineer fiercely stood up and exclaimed, “Well, that’s easy! You keep an array (a dynamic array or a vector) for each subset.”

You don’t want to be that person, believe me, and one of the goals of this book is making sure you don’t find yourself in that situation. Because the next thing to happen, hopefully, is that somebody else points out how, by using arrays, understanding whether two elements are in the same set could potentially require going over all the elements of all the subsets. Likewise, just understanding which subset an element belongs to could require the same number of element checks; that is, search could be linear in the total number of elements.

This would be a real performance concern, and it seems obvious we should be able to do better.

The next idea in this brainstorming could involve adding a map from elements to subsets, together with the list of subsets explained previously. This is a slightly better improvement for some operations; although, as we will see, it still forces operations such as merging two sets to potentially require `O(n)` assignments.

Performance, however, is not the main concern with that design. Using two independent data structures is a terrible idea, because you will have to manually sync them every time you face this problem in your application. This is very error-prone.

An already better solution is to provide a wrapper class that internally uses those two structures: it gives you encapsulation and isolation and as a result, you are able to write only once the code that syncs both structures on, say, add or merge (and so you gain reusability). Even more important, you are able to unit test your class in isolation and hence have a reasonable guarantee that it’s going to do its job without experiencing bugs every time you use it in your application (that is, provided you do write good and thorough unit tests, acing the edge cases and challenging the behavior of your class in all possible contexts).

So, let’s assume we agree on the need to write a class that takes care of the whole problem, keeping track of which (disjoint) subset an element belongs to, and encapsulating all the logic in it. We are going to delay the discussion on implementation for now: before focusing on the details, let’s discuss its public API and behavior.

Depending on the size of the catalog, we could even fit such a data structure in memory, but let’s instead assume that we set up a REST service (illustrated in figure 5.3) based on a persisted Memcached-like storage,[1](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020250) something like Redis. Durability of the data is important in this case, because in our example, the monitoring activity over the items will last for years, and we don’t want to recompute the whole disjoint set structure every time there is a change or a new product is added.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F3.png)

Figure 5.3 A possible design for an application using disjoint set. The disjoint set client can be anything ranging from a library to a REST client. The purpose of the (thin) client here is to be an interface between the in-memory storage (the Memcached node in the picture) and the server, which has persistent storage. The server can be a web server, but it could also just be another native application storing data on a disk. The disjoint set client will run on the same intranet, possibly even on the same machine, as the e-commerce server. It will have a cron job to keep the persistent storage in sync with the in-memory version (this could happen every few seconds or asynchronously after each operation). Moreover, it will respond to calls from the e-commerce site, querying the in-memory storage or, when needed, calling the web server (and, in case not all the data will fit in memory, it will also take care of swaps).

Alternatively, if the size of the universe[2](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020285) is small enough to fit in memory, it could be possible to imagine a synchronization mechanism that periodically serializes our in-memory data structure into a persistent database.

## 5.3 Describing the data structure API: Disjoint set

In our design, our data structure needs to offer only a few, though crucial, operations.

|Abstract data structure: `DisjointSet`|   |
|---|---|
|API|class DisjointSet {<br>  init(U);<br>  findPartition(x);<br>  merge(x, y);<br>  areDisjoint(x,y);<br>}|
|Contract with client|A disjoint set keeps track of the mutual relations between elements in a universe U.<br><br>The relation is not explicitly defined by the data structure; it is left to the client to define it.<br><br>However, it is assumed that such a relation ® has the reflexive, symmetric and transitive properties; this means that given x, y, z elements of U<br><br>- x® x<br>    <br>- if x® y, then y® x<br>    <br>- if x® y and y® z then x® z<br>    <br><br>The guarantees provided by the class are<br><br>- It’s possible to add a relation between any two elements.<br>    <br>- If two elements at any point are merged (that is, a relation between them is added), they will be part of the same disjoint set.<br>    <br>- If there is a chain of elements x1, x2, ..., x_n_ such that x1 has been merged to x2, x2 has been merged with x3 and so on, then x1 and x_n_ will be in the same partition.<br>    <br>- If two elements are not in the same partition, then there is no other element belonging to both elements’ disjoint sets.|

First, we’d like, obviously, to initialize our instance on construction. Without any loss of generality, we can restrict to the case where the Universe `U`, that is, the set of all possible elements, is known in advance and static. We also assume that initially every element is in its own partition. Workarounds to support violations to these assumptions are easily achievable by making wise use of dynamic arrays and the class’s very own methods.

Finally, throughout this chapter we assume the elements of our Universe `U` are the integers between `0` and `n-1`. This is not a real restriction, because we can easily associate an index to each of the actual elements of `U`.

Initialization, therefore, just takes care of allocation of the basic fields needed by the class and assigns each element into its own partition.

The `findPartition` method, when called on an element `x` of `U`, will return the partition to which `x` belongs. This output might not be meaningful outside of the instance of our data structure: think of this method mostly as a _protected method_ [3](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020302) for the class, or even consider restricting its visibility to _private_.

The two main operations that we’d like to perform are

- Given two elements `x` and `y`, both belonging to `U`, we’d like to check if they belong to different partitions (`areDisjoint`).
    
- Given two elements `x` and `y`, we’d like to merge their partitions into a single one (`merge`).
    

## 5.4 Naïve solution[4](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020319)

The most immediate solution for our problem is to represent each partition with a list (or array), as illustrated in figure 5.4. For each element, we need to keep track of the pointer to the head of the list.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F4.png)

Figure 5.4 Representing a disjoint set with lists. Each array element stores a pointer to the head of a linked list. Each linked list, in turn, represents a set. Here sets are numbered arbitrarily, as the index doesn’t really provide any information on the set (nor can it be retrieved).

To find out if two elements are in the same partition, we need to retrieve the list for one element, and check if it is the same list as for the other.[5](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020334)

To merge two partitions, `P1` and `P2`, modeled with two lists, `L1` and `L2`, we need to update the `next` pointer[6](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020365) of the last element in `L1` so that it points to the head of `L2` (or vice versa, with the last element of `L2` pointing to the head of `L1`). This operation, which is shown in figure 5.5, can be done in constant time by keeping an extra (constant space) pointer to the tail of each list. Unfortunately, though, we aren’t done just yet: for every element in `L2`, we also need to update its list pointer in our map to point to the head of the new merged list.

This operation requires linear time in the worst case, because we might have to update up to `n-1` elements (where `n` is the total number of elements in the Uni-verse `U`).

There is one way to slightly improve the expected number of assignments we have to perform: by always appending the shortest of the two lists, we will make sure that we won’t have to update more than `n/2` elements’ pointers. Unfortunately, this does not improve the asymptotic execution time.[7](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020383)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F5.png)

Figure 5.5 Merging two partitions. On the left: one of the two lists is appended to the other by adding a new edge tail-to-head and removing links from the array to the second list. On the right is shown the result after the merge, with all head-pointers for array elements belonging to the appended list (elements 1 and 5 in the example) updated.

Let’s delve into code to better explain how this works.

### 5.4.1 Implementing naïve solution

Let’s start with the pseudo-code for the constructor, and the class definition (see listing 5.1). All methods in the next sections will be assumed to be class methods for `DisjointSet`.

Listing 5.1 Naïve solution, `constructor`

**class** DisjointSet
  **#type** HashMap[Element, List[Element]]
  partitionsMap
 
**function** DisjointSet(initialSet=[])                        ❶
    **this**.partitionsMap ← **new** HashMap()                     ❷
    **for** elem in initialSet **do**                              ❸
      **throw-if** (elem == **null or** partitionsMap.has(elem)))  ❹
      partitionsMap[elem] ← **new** Set(elem)                  ❺

❶ The constructor takes a list of elements as argument, but by default initializes the disjoint set with an empty set.

❷ Creates a new map from elements to sets

❸ Goes over each element in the argument list

❹ Throws an exception if the element is `null` or a duplicate

❺ Adds a mapping between current element and a new singleton set containing current element only

Initialization is simple: we will check that the list passed as argument contains no duplicates and initialize the disjoint set with its elements.

In a real implementation, you should worry about how elements are compared. Depending on the programming language, it can use referential equality, an equality operator, or a method defined on the elements’ class. The following code is only meant to illustrate how a basic solution works, so we won’t worry about the details.

First, we initialize the associative array that is going to index the elements and map them to the partition they belong to (line #2).

Next, we just go through `initialSet`’s elements, one by one, check they are defined and that there is no duplicate, and then initialize their partition to the singleton containing the element itself (initially each element is disjoint from every other element).

Now that we have taken care of initializing our disjoint set, we can provide a couple of useful methods. For example, we can add a `size` public property, simply defined as the number of entries stored in the local partitions map.

You can find examples of such methods implemented in our repo. Here, instead, we will focus on the main API methods, starting with the `add` method, illustrated in figure 5.6, whose pseudo-code is shown in listing 5.2.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F6.png)

Figure 5.6 Adding a new element to the container. Provided the new element is not a duplicate of any element currently in our container, we can add it by creating a new singleton partition, which will only contain the newly added element.

Listing 5.2 Naïve solution, `add`

**function** add(elem)                       ❶
  **throw-if** elem == **null**                  ❷
  **if** partitionsMap.has(elem) **then**        ❸
    **return false**
  partitionsMap[elem] ← **new** Set(elem)    ❹
  **return true**

❶ Takes an element and returns `true` iff[8](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020408) the element was added successfully, `false` otherwise

❷ Checks that the element is valid

❸ If the element is already in the data structure, returns `false` without updating anything

❹ Otherwise just adds a mapping between the element and a newly created singleton[9](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020423) set and returns `true`

This method is used to allow the Universe to grow, with new (unique) elements that can be added at any time. Every time we add a new element, we just add a brand-new partition containing that element alone. But, of course, we need to check that the argument passed to `add` is well-defined, so that it’s not `null` and not a duplicate of another element already in our Universe.

Now we get to the really interesting stuff: first and foremost, the `findPartition` method, shown in listing 5.3.

Listing 5.3 Naïve solution, `findPartition`

**function** findPartition(elem)                               ❶
  **throw-if** (elem == **null or not** partitionsMap.has(elem))   ❷
  **return** partitionsMap[elem]                               ❸

❶ Takes an element and returns a `Set`, the partition (aka disjoint set) to which the element belongs

❷ Checks that the element is valid

❸ Returns the `Set` containing the argument

In this basic implementation, the method is particularly trivial: after the usual validation (including checking that the element is actually stored in the disjoint set), we just need to return the partition containing `elem`.

As we mentioned, this implementation of the `findPartition` method only requires constant time (assuming that the hash for `elem` can be computed in constant time).

Another easy-to-implement method, shown in listing 5.4, is the one checking if two elements belong to the same partition.

Listing 5.4 Naïve solution, `areDisjoint`

**function** areDisjoint(elem1, elem2)      ❶
  p1 ← **this**.findPartition(elem1)        ❷
  p2 ← **this**.findPartition(elem2)        ❸
  **return** p1 != p2                       ❹

❶ Takes two elements and returns `true` iff the elements are valid but don’t belong to the same partition, `false` iff the elements are valid but do belong to the same partition. Notice that if either element is `null` or hasn’t been added to this container, then this method will throw an error (because in turn `findPartition` will throw an error).

❷ Retrieves the disjoint set to which `elem1` belongs. If the argument is invalid or not found, this call will throw an error.

❸ Repeats the same operation for `elem2`

❹ Compares the two sets, and checks if they are the same, and hence if the elements belong to the same partition

We just need to reuse `findPartition`, call it for both elements, and check whether both calls return the same partition. Note that by reusing `findPartition`, we can make sure that the implementation of `areDisjoint` won’t need to change, no matter how our elements are stored or `findPartition` is implemented (as long as its interface remains the same, and partitions can be compared with the inequality operator).

Moreover, we decided to implement a check for elements belonging to different partitions rather than for elements belonging to the same one: this is because of how disjoint sets are normally used. We are normally interested in checking whether two elements don’t belong to the same partition, and if that’s the case, we merge the two partitions. But depending on the way you are going to use this container, it is possible that the other way around is more convenient, and there is nothing preventing you from defining a `samePartition` method instead.

All the methods we have seen so far run in constant time with respect to the size of the container. Now, it’s time to implement the method merging two partitions, shown in listing 5.5 (and illustrated in figure 5.5). As we have seen, the `merge` method requires `O(n)` assignments in the worst case.

Listing 5.5 Naïve solution, `merge`

**function** merge(elem1, elem2)           ❶
  p1 ← **this**.findPartition(elem1)       ❷
  p2 ← **this**.findPartition(elem2)       ❷
  **if** p1 == p2 **then**                     ❸
    **return false**
  **for** elem in p1 **do**                    ❹
    p2.add(elem)                       ❺
    **this**.partitions[elem] ← p2         ❻
  **return true**

❶ Takes two elements, merges their partitions, and returns `true` iff the two elements were in two different partitions that now are merged, or `false` if they were already in the same partition

❷ Retrieves the partitions to which `elem1` and `elem2` belong. If the argument is invalid or not found, these calls will throw.

❸ Compares `p1` and `p2`, and if they are the same, there is nothing left to do. The elements are already in the same partition, and so no merge happens: `false` is returned.

❹ Loops over the elements in the first partition. For each element in `p1` do:

❺ . . . add the element to `p2` . . .

❻ . . . then update the mapping for that element, which now belongs to `p2`.

This method is more complex than the previous ones. And yet, by reusing `findPartition`, it still looks quite simple.

We first check if elements belong to the same partition by calling `findPartition` on both and checking the result. Those calls also take care of validating the input.

Once we’ve established that we actually need to perform an action, we proceed and merge the two sets, correcting the pointers in the partitions map when needed. If the partitions were implemented with linked lists instead of `Set`, we could have just appended the head of a list to the tail of the other. Sets, instead, force us to actually add elements one by one. An extra linear number of assignments is needed (worst case), but this doesn’t change the order of the function’s runtime; we still need to update the references for elements of one of the two lists (that is, sets) anyway.

Here, we show the simplest code, always pouring the first partition’s elements into the second one. On our repo on GitHub[10](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020438) you can find a slightly better version that checks which set is smaller and adds its elements to the larger sets; however, this is just a constant-time improvement on the simplest version and the running time remains linear in the minimum of the sets’ sizes.

## 5.5 Using a tree-like structure[11](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020455)

To recap what we attained with our basic implementation, we managed to write a constant-time `findPartition` method and a worst-case linear-time `merge` method.

Now, can we do any better than linear, not just for `findPartition`, but for all the operations on a disjoint set? Well, turns out that yes, we can!

### 5.5.1 From list to trees

The idea is simple: instead of using lists to represent each partition, we will use trees, as shown in figure 5.7. Each partition is then uniquely identified by the root of the tree associated with the partition. The advantage of trees over lists is that if the tree is balanced, any operation on the tree is logarithmic (instead of linear, as for a list).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F7.png)

Figure 5.7 Representing a disjoint set with trees. Trees are named after their root, because we use the tree root as a unique identifier for the partition (we can do so under the assumption that elements are unique). Each element in the array points to a tree node: in the naïve implementation there is a 1:1 mapping between elements and tree nodes. This means that to get to the root of the tree, we might need to climb up the whole tree (and, on average, half of the height of the tree).

When we merge two partitions, we will set one tree root as the child of the root of the other tree; see figure 5.8 to see an example.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F8.png)

Figure 5.8 Merging two sets when using the tree representation. It only requires creating one new link (plus some tree traversing). In the figure on the left, we add one edge from the root tree of Tree 1 to the root of Tree 0, to merge them. On the right, we show how this changes the data structure: now we only have two trees, but the height of tree 0 is now larger than before the merge.

This is a huge improvement over the naïve solution, because we won’t have to update the partition map for any of the other elements in the partitions merged. Instead, each node in the tree will maintain a link to its parent (we don’t need to save links to children, because they are of no use in this case).

The roots of the tree, as mentioned, uniquely identify each partition. So, when we need to find out which partition an element belongs to, we just retrieve the tree node it’s pointing to and walk up to the root of its tree. In method `areDisjoint`, if we do the same for both elements and then compare the roots found, we can easily see if two elements belong to the same partition (if and only if the two roots are equal).

So, merging two partitions now requires a constant number of changes, plus the number of look-ups needed to find the two roots. Finding the set to which an element belongs (or seeing if two elements belong to the same partition) requires logarithmic time on average (remember when we introduced trees?[12](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020471)) but linear time in the worst case. That’s because when merging partitions, we might get unlucky with the choice of which tree’s root we set as child of the other. By randomly choosing each time which root is going to be used as a child of the other, we can make this worst-case scenario unlikely . . . but it would still be possible (although extremely unlucky) to find ourselves in an edge situation such as the one depicted in figure 5.9. This means that our worst-case scenario for `merge` still requires `O(n)` lookups and, what’s worst, now even `findPartition` has linear running time.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F9.png)

Figure 5.9 Worst case scenario for naive tree implementation: the height of the final tree is equal to `n`, the total number of elements, because the tree degenerated into a list.

Before seeing how we can improve this further, let’s check out some code for our improved version.

### 5.5.2 Implementing the tree version

Let’s see in detail how this improved implementation works. Most code remains unchanged from the previous section, so we won’t show it here. In the methods that have changes with respect to the naïve version, we will underline those changes to help readers quickly compare the two versions.

Instead of mapping to an actual set, elements in the partition map point to each element’s parent in the tree. That’s why, as you can see in the book’s repo, we can rename our `partitionsMap` to `parentsMap` to make its purpose explicit.

At initialization, we conveniently set each element as its own parent. Trust me on that one; we’ll see why later.

The same change applies to the `add` method, which otherwise stays unchanged.

The `findPartition` method (described in listing 5.6) needs quite a bit of tuning to work properly. Two notes on its implementation:

- With respect to the basic implementation, in this case we won’t return a list anymore, but rather the element at the root of the partition’s tree.
    
- The return value of `findPartition` might not immediately make sense to an external caller, and in fact this method will mostly be used internally when called by merge and `areDisjoint` methods.
    

Listing 5.6 Tree-based solution, `findPartition`

**class** DisjointSet
  **#type** HashMap[Element, Tree[Element]]
  parentsMap
 
**function** findPartition(elem)                             ❶
  **throw-if** (elem == **null or not** parentsMap.has(elem)))   ❷
  parent ← **this**.parentsMap[elem]                         ❸
  **if** parent != elem **then**                                 ❹
    parent ← **this**.findPartition(parent)                  ❺
  **return** parent                                          ❻

❶ Takes an element and returns another element, the one element at the root of the tree for the partition to which `elem` belongs

❷ Checks that the element is valid

❸ Retrieves the parent of the element

❹ If the current element’s parent is `elem` itself, then we’ve already gotten to the root of the tree; otherwise . . .

❺ . . . we need to climb up recursively to the root, by looking for `parent`’s partition.

❻ At this point, `parent` stores the root of the tree for the partition containing `elem`, so we can return it.

After getting the element’s parent, we check to see if it’s the element itself. If an element is its own parent, then we know this means we’ve reached the root of the partition’s tree, because of the way we initialize this field, and because, in `merge`, we never change a root’s parent.

Otherwise, if the current element does have a parent, we walk up the tree toward its root and perform a recursive call to `findPartition`, returning its result (line #5).

This new implementation of `findPartition`, as we already mentioned, is not running in constant time anymore. We will have as many recursive calls as the height of the partition’s tree. Since we can’t make any assumption about the trees so far, this means that we possibly have a number of calls proportional to the number of elements in the Universe `U,` although this is the worst case and, on average, we can expect far better performance.

It might seem so far that we have only made our data structure’s performance worse. We need to define our new implementation of the `merge` operation, shown in listing 5.7, to see the advantage of using trees.

Listing 5.7 Tree-based solution, `merge`

**function** merge(elem1, elem2)               ❶
  p1 ← **this**.findPartition(elem1)           ❷
  p2 ← **this**.findPartition(elem2)           ❷
  **if** p1 == p2 **then**                         ❸
    **return false**
  **this**.parentsMap[p2] ← p1                 ❹
  **return true**

❶ Takes two elements, merges their partitions, and returns `true` iff the two elements were in two different partitions that now are merged, `false` if they were already in the same partition

❷ Retrieves the partitions to which `elem1` and `elem2` belong to. If the argument is invalid or not found, this call will throw.

❸ Compares `p1` and `p2`, and if they are the same there is nothing left to do. The elements are already in the same partition, and so no merge happens: `false` is returned.

❹ Sets the parent of `p2` to be equal to `p1`, so that now both `p1` and `p2` have the same parent, but also all elements in `p2` will ultimately share `p1` as the root of their tree

By comparing the two implementations you can immediately see that this is simpler, although only the last few lines changed. The good news is that we no longer need to iterate through a list of elements! To merge two partitions, we simply need to get to both trees’ roots, and then set one root as the parent of the other. We still need to find those tree roots, though.

The new line we added only requires constant time, so the method runtime is dominated by the two calls to `findPartition`. As we have seen, they require time proportional to the height of the tree they are called on, and in the worst case this can still be linear. However, in the average case, and especially in the early stages after initialization, we know the height of the trees will be much smaller.

So, in summary, with this implementation we have a disjoint set for which all the operations still require linear time in the worst case, but on average will only need logarithmic time—even for those operations that were constant-time in the naïve implementation. Admittedly, that doesn’t sound like a great result if we focus on worst cases. Nevertheless, if you look at it from a different perspective, we’ve already managed to have a more balanced set of operations on our disjoint set, which is especially nice in those contexts where `merge` is a common operation (while in read-intensive applications, where `merge` is only executed rarely, the naïve implementation could overall be preferable).

Just read through the next section before dismissing the tree solution; it will be worth it.

## 5.6 Heuristics to improve the running time[13](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020486)

The next step in our quest for optimal performance is to make sure `findPartition` is logarithmic even in the worst-case scenario. Luckily, this is pretty easy! We discussed balanced trees in appendix C; feel free to check it out if you feel you could use a refresher.

Long story short, here we can easily keep track of the _rank_ (aka size) of each tree, using linear extra space and performing constant-time extra operations in method `merge`, where we will update rank only for trees’ roots.

When we merge two trees, we will make sure to set as child the tree with the smallest number of nodes, as shown in figure 5.10.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F10.png)

Figure 5.10 Merging two set trees: examples of balanced vs unbalanced merges. Arrows from the array are omitted for convenience, because each array element points to the corresponding tree element (so all light red elements point to the red [light] tree, and so on).

It can be proven by induction that this tree will also be the one with the smallest height: this means the new tree will either have the same height as the old one, or just have its height increased by 1. It is also provable that the height of a tree can’t be increased more than a logarithmic number of times.

As a logarithm grows really slowly (for instance `ln(1000) ~= 10, ln(1000000) ~= 20)`, this is, in practice, already a good result, sufficient for most applications.

However, if you are writing some really critical core code, such as a kernel or firmware code, you might want to do even better.

Why? Well, because you can. And sometimes also because you need to. If you shave 0.001ms over an operation you will repeat a billion times, you’ve already saved 16 minutes of computation.

Note Most of the time, in our job as developers, performance isn’t the only metric to consider regarding this kind of improvement. First, it depends on whether you are saving those 16 minutes over a computation taking an hour or a day (needless to say, in the latter situation the gain would be irrelevant). But it also depends at what price you get the savings. If it makes your code terribly more fragile, more complicated, and harder to maintain, or just requires weeks of development time, you will have to weigh the pros and cons before going down this path. Luckily for disjoint sets, this is not the case, and path compression is easy to implement, while it gives a big gain.

Let’s see how we could have this further improvement before delving into code.

### 5.6.1 Path compression

As Hinted in the previous section, we can do even better than just having balanced trees and logarithmic-time methods.

To improve our results further, we can use a heuristic called _path compression_. The idea, shown in figure 5.11, is even simpler: for each node in the trees, instead of storing a link to its parent, we can store one to the root of the tree. After all, we don’t really need to keep track of the history of the merges we performed; we just need to know at the current time what the root is for an element’s partition—and find that out as quickly as we can.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F11.png)

Figure 5.11 Disjoint set represented using a tree with path compression. Internal representation is shown next to the elements’ array. In tree representation, dashed arrows are parent links, while solid arrows are pointers to the set’s root. The structure initially holds two sets, colored in light red and dark blue and whose roots are respectively 0 and 1.

Now, if you were to update all the root pointers as part of merge, it wouldn’t be a logarithmic method anymore; we would need linear time to update each node in the tree.

But let’s see—what happens if we don’t update immediately the parent pointers in the nodes of the tree set as child? Simply put, next time we run `findPartition` on one of the elements in that tree—call it `x`—we need to walk the tree from `x` up to its old root `xR`, and then from `xR` to the new root `R`.

Keep in mind that the pointers for the elements in the old tree could have been in sync before the merge (and then we would just need two hops to get to the new root; see figure 5.12), or they might have never been updated.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F12.png)

Figure 5.12 Call to `find` on the same disjoint set of figure 5.11. Notice that the dark blue tree is out of sync. If we call `find` on the element `6`, the algorithm starts slowly crawling up the blue tree, until it finds its root (third diagram). Then (bottom diagrams), the algorithm backtracks, updating the root for the intermediate elements `9` and `6`.

Because we will have to walk up the tree anyway, we can then retrace our steps from the top, `R`, down to `x` and update the root pointers for all those elements. This won’t influence our asymptotic performance for `findPartition`, since by retracing the same path we just double the number of steps (and constant factors are irrelevant in asymptotic analysis; see appendix B).

But as a consequence of these extra steps that we take, next time we call `findPartition` on any elements in the path from `x` to `root(x)`, we know for sure that those pointers will be up to date and we will just need one single step to find their root.

At this point, we would like to understand how many times we will need to update the root pointers, on average, for a single operation or, in amortized analysis, over a certain number `k` of operations. This is where the analysis of the algorithm gets a bit complicated.

We won’t go into its details. Just know that it is proven that the running amortized time for `m` calls to `findPartition` and `merge` on a set of `n` elements will require `O(m * Ack(n))` array accesses.

Here `Ack(n)` is an approximation of the _inverse Ackermann_ function, a function growing so slowly that it can be considered a constant (its value will be lower than 5 for any integer that can be stored on a computer).

So, we managed to obtain an amortized constant bound for all the operations on this data structure! If you are not impressed by this result . . . you should be!

It is not yet known if this is the lowest bound for the Union-Find data structure. It has been proven,[14](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020499) though, that `O(m * InvAck(m, n))` is a strict lower bound, where `InvAck(m, n)` is the true inverse Ackermann function.

I know, this is a lot to take in. But do not despair; it turns out that we only need a few small changes to implement the path compression heuristic.

### 5.6.2 Implementing balancing and path compression

We will now discuss the final implementation of our disjoint set structure, including both the “tree balancing by rank” and “path compression” heuristics.

For each element, we’ll have to store some information about its subtree. Therefore, we’ll use a helper (private) class `Info` to gather all the info together, as shown in listing 5.8.

Listing 5.8 Class `Info`

**class** Info
  **function** Info(elem)       ❶
    **throw-if** elem == **null**   ❷
    **this**.root ← elem        ❸
    **this**.rank ← 1           ❹
 
**class** DisjointSet
  **#type** HashMap[Element, Info]
  parentsMap  

❶ The constructor for the `Info` class just takes an element of the disjoint set.

❷ Validates the argument

❸ Initially, an element is assigned to the singleton tree rooted at the element itself.

❹ The rank of the subtree is initially 1, because it only contains one element.

This `Info` class models (the info associated with) a node of the partitions’ tree. It is, to all purposes, just a container for two values: the root of the tree and the rank (that is, size) of the tree rooted at the current element.

In the root property, we won’t store references to other nodes. Instead, we will directly store the (index of) the element itself, that we then use as a key to a `HashMap`, exactly as we have shown in the previous sections.

If we were actually modeling a tree data structure, this design would result in imperfect encapsulation. But we are just using the `Info` class as a tuple to gather all the properties associated with an element.

Most implementations for disjoint set would use two arrays for this.Since our implementation does not restrict keys to integers and we are using hash maps, we could define two `Maps` for the element’s roots and ranks. In doing so, however, we would store each element three times: twice as a key of each map and once as a root of some tree (this last entry could store, of course, some keys several times and some others not once).

By using this extra wrapper and a single “info” map, we make sure to store elements only once as keys.

While objects would be stored as reference, with minimal overhead, immutable values, and especially strings, would be stored by value. Therefore, even avoiding storeing each element once more can lead to consistent memory saving.

We could, in theory, do even better by storing each element in object wrappers and using those wrappers as keys. This way, we would only store each key once, and use wrappers’ references all the time, both as keys for our map(s) and as values.

Is the overhead and increased complexity of the wrapper solution worth it? This depends on the assumptions you can make on the type and size of the keys. In most cases, it is probably not, so be sure to properly profile your application and analyze your input before embarking on such optimizations.

To go back to our implementation: once again, changes are minimal. In both constructor and `add`, we just need to update the very last line:

parentsMap[elem] = new Info(elem)

We use the constructor for `Info` and create a new instance associated with each element.

Things definitely get more interesting when we move to `findPartition`, implemented in listing 5.9.

Listing 5.9 Tree-with-heuristics solution, `findPartition`

**function** findPartition(elem)                              ❶
  **throw-if** (elem == **null or not** parentsMap.has(elem)))    ❷
  info ← **this**.parentsMap[elem]                            ❸
  **if** (info.root == elem) **then**                             ❹
    **return** elem
  info.root ← **this**.findPartition(info.root)               ❺
  **return** info.root

❶ Takes an element and `return` another element, the one element at the root of the tree for the partition to which elem belongs

❷ Checks that the element is valid

❸ Retrieves the info node stored for current element

❹ If the element’s root is the element itself, then we already got to the root of the tree, and we can return it.

❺ Otherwise, we need to climb up recursively to the root, but meanwhile we can update the root link for the current element so that it points to the actual root of the tree.

As described at the beginning of the section, when using the path compression heuristic, we don’t update the root of all elements on `merge`, but we do update it on `findPartition`. So, the main difference from the older version is that we save the result of the recursive calls to `findPartition` at line #6, and we use it to update the current element’s root. Everything else remains exactly the same.

It goes without saying that the largest portion of changes will be implemented in the `merge` method, as shown in listing 5.10.

Listing 5.10 Tree-with-heuristics solution, `merge`

**function** merge(elem1, elem2)               ❶
  r1 ← **this**.findPartition(elem1)           ❷
  r2 ← **this**.findPartition(elem2)           ❷
  **if** r1 == r2 **then**                         ❸
    **return false**
  info1 ← **this**.parentsMap[r1]              ❹
  info2 ← **this**.parentsMap[r2]
  **if** info1.rank >= info2.rank **then**         ❺
    info2.root ← **inf**o1.root;               ❻
    info1.rank += info2.rank;              ❼
  **else**
    info1.root ← **inf**o2.root;               ❻
    info2.rank += info1.rank;              ❼
 **return true**

❶ Takes two elements, merges their partitions, and returns `true` iff the two elements were in two different partitions that now are merged, `false` if they were already in the same partition

❷ Retrieves the roots of the trees to which `elem1` and `elem2` belong to. If its argument is invalid or not found, these calls will throw.

❸ Compares `R1` and `r2`, and if they are the same there is nothing left to do. The elements are already in the same partition, and so no merge happens: `false` is returned.

❹ At this point, we know we need to merge the two partitions, so it looks for the info nodes for both roots.

❺ Checks if the first tree has a larger rank (more elements) or vice versa. The smallest tree will become a subtree of the root of the largest tree.

❻ Sets the root of the smallest tree to the other root

❼ Updates the rank of the root (of both trees, now). It’s not worth updating the rank of the other (former) root, because it will never be checked again.

We still retrieve the elements at the roots of the trees as before, and still check that they are not the same.

But after that, we actually have to retrieve the info for both roots, and we check which tree is larger: the smaller one will end up as the child, and we will reassign its root. Moreover, we need to update the rank for the larger tree’s root; its subtree will now also contain all the elements in its new child.

If you’d like to look at some code for the heuristics implementation, we have an example on GitHub.

This is all we need to change in order to achieve a tremendous boost in performance. The simplicity of the code shows you how clever this solution is, and in the next sections we will also see why it is so important to get it right.

## 5.7 Applications

Applications for disjoint set are ubiquitous, and the reason they have been studied at length is exactly due to the number of cases where they prove useful.

### 5.7.1 Graphs: Connected components

For _undirected graphs_, there is a simple algorithm that uses disjoint sets to keep track of their _connected components_, that is, areas of the graph that are interconnected.

While connected components are usually computed using Depth First Search (_DFS_), we can use a disjoint set to keep track of the components while we scan all the graph’s edges. An example is shown in listing 5.11.

Listing 5.11 Computing connected components of a graph with a disjoint set

 disjointSet = new DisjointSet(graph.vertices)       ❶
 **for** edge **in** graph.edges **do**                          ❷
   disjointSet.merge(edge.source, edge.destination)  ❸

❶ Creates a new disjoint set where each vertex of the graph is initially in a different partition

❷ Loops over each edge in the graph

❸ Merges the partitions to which source and destination vertices belong to

At the end, each partition of vertices in `disjointSet` will be a connected component.

It’s worth noting that this algorithm can not be used for _directed graphs_ and _strongly connected_ _components_.

### 5.7.2 Graphs:[15](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020534) Kruskal’s algorithm for minimum spanning tree

A spanning tree for a connected undirected _graph_ `G` is a tree whose nodes are the vertex of the graphs, and whose edges are a subset of `G`’s edges. If `G` is connected, then it certainly has at least one spanning tree—possibly many, if it also has cycles (see figure 5.13).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F13.png)

Figure 5.13 An example of a graph with several spanning trees. (A) An undirected, connected graph, containing cycles. (B) Since the graph has cycles, there are several spanning trees covering all nodes. A few examples are shown, each of them selecting only the smallest set of edges (the thick ones) that “span” all vertices. (C) For each set of edges, several trees can be obtained, depending on the root of the tree and the order of the children. (Only a few examples are shown. Notice, though, how they are not limited to binary trees.)

Among all possible spanning trees, a _minimum spanning tree_ (_MST_) is the one for which the sum of edges’ weights is minimal.

Kruskal’s algorithm is beyond the scope of this book. Here it suffices to say that it constructs the MST for a graph by

1. Starting with each vertex in a difference set.
    
2. Keeping a disjoint set of the graph vertices.
    
3. Going through the graph’s edges _in order of increasing weight_.
    
4. For each edge, if its extremes are not in the same partition, merge their components.
    
5. If all vertices belong to the same component, stop.
    

The MST will be defined by the list of edges that triggers, at point (4), merge calls for the disjoint set.

### 5.7.3 Clustering

Clustering is the most-used unsupervised learning[16](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020549) algorithm. The problem here is that we would like to get a partitioning of a set of points into a few, usually disjoint subsets, as shown in figure 5.14.

There are several types of clustering algorithms. Although a taxonomy of clustering is beyond the scope of this chapter (we will devote chapter 12 to this topic), here we will mention one particular class of these algorithms.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F14.png)

Figure 5.14 An example of clustering. On the left, we have a raw dataset of 2-D points. We have no extra information about the points or the relationships between them. After clustering the dataset, on the right we can see that we have inferred some relationships between the points, and in particular we grouped them in three subsets whose points seem to show higher correlation.

_Agglomerative hierarchical clustering_ starts with each point in its own _cluster_ (partition) and continuously merges two points (and their clusters) until all clusters are merged into one; figure 5.15 shows an example of how it works. The algorithm keeps a history of this process, and it is possible to get a snapshot of the clusters created at any of the steps. The exact point where this snapshot is taken is controlled by a few parameters and determines the result of the algorithm.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch05_F15.png)

Figure 5.15 An example of hierarchical clustering. On the left, the dataset (a collection of 2-D points) is shown with progressive grouping shown as ellipses. From the figure, it can be inferred that, for instance, A and B are grouped together before C is added to the two of them to form a larger cluster. Hence, the relationship between A and B is inferred to be stronger than the one between A and C or B and C. On the right, the same process is shown using a dendrogram.[17](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch05.htm#pgfId-1020596) Note that both figures could be the result either of agglomerative or divisive clustering: the former produces the dendrogram starting from the bottom, the latter starting from the top.

The description of the algorithm should ring a bell: at each step we need to find two points belonging to two different clusters. You can easily imagine what the best data structure is to compute and find that information. In chapter 13, we’ll see a practical application of disjoint set as part of a distributed clustering algorithm.

### 5.7.4 Unification

Unification is the process of solving equations between symbolic expressions. One of the ways of solving such an equation is finding terms on both sides that are equivalent, and removing them from the equation.

Of course, solution strategies depend on which expressions (terms) can appear in the equation and how you can compare them or when they are considered to be equal. For instance, they might be evaluated and considered equal if they have the same value, or they might be symbolic and considered equal if they are equivalent, possibly net of some variable substitution.

As you can imagine, the disjoint set data structure is perfect for high-performance algorithms solving this problem.

## Summary

- The beauty of a disjoint set is that we can build increasingly complex and efficient solutions to solve it, just adding small incremental changes.
    
- We can sometimes settle for a sub-optimal implementation if that’s efficient enough and performance is not critical.
    
- Probably we could settle for the naïve linear time solution, but it is such a fundamental part of many graph algorithms that we just need to optimize it as much as possible.
    
- We do know a theoretical lower bound for the running time of operations of a disjoint set, but we don’t know if there exists an algorithm that runs with that bound, or even any other algorithm faster than the ones we know.
    
- The _inverse Ackermann function_, whose value won’t be greater than 5 for any integer that could fit on a computer, models the order of magnitude of the running time for the `merge` operation on disjoint sets. Merging two subsets will, on average, only require at most five swaps.
    

---

1. A (no-SQL) key-value store used as a distributed object caching system.

2. The set of all possible items—traditionally denoted as Universe (`U`) in set theory.

3. Definition of protected visibility varies depending on the programming language. Here, we assume a protected method or attribute is only visible to the class declaring it and its sub-classes. A private method, conversely, is not visible to any classes inheriting from the one in which it is declared.

4. This is a theory-intensive section.

5. As an implementation detail, we likely need to use referential equality here when comparing lists.

6. For a refresher on linked lists, or if you are not sure what the `next` pointer is, check out appendix C.

7. Remember that constant factors are irrelevant in big-O analysis, so `O(n/2) = O(n)`.

8. Abbreviation for “if and only if.”

9. A singleton is a set with exactly one element.

10. See [https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction#disjoint-set](https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction#disjoint-set).

11. This section is theory-intensive and features advanced concepts.

12. Check out appendix C, section C.1.3.

13. This section is theory-intensive and features advanced concepts.

14. It is possible to find plenty of literature on the subject. Be aware, though, that it’s very interesting reading, but challenging.

15. For an introduction to graphs, see appendix G.

16. Unsupervised machine learning deals with making sense of “unlabeled” data (that is, data that has not been classified or categorized), with the goal of finding a structure in the raw data.

17. A dendrogram (from Greek dendro “tree” and gramma “drawing”) is a tree diagram specifically used to illustrate the arrangement of the clusters produced by hierarchical clustering.