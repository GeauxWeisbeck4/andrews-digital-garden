---
id: 01JATTP05CQ4FRKMMSVNSNYMJY
modified: 2024-10-22T15:19:43-04:00
title: B B+ Trees
tags:
  - tree
  - b-trees
  - b-plus
  - complex
  - data-structures
  - dsa
  - programming
  - roadmaps
---
# B/B+ Trees

`B trees` and `B+ trees` are both types of self-balancing, sorted, tree-based data structures that maintain sorted data in a way that allows for efficient insertion, deletion, and search operations. A `B tree` is a tree data structure in which each node has multiple keys and can be in more than two children nodes. Each internal node in a `B tree` can contain a variable number of keys and pointers. The keys act as separation values which divide its subtrees. One important aspect of a `B tree` is that every key in the node also appears in the parent node. On the other hand, a `B+ tree` is an extension of a `B tree` which allows for efficient traversal of data. In a `B+ tree`, data pointers are stored only at the leaf nodes of the tree, making every leaf node of a `B+ tree` a linked list. The intermediary nodes only use the keys to aid with the search.