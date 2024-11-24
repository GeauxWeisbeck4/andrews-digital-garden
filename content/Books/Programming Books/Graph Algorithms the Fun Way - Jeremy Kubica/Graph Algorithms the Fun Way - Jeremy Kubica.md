---
title: Graph Algorithms the Fun Way
subtitle: Powerful Algorithms Decoded, Not Oversimplified
author: Jeremy Kubica
authors: Jeremy Kubica
category: Computers
categories: Computers
publisher: No Starch Press
publishDate: 2024-11-19
totalPage: 418
coverUrl: http://books.google.com/books/content?id=7Z39EAAAQBAJ&printsec=frontcover&img=1&zoom=1&edge=curl&source=gbs_api
coverSmallUrl: http://books.google.com/books/content?id=7Z39EAAAQBAJ&printsec=frontcover&img=1&zoom=5&edge=curl&source=gbs_api
description: "Enter the wonderful world of graph algorithms, where you’ll learn when and how to apply these highly useful data structures to solve a wide range of fascinating (and fantastical) computational problems. Graph Algorithms the Fun Way offers a refreshing approach to complex concepts by blending humor, imaginative examples, and practical Python implementations to reveal the power and versatility of graph based problem-solving in the real world. Through clear diagrams, engaging examples, and Python code, you’ll build a solid foundation for addressing graph problems in your own projects. Explore a rich landscape of cleverly constructed scenarios where: Hedge mazes illuminate depth-first search Urban explorations demonstrate breadth-first search Intricate labyrinths reveal bridges and articulation points Strategic planning illustrates bipartite matching From fundamental graph structures to advanced topics, you will: Implement powerful algorithms, including Dijkstra’s, A*, and Floyd-Warshall Tackle puzzles and optimize pathfinding with newfound confidence Uncover real-world applications in social networks and transportation systems Develop robust intuition for when and why to apply specific graph techniques Delve into topological sorting, minimum spanning trees, strongly connected components, and random walks. Confront challenges like graph coloring and the traveling salesperson problem. Prepare to view the world through the lens of graphs—where connections reveal insights and algorithms unlock new possibilities."
link: https://books.google.com/books/about/Graph_Algorithms_the_Fun_Way.html?hl=&id=7Z39EAAAQBAJ
previewLink: http://books.google.com/books?id=7Z39EAAAQBAJ&pg=PR21&dq=graph+algorithms+the+fun+way&hl=&as_pt=BOOKS&cd=1&source=gbs_api
isbn13: 9781718503878
isbn10: 1718503873
id: 01JCHAEXXGPM45ZRTXYVNR2TC2
modified: 2024-11-12T18:15:12-05:00
---

# Table of Contents


# INTRODUCTION

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098182519/files/images/opener.jpg)

This book is an introduction to graphs and their algorithms for programmers who want to understand and apply them. Graphs are a type of data structure used throughout mathematics, computer science, and numerous other fields to model and solve a wide range of real-world problems. The structure of a graph allows us to represent connections or associations between items. Understanding this structure is critical to harnessing the power of graphs and using them efficiently.

_Graph Algorithms the Fun Way_ grew out of the chapter on graphs in my previous book, _Data Structures the Fun Way_ (No Starch Press, 2022), where I wrote, “We could devote an entire book to this single vastly impactful data structure.” Yet this book still only scratches the surface of the exciting and powerful world of graph algorithms, an area of study with a long history and ongoing research. A comprehensive coverage of all graph techniques and their relative advantages would require numerous volumes and be out of date the moment it was printed. Instead, this book is meant to serve as a foundation for people approaching this exciting field for the first time.

The book starts by introducing the components of graphs, then dives into exploring a variety of graph algorithms and how they apply to real-world problems. It is more than a cookbook of common algorithms. Its goal is to help readers understand the ideas behind the algorithms and build the intuitions to adapt the concepts covered here to techniques beyond this book.

## Who Is This Book For?

This book is for programmers who want to learn more about graphs, graph algorithms, and the computational thinking behind such techniques. I assume no prior knowledge of graphs or graph algorithms. However, readers should have the kind of basic familiarity with Python that can be expected after an introductory course, book, or boot camp. They should be familiar with fundamental Python programming concepts, including basic data structures such as lists and dictionaries.

I hope this book will be useful to a wide range of audiences, not just programmers learning graph algorithms for the first time. The examples and metaphors used throughout the book are designed to provide an alternative way to view the topics from their standard mathematical definitions. Advanced students and experienced computer scientists may find a new perspective to understand particularly difficult or tricky topics.

