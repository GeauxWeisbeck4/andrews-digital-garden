---
id: 01J9QF5778WXW1MXEX1GRXGHC5
modified: 2024-10-08T21:44:16-04:00
---
_Priority queues_ are a class of data structures that retrieve items ordered by given scores for each item. Whereas both stacks and queues from Chapter 4 depended solely on the order in which data was inserted, priority queues use an additional piece of information to determine the retrieval order—the item’s priority. As we will see, this new information allows us to further adapt to the data and, among other useful applications, allows us to process urgent requests first.

For example, imagine a new coffee shop, Dynamic Selection Coffee, has opened in your neighborhood. Bursting with excitement, you venture into the store and see 10 types of coffee beans you’ve never sampled. Before diving in and trying them, you spend the next hour carefully charting the relative merits of each new brand based on their woefully inadequate menu descriptions, leaving you with a ranked list of coffees to try. You select the most promising brand from the top of the list, purchase it, and go home to savor the experience.

The next day, you return to Dynamic Selection Coffee to try the second item on your list, only to find they’ve added another two coffees to the menu. When you ask the barista why they’ve made this change, they point to the shop’s sign, which explains that Dynamic Selection Coffee serves a constantly expanding selection of coffees. Their goal is to eventually serve over a thousand varieties. You are at once thrilled and terrified. Every day you will need to prioritize the new coffees and insert them into your list so that you know which coffee to try next.

The task of retrieving items from a prioritized list is one that pops up regularly in computer programs: given a list of items and associated priorities, how can we efficiently retrieve the next item in priority order? Often, we need to do this retrieval in a dynamic context where new items arrive all the time. We might need to choose which network packet to process based on priority, offer the best suggestion in spellcheck based on common spelling errors, or choose the next option in a best-first search. In the real world, we might use our own mental priority queues to decide which urgent task to perform next, which movie to watch, or which patient to see first in a crowded emergency room. Once you start looking, prioritized retrievals are everywhere.

In this chapter, we introduce the priority queue, a class of data structures for retrieving prioritized items from a set, and then discuss the most common data structure for implementing this useful tool: the heap. Heaps make the central operations for a priority queue extremely efficient.

## Priority Queues

Priority queues store a set of items and enable the user to easily retrieve the item with the highest priority. They are dynamic, allowing insertions and retrievals to be intermixed. We need to be able to add and remove items from our prioritized task list. If we were stuck using a fixed data structure, the author might spend the day consulting his static list and repeatedly performing the highest-priority task of “Get morning coffee.” Without the ability to remove that task after it was accomplished, it would stay at the top of the author’s list. While this might make for an enjoyable day, it is unlikely to be a productive one.

In their most basic form, priority queues support a few primary operations:

- Add an item and its associated priority score.
- Look up the item with the highest priority (or null if the queue is empty).
- Remove the item with the highest priority (or null if the queue is empty).

We can also add other useful functions that allow us to check whether a priority queue is empty or to return the number of items currently stored.

We set the items’ priorities according to the problem at hand. In some cases, the priority values might be obvious or determined by the algorithm. When processing network requests, for example, each packet might come with an explicit priority, or we might choose to process the oldest request first. Deciding what to prioritize isn’t always a simple endeavor, however. When prioritizing which brand of coffee to try next, we might want to create a priority based on price, availability, or caffeine content—it depends on how we plan to use our priority queue.

It’s possible, but not ideal, to implement priority queues with primitive data structures like sorted linked lists or sorted arrays, adding new items into the list according to their priority. [Figure 7-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-1) shows an example of adding the value 21 into a sorted linked list.

![A linked list before and after the addition of an eighth element into the middle of the list where values are in decreasing order. The head of the list has the value 50, and the end of the list has value 9. Value 21 is inserted between value 28 and value 15.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07001.png)

Figure 7-1: Adding an element (21) into a sorted linked list representing a priority queue

