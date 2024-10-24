---
id: 01JATTBSCP0SJFDM369W1Q7B2R
title: Fenwick Trees
tags:
  - advanced
  - data-structures
  - dsa
  - programming
  - roadmaps
  - fenwick
  - tree
modified: 2024-10-22T15:15:33-04:00
---
# Fenwick Trees

Fenwick Trees, also known as Binary Indexed Trees, are data structures that can efficiently support the operation of updating elements and calculating prefix sums in a table of numbers. This makes it particularly useful in situations where the table gets updated often and different kinds of queries (such as sum of elements) need to be answered fast. A Fenwick Tree typically takes O(log n) time for both updation and query operations, which is more efficient than an array or a segment tree. It achieves this efficiency by storing partial sum information in the array. This allows for efficient calculation of sum ranges, with the operation of adding an element and getting the sum of a range both achieve in O(log n) time.