## Analogies and Examples

This book supplements formal descriptions and code with a range of real-world and absurd examples and analogies. The structure of graphs makes them perfect for illustrating algorithms with stories of adventurers searching labyrinths or planning vacations through unknown cities. The goal of these examples and analogies is twofold. First, they motivate the algorithms themselves and why we care about the problems they solve. Second, they provide an alternate approach to visualizing these problems that will help readers break free of technicalities and minutiae.

## Language and Coding Conventions

I chose to present example code in Python due to the language’s wide use and readability. However, aficionados of other languages need not fear, as the concepts behind the code are language-agnostic. Graph algorithms have been implemented in a wide range of languages, and all code examples in this book can be adapted beyond Python.

The code throughout the book uses common Python conventions. To make the code clearer, I use type hints, as in the following code block:

```
def is_edge(self, from_node: int, to_node: int) -> bool:
    return self.get_edge(from_node, to_node) is not None
```

The input arguments list the expected types, such as int, and the function definition describes the expected return type (bool).

The code in this book uses multiple core Python libraries. Since functions throughout a file often use the same library, individual code snippets do not explicitly include the import statements. Users implementing the code will need to make sure to import the relevant libraries. Where ambiguous, I identify the needed libraries in the code’s text description.

Standard Python libraries used in this book include:

csv

copy

itertools

math

queue

random

typing (for Union)

The typing library in particular is needed for a number of code snippets, in order to support type hints for functions with multiple return values.

In addition, Appendix B defines a custom PriorityQueue class used in multiple examples, and Appendix C defines a minimal UnionFind data structure.

The code in this book is structured to stand alone as much as possible and requires only these core Python libraries. This means I sometimes don’t take advantage of good existing libraries. For example, in [Chapter 1](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter1.xhtml) I represent a matrix as a list of lists instead of leveraging the numpy library optimized for matrix operations. I call out instances where existing libraries would be a good fit, but I leave their integration into the code as an exercise for the reader.

I’ve also made many of the implementations in the book more verbose than strictly necessary in order to focus on the computational ideas behind them. This means that individual implementations may be broken into extra stages to illustrate the computational concepts, structured in a way that matches the explanation, or otherwise may not be fully optimized. In addition, to keep the examples simple, I often leave out the basic validity checks that are vital for production programs. Treat the examples as illustrations of general concepts rather than code to use verbatim in your own projects.

## Terminology and Definitions

Since graph algorithms have been studied in a variety of fields, multiple terms sometimes exist for the same underlying concept. For example, links in a graph are also commonly referred to as _edges_ or _arcs_. I define each concept when it is introduced and note some of the alternative names that readers might find in other references.

In other cases, the same term is used differently within different fields. In particular, the definitions of several key terms have deviated over the years between formal graph theory in mathematics and computer science study. For example, in mathematics, a _path_ through a graph cannot include repeated nodes, while in computer science it often can. Where definitions differ, I default to the common computer science usages and note the difference in the text.

## How to Use This Book

This book is structured progressively, with later chapters building on earlier ones. [Part I](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/part1.xhtml) sets up the conceptual foundations on which later chapters rely:

**[Chapter 1](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter1.xhtml): Representing Graphs** Introduces the structure of graphs, discusses the graph representations of adjacency lists and adjacency matrices, and provides the implementations used throughout the rest of the book.

**[Chapter 2](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter2.xhtml): Neighbors and Neighborhoods** Covers the core concept of neighboring nodes, basic algorithms to build sets of neighbors, and some basic metrics for understanding the local connectivity around a node.

**[Chapter 3](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter3.xhtml): Paths Through Graphs** Discusses paths through graphs and introduces multiple representations including lists of nodes, lists of edges, and lists of back pointers.

Later sections are less interdependent but still call upon concepts in earlier chapters. Each is organized around a theme. [Part II](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/part2.xhtml) focuses on searches and shortest paths in a graph:

**[Chapter 4](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter4.xhtml): Depth-First Search** Introduces two implementations of depth-first search, a recursive approach and an iterative stack-based approach, and also discusses how search information can be encoded in a depth-first search tree.

**[Chapter 5](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter5.xhtml): Breadth-First Search** Explores breadth-first search, discusses its properties, and shows how we can use it to find the shortest paths through unweighted graphs.

