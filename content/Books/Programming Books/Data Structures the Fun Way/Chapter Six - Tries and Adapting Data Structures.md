---
id: 01J9QF48B36JZ3WCWMGBRXFH3N
modified: 2024-10-08T21:43:45-04:00
---
Binary search trees, while incredibly powerful, are only one way of using a tree structure to better organize data. Instead of splitting our data sets based on less-than or greater-than comparisons, we can optimize how the tree splits the data for the specific search problem at hand. In this chapter, for example, we tackle the problem of storing and searching strings in trees. Extending the binary search tree’s general branching approach to capture additional structure within the data enables us to search efficiently for target strings in a set of strings.

We begin by discussing how binary search trees can be directly applied to string data but have greater cost than other datatypes. Taking the sequential nature of strings into account, we’ll then adapt our search trees to store strings more efficiently. The result is a branching structure that may one day motivate the world’s most gloriously excessive filing cabinet: the trie (pronounced “try”).

Tries are data structures that branch on a _single_ character of the string at each level. This splitting strategy greatly reduces the cost of comparisons at each node. Through this lens, we explore how foundational concepts of various algorithms and data structures can be adapted to a new type of problem.

## Binary Search Trees of Strings

When considering whether we can improve an algorithm, we should first understand the limitations of our current approach—otherwise, there’s no reason to build a more complex data structure. Therefore, before we dive into string-specific data structures, we’ll examine where binary search trees fall short when used to store strings. First, let’s see how binary search trees can be used for this search.

### Strings in Trees

Binary search trees can store not just numbers but anything sortable, from shoes (sorted by size or smell) to zombie movies (sorted by box office revenue or scariness) to food items (sorted by price, spiciness, or likelihood to cause vomiting within the next 24 hours). Binary search trees are quite versatile that way. All we need is the ability to order the items.

To store strings in a binary search tree, we can sort elements in alphabetical order. For example, each node of the binary search tree in [Figure 6-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-1) is a single string partitioning its subtree into strings that come before and after it in the dictionary. We reuse the greater than and less than notation from binary search trees, where X < Y indicates that string X comes before string Y in the alphabet.

![The root node of this binary search tree is Laugh. Its left child is Feet, and its right child is Rock. Various subtrees are also organized alphabetically.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06001.png)

Figure 6-1: A binary search tree constructed with words

We search binary search trees of strings just as we did for numbers. For example, to find the string LIGHT in [Figure 6-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-1), we begin at the root node. We then compare the target value to the node’s value, using alphabetical ordering, and progress down either the left or right branch:

1. LIGHT > LAUGH: We proceed down the right branch.
2. LIGHT < ROCK: We proceed down the left branch.
3. LIGHT < MAIN: We proceed down the left branch.
4. LIGHT == LIGHT: We have found the target value and can terminate the search.

In [Figure 6-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-2), the shaded ovals indicate the 4 out of 12 nodes explored during the search.

![Diagram marking the path of a search for the string LIGHT. The search moves down the tree from LAUGH to ROCK to MAIN to LIGHT. The nodes containing these strings are grayed out.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06002.png)

Figure 6-2: The path of traversal when searching a binary search tree of strings for the string LIGHT

At first glance, binary search trees seem to provide a simple, efficient mechanism for searching string data—no modification required. If our tree is balanced, the worst-case cost of the search will scale proportional to the logarithm of the number of entries. In [Figure 6-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-2), we were able to limit our search to checking the target against only 4 out of 12 nodes.

However, we are forgetting one critical factor—the cost of each comparison. As we saw in Chapter 1, comparing two strings is more expensive than comparing a pair of numbers. In fact, in the worst case, the cost of the string comparison operation scales with the length of the strings themselves. Now the cost of our tree search depends on both the number of strings and their lengths, meaning we’ve added a new dimension of complexity.

To fix this problem and achieve even greater computational savings than provided by a binary search tree, we must take into account two important aspects of string data’s structure: the sequential ordering of strings and the limited number of letters or characters involved.

### The Cost of String Comparison

