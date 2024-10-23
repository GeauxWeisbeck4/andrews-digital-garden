---
id: 01JATTBSCP0SJFDM369W1Q7B2R
title: Disjoint Set (Union-Find)
tags:
  - advanced
  - data-structures
  - dsa
  - programming
  - roadmaps
  - disjoint-set
  - union-find
modified: 2024-10-22T15:16:07-04:00
---
# Disjoint Set (Union-Find)

A **disjoint-set** data structure, also called a union-find data structure or merge-find set, is a data structure that tracks a partition of a set into numerous non-overlapping subsets. It provides near-constant-time operations to add new sets, to merge existing sets, and to determine whether elements are in the same set. The underlying algorithm uses two main techniques, `Union by Rank` and `Path Compression`, to achieve the efficient time complexity. Each element is represented as a node, and each group of disjoint sets forms a tree structure. Disjoint sets are useful in multitude of graph algorithms like Kruskal’s algorithm and many more.