A sorted linked list keeps the highest-priority item at the front of the list for easy lookup. In fact, in this case, a lookup takes constant time regardless of the length of the priority queue—we just look at the first element. Unfortunately, adding new elements can be expensive. We might need to traverse the entire list each time we add a new item, costing us time proportional to the length of the priority queue.

The author uses a real-world version of this sorted-list approach to organize his refrigerator, storing items front-to-back in order of increasing expiration date. The closest item is always the highest priority, the one that will expire the soonest. Retrieving the correct item is easy—just grab whatever is in front. This scheme is particularly beneficial when storing milk or cream for morning coffee: no one wants to spend the first few bleary moments of the morning reading expiration dates. However, inserting new items behind old ones can take time and require an annoying amount of shifting.

Similarly, we could maintain the priority queue in an _unsorted_ linked list or array. New additions are trivial—just tag the element onto the back of the list, as illustrated in [Figure 7-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-2).

![An unsorted array before and after the addition of the eighth element. The value 21 is added to the end of the unsorted array after the value 39.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07002.png)

Figure 7-2: Adding an element (21) into an unsorted array representing a priority queue

Unfortunately, we now pay a high cost in looking up the next element. We must scan the entire list in order to determine which element has the highest priority. If we are removing it, we also shift everything over to fill in the gap. This approach corresponds to scanning the full set of items in the author’s fridge to find whichever is closest to expiring. This might work for a fridge with just a few milk cartons, but imagine the overhead of picking through every single item of food or drink in a large grocery store.

Implementing priority queues as sorted lists may work better than using unsorted lists or vice versa, depending on how we plan to use the priority queue. If additions are more common than lookups, we prefer the unsorted list. If lookups are more common, we should pay the cost of keeping the elements sorted. In the case of refrigerators, lookups are much more common—we pick up a single carton of milk to use it more often than we buy a new carton, so it pays to keep the milk in sorted order. The challenge arises when both additions and lookups are common; prioritizing one operation over the other will lead to overall inefficiency, so we need a method that balances their costs. A clever data structure, the heap, helps us solve this problem.

## Max Heaps

A _max heap_ is a variant of the binary tree that maintains a special ordered relationship between a node and its children. Specifically, a max heap stores the elements according to the _max heap property_, which states that the value at any node in the tree is larger than or equal to the values of its child nodes. For simplicity’s sake, we will often use the more general terms _heaps_ and _heap property_ to refer to max heaps and the max heap property throughout the remainder of the chapter.

[Figure 7-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-3) shows a representation of a binary tree organized according to the max heap property.

![The binary tree for a heap starts with the largest element (99) at the root with two children 67 and 97. For any node, the values of both the left and right children are less than or equal to the value of the node itself.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07003.png)

Figure 7-3: A heap represented as a binary tree

There’s no special preference or ordering between the left and right children, aside from their being lower priority than their parent. For comparison, imagine an elite coffee lover’s mentorship program, the Society for the Improvement of Coffee-Related Knowledge. Each member (node) agrees to mentor up to two other coffee lovers (child nodes). The only condition is that each of the mentees must not know more about coffee than their mentor does—otherwise, the mentorship would be a waste.

Computer scientist J. W. J. Williams originally invented heaps as a component of a new sorting algorithm, heapsort, which we’ll discuss later in the chapter. However, he recognized that heaps are useful data structures for other tasks. The max heap’s simple structure allows it to efficiently support the operations required for priority queues: (1) allowing a user to efficiently look up the largest element, (2) removing the largest element, and (3) adding an arbitrary element.

Heaps are often visualized as trees but are often implemented with arrays for efficiency. In this chapter, we present these two representations in parallel to allow the reader to make mental connections between them. However, it is not required that we use arrays for implementation.

In the array-based implementation, each element in the array corresponds to a node in the tree with the root node at index 1 (we skip index 0 to stay consistent with common convention for heaps). Child node indexes are defined relative to the indexes of their parents, so a node at index _i_ has children at indexes 2_i_ and 2_i_ + 1. For example, the node at index 2 has a child at index 2 × 2 = 4 and index 2 × 2 + 1 = 5, as shown in [Figure 7-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-4).