**[Chapter 6](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter6.xhtml): Solving Puzzles** Shows how we can use graphs to encode puzzles and use the search algorithms from [Chapters 4](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter4.xhtml) and [5](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter5.xhtml) to solve these puzzles.

**[Chapter 7](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter7.xhtml): Shortest Paths** Introduces three algorithms for finding shortest paths through weighted graphs: Dijkstra’s algorithm, the Bellman-Ford algorithm, and the Floyd-Warshall algorithm.

**[Chapter 8](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter8.xhtml): Heuristic-Guided Searches** Describes two heuristic-based searches, heuristic greedy search and A* search, and shows how they can make use of heuristic information about how promising the nodes are.

[Part III](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/part3.xhtml) focuses on connectivity and ordering in graphs:

**[Chapter 9](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter9.xhtml): Topological Sort** Discusses the problem of sorting a graph’s nodes in topological order and introduces two algorithms for this task: Kahn’s algorithm and an extension of depth-first search.

**[Chapter 10](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter10.xhtml): Minimum Spanning Trees** Describes two algorithms for finding minimum spanning trees on graphs, Prim’s algorithm and Kruskal’s algorithm, and also shows how the ideas behind Kruskal’s algorithm can be extended to problems such as generating solvable mazes or clustering spatial data.

**[Chapter 11](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter11.xhtml): Bridges and Articulation Points** Examines algorithms based on depth-first search for finding bridges and articulation points in graphs.

**[Chapter 12](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter12.xhtml): Strongly Connected Components** Explores Kosaraju-Sharir’s algorithm to identify strongly connected components in directed graphs.

**[Chapter 13](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter13.xhtml): Random Walks** Introduces into random walks on graphs and discusses the concept of Markov chains, then shows how to implement random walk behavior on graphs and estimate the underlying graphs from observed data.

[Part IV](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/part4.xhtml) introduces the concept of flow within graphs and uses it to solve a particular matching problem:

**[Chapter 14](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter14.xhtml): Max-Flow Algorithms** Defines the concepts of flow through a graph and the max-flow problem, introduces an extended version of the graph data structure to support this problem, and describes the Ford-Fulkerson and Edmond-Karp algorithms for solving the maximum-flow problem.

**[Chapter 15](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter15.xhtml): Bipartite Graph Matching** Introduces the task of matching in graphs and the concept of bipartite graphs before focusing on the specialization of matching within bipartite graphs. We show how to use maximum-flow algorithms to solve one variety of the matching problem on bipartite graphs.

[Part V](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/part5.xhtml) covers various node assignment and path planning problems through graphs:

**[Chapter 16](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter16.xhtml): Graph Coloring** Introduces the problem of assigning colors to graph nodes such that no two neighbors share a color and considers a range of algorithms to solve this problem.

**[Chapter 17](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter17.xhtml): Cliques, Independent Sets, and Vertex Covers** Introduces algorithms for three computationally challenging assignment problems: finding a maximum clique, finding a maximum independent set, and finding a minimum vertex cover.

**[Chapter 18](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter18.xhtml): Tours Through Graphs** Considers three path-planning problems: finding paths that visit each node exactly once, finding paths that visit each node exactly once while minimizing the edge weights traversed, and finding paths that cross each edge exactly once. We describe why the first two problems are difficult, but there exists an efficient solution for the third.

The appendices provide additional functions and data structures that are helpful for implementing the algorithms in this book:

**[Appendix A](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/appendix_A.xhtml)** Describes functions for programmatically creating graphs, including loading them from files.

**[Appendix B](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/appendix_B.xhtml)** Defines the modifiable priority queue data structure used in algorithms throughout the book.

**[Appendix C](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/appendix_C.xhtml)** Introduces a minimal UnionFind data structure necessary to implement some of the algorithms in [Chapter 10](https://learning.oreilly.com/library/view/graph-algorithms-the/9781098182519/xhtml/chapter10.xhtml)

Throughout the book, the reader should focus on the questions _How?_ and _Why? How_ does this real-world problem map onto a graph formulation? _Why_ does a certain approach help us compute the solution? _How_ does an algorithm use the graph’s structure to solve the problem? _Why_ do we care about this problem? _How_ do these algorithms apply to different problems? _Why_ is the author using that ridiculous analogy? Understanding the answers to these questions will provide the foundation you need to effectively use existing algorithms and develop your own techniques in the future.