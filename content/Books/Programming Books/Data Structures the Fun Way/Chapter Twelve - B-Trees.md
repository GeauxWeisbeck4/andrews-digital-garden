---
id: 01J9QFA72QJTWM2KYVKP8KTZJ6
modified: 2024-10-08T21:47:00-04:00
---
Chapter 11 showed how the cost of memory access can vary across mediums. In this chapter, we’ll discuss how this problem extends beyond accessing individual values to the cost of accessing new blocks of data, introducing a new data structure to handle this situation.

Computer science is full of instances where accessing data within a block is cheap, but retrieving a new block is comparatively expensive. A computer might read an entire block of information, known as a _page_, from the hard drive and store it in memory. In the days of floppy disk–based video games, you might see a message directing you to “Insert disk 5 of 7” or wait while the game loaded the next chunk of data from a CD. Similarly, online applications might download coherent blocks of data from a server across the internet, allowing you to start watching a video before you’ve downloaded the entire thing.

This chapter covers the _B-tree_, a self-balancing tree-based data structure that computer scientists Rudolf Bayer and Edward McCreight designed to account for the cost of retrieving new blocks of data. B-trees store multiple pieces of data in a single node, allowing us to pay the expensive retrieval cost exactly once to extract all those values. Once the node is in local memory, we can quickly access the values within it. The tradeoff is additional complexity when dealing with the nodes.

In the computational domain, we may run into this problem when trying to index a massive data set. Consider the index for a literally astronomical data set that contains pointers to images of every star, galaxy, nebula, comet, asteroid, and other celestial body ever observed. The data set is still larger than the index, but the index itself might need to reside across many blocks of slow storage. B-trees provide an inventive way of combining the indexing and keys while minimizing retrieval costs.

B-trees are also one example of how to define trees’ operations in such a way that they don’t become horribly imbalanced. As we will see later in the chapter, the B-tree always remains perfectly balanced with all the leaf nodes at exactly the same depth.

## B-tree Structure

B-trees apply the multi-way branching structure that we saw in tries or quadtrees to storing individual keys. Practically, this means that they allow internal nodes many more than two branches, so they’re practically bristling with pointers. They also need to store more than a single key in each node. B-tree nodes are packed with keys, allowing them to both track multi-way partitions and, more importantly, maximize the amount of data we can retrieve by fetching a single node.

We see the benefit of packing multiple items into a node in the everyday context of online shipping. We pay a cost for every box shipped, and these costs can add up quickly if we are shipping many small boxes. This is the equivalent of retrieving many small tree nodes from expensive storage. If we pack several items into the same box, however, we can reduce the cost by shipping them together. Similarly, B-trees reduce the cost of retrieving multiple keys by retrieving a block of them together.

Formally, we define the size of B-tree node with a size parameter _k_, which provides bounds on how many elements a non-root node can store. All non-root nodes store between _k_ and 2_k_ keys in sorted order. The root node is more flexible and is allowed to contain between 0 and 2_k_ keys. Like a binary search tree, internal nodes use the values of these keys to define the ranges for the branches. Internal nodes store pointers for each possible split point, before and after each key in the node, allowing for between _k_ + 1 and 2_k_ + 1 children for all internal nodes except the root node, which can have between 0 to 2_k_ + 1 children. These split points are conceptually the same as what we do in a binary search tree—they divide the space into keys that come before the split and keys that come after.

[Figure 12-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-1) shows an example of this structure. The node with keys 12, 31, and 45 defines four separate partitions: keys that come before 12, keys after 12 but before 31, keys after 31 but before 45, and keys after 45. The subtree containing 13, 17, and 26 is defined by two split points in the parent node. All the keys in that node must be greater than 12 since their node’s child pointer is to the right of key 12. Similarly, the keys must all be less than 31 since the pointer is to the left of key 31.

![The example B‐tree has 3 layers. The top node has a single key (51) and pointers to 2 children. The left child has keys 12, 31, and 45 and pointers to 4 children, one of which is a subtree containing 13, 17, 26.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12001.png)

Figure 12-1: An example B-tree

## Note

The notation used to describe the parameters of a B-tree varies from reference to reference, such as defining the size parameter as the maximum number of keys (or children) per node or varying how the children are indexed relative to the keys. When comparing implementations, it is important to account for these differences.

Picture this structure in the context of indexing the vast collection at the Comprehensive Museum of Fun and Informative Collectibles. In an attempt to allow a dynamic collection, we store the index entry to each item on a small paper card with the name, a brief description, and the location in our massive collectibles warehouse. We can fit nine hundred cards into a single binder, so, for our collection of a hundred million items, we need over a hundred thousand binders to store the entire index.

While we would like to store the entire index locally, we simply lack enough space in our office. Once we have requisitioned and retrieved a binder containing a slice of the index, we can browse through its entries relatively easily. However, asking for each new volume involves a trip to the archives and a requisition form.

In this organization scheme, each index card corresponds to a single entry in the B-tree where the name string is the key. The binders correspond to B-tree nodes, each with 900 pockets and thus holding a maximum of 900 keys. The entries within the binder are in sorted order, allowing us to search for a key with a linear scan or binary search. In addition, we keep one more piece of data in each of the binder’s pockets, a pointer to the binder containing entries between the current index card’s key and the key of the previous index card. As shown in [Figure 12-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-2), if we are looking for the target “Caffeine Unlimited Coffee Mug,” we would first scan past “Caffeine Ten Coffee Mug," which comes before our target in alphabetical order, and then hit “Jeremy’s Gourmet High-Caffeine Experience,” which comes after the target. At this point, we have passed the potential location for our target key and know that we need to search the binder before the current entry.

![Three entries from our collectibles index: Caffeine Ten Coffee Mug, Jeremy’s Gourmet High‐Caffeine Experience Coffee Mug, and Morning Zap Brand Coffee Mug . Each entry has two pointers, an item pointer and a binder pointer. The binder pointer for Jeremy’s Gourmet High‐Caffeine Experience Coffee Mug is shown as an arrow.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12002.png)

Figure 12-2: The binder pointer in our index cards indicates which binder to use to continue our search.