![A heap represented both as an array and as a tree with arrows indicating where each node sits in the array. The root node 98 corresponds to the first element in the array. The node’s two children, 95 and 50, are the second and third elements of the array.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07004.png)

Figure 7-4: The heap’s position corresponds to index locations.

Similarly, we compute the index of a node’s parent as `Floor(i/2)`. The indexing scheme allows the algorithm to trivially compute the index of a child based on that of the parent, and the index of a parent based on its child.

The root node always corresponds to the maximum value in a max heap. Since we store the root node in a fixed spot in the array (index = 1), we can always find this maximum value in constant time. It’s just an array lookup. The layout of the data itself thus addresses one of the necessary priority queue operations.

Since we’ll be adding and removing arbitrary elements in our priority queue, in order to avoid constantly resizing the array, we want to preallocate an array that is large enough to accommodate the number of items we expect to add. Remember from Chapter 3 that dynamically resizing an array can be expensive, forcing us to create a new array and copy over the values, and in this case wasting the precious efficiency of the heap. Instead, we can initially allocate a large array, track the index of the last filled element of the array, and call that index the virtual end of the array. This allows us to append new elements by simply updating the index for the last element.

```
Heap {
    Array: array
    Integer: array_size
    Integer: last_index
}
```

Of course, the cost of this overallocation is a chunk of potentially unused memory if our heap doesn’t grow as large as expected.

Using an array to represent a tree-based data structure is an interesting step in its own right. We can use packed arrays and a mathematical mapping to represent a heap instead of relying on pointers, allowing us to store the heap with less memory. By maintaining a mapping from a node’s index to its children’s, we can re-create a tree-based data structure without the pointers. As we will see below, this array-based representation is feasible for heaps because the data structure always maintains a nearly complete and balanced tree. This results in a packed array with no gaps. While we could use the same array representation for other trees, such as binary search trees, those data structures often have gaps throughout and would require very large (and possibly mostly empty) arrays to store trees with deep branches.

### Adding Elements to a Heap

When adding a new element to a heap, we must ensure that the structure retains the heap property. Just as you would not assign a decorated general to report to a fresh lieutenant, you wouldn’t put a heap node with high priority under a low priority node. We must add the element to the heap’s tree structure such that all elements below the addition have a priority less than or equal to that of the new node. Similarly, all nodes above the new addition should have priorities that are greater than or equal to that of the new node.

Part of the brilliance of the array implementation of a heap is that it retains this property while storing the nodes as a packed array. In previous chapters, adding nodes to the middle of an array was expensive, requiring us to shift later entries down. Fortunately, we don’t need to pay this linear cost each time we add a new element to a heap. Instead, we add elements by first breaking the heap property and then swapping elements along a single branch of the tree to restore it.

In other words, to add a new element to the heap, we add it to the first empty space in the bottom level of the tree. If this new value is larger than the value of its parent node, we bubble it up the tree until it is smaller than or equal to its parent, restoring the heap property. The structure of the heap itself allows us to do this efficiently. In the array implementation of a heap, this corresponds to appending the new element to the back of the array and swapping it forward.

Consider the example in [Figure 7-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-5), which shows the structure of the heap as both an array and a tree at each step. [Figure 7-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-5)(a) shows the heap before the new element is added. In [Figure 7-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-5)(b), we append the new element, 85, to the back of the array, effectively inserting it in the bottom of the tree. After the first comparison in [Figure 7-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-5)(c), we swap the new element with its parent node, since 85 is greater than 50. The swap is shown in [Figure 7-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-5)(d). The second comparison, in [Figure 7-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-5)(e), reveals that the new node is now in the correct place in the hierarchy: 98 is greater than 85, so there is no need for the new node to switch with its parent a second time. [Figure 7-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-5)(f) shows the heap after the addition is complete.