So far, we have been ignoring two important pieces of information in our quest to search strings. The first is the sequential nature of string comparisons. To determine the alphabetical order of strings, we start at the first character in a string and sequentially compare characters until we find a difference. That one difference then determines the string’s relative order in the search tree—it doesn’t matter what the rest of the characters are.

In the example in [Figure 6-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-3), ZOMBIE comes before ZOOM because of the characters in the third position: M comes before O. We don’t care about the relationship of remaining characters, BIE and M, and can ignore them.

![A figure showing the comparison of the strings ZOOM and ZOMBIE. The first two characters are marked with an equal sign to show they are the same. The third is marked with a greater than sign to indicate that O comes after M. The final characters are grayed out to indicate they are ignored.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06003.png)

Figure 6-3: The comparison of two strings progresses character by character until the first nonmatching pair is found.

As we saw in Chapter 1, the sequential comparison required for strings in binary search trees is inherently more expensive than the comparison of two numbers. Even comparing the two relatively short strings in [Figure 6-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-3) requires three separate comparisons: Z versus Z, O versus O, and O versus M. By the time we get to longer strings, such as our favorite movie quotes, the cost can become significant. The situation becomes even more dire when the strings have a high degree of overlap. Imagine a binary search tree that indexes our coffee collection by name. If we insert a few hundred coffees under the brand name “Jeremy’s Gourmet High Caffeine Experience,” we would need to compare quite a few characters to decide whether “Jeremy’s Gourmet High Caffeine Experience: Medium Roast” comes before or after “Jeremy’s Gourmet High Caffeine Experience: City Roast.” Our binary search tree algorithm pays that cost at each node.

The second key piece of information we haven’t considered as we strive to improve search efficiency is that, in many languages, each position can only contain a small number of letters. English words use just 26 letters (ignoring capitalization). Even if we include numbers and other characters, the set of valid characters is limited in practice. This means that, at each position, we have a limited number of ways the string can proceed—a limited number of next steps. As we’ll see shortly, this insight allows us to define a partitioning function that creates multi-way splits over the next character in the string.

We can combine these insights to build a data structure that operates similarly to comparing strings in the real world. The resulting data structure, a trie, is optimized to account for the additional structure in strings.

## Tries

_Tries_ are tree-based data structures that partition strings along different branches based on their prefixes. Computer scientist René de la Briandais proposed the general approach behind tries as a method to improve file searching on computers with slow memory access, and Edward Fredkin, a computer scientist and physicist, proposed their name. Instead of partitioning the data into two sets at each node, we branch the tree based on the prefix so far (sequential comparisons). Further, we allow the tree to split into more than a measly two branches (limited number of characters). In fact, for English words, we can let the tree branch 26 ways at each node—one branch for each of the possible next letters. Each node in the trie thus represents a prefix so far.

Like binary search trees, the trie starts at a root node, in this case representing the empty prefix. Each branch then defines the next character in the string. Naturally, this means each node can have more than two children, as shown in [Figure 6-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-4).

![A trie node with branches to child node pointers for strings starting with A, B, C, and so forth.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06004.png)

Figure 6-4: A trie node branches out on each possible character in the current position.

We can implement the branches of each trie node as using an array of pointers, with one array bin for each character. In some cases, we may be able to use representations more memory-efficient than arrays within our data structure. After all, even for English words, most non-trivial prefixes will have significantly fewer than 26 valid options for the next letter. For now, however, we will stick with an array implementation for simplicity’s sake. The discussions and example implementations in this chapter also focus primarily on English words (26 letters and a 26-element array of children), though the algorithms apply to other character sets as well.

Think of a trie’s branching structure as the registration table of a major event. Upon showing up at the world’s premier conference on Coffee Analogies for Computer Science, we visit the registration table to get our own personalized packet of information and free conference goodie (hopefully a coffee mug). With a huge number of participants, the organizers split up the packets into manageable groups so as to prevent a single line from snaking around the convention center. Instead of creating a series of pairwise splits (such as “Does your last name come before or after Smith?”), the organizers divide attendees into 26 different lines according to the first letter of their last name. In a single step, the crush of attendees shrinks to 26 more manageable lines. A trie performs that spectacular many-way branching at every node.