We store a single additional pointer at the very back of the binder that points to another binder containing only keys that come after the last key in the current binder. In total, our binder can contain up to 900 keys (with their pointers to the associated collectibles) and 901 pointers to other binders.

As with the other tree-based data structures, we define B-tree structure using both a top-level composite data structure and a node-specific data structure:

```
BTree {
    BTreeNode: root
    Integer: k
}

BTreeNode {
    Integer: k
    Integer: size
    Boolean: is_leaf
    Array of Type: keys
    Array of BTreeNodes: children
}
```

In this data structure and the examples below, we store and retrieve individual keys to keep the code simple. As with the other data structures we have introduced, in most cases it will be useful to store a composite data structure with both the key and a pointer to the key’s data, such as with the item pointers in [Figure 12-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-2).

One complication of the B-tree structure is that we store the keys and children in two differently sized arrays. This means that we need to define how a key at index _i_ maps to its adjacent child pointers. For any given index _i_, we can access the key at _keys_[_i_], but we also need to be able to access the node pointers before and after that key. We define the pointers such that the value of all keys in or below _children_[_i_] are less than _keys_[_i_] and greater than _keys_[_i_ _−_ 1] (if _i_ > 0), as shown in [Figure 12-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-3).

![The keys array contains 12, 31, and 45. The children array has 4 pointers: keys less than 12, keys less than 31 and greater than 12, keys less than 45 and greater than 31, and keys greater than 45.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12003.png)

Figure 12-3: A mapping from the entries in the keys array to the corresponding elements in the children array

By definition, B-trees are balanced data structures. Every leaf node is exactly the same depth from the root. In a later section, we’ll show how this structure is maintained by updating nodes as we insert and delete new keys.

## Searching B-trees

We search the B-tree using the same general procedure that we use for all tree-based data structures: we start at the top of the tree and work our way down until we find the key of interest. The main difference between the B-tree and a binary search tree is that we might need to check more than one key per node. We scan along the keys in each node until we either find the target key or find a key with a value larger than our target. In the latter case, if we are at an internal node, we drop down to the appropriate child and continue the search, as follows:

```
BTreeSearch(BTree: tree, Type: target):
    return BTreeNodeSearch(tree.root, target)

BTreeNodeSearch(BTreeNode: node, Type: target):
    # Search the node's key list for the target.
    Integer: i = 0
  ❶ WHILE i < node.size AND target >= node.keys[i]:
      ❷ IF target == node.keys[i]:
            return node.keys[i]
        i = i + 1

    # Descend to the correct child.
  ❸ IF node.is_leaf:
        return null
  ❹ return BTreeNodeSearch(node.children[i], target)
```

The code for this search starts by scanning through the keys stored at the current node using a `WHILE` loop ❶. The loop continues until it hits the end of the key list (`i == node.size`) or hits a key larger than the target (`target < node.keys[i]`). The code checks whether it has found a matching key in the current node, and, if so, returns that key ❷. While the example code uses a linear scan to search the node for the purpose of simplicity in this example, we could also use binary search for better efficiency.

If the code does not find a match in the current node and the current node is a leaf, there is no match in the tree, and the code returns `null` ❸. Otherwise, the code recursively explores the correct child node. It can access the correct child directly with the loop iterator `i` ❹, because the loop stops when either `i` represents the last child or `key[i] > target`.

Consider the example of searching the B-tree shown earlier in [Figure 12-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-1) for key 17. In the root node, we check the first key (51) and see that it is greater than 17, so we drop down a level using the first child pointer. At the next level, we check two keys: 12 is less than our target, so we proceed past it; 31 is greater than our target, so we drop down to the child node whose keys come before 31 using the second child pointer. This process continues at the leaf node. [Figure 12-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-4) illustrates this search, indicating with gray the array bins we have accessed and compared.

![The search of the B‐tree visits the key 51 in the top node, 12 and 31 in the second‐level child, and 13 and 17 in the third‐level child.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12004.png)

Figure 12-4: An example search of a B-tree. The shaded cells are the ones checked by the algorithm.

We should consider how searching the keys within a node impacts the runtime: instead of dropping down to the next level after a single comparison, we might now have to perform multiple comparisons per node. This is an acceptable tradeoff for two reasons. First, remember the B-tree is optimized to reduce the number of nodes fetched. In comparison, the data accesses within a node are expected to be relatively cheap because they are happening in local memory and do not require us to fetch another block of data from the expensive storage. Second, and equally important, the branching structure of the B-tree still provides ample opportunities for pruning. Each comparison still eliminates entire subtrees. And, like the current node, each skipped node may contain up to 2_k_ keys and 2_k_ + 1 children.

Returning to our collectibles example, consider a search for a particular collectible. We start at the root binder. Since the keys are stored in alphabetical order, we can rapidly skim down the rows until we find our desired key or pass where it should be. If we don’t see our desired key, we know it isn’t in this binder. We mumble a few choice words about the unfairness of finite storage space and note the first key we encounter that comes after our target key has an item pointer reading “Binder #300.” We mumble a few more complaints and ask the archivist for binder #300.

Let’s contrast this storage approach with what happens if we instead store all our index cards in sorted order. Binder #1 contains the first set of cards _Aa_ through _Ab_, binder #2 contains _Ac_ through _Ad_, and so forth. This could work well for a static data set. We could perform a binary search over the binders, requesting the middle binder in our current range each time and limiting ourselves to a logarithmic number of requests. However, this approach begins to break down as we add or remove cards. Binders become overfull, requiring us to shift cards from one binder to the next. Updates to our collection might require cascading updates to many binders as cards must be shifted over. In the worst case, if we pack our binders full, we could end up needing to access every binder in our index. As we will see next, B-trees structures facilitate dynamic changes to the data set.

## Adding Keys

Adding keys to a B-tree is more complex than adding to the tree-based data structures we’ve previously considered. In this case, we need to keep the structure balanced and limit the number of keys stored in each node (between _k_ and 2_k_). There are two approaches for handling full nodes. First, we could split as we proceed down the tree, making sure we never call insertion on a full node. Second, we can temporarily insert into a full node (allowing it to be overfull) and then split it on the way back up. We’ll explore the later approach, which results in a two-stage algorithm for inserting new keys.

