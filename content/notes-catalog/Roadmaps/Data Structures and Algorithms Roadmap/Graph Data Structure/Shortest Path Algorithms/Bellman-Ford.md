---
id: 01JATSQD447DJ4NC5W6RDD5674
title: Bellman-Ford
modified: 2024-10-22T15:08:52-04:00
tags:
  - graph
  - data-structures
  - dsa
  - roadmaps
  - programming
  - bellman-ford
---
# Bellman-Ford

The **Bellman Ford algorithm** is a method used in graph theory for finding the shortest path between a single source vertex and all other vertices in a weighted graph. This algorithm is significant because it is capable of handling graphs with negative weight edges, unlike Dijkstra’s algorithm. It follows a bottom-up approach, filling up the distance table gradually while relaxing edges. The algorithm gets its name from its developers, Richard Bellman and Lester Ford. However, it can lead to an infinite loop if there are negative weight cycles in the graph, which should be addressed separately using another check.