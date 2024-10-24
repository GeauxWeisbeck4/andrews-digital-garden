---
id: 01JATTBSCP0SJFDM369W1Q7B2R
title: Segment Trees
tags:
  - advanced
  - data-structures
  - dsa
  - programming
  - roadmaps
  - segment-trees
modified: 2024-10-22T15:14:28-04:00
---
# Segment Trees

A **Segment Tree** is a data structure designed for efficient processing of range queries and updates on array elements. In a situation where you have an array and you need to execute several types of queries including updating array elements and computing sum or minimum or maximum of elements in a given range, segment trees could be a great choice. The tree itself is a height-balanced binary tree and is filled with data from the input array. The leaves of the Segment Tree contain the array elements, while the internal nodes store information needed for processing the query, often the sum, minimum, or maximum of the elements represented by their child nodes. Efficient implementation of segment trees can achieve query and update operations in logarithmic time.