![We start with the array from Figure 7‐4. Node 85 is added to the end of the array and then bubbled up to the correct position. We first compare 85 with its parent 50 and swap them. In the next step, we compare 85 with its new parent 98 and keep the ordering.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07005.png)

Figure 7-5: Adding an element (85) to a heap

The code for implementing this addition uses a single `WHILE` loop to progress up the levels of the heap until it has reached the root or found a parent larger than or equal to the new node’s value:

```
HeapInsert(Heap: heap, Type: value):
  ❶ IF heap.last_index == heap.array_size - 1:
        Increase Heap size.

  ❷ heap.last_index = heap.last_index + 1
    heap.array[heap.last_index] = value

    # Swap the new node up the heap.
  ❸ Integer: current = heap.last_index
    Integer: parent = Floor(current / 2)
  ❹ WHILE parent >= 1 AND (heap.array[parent] < 
                           heap.array[current]):
      ❺ Type: temp = heap.array[parent]
        heap.array[parent] = heap.array[current]
        heap.array[current] = temp
        current = parent
        parent = Floor(current / 2)
```

Since we are using an array to store the heap, the code starts by checking that it still has room in the array to add a new element ❶. If not, it increases the size of the heap, perhaps by applying the array-doubling technique described in Chapter 3. Next, the code appends the new element to the end of the array and updates the position of the last element ❷. The `WHILE` loop starts at the freshly added element ❸, progresses up each layer of the heap by comparing the current value to that of its parent ❹, and switching if necessary ❺. The loop terminates when we either reach the top of the heap (`parent == 0`) or find a parent greater than or equal to the child.

We might compare this process to an oddly designed, yet efficient, package distribution center as shown in [Figure 7-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-6). The employees organize packages in neat rows on the floor using the heap property: in the front row is a single package with the highest priority, the next to be shipped. Behind that package sit two lower-priority packages. Behind each of these are two more packages (for a total of four in that row) such that each pair has priorities less than or equal to the corresponding package in front of them. With each new row, the number of packages doubles so that the arrangement spreads out wider and wider as you move toward the back of the warehouse. Each package has at most two lower- or equal-priority packages sitting behind it and at most one higher- or equal-priority package in front. Painted rectangles on the warehouse floor helpfully indicate the possible locations for packages in each of the rows.

![A diagram showing rows of packages on a warehouse floor, with one package in the top row, two in the second row, four in the third, and eight in the fourth. The fifth and last row is not completely filled.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07006.png)

Figure 7-6: A warehouse floor organized as a heap

New packages are brought in from the back of the warehouse. Each delivery person mumbles something about the weird sorting scheme, relieved that at least they don’t have to carry their package to the very front, drops the package in the frontmost available spot, and leaves as quickly as possible. The warehouse employees then spring into action, comparing the priority of the new package with the one immediately in front of it (ignoring the rest of the packages in that row). If they find an inversion, they switch the packages. Otherwise, they leave the package where it is. This continues until the new package occupies the appropriate place in the hierarchy, with the package in front of it having a higher or an equal priority. Since the packages are heavy and spread out, the employees minimize their work with at most one comparison and switch per row. They never shuffle packages within a row. After all, nobody wants to move boxes unnecessarily.

Intuitively, we can see that heap additions are not terribly expensive. In the worst case, we would have to swap the new node all the way to the root of the tree, but this only means swapping a small fraction of the array’s values. By design, heaps are _balanced_ binary trees: we fill out a complete level of the tree before inserting a node into the next level. Since the number of nodes in a full binary tree doubles with each level, the addition operation requires, in the worst case, log2(_N_) swaps. This is significantly better than the worst case of _N_ swaps required to maintain a sorted list.

### Removing the Highest-Priority Elements from Heaps

Looking up and removing the highest-priority element from a priority queue is a core operation that allows us to process items in order of their priority. Perhaps we are storing a list of pending network requests and want to process the highest-priority one. Or we could be running an emergency room and looking to see the most urgent patient. In both cases, we want to remove this element from our priority queue so that we can go on to extract the next highest priority element.