As with a binary search tree, we do not need to create nodes for empty branches of the tree. [Figure 6-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-5) shows an example of this structure, where the shaded letters represent empty branches of the trie. We do not create child nodes for those branches. Using the same terminology as for binary search trees, we call nodes with at least one child _internal nodes_ and nodes without any children _leaf node__s_. As a result, although we have the potential to branch 26 times at each node (when using only English letters), our trie will be relatively sparse. Later nodes may only branch a small number of times.

![The first three levels of a trie. Each node shows possible branches to children starting with A, B, or C. Certain characters are grayed out to indicate the absence of children in various branches.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06005.png)

Figure 6-5: Tries only create nodes for non-empty branches of the tree, allowing the data structure to avoid wasting space on unused branches.

Unlike in binary search trees, not every node in a trie represents a valid entry. While every leaf node is a valid entry, some internal nodes might be prefixes along the route to a full string. For example, a trie containing the string COFFEE would include an intermediate node representing the prefix COFFE. Although this is the spelling the author often uses when he’s consumed too much coffee, it is not actually a valid word or entry. We don’t want our data structure to imply that COFFE is acceptable just because there is a node corresponding to that prefix. At other points, however, an internal node might be a completely valid entry. If a trie contains the strings CAT and CATALOG, the node for CAT will be internal, because it has at least one child that lies along the path to the node for CATALOG.

To resolve this ambiguity, we store within the trie node an indicator of whether the current prefix represents a valid entry. This could be as simple as a Boolean `is_entry` that is `True` for any node corresponding to a valid entry and `False` otherwise:

```
TrieNode {
    Boolean: is_entry
    Array of TrieNodes: children
}
```

In this example, the node for COFFE would have `is_entry = False`, while the node for COFFEE would have `is_entry = True`.

Alternately, we could store something more useful in the trie node, such as the count of how many times a given entry has been inserted. Or, if we are tracking auxiliary data for each entry, such as a word’s definition or a list of hysterical puns, we could use the existence of this data as an indicator itself. Prefixes that do not represent valid entries can point to `null` or empty data structures. It may seem harsh, but only real words qualify for our best puns.

As with the binary search tree, we can clean up the trie’s interface by wrapping the root node in a trie object:

```
Trie {
    TrieNode: root
}
```

Unlike the binary search tree, our trie always has a (non-null) root. We create this root at the same time we create the trie data structure itself. Even for completely empty tries, we allocate a root node (with `is_entry = False`) that indicates the start of the string. Not only will the `Trie` data structure wrap the root node into a convenient container, but it will allow us to hide some additional bookkeeping needed for various operations.

One useful physical analogy for tries is an ultimate real-world filing system. Imagine a building that acts as storage system for detailed files on every topic in the world—a monument to efficient filing schemes. We partition the topics based on their first letter, like books of an encyclopedia, so our building has 26 stories. We reserve each floor of the building for each first letter, so the floors provide our first-level split. We then pack each floor with 26 rooms, one for each second letter of the topic. Each room would contain 26 filing cabinets that split for the third letter; each cabinet has 26 drawers (fourth letter), and 26 sections per drawer (fifth letter), and so on. At each level we are grouping together entries by their common prefixes. As long as we have high-speed elevators, we can find any topic relatively easily.

### Searching Tries

Searching tries is similar to searching binary search trees, in that we start at the top of the tree at the root node and progress downward, choosing branches that lead to the search target. In the case of a trie, however, we choose the branch that corresponds to the next letter in the string. We don’t need to compare full strings, or even the beginning of the prefix. That was done at previous nodes. We only need to consider the next character—a single comparison at each level.

Returning to the filing building analogy, imagine searching for information on your favorite author. After arriving at the floor K, you face 26 rooms labeled A through Z that represent the prefixes KA through KZ. Your next step depends only on the second letter of the author’s name. You do not need to waste time even considering the first letter again—that was already done on the elevator. Every room on this floor starts with K. You confidently head toward the room labeled U.

One complication with implementing this approach in code is that the comparisons we perform change at each level of the search. At the first level, we check the first character for a match—but at the second level, we need to check the second character. Our search no longer compares the entire target against the value at the node. We need additional information, the placement of the character that we are checking for at this level. We can track this additional state by passing the index to check into our recursive search function and incrementing it with each level of recursion.

