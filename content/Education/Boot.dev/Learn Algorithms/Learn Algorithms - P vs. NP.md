---
id: 01J6E1KTGQ2ZMWJ8N99VXGB6E7
modified: 2024-08-28T23:36:53-04:00
title: Learn Algorithms - P vs. NP
description: Learning about n vs np
tags:
  - algorithms
  - python
  - programming
  - boot-dev
---
  
A good way of thinking about problems in `NP` is to imagine that we have a magic oracle that gives us potential solutions to problems. Here would be our process for finding if a problem is in `NP`:

- Present the problem to the magic oracle
- The magic oracle gives us a potential solution
- We verify in polynomial time that the solution is correct

If we can do the verification in polynomial time, the problem is in `NP`, otherwise, it isn't.

# Traveling Salesman Problem

A famous example of a problem in `NP` is the Traveling Salesman Problem, also known as `TSP`.

The version of the problem that we will solve can be stated:

> Given a list of cities, the distances between each pair of cities, and a total distance, is there a path through all the cities that is less than the distance given?

![TSP](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/lOCwZI5.png)

For example, with the above graph, the problem could be, "Is there a way to travel through A, B, C, and D in less than a distance of 67?" The answer would be "yes" by way of `A -> B -> D -> C`

## Assignment

Our Socialytics Instagrammers need to travel to trade shows to shill their sponsor's products! Since none of them trust Google Maps, they want to put in their proposed route to Socialytics, and our app will tell them if their route is short enough to be worth their time.

Complete the `tsp` function by performing a brute-force search using the provided algorithm. The brute-force search will, unfortunately, take factorial time, `O(n!)`, because you will need to try all possible paths and keep track of the shortest.

There is a helper function, `permutations()`, provided for you that will provide all possible permutations of a list. For example, `permutations([1,2,3])` would result in:

```json
[
    [1, 2, 3],
    [2, 1, 3],
    [3, 2, 1],
    [2, 3, 1],
    [3, 1, 2],
    [1, 3, 2]
]
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

### Pseudocode: `tsp(cities, paths, dist)`

#### Inputs

1. `cities`: A list of city identifiers (integers 0-n)
2. `paths`: A matrix where each point represents the distance between the two cities. For example, `paths[cityA][cityB]` holds the distance from cityA to cityB. `paths[cityA][cityB]` = `paths[cityB][cityA]`
3. `dist`: The distance we are trying to find a path shorter than

#### Output

`True` if there is a path shorter than `dist`. `False` otherwise.

#### Algorithm

1. Use the `permutations` function to get the matrix of all possible paths through the given `cities`.

Where the first path, `[1, 2, 3]` represents moving from `city 1 -> city 2 -> city 3`

1. Loop over each resulted permutation
    1. Loop over each city in the path keeping a sum of all the distances between each city.
    2. If the total distance in the path is less than the given `dist` return `True`
2. If no short paths were found, return `False`

# Traveling Salesman Problem - Verify

We know that `TSP` takes factorial time to solve. The reason is that we need to try every permutation of all the possible cities. For example, given 4 cities, `A, B, C, D`, we need to try:

```
A,B,C,D
B,A,C,D
C,A,B,D
A,C,B,D
B,C,A,D
C,B,A,D
C,B,D,A
B,C,D,A
D,C,B,A
C,D,B,A
B,D,C,A
D,B,C,A
D,A,C,B
A,D,C,B
C,D,A,B
D,C,A,B
A,C,D,B
C,A,D,B
B,A,D,C
A,B,D,C
D,B,A,C
B,D,A,C
A,D,B,C
D,A,B,C
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

That's `4!` or `4*3*2*1` or `24` permutations.

## TSP in NP

Even though it takes factorial time to solve `TSP`, `TSP` is in `NP` because we can verify a solution much faster. Let's write a TSP verifier!

## Assignment

Complete the `verify_tsp` function by implementing the algorithm below. Notice that it runs in polynomial time.

### Pseudocode: `verify_tsp(paths, dist, actual_path)`

#### Inputs

1. `paths`: A matrix where each point represents the distance between the two cities. For example, `paths[cityA][cityB]` holds the distance from cityA to cityB. `paths[cityA][cityB]` = `paths[cityB][cityA]`
2. `dist`: The distance we are trying to find a path shorter than
3. `actual_path`: The path we are trying to verify

#### Output

`True` if the distance of the path is actually shorter than `dist`. `False` otherwise.

#### Algorithm

1. Loop over each city in the actual path
2. Sum the distance between each city in the actual path
3. If the sum is less than `dist`, return `True`, otherwise return `False`

## Answer
```python
def verify_tsp(paths, dist, actual_path):
    total = 0
    for i in range(len(actual_path)):
        if i != 0:
            total += paths[actual_path[i - 1]][actual_path[i]]
    return total < dist

```

# TSP Review

Consider the `TSP` algorithms that we wrote:

## Solving TSP

```python
def tsp(cities, paths, dist):
    perms = permutations(cities)
    for perm in perms:
        total_dist = 0
        for i in range(1, len(perm)):
            total_dist += paths[perm[i - 1]][perm[i]]
        if total_dist < dist:
            return True
    return False
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Verifying TSP

```python
def verify_tsp(paths, dist, actual_path):
	total = 0
	for i in range(len(actual_path)):
		if i != 0:
			total += paths[actual_path[i-1]][actual_path[i]]
	return total < dist
```

## Answer

`O(n!)` and `O(n)`

# NP-Complete

Some, but not all problems in `NP` are also considered to be [NP-complete](https://en.wikipedia.org/wiki/NP-completeness).

A problem in `NP` is also `NP-complete` if _every_ other problem in `NP` can be [reduced](https://en.wikipedia.org/wiki/Reduction_(complexity)) into it in polynomial time.

## How is a problem "reduced"?

We won't go in deep on the subject of reductions in this course, but we can cover the basics.

Any problem, let's call it `Problem A`, can be reduced to a different problem, `Problem B`, if there is an algorithm (called a reducer) that changes an algorithm that solves `Problem B` into an algorithm that solves `Problem A`.

`Algorithm to solve B` -> reducer -> `Algorithm to solve A`

We say that "Problem A is reducible to problem B" if the `reducer` from the above can be run in polynomial time.

## What Does That Mean With NP-Complete?

If we have an algorithm that solves an `NP-complete` problem, it has been proven that we can _quickly_ reduce that algorithm into a new algorithm to solve any other problem in `NP`.

## NP can be reduced means
to reduce it in polynomial time, which is quickly

A solver for NP-problems can quickly be reduced to solve problems in NP