To remove the highest-priority node, we must first break, then restore, the heap property. Consider the example in [Figure 7-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-7). We start by swapping the highest-priority node with the last node in the lowest level of the tree ([Figure 7-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-7)(b)), effectively making the last element the new root node. However, this epic promotion for the new root node is almost guaranteed not to stand up to scrutiny. In the array implementation, this corresponds to swapping the first and last elements in the array. This swap plugs the gap at the front of the array that would be created by removing the first element and thus maintains a packed array.

Next, the original max value of 98, which is currently the last element in the tree, is deleted as in [Figure 7-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-7)(c). We have now deleted the correct node, but likely broken the heap property in the process. We may have moved a low-urgency package all the way to the front of the warehouse.

## NOTE

It is also possible that swapping the first and last elements in the heap will not break the heap property, especially in the case of duplicate priorities or small heaps.

![Node 98 is removed from the heap by first swapping it with the last element (23). The new root bubbles down the heap by swapping with the larger of its two children. In the first step, node 23 is compared with 95 and 50 and then swapped with 95.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07007.png)

Figure 7-7: Removing the highest-priority element from a heap

To fix the heap property, we start at the new (incorrect) root node of 23 and walk down the tree, restoring the heap property at every level. The incorrectly placed package is shifted one row at a time toward the back of the warehouse, the opposite of an added package’s shifting forward row by row. Admittedly, this traversal isn’t as exciting as moving an urgent package to the front of the line, but it’s for the good of the heap structure. At each level, we compare our moving package’s priority to that of both its children, the two packages in the next row ([Figure 7-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-7)(d)). If it’s smaller than either of its children, we move the new root node backward to restore the heap property by swapping places with the larger of its two children ([Figure 7-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-7)(e)). This represents the package’s higher-priority successor moving forward to take its place.

The downward swaps terminate when there are no larger children. [Figure 7-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-7)(f) shows a comparison made when the current node is in the correct position. The heap property has been restored, and all the nodes are satisfied with their relative position. [Figure 7-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-7)(g) shows the final heap, once the removal is complete.

While fixing the location for the root node, we follow a single path down the tree, checking and restoring the heap property for only the descendant branch where we made the swap. There is no need to check the other branch. It is already guaranteed to maintain the heap property since we haven’t done anything to break it. Again, this means that, in the worst case, we need to make log2(_N_) swaps.

Here’s the code for removing the max element:

```
HeapRemoveMax(Heap: heap):
  ❶ IF heap.last_index == 0:
        return null
    
    # Swap out the root for the last element and shrink heap.  
  ❷ Type: result = heap.array[1]
    heap.array[1] = heap.array[heap.last_index]
    heap.array[heap.last_index] = null
    heap.last_index = heap.last_index - 1

    # Bubble the new root down.
    Integer: i = 1
  ❸ WHILE i <= heap.last_index:
        Integer: swap = i
      ❹ IF 2*i <= heap.last_index AND (heap.array[swap] < 
                                       heap.array[2*i]):
            swap = 2*i
      ❺ IF 2*i+1 <= heap.last_index AND (heap.array[swap] <
                                         heap.array[2*i+1]):
            swap = 2*i+1

      ❻ IF i != swap:
            Type: temp = heap.array[i]
            heap.array[i] = heap.array[swap]
            heap.array[swap] = temp
            i = swap
        ELSE:
            break
    return result
```

This code starts by checking that the heap is not empty ❶. If it is, there is nothing to return. The code then swaps the first element (`index == 1`) with the last element (`index == heap.last_index`), breaking the heap property to prepare the max element for removal ❷. The code then uses a `WHILE` loop to traverse down the heap in a series of comparisons, repairing the heap property ❸. During each iteration, it compares the current value to both children and swaps with the larger one if necessary ❻. We must add additional checks ❹ ❺ to ensure that the code is only comparing the current value to an existing child. We don’t want it to try to compare against an entry that is past the array’s last valid index. The loop terminates when it has hit the bottom of the heap or has gone an iteration without a swap (via the `break` statement).