To perform insertions, we first proceed down the tree, searching for the position to insert the new key. Second, we return back up the tree, splitting nodes that have become overfull. Each split increases the branching factor of a node, but not necessarily the height. In fact, the only time we increase the height of the tree is when we split the root node itself. Because we only increase the height by splitting the root node (adding a depth of 1 to every leaf simultaneously), we can guarantee that the tree always remains balanced.

### The Addition Algorithm

During the first stage of the algorithm, we recursively descend the tree, searching for the correct location to insert the new key. If we find a matching key along the way, we can update that key’s data. Otherwise, we proceed down to a leaf node, where we insert the key into our array.

We start off by defining a simple helper function `BTreeNodeAddKey` to insert a key into a non-full node. For convenience, we also take a pointer to a child node (representing the child _after_ the new key) so that we can reuse this function when splitting nodes. If we are at a leaf node, which doesn’t store pointers to children, this `next_child` pointer is ignored.

```
BTreeNodeAddKey(BTreeNode: node, Type: key, 
                BTreeNode: next_child):
  ❶ Integer: i = node.size - 1
    WHILE i >= 0 AND key < node.keys[i]:
        node.keys[i+1] = node.keys[i]
        IF NOT node.is_leaf:
            node.children[i+2] = node.children[i+1]
        i = i - 1

    # Insert both the key and the pointer to the child node.
  ❷ node.keys[i+1] = key
    IF NOT node.is_leaf:
        node.children[i+2] = next_child
  ❸ node.size = node.size + 1
```

The code starts at the _end_ of the `keys` array (index `node.size – 1`) and proceeds toward index 0 using a `WHILE` loop ❶. At each step, it checks whether the new key should be inserted here and, if not, shifts the current element of both `keys` and `children` back one space. The loop terminates when it has gone one step past the correct location, which might be the start of the array. Once we have found the correct location for the new key, we have already moved that and the following elements out of the way. We can directly insert the new key and child ❷. The code finishes by adjusting the size of the node to account for the insertion ❸.

Here we may gasp in dismay at the cost of linearly shifting down items in the array to make room for our new element, as shown in [Figure 12-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-5). This is everything we warned about in Chapter 3. But remember, we are trading off these (bounded) linear costs to minimize node accesses. We are willing to put up with the hassle of moving down the cards within our binder to minimize future requisitions to other binders.

![An array for four spots. The key 26 is inserted into the second spot in the array, and the following two keys 31 and 45 are each shifted back one position.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12005.png)

Figure 12-5: Shifting the elements over to insert 26 in `BTreeNodeAddKey`

We require a few more helper functions to handle the case where a node fills up. Remember that we are restricted to at most 2_k_ elements per node—any more than that means we need to split the node. First, we start with a simple accessor function `BTreeNodeIsOverFull`, which returns a Boolean indicating whether the node contains more than 2_k_ items:

```
BTreeNodeIsOverFull(BTreeNode: node):
    return node.size == (2 * node.k + 1)
```

This is equivalent to checking if we have used up all the pockets in our binder.

We also add a second helper `BTreeNodeSplit`, which takes a node and the index of a child and splits that child. Everything before that index is retained in the original child. Everything after that index is cleared from the child and added to a newly created sibling node. The key at the index itself is cleared from the child and added to the current (parent) node.

```
BTreeNodeSplit(BTreeNode: node, Integer: child_index): 
  ❶ BTreeNode: old_child = node.children[child_index]
    BTreeNode: new_child = BTreeNode(node.k)
    new_child.is_leaf = old_child.is_leaf

    # Get the index and key used for the split.
  ❷ Integer: split_index = Floor(old_child.size / 2.0)
    Type: split_key = old_child.keys[split_index]    

    # Copy the larger half of the keys (and their children) to 
    # new_child and erase them from old_child.
    Integer: new_index = 0
    Integer: old_index = split_index + 1
  ❸ WHILE old_index < old_child.size:
        new_child.keys[new_index] = old_child.keys[old_index]
        old_child.keys[old_index] = null

        IF NOT old_child.is_leaf:
            new_child.children[new_index] = old_child.children[old_index]
            old_child.children[old_index] = null
        new_index = new_index + 1
        old_index = old_index + 1

    # Copy the remaining child (after the last key).
  ❹ IF NOT old_child.is_leaf:
        new_child.children[new_index] = old_child.children[old_child.size]
        old_child.children[old_child.size] = null

    # Remove the key at index and add it to the current node.
  ❺ old_child.keys[split_index] = null
  ❻ BTreeNodeAddKey(node, split_key, new_child)

    # Update the sizes of the nodes.
  ❼ new_child.size = old_child.size - split_index - 1
    old_child.size = split_index
```

The code for `BTreeNodeSplit` starts by looking up the node to split (`old_child`) and creating a new (empty) sibling node (`new_child`) ❶. This node will be at the same level as the child to split, so we copy over the value of `is_leaf`. Next the code determines what index and key to use as a split point for `old_child` ❷. The code then uses a `WHILE` loop to copy everything after `split_index` from both the `keys` and `children` in `old_child` to the corresponding arrays in `new_child` ❸. The code uses a pair of indices to capture the index of the old location (`old_index`) and the corresponding new location (`new_index`). At the same time, the code removes the elements from `old_child`’s arrays by setting the entries to `null`. Because the `children` array is one element longer, we need to copy that last element separately ❹. Finally, we remove the key at `split_index` ❺, add both `split_key` and the new child pointer to the current node ❻, and set both the children’s sizes ❼.

Let’s view this operation in the context of our collectibles storage index as shown in [Figure 12-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-6). When a binder reaches capacity, we repartition its contents into two binders. First, we buy a new empty binder. This sibling will store approximately half the contents of the overfull binder. Second, we carefully relocate the back half of the overfull binder’s contents into the new binder, preserving the sorted ordering. Third, we remove the single index card whose key lies between the keys in each binder and insert it in the parent binder in order to indicate the divide between the two child binders. The previously overfull child binder will contain cards whose keys come before this split, and the new binder will contain cards whose keys come after this split.