The trie wrapper allows us to hide both the reference to the root node and the initial counter required by the recursive function, simplifying the code seen by the trie’s users:

```
TrieSearch(Trie: trie, String: target):
    return TrieNodeSearch(tr.root, target, 0)
```

This wrapper guarantees that subsequent search function is called with a non-null node and the correct initial index.

The code for recursively searching a trie is a bit more complex than the code for searching a binary search tree, because we must deal with target values of different lengths:

```
TrieNodeSearch(TrieNode: current, String: target, Integer: index):
  ❶ IF index == length(target):
        IF current.is_entry:
            return current
        ELSE:
            return null

  ❷ Character: next_letter = target[index]
  ❸ Integer: next_index = LetterToIndex(next_letter)
    TrieNode: next_child = current.children[next_index]
  ❹ IF next_child == null:
        return null
    ELSE:
        return TrieNodeSearch(next_child, target, index+1)
```

This code starts by checking the length of the target string against the current depth in order to determine whether the target should be located at this level ❶. If the index is equal to the length of the string (and thus one _past_ the last character in the string), the code then checks whether the current node is itself a valid entry. This check is particularly necessary when the search terminates at an internal node, since we need to confirm that this node represents a valid entry in its own right, not just the prefix of another entry.

If the code has not reached the end of the target string, it continues the search by examining the next character in our target ❷. We can define a helper function to map the character to the correct index in the array ❸. The code then checks whether the corresponding child exists ❹. If there isn’t a corresponding child, the code returns `null`, confident that `target` is not in the trie. If there is a corresponding child, the code follows that branch.

For an example of this search procedure, consider a trie of exclamations like YIKES and ZOUNDS from a recent episode of our favorite Saturday morning cartoon, as in [Figure 6-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-6). We can record auxiliary data such as the frequency of the word and who said it, allowing us to correct people’s references at parties. After all, what’s the use of data structures if they don’t help us win pedantic arguments?

![A trie that includes the strings EGADS, YIKES, YIP, YIPPEE, ZONK, and ZOUNDS.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06006.png)

Figure 6-6: A trie constructed from cartoon phrases

To check whether this week’s episode contained our all-time favorite cartoon word, ZONK, we can simply search the trie. We start at the top of the trie and take the corresponding branches for each character, as shown in [Figure 6-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-7).

![The trie from Figure 6‐6 with the following nodes highlighted: root, Z, ZO, ZON, and ZONK. The highlighted nodes show the path from the root to the node containing ZONK.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06007.png)

Figure 6-7: A search of the trie of cartoon phrases for ZONK. The shaded nodes indicate the path taken during the search.

Since tries include only nodes that have data, we can determine whether a string is not in the trie by watching for dead ends. For example, we know ZIPPY did not occur in the episode because we hit a dead end after the prefix Z. There is no branch for the prefix ZI. If some know-it-all proclaims that their favorite line contained the exclamation ZIPPY, we can prove them wrong with a simple search.

At first glance, it may appear that adding a large number of internal nodes increases the cost of searching. However, this new structure actually improves our search enormously. At each character in our target string, we perform a single lookup in the current node, checking for an existing child for that character, then proceed to the appropriate child node; thus, the number of lookups and comparisons scales with the length of our target string. Unlike a binary search tree, the number of comparisons for a successful trie search is independent of the number of strings stored in the trie. We could fill the trie with an entire dictionary and still only need to visit six nodes to check for the string EGADS.

Of course, as with everything in computer science, this efficiency does not come for free. We pay a significant cost in memory usage. Instead of storing one node for each string and pointers to two children, we now store one node for each character in the string and a large number of pointers to potential children. Overlapping prefixes help reduce the memory cost per string. If multiple entries share the same prefix, such as ZO for ZOUNDS and ZONK in [Figure 6-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-6), those entries share nodes for those initial overlapping characters.

### Adding and Removing Nodes