### Storing Auxiliary Information

Much of the time, we need our heap to store additional information for each entry. In our task list, for example, we need to store information about the tasks to be done, not just their priority. It doesn’t help us to know that we need to do the priority = 99 task next, if we don’t know what that task is. We might as well just scan through the original list manually.

Augmenting the heap to store composite data structures or objects, such as a `TaskRecord`, is simple:

```
TaskRecord {
    Float: Priority
    String: TaskName
    String: Instructions
    String: PersonWhoWillYellIfThisIsNotDone
    Boolean: Completed
}
```

We modify the previous code to handle comparisons based on the priority field of this composite record. We could do this by directly modifying the code (such in the `HeapInsert` function):

```
      WHILE parent >= 1 AND (heap.array[parent].priority < 
                             heap.array[current].priority):
```

However, this requires us to potentially specialize the heap implementation to the particular composite data structure. A cleaner approach is to add a composite data structure specific helper function, such as:

```
IsLessThan(Type: a, Type: b):
  return a.priority < b.priority
```

We’d use this function instead of the mathematical less-than sign in the code for `HeapInsert`:

```
    WHILE parent >= 1 AND IsLessThan(heap.array[parent], 
                                     heap.array[current]):
```

Similarly, we would modify the comparisons in the `HeapRemoveMax` function to use the helper function.

```
        IF 2*i <= heap.last_index AND IsLessThan(heap.array[swap],
                                                 heap.array[2*i]):
            swap = 2*i
        IF 2*i+1 <= heap.last_index AND IsLessThan(heap.array[swap],
                                                   heap.array[2*i+1]):
            swap = 2*i+1
```

These small changes allow us to build heaps from composite data structures. As long as we can define an `IsLessThan` function to order the elements, we can build an efficient priority queue for them.

## Updating Priorities

Some use cases might demand another mode of dynamic behavior: allowing the algorithm to update the priorities of elements within the priority queue. Consider a bookstore database that prioritizes which books to restock by the number of patrons who have requested each title. The system builds the heap over an initial list of books and uses that to determine which title to order next. But, after a popular blog article points out the vital importance of data structures in computational thinking, the store suddenly sees a dramatic—though entirely understandable—increase in patrons requesting books about data structures. Its priority queue must be equipped to handle this sudden influx.

To meet this need, we use the same approaches we applied to addition and removal. When we change an item’s value, we check whether we are increasing or decreasing the priority. If we are increasing the item’s value, we need to bubble the item up the max heap in order to restore the heap property. Similarly, if we are decreasing the item’s value, we let it sink down the max heap into its rightful position.

```
UpdateValue(Heap: heap, Integer: index, Float: value):
    Type: old_value = heap.array[index]
    heap.array[index] = value

    IF old_value < value:
        Bubble the element up the heap using the
        procedure from inserting new elements 
        (swapping with parent).
    ELSE:
        Drop the element down the heap using the
        procedure from removing the max element 
        (swapping with the larger child).
```

We can even break out the code for letting elements bubble up or sink down so that the exact same code can be used for updating as for addition and removing the max.

How do we find the element we want to update in the first place? As mentioned, heaps aren’t optimized for finding specific elements. If we have no information on our element of interest other than its value, we might need to search through a substantial portion of the array to find it. Often, we can solve this problem by using a secondary data structure, such as a hash table (discussed in Chapter 10), to map from the item’s key to its element in the heap. In the example in this section, we assume that the program already has the item’s current index.

## Min Heaps

