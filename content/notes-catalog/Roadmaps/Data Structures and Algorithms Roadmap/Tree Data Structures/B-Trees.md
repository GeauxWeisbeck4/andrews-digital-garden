---
id: 01JATS0DWBNBB5673VBJ0BYCX6
title: B-Trees
tags:
  - tree
  - data-structures
  - dsa
  - programming
  - roadmaps
  - binary-search
  - b-trees
modified: 2024-10-22T14:52:34-04:00
---
# B-Trees

B-Tree is a self-balanced search tree data structure that maintains sorted data and allows for efficient insertion, deletion, and search operations. It is most commonly used in systems where read and write operations are performed on disk, such as databases and file systems. The main characteristic of a B-Tree is that all leaves are at the same level, and the internal nodes can store more than one key. Each node in a B-Tree contains a certain number of keys and pointers which navigate the tree. The keys act as separation values which divide its subtrees. For example, if a node contains the values [10,20,30] it has four children: the first contains values less than 10, the second contains values between 10 and 20, the third contains values between 20 and 30, and the fourth contains values greater than 30.