![A row  with three entries: Caffeine Ten Coffee Mug, Jeremy’s Gourmet High‐Caffeine Experience Coffee Mug, and Morning Zap Brand Coffee Mug. An arrow indicates that the card on the left (and all preceding cards) stays in the binder. A second arrow indicates the middle card goes to the parent binder. A third arrow indicates that the card on the right and all following cards goes to the new binder.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12006.png)

Figure 12-6: We repartition the binder by splitting it on the key of the middle card.

Given these helpers, we can now define an insertion function that performs the recursive search and the subsequent addition. We perform the addition at the leaf node. As the recursion returns up the tree, we check whether the recently accessed child node is full and thus needs to be split.

```
BTreeNodeInsert(BTreeNode: node, Type: key):
    Integer: i = 0
  ❶ WHILE i < node.size AND key >= node.keys[i]:
      ❷ IF key == node.keys[i]:
            Update data.
            return
        i = i + 1

    IF node.is_leaf:
      ❸ BTreeNodeAddKey(node, key, null)
    ELSE:
      ❹ BTreeNodeInsert(node.children[i], key)
      ❺ IF BTreeNodeIsOverFull(node.children[i]):
          ❻ BTreeNodeSplit(node, i)
```

The code starts by finding the correct location for `key` in the `keys` array ❶. The `WHILE` loop iterates through the array until it hits the end of the key list (`i == node.size`) or hits a key that is larger than the target (`key < node.keys[i]`). If the code finds an exact match, it updates any data for this key and returns ❷. Otherwise, it needs to insert new data.

If the key is being inserted into a leaf, the code uses the `BTreeNodeAddKey` function ❸, which shifts the array elements over and adds the new key to the correct location. If the key is being inserted into an internal node, the index `i` provides the pointer of the correct child for insertion. The code recursively inserts into that child ❹, then checks whether it broke the properties of a B-tree (specifically the size of nodes falling between _k_ and 2_k_) with the insertion ❺.

The code breaks the B-tree property if it inserts too many elements into a node. We can use our helper function `BTreeNodeIsOverFull` to check if the recently modified node has too many elements. The code conducts this check from the parent node, so we can keep the logic of repairing the B-tree simple. It uses `BTreeNodeSplit` to split the overfull child into two nodes ❻. In the process of this insertion, we might break the current node when inserting the new separating key, but that’s okay; we’ll take care of it when we return to this node’s parent.

We use a little extra storage in order to simplify the code. The code allows a node to temporarily overfill, storing 2_k_ + 1 keys and 2_k_ + 2 children while waiting for its parent node to call `BTreeNodeSplit`. We can create this buffer by simply allocating large enough arrays for the keys and children.

We can think of the first phase of the code as receiving a new coffee mug for our collection. We create an index card for the mug and insert it into our indexing binders. We start at the root binder and search for the location to put the card. During our search, we follow the appropriate pointers to child binders. Once we end up at a leaf binder, with no children indicated on the index cards, we add the new card there. We check whether the binder is now (over)full and, if it is, start repartitioning its contents. Afterward, we return the binders to storage in the reverse order that we requisitioned them. If we just split a binder and thus transferred a new card to its parent, we also check whether we need to split the parent binder. This process continues until we’ve returned to the root binder.

We need to define one additional special case for the root node. Remember that splitting the root node is the only way we are allowed to increase the height of the tree. We need to define a wrapper function to do exactly that. Luckily, we can reuse our previous helper functions:

```
BTreeInsert(BTree: tree, Type: key): 
  ❶ BTreeNodeInsert(tree.root, key)

  ❷ IF BTreeNodeIsOverFull(tree.root):
      ❸ BTreeNode: new_root = BTreeNode(tree.k)
        new_root.is_leaf = False
        new_root.size = 0

      ❹ new_root.children[0] = tree.root
      ❺ BTreeNodeSplit(new_root, 0)
      ❻ tree.root = new_root
```

The code starts by inserting the `key` into the root node using `BTreeNodeInsert` ❶. This function recursively descends the tree, finds the correct location to insert the new key, and returns through the levels fixing the broken B-tree property at all but the root node. Then the code checks if the root node has too many elements by using `BTreeNodeIsOverFull` on the root node ❷. If the root node has too many elements, the code adds a level to the tree by creating a new empty root node ❸, assigning the old root to be the first child of the new root ❹, splitting this (overfull) child ❺, and updating the tree’s root ❻. After a split, the new root node will contain exactly one key and two children.

In the process of inserting a key, we complete a single round trip from the root node to a leaf node and back. The number of nodes we need to access (and modify) is thus proportional to the depth of the tree. Since our B-tree always remains balanced, with all leaf nodes at the same depth, and the branching factor of all non-root, internal nodes is at least _k_ + 1, the node retrievals scales logarithmically in _N_. The total work also includes linear operations within the node, such as copying or shifting keys, so the total work required scales proportional to _k_ × log_k_(_N_).

### Examples of Adding Keys

Let’s consider a few examples to better understand the functions we just covered. First, take the simplest case, shown in [Figure 12-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-7), of adding a key to leaf node that will not be overfull. Suppose _k_ = 2, where our non-root nodes can contain between 2 and 2_k_ = 4 items. If we add the key 30 to the subtree in [Figure 12-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-7)(a), we simply proceed down to the leaf node and add the new key in the correct part of the array with the `BTreeNodeAddKey` helper function. Since the leaf has four elements, we do not need to split it. We get the subtree shown in [Figure 12-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-7)(b).

![Figure A shows a subtree with one internal node and four leaf nodes. The second leaf node has keys 13, 17, and 26. Figure B shows the same tree but with the key 30 added to the end of the second child node. That node now has keys 13, 17, 26, and 30.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12007.png)

Figure 12-7: Inserting the key 30 into a non-full B-tree leaf (a) results in a leaf node with four elements (b).

The logic becomes more complex as we fill up nodes. Consider the example shown in [Figure 12-8](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-8)(a) of adding the key 29 to the same tree. After inserting the new key in [Figure 12-8](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-8)(b), the leaf node is overfull. We handle this by identifying the split point of the overfull node (key = 26) and promoting that to the parent node. We then use the helper function `BTreeNodeSplit` to divide the leaf into two siblings as shown in [Figure 12-8](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-8)(c). If the promotion of the middle element fills up the internal node, we need to split that as well.

