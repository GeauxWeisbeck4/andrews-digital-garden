---
id: 01JATSQD447DJ4NC5W6RDD5674
title: Graph Search Algorithms
modified: 2024-10-22T15:05:10-04:00
tags:
  - graph
  - data-structures
  - dsa
  - roadmaps
  - programming
  - search-algorithms
---
# Graph Search Algorithms

Search algorithms are fundamental techniques used for exploring a graph. Two classical methods are **Depth-First Search** (DFS) and **Breadth-First Search** (BFS). **DFS** relies on a stack and the concept of backtracking. Starting from a given node, it explores as far down a path as possible before backtracking. **BFS**, on the other hand, uses a queue and visits all of a node’s neighbors at one level before moving on to the next. For discovering the shortest path between two nodes, Dijkstra’s algorithm and the A* search algorithm are often used. **Dijkstra’s algorithm** builds up a table that provides the shortest distances to each reachable node from a selected starting node. __A_ search algorithm_*, a modification of Dijkstra’s algorithm, uses a heuristic to provide a best estimate of the path from the current node to the goal, thus often increasing the algorithm’s efficiency.