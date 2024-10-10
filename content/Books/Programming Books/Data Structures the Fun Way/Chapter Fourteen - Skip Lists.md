---
id: 01J9QFBS7GAGKP22X343MFNP01
modified: 2024-10-08T21:47:51-04:00
---
This chapter introduces the _skip list_, a sorted linked list with multiple pointers that allow us to occasionally jump forward to an element further ahead in the list during operations like search, insertion, or deletion. This potential to jump mitigates one of the major concerns with linked lists—that we have to scan through all the elements to find a single target. Skipping some elements saves precious time.

To envision how skip lists work, consider the strategy I employ every time I lose my place in a book. Determined to avoid spoilers, I do not use binary search, which may jump to parts of the text I haven’t read yet. Instead, I start at the beginning of the book and skip forward multiple pages at a time—in sections large enough that I’m not scanning every page, but small enough that, if I overshoot, it won’t ruin the story. I use larger jumps in the beginning of my search but shift to smaller and smaller jumps as I near where I left off. Skip lists use a similar approach to dramatically change the behavior of the linked list, enabling it to take on problems that we’d previously reserved for tree-based data structures.

Skip lists, proposed by computer scientist William Pugh, are probabilistic data structures that make operations such as insertion, deletion, and searching significantly more efficient in the _average_ case. Instead of storing a single linked list, skip lists effectively create a layer of linked lists, each with only a subset of the nodes at the layer below. This means we start our search at higher levels of the skip list, where there are fewer nodes, and take large steps across the list to _skip_ unnecessary nodes. As we get closer to our target and refine our search, we drop down in the multilevel hierarchy. In the case of searching for our place in a book, this corresponds to using smaller and smaller jumps as we near our most recent location.

I’ve included skip lists in this book for two reasons. First, in keeping with almost every other data structure presented here, they demonstrate how additional information or structure can provide significant algorithmic advantages. In this case, multiple levels of links decrease the cost of a search. Second, and perhaps more exciting, skip lists are randomized data structures. Unlike Bloom filters, which are deterministic given the data, skip lists push the use of randomness a step further: their very structure is probabilistically determined in order to balance out performance in the average case. We use a random number generator to choose the level of each node and thus how far ahead it will let us skip.

## Randomized vs. Deterministic Structures

The change from a deterministically generated data structure to a randomized one introduces both complexities and benefits. The structure of every data structure we have examined so far is fully determined by the data we insert. For example, if we insert the same data in the same order into a binary search tree, we always get the same structure. The same applies for heaps, tries, grids, quadtrees, and so forth. Even two hash tables or Bloom filters will be identical if we use the same hash functions and insert the same set of items.

This determinism can lead to problems in the face of worst-case data. As we saw in Chapter 5, if we start with an empty binary search tree and insert elements in sorted order, our tree effectively becomes a sorted linked list. Each node will have a single child node in the same direction. One potential way to mitigate this problem is to insert the data in random order. While we still might happen to choose a bad ordering, the probability is significantly lower.

We can extend this randomized approach to constructing the data structures themselves by randomly choosing parameters during each insertion. Instead of varying the order of the data, we are varying how we link that data into our structure.

At first, the randomized approach can seem unintuitive. If we don’t know our input distribution, we may easily end up making bad structural choices for that distribution. We might worry that we will always choose the worst-case parameter. However, if we use a good randomization strategy, this level of failure will be exceedingly rare. On the other hand, the randomized design prevents us from making consistently suboptimal choices. While it might not lead to an optimal solution, it will often produce a reasonable one. The randomness can provide good average case performance. The randomness also helps smooth out cases where the data arrives in a pathologically bad ordering.

## Introducing Skip Lists

As we saw in Chapter 3, certain operations on linked lists are inherently limited by the list’s structure. We can’t efficiently search a linked list because we can’t randomly access elements. This has tragic consequences; even when we know the nodes are sorted, we can’t use binary search. We’re forced to crawl along the pointers from one node to another until we reach the target node. This frustrating limitation has caused many new computer scientists to tear out their hair, muttering unkind things.

Skip lists help alleviate this inefficiency by providing the ability to jump ahead multiple entries. At its heart, a skip list is simply a sorted linked list with multi-height nodes:

```
SkipList {
    Integer: top_level
    Integer: max_level
    SkipListNode: front
}
```