![Figure A shows a subtree with one internal node and four leaf nodes. The second leaf node has keys 13, 17, 26, and 30. In figure B, the key 29 has been added to the second child, and the node is overfull with keys 13, 17, 26, 29, and 30. In Figure C, the second child has been split into two nodes (13, 17) and (29, 30), and the key 26 has been promoted to its parent.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12008.png)

Figure 12-8: Inserting the key 29 into an already-full leaf node (a) gives the leaf too many elements (b). We must split the overfull leaf to restore the B-tree conditions (c).

Finally, consider what happens if our splits propagate all the way back to the root node. Suppose that, after an insertion, the root node itself overfills as shown in [Figure 12-9](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-9)(a). We solve this problem in [Figure 12-9](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-9)(b) by splitting the root node and creating a new level for the tree. The new root node contains exactly one element, the middle key of the previous root node. Note that, unlike all the other nodes, the root node is allowed to have less than _k_ items. In fact, every time we split the root node, we create a new root with exactly one item.

As the examples in this section show, the modifications to the B-tree are limited to only those nodes explored during the initial search for the insertion location. Since we do not need to update or repair other branches, the total number of modified nodes is limited by the depth of the tree and thus scales proportional to log_k_(_N_).

![Figure A shows a B‐tree with an overfull root node containing the keys 12, 31, 51, 61, and  86. The root node has five children, which are all leaves. Figure B shows the resulting tree after the split, with a new root node with key 51 and two children. The root's left child contains keys 12 and 31. The root’s right child contains keys 61 and 86. The tree has the same five leaves.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12009.png)

Figure 12-9: When the root node of a B-tree becomes overfull (a), we split it into two siblings and promote the middle element to the new root node (b).

## Removing Keys

Removing keys follows a similar approach to adding keys. We again need to keep the structure balanced and limit the number of keys stored in each node (between _k_ and 2_k_). This results in a multi-stage algorithm for deleting keys. First, we proceed down the tree as though we were searching for the key. Once we find it, we delete the key. Finally, we return back up the tree checking for and fixing nodes with too few keys. Since we never remove a node except an empty root node (decreasing the depth of all the leaves by one), we again guarantee that the tree always remains balanced.

### Fixing Under-full Nodes

When we remove keys from a B-tree, we run the risk of nodes dropping below the minimum of _k_ keys. We can check this condition with a simple helper function:

```
BTreeNodeIsUnderFull(BTreeNode: node):
    return node.size < node.k
```

Depending on the structure of the B-tree, there are two different approaches we may need to use to fix an under-full node, both of which I’ll discuss in this section. Each approach relies on augmenting the node’s keys with keys from an adjacent sibling. In the first case, we directly merge two small sibling nodes into a single node. In the second case, we transfer a key from a larger sibling to the under-full node. Which function we use depends on how many keys in total the two siblings have. Both of these helper functions are called from the parent node with the index for the key that separates the adjacent sibling nodes.

The merge operation takes two adjacent sibling nodes, along with the key separating them, and concatenates them into a single large child node. As such, it requires that the combined number of keys in the two siblings be _less_ than 2_k_ so that the new child is guaranteed to be valid. [Figure 12-10](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-10) shows this procedure, with [Figure 12-10](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-10)(a) depicting a subtree before the merge operation, where the middle child has a single key. [Figure 12-10](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-10)(b) shows the same subtree after the merge.

![Figure A shows a node with three leaves and a single internal node. The internal node has keys 26 and 31. The middle leaf has only a single key 29, and the right‐most leaf has two keys 32 and 42. Figure B shows a node with two leaves. The internal node now has a single key, 26. The right‐most node now has keys 29, 31, 32, and 42.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12010.png)

Figure 12-10: The merge operation on B-tree nodes

[Listing 12-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#listing12-1) shows the code for merging two adjacent siblings.

```
BTreeNodeMerge(BTreeNode: node, Integer: index):
  ❶ BTreeNode: childL = node.children[index]
    BTreeNode: childR = node.children[index + 1]

    # Copy over the parent's key and the right child's first child pointer.
  ❷ Integer: loc = childL.size
    childL.keys[loc] = node.keys[index]
    IF NOT childL.is_leaf:
        childL.children[loc + 1] = childR.children[0]
    loc = loc + 1

  ❸ # Copy over the right child's keys and children.
    Integer: i = 0
    WHILE i < childR.size:
        childL.keys[loc + i] = childR.keys[i]
        IF NOT childL.is_leaf:
            childL.children[loc + i + 1] = childR.children[i + 1]
        i = i + 1
    childL.size = childL.size + childR.size + 1

    # Remove the key from the current node.
    i = index
  ❹ WHILE i < node.size - 1:
        node.keys[i] = node.keys[i + 1]
        node.children[i + 1] = node.children[i + 2]
        i = i + 1
    node.keys[i] = null
    node.children[i + 1] = null
    node.size = node.size – 1
```

Listing 12-1: Code for merging two child nodes

The code appends the keys from the right child and the separating key from the parent onto the left child. It starts by retrieving both child nodes, which we call `childL` and `childR` for left and right respectively ❶. By definition, any key in `childL` is less than the separating key, and any key in `childR` is greater than the separating key. The code then appends the separating key from the parent and the first child pointer from the right child to the end of the left child ❷. It uses a `WHILE` loop to copy the remaining keys and pointers from the right child ❸. It also updates the left child’s size. At this point, it has successfully created the merged node out of the two children. The merged child’s pointer is stored in `node.children[index]`.

The code finishes by cleaning up the parent node ❹. It removes the previous separating key and the pointer to the right child by shifting the subsequent keys and pointers over, setting the final bins to `null`, and updating the current node’s size.

In the process of merging two nodes, we are taking a key from their parent. This could leave the parent node with less than _k_ keys, and our repairs would need to continue at the next higher level of the tree.

This process is directly analogous to merging binders in our storage indexing example. If an index binder contains too few keys, it is a waste of space and requisition time. We wouldn’t want to requisition a binder with a single index card. Merging binders consists of taking the cards from one child binder, along with the separating card from the parent binder, and putting them in the other child binder in the correct order. Since we have already requisitioned the parent and one child (and thus have them in local memory), we can do this merge quickly with only a single additional requisition for the other child.

The second approach to fixing an under-full node is to shift one of the keys (and potentially children) from its adjacent sibling. This works only when the sibling can afford to lose a key and thus applies to cases where the combined number of keys of the siblings must be at least 2_k_. While we could merge and optimally resplit the adjacent siblings, for illustrative purposes we use a simpler approach of transferring only one key. Since we only ever remove a single key from a node during deletion or the merge operation, transferring a single key is sufficient to fix our under-full node.

However, as shown in [Figure 12-11](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-11), we can’t just take a key from one child and give it to the other. The separating key in the parent node enforces the bounds of the split. Instead, we do a two-stage transfer. First, we transfer the current separating key from the parent to the under-full node. Second, we replace the separating key in the parent node with a key from the other sibling.

![Figure A shows a node with three leaves and one internal node. The internal node has keys 26 and 31. The middle leaf has only a single key 29, and the right‐most leaf has three keys 32, 42, and 45. Figure B shows the same subtree. The internal node now has keys 26 and 32. The middle leaf now has two keys 29 and 31, and the right‐most node has two keys 42 and 45.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12011.png)

Figure 12-11: The transfer left operation on B-tree nodes

As shown in [Listing 12-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#listing12-2), we break the code into two helper functions, one to transfer a key from the right child to the left child and one to transfer the other way. The code to transfer a key from the right child to the left child transfers two keys: one from the right child to the parent and one from the parent to the left child.

```
BTreeNodeTransferLeft(BTreeNode: node, Integer: index):
  ❶ BTreeNode: childL = node.children[index]
    BTreeNode: childR = node.children[index + 1]
    Type: middle_key = node.keys[index]

  ❷ node.keys[index] = childR.keys[0]
  ❸ childL.keys[childL.size] = middle_key
    IF NOT childR.is_leaf:
        childL.children[childL.size + 1] = childR.children[0]
    childL.size = childL.size + 1

  ❹ Integer: i = 0
    WHILE i < childR.size - 1:
        childR.keys[i] = childR.keys[i + 1]
        IF NOT childR.is_leaf:
            childR.children[i] = childR.children[i + 1]
        i = i + 1
  
  ❺ childR.keys[i] = null
    IF NOT childR.is_leaf:
        childR.children[i] = childR.children[i + 1]
        childR.children[i + 1] = null
  ❻ childR.size = childR.size – 1
```

Listing 12-2: Code for transferring a key and child pointer to an under-full node from its right-hand sibling

The code starts by retrieving the two adjacent siblings and the separating key ❶. It moves the first key from the right-hand child to replace the previous separating key ❷. It adds the previous separating key from the parent (`middle_key`) and the first child pointer from the right-hand child to the end of the arrays in the left-hand child ❸. Both the left-hand child and parent are now updated. The code then cleans up the right-hand child. It uses a `WHILE` loop to shift over the remaining elements ❹, marks the now empty spots as `null` ❺, and adjusts the size ❻.

The code for transferring a key from the left child to the right child is similar to that shown in [Listing 12-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#listing12-3). The two-key transfer works in the opposite direction: one from the left child to the parent and one from the parent to the right child.

```
BTreeNodeTransferRight(BTreeNode: node, Integer: index):
  ❶ BTreeNode: childL = node.children[index]
    BTreeNode: childR = node.children[index + 1]
    Type: middle_key = node.keys[index]

    # Make space in childR for the new key and pointer.
  ❷ Integer: i = childR.size - 1
    WHILE i >= 0:
        childR.keys[i+1] = childR.keys[i]
        IF NOT childR.is_leaf:
            childR.children[i+2] = childR.children[i+1]
        i = i – 1
    IF NOT childR.is_leaf:
        childR.children[1] = childR.children[0]

  ❸ childR.keys[0] = middle_key
    IF NOT childR.is_leaf:
        childR.children[0] = childL.children[childL.size]
    childR.size = childR.size + 1

  ❹ node.keys[index] = childL.keys[childL.size – 1]

  ❺ childL.keys[childL.size - 1] = null
    IF NOT childL.is_leaf:
        childL.children[childL.size] = null
    childL.size = childL.size – 1
```

Listing 12-3: Code for transferring a key and child pointer to an under-full node from its left-hand sibling

The code again starts by retrieving the two adjacent siblings and the separating key ❶. The code then shifts over the keys and children in the right-hand node to make room for the new addition ❷. It appends the previous separating key from the parent (`middle_key`) and the last child pointer from the left child to the _beginning_ of the right-hand node ❸, increasing its size by 1. The code then moves the last key in the left-hand child to replace the separating key in the parent ❹. The code completes by cleaning up the left-hand child by marking the now empty entries `null` and updating the size ❺.

Unlike the merge operation, neither transfer operation reduces the number of keys in the parent. Thus, we do not need to perform repairs at higher levels of the tree. The physical corollary of these transfer operations is requesting a sibling storage binder and shifting two index cards between the two children and the parent. We take the intermediate card (that falls between the range of the two binders) from the parent and add it to the less full child binder. We replace this card in the parent with the appropriate one from the child that has more cards.

We can encapsulate all three of these repair functions as well as the logic to choose them into a helper function that takes in the current node and the index of the under-full child:

```
BTreeNodeRepairUnderFull(BTreeNode: node, Integer: child):
  ❶ IF child == node.size:
        child = child - 1
  ❷ Integer: total = (node.children[child].size + 
                      node.children[child + 1].size)
 
    IF total < 2 * node.k:
      ❸ BTreeNodeMerge(node, child)
        return

  ❹ IF node.children[child].size < node.children[child + 1].size:
        BTreeNodeTransferLeft(node, child)
    ELSE:
        BTreeNodeTransferRight(node, child)
```

To know which repair strategy to employ, the code needs to find an adjacent sibling and check the two children’s total number of keys. Here, for illustrative purposes, we use a simplistic strategy of always using the next child (`child + 1`) as the sibling unless we are repairing the last child in the array ❶. If we are repairing the last child in the array, we use the previous child for its sibling. The code checks the total count of keys in these two child nodes ❷. If the number of keys is small enough (under 2_k_), it merges those nodes with the `BTreeNodeMerge` function ❸. Otherwise, if the nodes have 2_k_ or more keys, the code uses either `BTreeNodeTransferLeft` or `BTreeNodeTransferRight` to move a single key to the smaller node ❹.

### Finding the Minimum Value Key

We use one more helper function as part of the deletion operation—code to find and return the minimum key at or below a given node. This code, in [Listing 12-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#listing12-4), can also be useful in its own right, such as for computing the bounds of the keys in the B-tree.

```
BTreeNodeFindMin(BTreeNode: node):
  ❶ IF node.size == 0:
        return null
  ❷ IF node.is_leaf:
        return node.keys[0]
    ELSE:
      ❸ return BTreeNodeFindMin(node.children[0])
```

Listing 12-4: Code to find the minimum key at or below a given node

The code consists of three possible conditions. If the node is empty, the code returns `null` to indicate that there is no minimum key there ❶. This should occur only in an empty root node, as all other nodes will have at least _k_ keys. If the node is a non-empty leaf, the code returns the first (and thus minimum) key in the node’s array ❷. Finally, if the node is internal, the code recursively checks the first child ❸.

### The Removal Algorithm

We start the description of the deletion algorithm with the top-level wrapper function. This function is relatively simple. It calls the recursive deletion function using the tree’s root node.

```
BTreeDelete(BTree: tree, Type: key):
    BTreeNodeDelete(tree.root, key)

    IF tree.root.size == 0 AND NOT tree.root.is_leaf:
        tree.root = tree.root.children[0]
```

Just as we added a level only when we split the node, we remove a level from the tree only when the root node becomes empty. If the B-tree is not completely empty, the empty root node will still have a single valid child in array position 0. We use this child to replace the former root node.

The core deletion algorithm recursively descends the tree, searching for the key to delete. Since we might reduce the number of keys below the required _k_, we need to know check whether the modified child is now under-full and, if so, repair it.

```
BTreeNodeDelete(BTreeNode: node, Type: key):
  ❶ Integer: i = 0
    WHILE i < node.size AND key > node.keys[i]:
        i = i + 1

    # Deletion from a leaf node.
    IF node.is_leaf:
        IF i < node.size AND key == node.keys[i]:
          ❷ WHILE i < node.size - 1:
                node.keys[i] = node.keys[i + 1]
                i = i + 1
            node.keys[i] = null
            node.size = node.size - 1
        return

    # Deletion at an internal node.
    IF i < node.size AND key == node.keys[i]:
      ❸ Type: min_key = BTreeNodeFindMin(node.children[i+1])
        node.keys[i] = min_key

      ❹ BTreeNodeDelete(node.children[i+1], min_key)
        IF BTreeNodeIsUnderFull(node.children[i+1]):
            BTreeNodeRepairUnderFull(node, i+1)
    ELSE:
      ❺ BTreeNodeDelete(node.children[i], key)
        IF BTreeNodeIsUnderFull(node.children[i]):
            BTreeNodeRepairUnderFull(node, i)
```

The code starts by searching for the key to delete in the current node by scanning across the array of keys ❶. If there is a matching key in this node, the `WHILE` loop terminates such that `i` is the index matching the key.

The code then considers the leaf case. If the node is a leaf and the key is found, the code deletes it by shifting over the keys ❷. The code also sets the last element to `null` and updates the size. The code doesn’t need to change the child pointers because they are not set for leaf nodes. If the node is a leaf and the key is not found, then the code simply returns.

The code next handles the case of internal nodes. There are two cases to consider: the key is in the node, or it is not. If the code finds the key in the internal node, it replaces the key with the key that immediately _follows_ the target key in sorted order ❸. The code finds this subsequent key using `BTreeNodeFindMin` from [Listing 12-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#listing12-4), called on the child node immediately after the target key. The code deletes this following key from the child’s subtree by calling `BTreeNodeDelete` ❹. The code then checks whether the child node is under-full and, if so, fixes it.

If the target key is not in an internal node, then the code recursively calls `BTreeNodeDelete` on the appropriate child ❺. Again, it needs to check whether that child node is now under-full and, if so, fix it.

As with insertion, our goal is to limit the number of nodes retrieved during this operation. Deletion will make at most a single pass from the root node to a leaf. Even if we delete from an internal node, the subsequent replacement and deletion operations still only continue the trek to a single leaf. We pay one additional requisition whenever we repair a node to retrieve the under-full node’s sibling.

### Examples of Removing Keys

Let’s look at a few examples of the removals that we just covered. First, take the simplest case, shown in [Figure 12-12](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-12), of removing a key from a leaf node with more than _k_ + 1 keys. Suppose _k_ = 2, where our non-root nodes can contain between 2 and 2_k_ = 4 items. If we remove key 5 from the subtree in [Figure 12-12](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-12)(a), we simply proceed down to the leaf node and remove the key in the array. Since the resulting leaf has three elements, we do not need to repair it. We get the subtree shown in [Figure 12-12](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-12)(b).

![Figure A shows a subtree with one internal node and five leaf nodes. The first leaf has keys 1, 3, 5, and 6. Figure B shows the same tree as Figure A with the key 5 deleted from the middle of the first leaf node. The node now contains keys 1, 3, and 6.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12012.png)

Figure 12-12: Deleting the key 5 from a B-tree leaf (a) results in a leaf node with three elements (b).

Next, we consider the case of removing a key from an internal node without needing repairs as shown in [Figure 12-13](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-13). If we remove key 45 from the subtree in [Figure 12-13](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-13)(a), we find that key in an internal node. To remove it, we replace it with the next key in order, which is 47. Since the resulting nodes all have at least two elements, we do not need to perform any repairs. We get the subtree shown in [Figure 12-13](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-13)(b).

![Figure A shows a subtree with one internal node and five leaf nodes. The internal node contains keys 12, 26, 31, and 45. The fifth (right‐most) leaf contains keys 47, 48, and 49. Figure B shows the same tree as Figure A with the key 45 deleted from the internal node. The key 47, which previously resided in the right‐most node in Figure A, replaces the key 45. The fifth leaf node has keys 48 and 49.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12013.png)

Figure 12-13: Deleting the key 45 from an internal B-tree node (a) results in taking a key from one of the children (b).

Finally, we consider the different cases where removing a key requires us to repair an under-full node. [Figure 12-14](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-14) shows a case where we can merge two nodes. We start by deleting the key 32 in [Figure 12-14](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-14)(a). [Figure 12-14](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-14)(b) shows the keys that we use for the merge operation: the keys in the under-full node, the keys in its right-hand adjacent sibling, and the key in the parent separating the two. [Figure 12-14](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-14)(c) shows the repaired tree. The new child node has four keys, and the previous parent has three.

![Figure A shows a subtree with one internal node and five leaf nodes. The internal node contains keys 12, 26, 31, and 45. The fourth leaf contains keys 32 and 42. The fifth leaf contains keys 47 and 48. In Figure A, the key 32 is removed from a leaf node, leaving a single key 42. Figure B shows a dashed line around the right‐most two children and the separating key 45. Figure C shows the final tree with the merged child. The internal node now has four children and contains the keys 12, 26, and 31. The right‐most leaf node has keys 42, 45, 47, and 48.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12014.png)

Figure 12-14: Deleting the key 32 from an almost empty node (a) gives the leaf too few elements (b). We must merge with an adjacent sibling to restore the B-tree conditions (c).

[Figure 12-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-15) shows a case where we can transfer a key from a larger sibling node. We start by deleting the key 32 in [Figure 12-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-15)(a). [Figure 12-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-15)(b) shows the keys that we use to restore balance: the keys in the under-full node, the keys in its right-hand adjacent sibling, and the key in the parent separating the two. [Figure 12-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-15)(c) shows which keys will move and where. The repaired tree is shown in [Figure 12-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-15)(d).

![Figure A shows a subtree with one internal node and five leaf nodes. The internal node contains keys 12, 26, 31, and 45. The fourth leaf contains keys 32 and 42. The fifth leaf contains keys 47, 48, and 49. In Figure A, the key 32 is removed from a leaf node, leaving a single key 42. Figure B shows a dashed line around the right‐most two children and the separating key 45. Figure C includes arrows showing the key 47 from the right‐most child will move into the parent and key 45 from the parent will move to the under‐full child. The final tree is shown in Figure D. The internal node has four children and contains the keys 12, 26, 31, and 47. The fourth leaf node contains keys 42 and 45. The fifth leaf node contains keys 48 and 49.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12015.png)

Figure 12-15: Deleting the key 32 from an almost empty node (a) gives the leaf too few elements (b). We can repair this by taking a key from an adjacent sibling (c) to restore the B-tree conditions (d).

Finally, [Figure 12-16](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-16) shows a case where we remove a level from the tree by merging the only two children below the root. [Figure 12-16](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-16)(b) shows that after the merge, we are left with an empty root node. Its one key has been moved to the merged node. We repair this in [Figure 12-16](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c12.xhtml#figure12-16)(c) by removing the old root node and promoting that node’s single child to be the new root node.

Unlike addition, where the modifications to the B-tree are limited to only those nodes explored during the initial search for the insertion location, deletion can modify nodes in other branches. Both merging nodes and transferring keys use a sibling node at the same level. However, the total number of nodes modified is still limited by the depth of the tree. At most, we access one sibling node per level, and the number of nodes accessed scales logarithmically with _N_.

![In Figure A, the root node has the key 51 and has two children with keys 12 and 31 in the left node and key 61 in the right node. Figure B shows the result of the merge operation. The root is empty and has a single child with keys 12, 31, 51, and 61. Figure C shows the final tree where the old root node has been removed. The new root is the node with keys 12, 31, 51, and 61.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c12/f12016.png)

Figure 12-16: Merging the only two children under the root node (a) results in an empty root (b). We repair this by promoting the root node’s only child to be the new root (c).

## Why This Matters

B-trees illustrate a few important concepts. First, they show how we can adapt the behavior of previous data structures to handle cases where memory accesses between nodes are more expensive than accesses within a node. B-trees combine indexing and storage in such a way as minimize the number of accesses we need. This is critical for large data sets that might keep information on a disk or an external server. By enforcing a minimum of _k_ keys for each non-root node, we ensure a branching factor of at least _k_ + 1 at each node and thus flatten out the overall data structure. This helps limit the overall depth of the tree and thus the number of retrievals needed for search, insertion, or deletion. We also guarantee that each non-root node always remains at least half-full, meaning that we do not waste time retrieving nodes with only a few elements (except possibly the root).

It is useful to contrast the B-tree’s approach with a more specialized indexing scheme for our collectibles. We could develop a data structure that initially splits on category. The top-level index maps the collectible’s category, such as coffee-related collectibles, to a binder for that specific category. Each category’s binder then maps to all the subcategories, such as coffee mugs or coffee posters. And so forth. This is also a valid approach that builds off the branching structures we have seen throughout this book. The tradeoff becomes one of generalizability versus efficiency. In many cases, we can further optimize a data structure to a particular task at hand but lose the ability to apply it to other problems. In some cases, this tradeoff might be worth it. In others, it might not. Compared to a categorical focused indexing scheme, B-trees provide a more general approach that can work for any sortable set of keys.

The second concept B-trees illustrate is a second level of dynamism within the data structure itself. The B-tree constantly rearranges its structure to adapt to the distribution of data it stores and thus remain balanced. As we saw in Chapter 5, we lose the advantages of the tree-based structure if our tree becomes highly unbalanced. B-trees programmatically prevent this through a combination of bounds on the number of keys in each node (_k_ to 2_k_) and a guarantee that all leaf nodes have exactly the same depth. They adapt to “bad” distributions of input data by rebalancing—fixing nodes with too many or too few items. While there are a wide range of other balancing strategies for trees, B-trees provide a simple and clear example of how we can use additional structure (in this case, multiple keys per node) to avoid worst-case scenarios.