Like binary search trees, tries are dynamic data structures that adapt as we add or remove nodes, allowing them to accurately represent the data as it changes. Adding a string to a trie works much like adding data to a binary search tree. We progress down the tree as though searching for the string. Once we hit a dead end, we can create a subtree below that node to capture the remaining characters in that string. Unlike insertions in binary search trees, we may add multiple new internal nodes while inserting a single entry.

The top-level `Trie` function sets up the insertion by calling the recursive search function with the (non-null) root node and the correct initial depth:

```
TrieInsert(Trie: tr, String: new_value):
    TrieNodeInsert(tr.root, new_value, 0)
```

We don’t need to treat the creation of the root node as a special case, as we allocate an initial root node during the creation of the trie itself.

The code for insertion is similar to that of the search function:

```
TrieNodeInsert(TrieNode: current, String: new_value, Integer: index):
  ❶ IF index == length(new_value):
        current.is_entry = True
    ELSE:
        Character: next_letter = new_value[index]
        Integer: next_index = LetterToIndex(next_letter)
        TrieNode: next_child = current.children[next_index]
      ❷ IF next_child == null:
            current.children[next_index] = TrieNode()
          ❸ TrieNodeInsert(current.children[next_index], 
                           new_value, index + 1)
        ELSE:
          ❹ TrieNodeInsert(next_child, new_value, index + 1)
```

This code starts by checking the current position against the length of the inserted string ❶. When it has hit the end of the string, it marks the current node as a valid entry. Depending on the use case, the code might also need to update the node’s auxiliary data. Where the code has not yet hit the end of the string, it looks up the next character and checks whether the corresponding child exists ❷. If not, the code creates a new child node. Then it recursively proceeds to call `TrieNodeInsert` on the correct child node (❸ or ❹).

For example, if we wanted to add the string EEK to our list of cartoon exclamations, we would add two nodes: an internal node for the prefix EE and a leaf node for the full string EEK. [Figure 6-8](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-8) illustrates this addition, with shaded nodes indicating the trie nodes created during the insertion.

Removing nodes follows a similar process, but in reverse: starting at the node for the final character, we progress up the tree, deleting nodes that we no longer need. We stop deleting nodes once we hit an internal node that either has at least one non-empty child branch, thus representing a valid prefix for other strings in the trie, or is itself a string stored in the tree and thus represents a valid leaf node in its own right.

As with search and insertion, we start with the wrapper code that starts the deletion at the root node and with the correct index:

```
TrieDelete(Trie: tr, String: target):
    TrieNodeDelete(tr.root, target, 0)
```

The function does not return a value.

The code for removing nodes builds on the code for search and insertion, initially walking down the tree until it reaches the entry to delete. As it returns back up the trie, additional logic prunes empty branches. The code returns a Boolean value indicating whether or not the current node can safely be deleted, allowing the parent node to prune the branch.

![The trie from Figure 6‐6 with nodes added to store the strings EE and EEK. The additional nodes are in a branch below the node for the prefix E.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06008.png)

Figure 6-8: The addition of EEK to the trie of cartoon phrases. The new nodes are shaded.

```
TrieNodeDelete(TrieNode: current, String: target, Integer: index):
  ❶ IF index == length(target):
        IF current.is_entry:
            current.is_entry = False
    ELSE:
      ❷ Character: next_letter = target[index]
        Integer: next_index = LetterToIndex(next_letter)
        TrieNode: next_child = current.children[next_index]
        IF next_child != null:
          ❸ IF TrieNodeDelete(next_child, target, index+1):
               current.children[next_index] = null

    # Do not delete this node if it has either an entry or a child.
  ❹ IF current.is_entry:
        return False
  ❺ FOR EACH ptr IN current.children:
        IF ptr != null:
            return False
    return True
```

This code starts by comparing the length of the deleted string to the current level and changing the value of `is_entry` if the current node is being removed ❶. Otherwise, the algorithm recursively progresses down the tree, using the same logic as both our search and insertion functions ❷. It looks up the next character, finds the corresponding node, checks whether the node exists, and, if so, recursively descends to that node. If the node doesn’t exist, the target string is not in the trie, and the code will not continue downward. The code then deletes empty branches from the parent node. Each `TrieNodeDelete` call returns a Boolean to indicate whether it is safe to delete the corresponding node. If `TrieNodeDelete` returns `True`, then the parent immediately deletes the child node ❸.