So far, we have focused on the max heap, which uses the property that the value at any node in the tree is larger than (or equal to) the values of its children. The _min heap_ is a version of the heap that facilitates finding the item with the lowest value. With a min heap, the root of the tree is the smallest value, allowing us to easily find the item with the lowest score. For example, instead of ordering network packets by priority, we might want to sort them by arrival time, processing packets with an earlier time of arrival before packets received more recently. More importantly, if we run out of space on our coffee shelf, we’ll need to remove our least favorite brand. After a gut-wrenching internal debate on whether it would be better to increase the size of our coffee storage by discarding a shelf’s worth of plates or bowls rather than part with any of our precious coffee grounds, we instead decide to discard the lowest-ranked coffee. We consult our list of enjoyability scores for each coffee and select the one with the lowest value.

We could, in theory, continue to use a max heap by just negating the values. However, a cleaner strategy is to make a minor tweak to our heap property and solve the problem outright. The _min heap property_ is that the value at any node in the tree is smaller than (or equal to) the values of its children. An example min heap is shown in [Figure 7-8](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-8). When we insert new elements, the ones with the lowest score bubble up the hierarchy. Similarly, we always extract and return the element with the lowest score.

![A min heap represented both as an array and as a tree with arrows indicating where each node sits in the array. The root node 10 corresponds to the first element in the array. The node’s two children, 23 and 17, are the second and third elements of the array.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07008.png)

Figure 7-8: The min heap’s position corresponds to index locations.

Of course, the algorithms for adding and removing elements in the min heap need to be modified accordingly. For insertion, we change the comparison function in the `WHILE` loop to check whether the parent value is greater than the current value:

```
MinHeapInsert(MinHeap: heap, Type: value):
    IF heap.last_index == heap.array_size - 1:
        Increase Heap size.

    heap.last_index = heap.last_index + 1
    heap.array[heap.last_index] = value

    # Swap the new node up the heap.
    Integer: current = heap.last_index
    Integer: parent = Floor(current / 2)
  ❶ WHILE parent >= 1 AND (heap.array[parent] > 
                           heap.array[current]):
        Type: temp = heap.array[parent]
        heap.array[parent] = heap.array[current]
        heap.array[current] = temp
        current = parent
        parent = Floor(current / 2)
```

The majority of the code is identical to that for insertion into a max heap. The only change is to replace `<` with `>` when comparing a node to its parent ❶.

We make a similar change for the two comparisons during the removal operation:

```
MinHeapRemoveMin(Heap: heap):
    IF heap.last_index == 0:
        return null
    
    # Swap out the root for the last element and shrink heap.  
    Type: result = heap.array[1]
    heap.array[1] = heap.array[heap.last_index]
    heap.array[heap.last_index] = null
    heap.last_index = heap.last_index - 1

    # Bubble the new root down.
    Integer: i = 1
    WHILE i <= heap.last_index:
        Integer: swap = i
      ❶ IF 2*i <= heap.last_index AND (heap.array[swap] > 
                                       heap.array[2*i]):
            swap = 2*i
      ❷ IF 2*i+1 <= heap.last_index AND (heap.array[swap] >
                                         heap.array[2*i+1]):
            swap = 2*i+1

        IF i != swap:
            Type: temp = heap.array[i]
            heap.array[i] = heap.array[swap]
            heap.array[swap] = temp
            i = swap
        ELSE:
            break
    return result
```

Here, the only changes are replacing `<` with `>` when determining whether to swap nodes ❶ ❷.

## Heapsort

Heaps are powerful data structures across a range of computer science tasks, not limited to implementing priority queues and efficiently returning the next item in a prioritized list. Another exciting lens through which to view heaps, and data structures in general, is in terms of the novel algorithms that they enable. J. W. J. Williams initially proposed heaps themselves in the context of a new algorithm to sort arrays: _heapsort_.

As its name implies, heapsort is an algorithm for sorting a list of items using the heap data structure. The input is an unsorted array. The output is an array containing those same elements, but in _decreasing_ sorted order (for a max heap). At its core, heapsort consists of two phases:

1. Building a max heap from all the items
2. Extracting all the items from the heap in decreasing sorted order and storing them in an array

It’s that simple.

Here’s the heapsort code:

