---
id: 01JATTP05CQ4FRKMMSVNSNYMJY
modified: 2024-10-22T15:20:26-04:00
title: Skip List
tags:
  - complex
  - data-structures
  - dsa
  - programming
  - roadmaps
  - skip-list
---
# Skip List

A **Skip List** is a probabilistic data structure that allows efficient search, insertion, and removal operations. It is a layered list that consists of a base list holding all the elements and several lists layered on top, each layer containing a random subset of the elements from the layer below. The highest level contains only one element, the maximum. Every element in the lists is connected by a link to the element of the same value in the list below. This structure provides a balance between the speed of binary search trees and the ease of implementation of linked lists, providing an efficient means for storing data while allowing fast retrieval, even within large sets of data.