The function ends with logic to determine whether it is safe for the parent to delete the current node. It returns `False` if `is_entry == True` ❹, indicating a valid entry, or if the current node has at least one non-null child ❺. It performs this last check using a `FOR` loop to iterate through each child and check whether it is `null`. If any child is not `null`, the code immediately returns `False` because it is a necessary internal node. Note that the code returns `False` in cases where the target string is not in the trie, because the code never sets `is_entry` to `False` for any node and thus there are no new pruning opportunities.

Consider removing the string YIPPEE from our example trie. If the trie also contains the word YIP as an entry, we’d delete all nodes following the one for YIP as shown in [Figure 6-9](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-9). The node for YIPPEE itself is marked safe to delete because it is a leaf and `is_entry` was marked `False` as part of the deletion. When the function returns to the node for YIPPE, it immediately deletes its only child (branch E). The node for YIPPE is now a leaf with `is_entry == False` and can be deleted by its parent. The process continues up the tree until we hit the node for YIP, which has `is_entry == True`, because the string YIP is in the trie.

Since deletion requires making a round trip from the root to a single leaf, the cost is again proportional to the length of the target string. As with search and insertion, cost is independent of the overall numbers of strings stored in the trie.

![The Trie from Figure 6‐8 with the nodes YIPP, YIPPE, and YIPPEE marked with grayed‐out, dashed lines to indicate their deletion.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06009.png)

Figure 6-9: Deletion of the string YIPPEE from the trie of cartoon phrases that also contains the string YIP. The deleted nodes have dashed lines and are grayed out.

## Why This Matters

We can now see how tries solve the problem with binary search trees that we posed earlier on: the cost of their search depends on both the number and length of words. In the brief examples in this chapter, tries do not provide much of an advantage. In fact, the overhead from additional branches may make them less efficient than binary search trees or sorted lists. However, tries become more and more cost-effective as we add more and more strings and the number of strings with similar prefixes increases. There are two reasons for this: first, the cost of a lookup in a trie scales independent of the number of entries, and second, string comparisons themselves can be expensive. In a binary search tree, these two factors compound, as we pay the cost of comparing two strings at each of the nodes.

In the real world, for example, we might use a trie inside a word processor to track the words in a dictionary. Auxiliary data at each node might include the definition or common misspellings. As the user types and edits, the program can efficiently check whether each word is in the dictionary and highlight it if otherwise. This program greatly benefits from the shared prefixes and limited number of characters in a natural language.

If we are composing an in-depth essay on the history of encyclopedias, for example, we do not want to pay excessive cost comparing _encyclopedia_ to the neighboring words _encyclopedias_, _encyclopedic_, _encyclopedist_, and so forth. Remember that, as shown in [Figure 6-10](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c06.xhtml#figure6-10), the algorithm to compare the alphabetical ordering of two strings consists of iterating over those strings, comparing each character. Although the program stops once it finds a differing character, the cost of comparing similar prefixes adds up. Within an active word processor document, we might need to modify the set of words constantly. Each insertion or edit will require a lookup in our data structure.

![A string comparison between the words encyclopedic and encyclopedias. The first 11 characters are marked as equal.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c06/f06010.png)

Figure 6-10: An example of an expensive string comparison

An even better use for a trie might be a data structure that tracks structured labels—such as serial numbers, model numbers, or SKU codes—that are often formatted as short alphanumeric strings. For example, we could create a simple trie to store product registration information indexed by serial number. Even allowing for billions of products sold, the cost of all operations scales linearly with the length of the serial number. If our serial numbers contain structure, such as a prefix representing the device’s model, we can realize further savings by limiting the branching factor at initial nodes (since many strings will use the same prefix). Auxiliary data could include information about where or when the device was purchased.

More importantly than any particular application, tries demonstrate how we can use further structure within the data to optimize the cost of operations. We adapted the branching structure of binary search trees to use the sequential nature of strings. This improvement once again illustrates a core theme of this book: we can often use the structure inherent in data to improve algorithmic efficiency.