```
Heapsort(Array: unsorted):
    Integer: N = length(unsorted)
    Heap: tmp_heap = Heap of size N
    Array: result = array of size N      

    Integer: j = 0
  ❶ WHILE j < N:
        HeapInsert(tmp_heap, unsorted[j])
        j = j + 1

    j = 0
  ❷ WHILE j < N:
        result[j] = HeapRemoveMax(tmp_heap)
        j = j + 1
    return result
```

This code consists of two `WHILE` loops. The first one inserts each item into a temporary heap ❶. The second loop removes the largest element using the `HeapRemoveMax` function and adds it to the next position in the array ❷. Alternatively, we can implement heapsort to produce an answer in _increasing_ sorted order by using a min heap and `HeapRemoveMin`.

Suppose we want to sort the following array: `[46, 35, 9, 28, 61, 8, 38, 40]` in decreasing order. We start by inserting each of these values into our heap. [Figure 7-9](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-9) shows the final arrangement of the array (and its equivalent tree representation) after the insertion. Remember that we always begin by inserting new items at the back of the array, then swap them forward until the heap property is restored. In [Figure 7-9](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-9), arrows represent the new element’s path through the array to its final position. The tree representation is also shown, with shading indicating the nodes that have been modified.

![An illustration showing each of the eight insertions during the first stage of heapsort, both in an array and in a tree. The numbers 46, 35, 9, and 28 are all inserted without swaps. When 61 is inserted at the top of the tree and front of the array, it swaps position first with 35 and then 46 to become the new root node.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07009.png)

Figure 7-9: The first stage of heapsort where elements in the unsorted array are added to the heap one at a time

We bound the runtime of creating the heap using the worst-case runtime of a single insertion. As we saw earlier in the chapter, in the worst case, inserting a new item into a heap containing _N_ items can scale proportional to log2(_N_). Thus, to construct a heap out of _N_ items, we bound the worst-case runtime as proportional to _N_log2(_N_).

Now that we’ve built our heap, we proceed to the second stage and extract each item, as shown in [Figure 7-10](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c07.xhtml#figure7-10).

![An illustration showing each of the removals during the second stage of heapsort, both in an array and in a tree. During the first removal, the node 61 is removed, 46 is swapped into its place, 40 is swapped into 46’s former place, and 28 is promoted into 40’s former place.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c07/f07010.png)

Figure 7-10: The second stage of heapsort, where the maximal element is repeatedly removed from the heap with each iteration

We remove the items from the heap one by one in order of decreasing priority. This is what produces the decreasing sorted order. At each step, the root is extracted, the last item in the heap is swapped to the root’s position, and then the new root sinks back down into a position that restores the heap property. The figure shows the state of the array (and the equivalent tree representation) at the end of each iteration. The arrows in the diagrams and the shaded nodes illustrate the overpromoted node’s journey down though the heap. As we extract the items, we add them directly to the array storing our result. Once we have emptied the heap, we throw it away. It has served its purpose.

As with the insertions, we can bound the worst-case runtime as proportional to _N_log2(_N_). Each extraction requires at most log2(_N_) to restore the heap property, and we need to extract all _N_ items for our sorted list. Thus, the total worst-case runtime of the heapsort algorithm is proportional to _N_log2(_N_).

## Why This Matters

Heaps are a simple variation of binary trees that allow a different set of computationally efficient operations. By changing the binary search tree property into the heap property, we can change the behavior of the data structure and support a different set of operations.

Both addition of new elements and deletion of the maximum element require us to walk at most one path between the top and bottom of the tree. Since we can approximately double the number of nodes in a heap while adding only a single level of new nodes to the bottom, even large heaps allow speedy operations. Doubling the number of nodes in this way adds only one additional iteration to insertion and deletion! Furthermore, both operations guarantee that the tree remains balanced so that future operations will be efficient.

However, there’s always a tradeoff: by moving from the binary search tree property to the heap property, we can no longer efficiently search for a specific value. Optimizing for one set of operations often prevents us from optimizing from others. We need to think carefully about how we will use the data and design its structure accordingly.