The field `top_level` represents the highest level currently in use, while the field `max_level` represents the highest allowable level. For simplicity, we specify `max_level` independently so we can preallocate an array of pointers at the start of our list.

Skip lists’ complexity, and thus their power, arises from the pointer structure within the nodes. Instead of storing a single pointer to the next node in the list, each node has a predefined level, or _height_, in which it stores pointers to the next node. Nodes of level _L_ maintain _L_ + 1 different forward pointers, one for each level [0, _L_]. Critically, the pointer at level _L_ links the current node to the next node _at the same height_, meaning that the pointers in `next` will often point to different skip list nodes.

```
SkipListNode {
    Type: key
    Type: value
    Integer: height
    Array of SkipListNodes: next
}
```

Since the higher levels of a skip list contain fewer nodes than the layers below them, nodes at these higher levels can link further than would have been possible at the lower layers. This allows algorithms to take larger steps at the higher layers and skip many intervening nodes. As the levels progress higher and higher, the number of nodes decreases, and these linkings jump further and further ahead.

Imagine the process of searching a skip list in terms of passing messages between buildings by signaling with flashlights. How far you can pass a message depends on what floor you are on and the heights of the buildings in your path. If you are stuck on the first floor, you can only pass messages to the adjacent building. Any building beyond that is blocked by the adjacent building itself. However, if you are lucky enough to be in a tall building, you can pass messages over the heads of closer, but shorter, buildings, as shown in [Figure 14-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c14.xhtml#figure14-1). Alternatively, if you need to send a message to your immediate neighbor, you can simply move down to the lowest floor.

![Five buildings of different heights. Building 2 is the highest, and building 5 is next highest. An arrow shows that someone in the second building can pass messages to the last building from a floor above the tops of the two buildings in between.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c14/f14001.png)

Figure 14-1: Moving between nodes in a skip list is like passing messages by flashlight between buildings in a city.

Skip lists create these linkings probabilistically. The program gives each node a random height, independent of the key stored in the node, and inserts the new node into its corresponding list for each level. Thus, a node with height 0 will only appear in the bottommost list, while a node with height 2 will appear in the lists for levels 0, 1, and 2. [Figure 14-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c14.xhtml#figure14-2) shows an example of this. In the message-passing example above, this would be equivalent to a building with only a single floor compared to a building with three floors. The building with three floors is able to pass messages at three different heights, potentially accessing up to three neighbors.

![The skip list contains the keys 0, 1, 8, 9, 12, and 17. Each node has a height between 0 and 2. The node for 1 has pointers to the node 8 at height 0, node 12 at height 1, and node 17 at height 2.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c14/f14002.png)

Figure 14-2: An example skip list

Since higher nodes provide the ability to skip further ahead, over the tops of the lower nodes, we ideally want to use them sparsely and sprinkle them throughout the list. In the message-passing example, we do not want our cityscape to include only buildings of the same height. We want a lot of single-story buildings with some medium buildings and a few taller buildings that allow us to jump our messages down the street. By choosing heights with the correct probability distribution, we can, on average, balance out the density at each level. Nodes of level _L_ + 1 are less numerous than nodes at level _L_. This leads to good average case performance and can help avoid the worst-case scenarios that can occur in other data structures.

As shown in [Figure 14-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c14.xhtml#figure14-2), this skip list implementation uses a dummy node `front` to store the pointers at the front of each level. The node `front` is a `SkipListNode` but contains no key or value. Tracking the front of the list in a `SkipListNode` makes the code for insertion and deletion significantly simpler, as we’ll see later in the chapter.

### Searching Skip Lists

To search a skip list, we start at the front of the top-most level and iterate through the list nodes. Speaking more informally based on the illustration in [Figure 14-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c14.xhtml#figure14-2), we start at the top left corner and proceed downward and to the right. At each iteration, we check whether there is another node along this level and, if so, whether its key is less than our target. If both those conditions are met, we move along to the next node at that level. If either of these conditions is false (we have hit the end of the level or found a node whose key is larger than or equal to our target), we drop down a level and continue our search from there. Our search terminates when we try to drop below the bottom level.

```
SkipListSearch(SkipList: list, Type: target):
    Integer: level = list.top_level
  ❶ SkipListNode: current = list.front

  ❷ WHILE level >= 0:
        WHILE (current.next[level] != null AND
               current.next[level].key < target):
           current = current.next[level]
        level = level - 1
      
  ❸ SkipListNode: result = current.next[0]
  ❹ IF result != null AND result.key == target:
        return result.value
    ELSE:
        return null
```

The code for a skip list search starts with the `current` node at the front of the topmost list ❶. Two nested `WHILE` loops handle the traversal. The inner loop iterates through the current linked list until it hits the end of the list (`current.next[level] == null`) or a node with a key larger than or equal to target (`current.next[level].key >= target`). The outer loop drops down a single level each iteration until we hit the bottom of the list ❷. If the target is in the list, it will be at the next node in the list ❸. However, we must check that that node both exists and has the correct key ❹. When the search loop terminates, we are guaranteed to be at the last node in the list with a key _less_ than the target. The target is either the next node in the list or does not exist.

Consider searching for a target of 14 in the list shown in [Figure 14-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c14.xhtml#figure14-3). We start at the front of level 3. The first node at this level has a key of 13, which is less than our target, so we progress to that node. At this point we've reached the end of the list for level 3. We can’t progress any further at this height. The next node pointer is null.

The search then drops down a level and continues on level 2. Here we find the next key in the list (14) is _not_ less than our target, so we drop down to level 1. The same condition holds true on level 1 and level 0—the next key in the list is not less than our target. Our search terminates once we have completed level 0. At this point, our current node’s (key = 13) `next` pointer leads to the target node.

![The search for 14 starts at the front, follows the pointer to node 13 at height 3, and then proceeds down the levels in that node.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c14/f14003.png)

Figure 14-3: Searching a skip list for the target of 14. The shaded entries and dashed pointers represent the ones traversed during the search.

Note that although we pointed to the target node for several iterations (at levels 2, 1, and 0), we continued the search until we passed the bottom level. This is due to our termination criteria in the code. We could add additional logic to halt the search earlier, but we keep the logic simpler here to match the logic used later for insertion.

In contrast, if we were searching the same list for a target of 12, as shown in [Figure 14-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c14.xhtml#figure14-4), we'd drop down to the bottom level significantly earlier in our search and progress along the bottom level.

![The search for 12 starts at the front, follows the pointer to node 2 at height 2, Proceeds down that node until height 0, and follows the pointer to node 9 at height 0.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c14/f14004.png)

Figure 14-4: Searching a skip list for the target of 12. The shaded entries and dashed pointers represent the ones traversed during the search.

We can picture this traversal as a squirrel’s navigation along a row of trees. Enjoying the views from higher branches, the squirrel jumps from tree to tree until there are no more trees of that height before its destination. Whenever possible it jumps between tall, old oak trees and sails over the shorter saplings in between. Since the taller trees are rarer, and thus further apart, the squirrel also covers more distance per jump. Moving from the wide branches of a tall tree to those of its tall neighbor requires fewer jumps than traversing all the small saplings in between.

However, the squirrel is unwilling to waste time backtracking and thus never overshoots its destination. Eventually the squirrel reaches a point where it will pass its destination if it jumps to the next tree at this level. Or perhaps there are no more trees of this height. Either way, the squirrel sighs and reluctantly descends to the lower level of branches before continuing forward. It progresses at the next level, taking jumps as large as possible and enjoying the scenic route, until it again hits a point where it needs to descend.

### Adding Nodes

The distribution we use to select the height of a new node can have a significant impact on the structure and performance of the skip list. If everything is the same level, whether minimum or maximum, our skip list devolves into nothing more than a sorted linked list with additional memory overhead. Worse, if we set all the heights to the maximum, we create multiple parallel lists without adding any search efficiencies. Ideally, we’d like fewer tall nodes with increasing numbers at each level below.

William Pugh’s original approach to selecting heights is to continually use a constant probability _p_ of adding another level. All nodes start at level 0. We keep flipping a weighted coin—choosing a random number from 0 to 1 and checking if it is less than _p_—until we either get a number greater than _p_ or hit the maximum allowable height. We count the number of flips less than _p_ and set that as our new level. For example, we could use _p_ = 0.5, in which case we’d expect approximately half the nodes at level _L_ to be promoted to level _L_ + 1. We can use the value of _p_ to balance search efficiency with memory usage. A smaller value of _p_ will mean fewer tall nodes and thus fewer pointers per node. We cap a node’s height at `max_level` to stay consistent with the preallocated array in the `front` node.

You can visualize this approach in terms of an inconsistent parent responding to a child’s request for more candy. When the child gets candy, they always gets a single piece, and then they always want more. Every time the child asks for candy, the parent randomly (with probability _p_) decides whether to grant the request. If so, they give the child another piece. This corresponds to increasing the height of the node by one. Naturally the child, seeing their victory, asks again immediately. The process continues until the parent finally gets annoyed, with probability (1 − _p_), and shouts a definitive “No more candy!” Similarly, we continue to increase the height of the node until either our random number generator or our max threshold tells us to stop altogether.

Adding nodes to a skip list follows the same flow as searching for a target node: we progress downward and to the right, looking for a place to insert the new node. In fact, we can reuse the basic structure of the search for our insertion. We just need to track one additional piece of data: the last node at each level that could point to our new node.

```
SkipListInsert(SkipList: list, Type: key, Type: value):
    Integer: level = list.top_level
  ❶ SkipListNode: current = list.front
  ❷ Array: last = a size list.max_level + 1 array of SkipListNode pointers 
                  initially set to list.front for all levels.

  ❸ WHILE level >= 0:
      ❹ WHILE (current.next[level] != null AND
               current.next[level].key < key):
           current = current.next[level]
      ❺ last[level] = current
        level = level - 1
      
    SkipListNode: result = current.next[0]
  ❻ IF result != null AND result.key == key:
       result.value = value
       return

  ❼ Integer: new_level = pick a random level
  ❽ IF new_level > list.top_level:
        list.top_level = new_level
    SkipListNode: new_node = SkipListNode(key, value, new_level)

    Integer: j = 0
  ❾ WHILE j <= new_level:
         new_node.next[j] = last[j].next[j]
         last[j].next[j] = new_node
         j = j + 1
```

We begin at the top left corner of the list (`list.top_level` of `list.front`) ❶. With a pair of nested `WHILE` loops, we progress downward and to the right as we search for the correct place to insert the node. The outer `WHILE` loop ❸ iterates through the list’s levels, saving the last node seen at each level and then dropping down to the next level. The inner `WHILE` loop ❹ traverses the skip list, moving forward whenever another node along this level has a key less than our target.

Every entry in the array `last` starts off at `list.front`, indicating that the node is inserted in the front of the list ❷. We update `last` for each level each time we drop down from that level ❺, because we have seen that the key of the next node at this level is greater than or equal to the key to be inserted (or the next node is `null`) and therefore we will need to insert before that node. If we happen to find a matching key while traversing the skip list, we simply update the data for that key ❻. This means that, like our other data structures, our skip list implementation treats each key as unique.

When we find the correct point to insert the new node, we pick a random level for this node ❼. As we discussed previously, the probability distribution we use to select this height will have a significant impact on the structure and performance of the skip list. Since we cap the new level to be less than `list.max_level`, we avoid an invalid access to the `last` array. We check whether the selected level represents a new top level for the list and, if so, update `list.top_level` ❽.

Finally, the code inserts the new element by using a `WHILE` loop to update the pointers from the new node to point to the correct following node ❾. Then it updates each of the nodes listed in `last` to point to our new node. Here we can see the benefit of using the dummy node `front` (with the maximum height) to store the pointers to the beginning of the list. We can track and update this “front of list” position as we would any other node. This greatly simplifies the code.

[Figure 14-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c14.xhtml#figure14-5) shows how we would insert the key of 10 into an example skip list. The shaded nodes indicate which entries we traverse during the search.

![The last array points to front at height 3, node 1 at height 2, Node 1 at height 1, and node 9 at height 0.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c14/f14005.png)

Figure 14-5: Inserting the key of 10 into a skip list. The array last tracks which node comes before the inserted node.

By tracking the last node at each level _before_ the target, we are effectively tracking which node needs to point to the new node. As we traverse the list, we are noting down the locations where we will need to insert new links. At each level, we reach a point where the next key is larger than or equal to our new key, and we can exclaim, “I see where we need to insert the new node. It’s right after this one!” We then descend a level to continue our work. By the time the search phase reaches the bottom layer, we already have a full list of all the nodes whose forward pointers we need to adjust.

### Deleting Nodes

Deleting nodes from a skip list follows almost the same algorithm as inserting nodes. We first search the skip list for the deletion target while tracking the last node at each level that comes _before_ the target node. Once the search phase completes, we update this list of previous nodes to remove the node we are deleting.

```
SkipListDelete(SkipList: list, Type: target):
    Integer: level = list.top_level
  ❶ SkipListNode: current = list.front
    Array: last = a size list.max_level + 1 array of SkipListNode pointers 
                  initially set to list.front for all levels.

  ❷ WHILE level >= 0:
        WHILE (current.next[level] != null AND
               current.next[level].key < target):
           current = current.next[level]
        last[level] = current
        level = level - 1

  ❸ SkipListNode: result = current.next[0]
    IF result == null OR result.key != target:
        return

    level = result.height
    Integer: j = 0
  ❹ WHILE j <= level:
         last[j].next[j] = result.next[j]
         result.next[j] = null
         j = j + 1

  ❺ IF level == list.top_level:
        Integer: top = list.top_level
        WHILE top > 0 AND list.front.next[top] == null:
             top = top - 1
        list.top_level = top
```

The initial block of deletion code is identical to the code for insertion. We start at the top left of the list ❶. A pair of nested `WHILE` loops ❷ searches across each level until we hit a node with a key greater than or equal to `target` or the end of the list (`null`). At this point, we record the last node visited and drop down to the next level to continue our search. At the end of the search, we check that we’ve found a node whose `key` matches `target` ❸. Otherwise, there is no match in the skip list and thus nothing to delete.

To remove the identified node, we use a `WHILE` loop ❹ to simply link the next pointers for each node in the skip list’s `last` to point past the current node: `last[j].next[j] = result.next[j]` for all levels `j`. This splices our node out of the list. We also set `result.next[j]` to `null` because `result` is no longer in the list.

Finally, we need to check whether the skip list’s top level is still valid ❺. If we deleted the sole node with height of `top_level`, then `top_level` should be decreased to reflect the current maximum height. We can update `top_level` by proceeding down the `front` node and checking the `next` pointers until we find one that is not `null`. The last block of code in our deletion function updates the list’s top level if needed. It finds the first level where our dummy front node points to a valid data node. If the list is empty, we simply default to a top level of zero.

Again, we can visualize the initial search required for deletion in terms of how we are viewing the next node and maintaining the list of nodes to update. At each level, we identify the node to delete (if it exists) while still at the node immediately preceding it at that level. The next node’s key is greater or equal to the key we need to delete. We pause: “I’d better mark down this current node, because I will need to change its pointers to skip the deleted node.” We record the pointer to the current node in `last` and proceed to the next level. At the end of our journey, we have collected a full list of the nodes whose pointers need to be updated.

## Runtimes

The cost of search, insertion, and deletion operations will all depend on the nodes’ locations and distributions of heights. In an ideal case, the nodes at level _L_ would include every other node from level _L_ − 1. We drop half the nodes at each level and space them evenly apart. In this case, the behavior of a skip list mirrors that of binary search. We can prune half the search space by looking at the single node in the top layer. We then drop down a layer and cut the space in half again. Thus, in the best case, our performance will be logarithmic in terms of the number of entries.

The worst-case performance of a skip list is equivalent to that of a standard linked list—it is linearly proportional to the number of nodes. If every node in the list is the same height, our skip list is nothing more than a sorted linked list. We’re forced to scan through each node in sequence to find a given target.

Assuming that we use a good probability distribution of heights, such as that provided by the previously described Pugh’s original technique (with _p_ = 0.5), the _expected_ costs of insertions, deletions, and searches all scale logarithmically with the number of entries. Unlike worst-case cost, expected cost provides an estimate of how the data structure will perform on average. This puts the average performance of skip lists on par with binary search trees.

## Why This Matters

Skip lists are intended to be a simpler alternative to balanced search trees, as yet another dynamic data structure that enables efficient search. However, unlike the other algorithms we've applied to this task, including sorted arrays and binary search trees, skip lists rely on randomized structure to provide good performance. The _expected_ computational cost of our common operations—search, insertion, and deletion—is logarithmic in the size of the list.

This naturally poses a question: why should we trust the performance of our algorithm to randomized behavior? We could easily run into cases where the tall nodes are clustered, or the distribution of heights is too flat. Yet the same is true of binary search trees. If we insert and remove nodes in a suboptimal order, we can end up with a linked list of tree nodes. Where more sophisticated extensions of binary search trees can be employed to avoid this worst-case behavior, skip lists rely on randomization to avoid terrible behavior. In exchange, they use much simpler code. Thus, skip lists demonstrate how randomization can provide both a robust defense against bad data and simplicity in a data structure’s implementation.