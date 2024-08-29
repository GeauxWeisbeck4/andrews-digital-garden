---
id: 01J6DMR085HA84Z0M1864NY811
title: Learn Algorithms - Exponential Time
modified: 2024-08-28T22:13:54-04:00
description: Learning about exponential time using algorithms
tags:
  - boot-dev
  - algorithms
  - programming
---
# Polynomial vs Exponential

Broadly speaking, algorithms can be classified into two categories:

- "Polynomial time"
- "Exponential time"

![polynomial vs exponential](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/TOEsAwr.png)

_Technically `O(n!)` is "factorial" time, but let's lump them together for simplicity_

An algorithm runs in "Polynomial time" if its runtime does _not_ grow faster than `n^k`, where `k` is any constant and `n` is the size of the input.

To put it simply, polynomial time algorithms _can_ be useful. Exponential time algorithms are almost always too slow to ever be practical. Even when `n` is as low as `20`, `2^n` is already over a million!

# Polynomial Time = P

Back in the 1970s, some computer scientist researchers wanted to come up with a good, descriptive name for the set of polynomial time algorithms. After much deliberation, they settled on the letter `P` ([naming things is hard](https://xkcd.com/910/)).

In general, the main important takeaway is that:

- Problems that fall into class `P` are _practical_ to solve on computers.
- Problems that **don't** fall into `P` are hard, slow, and _impractical_.

# Reduction to P

Let's take an exponential time algorithm and rewrite it to run in polynomial time!

The [Fibonacci sequence](https://www.mathsisfun.com/numbers/fibonacci-sequence.html) is a sequence of numbers where each number is the sum of the two numbers before it. Like this:

```
0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

We want a function that, given an index in the sequence, returns the Fibonacci number at that index. For example:

- `fib(0)` -> 0
- `fib(1)` -> 1
- `fib(2)` -> 1
- `fib(3)` -> 2
- `fib(4)` -> 3
- `fib(5)` -> 5
- `fib(6)` -> 8
- `fib(7)` -> 13

Here are the implementation details to do it in polynomial time:

1. The input `n` represents the index of the desired Fibonacci value
2. Initialize three values. One will hold the `current` value, one will hold the `parent` value (1 before), and one will hold the `grandparent` value (2 before the current). You'll need to hard-code the first two values of the sequence. `grandparent`=0 and `parent`=1
3. Write a loop that iterates `n-1` times. (For example, the loop should not repeat when `n=2`.)
4. Inside the loop update the `current` based on the parents, then update the parents to their appropriate values.
5. Return the current value once the loop completes

## Assignment

Our data scientists at Socialytics have found that the growth of the average influencer's follow count is roughly the same growth rate as the [Fibonacci](https://en.wikipedia.org/wiki/Fibonacci_number) sequence! In other words, after 6 weeks of good Instagram posts, the average influencer will have 8 followers. After 7 weeks, 13 followers, and so on.

The trouble is, our current implementation of the `fib` function takes _so long_ (exponential time!) to complete that when our influencers navigate to their analytics page it often never completes loading!

Adjust the `fib` function using the given algorithm to achieve polynomial runtime.

```python
def fib(n):
    if n <= 1:
        return n

    grandparent = 0
    parent = 1
    current = 1

    for _ in range(2, n + 1):
        current = grandparent + parent
        grandparent = parent
        parent = current

    return current
```
# Order 2^N - Exponential

`O(2^n)` is the first Big O class that we've dealt with that falls into the scary _exponential_ category of algorithms.

Algorithms that grow at an exponential rate become impossible to compute after so few iterations that they are almost worthless in practicality.

## Assignment

At Socialytics we need to be able to compute the [power set](https://en.wikipedia.org/wiki/Power_set) of a set of influencers. Something about targeting segments of an audience with ads. I don't know, I just do what I'm told.

A power set is the set of all possible subsets of a set. For example, the set `{1, 2, 3}` has the power set:

`{{}, {1}, {2}, {3}, {1, 2}, {1, 3}, {2, 3}, {1, 2, 3}}`.

_We'll work with lists instead of sets for simplicity._

Complete the `power_set` function using the following algorithm:

1. Check if the input list is empty. If it is, return a list containing an empty list. (The power set of an empty set is a set containing just the empty set)
2. Otherwise, create an empty list to hold all the final subsets of the input list.
3. Recursively call `power_set`. Pass in all of the elements in the input set _except the first one_.
4. Iterate over the list of subsets returned from the recursive call. For each subset, append two new subsets to the _final_ list of subsets:
    1. list_with_only_the_first_item_from_input_set + subset
    2. subset
5. Return the list of subsets

## Observe!

Notice how the `power_set()` output gets _exponentially_ larger with each iteration. This is because its complexity class is `O(2^n)`

If we could calculate one subset per millisecond, completing the `power_set()` of just 25 items would take approximately 9 hours, and that's not accounting for the massive amounts of memory we would need.

40 items would take over 34 years!

## Answer
```python
def power_set(input_set):
    if not input_set:
        return [[]]

    final_subsets = []

    subsets_without_first = power_set(input_set[1:])

    for subset in subsets_without_first:
        final_subsets.append(subset)
        final_subsets.append([input_set[0]] + subset)

    return final_subsets
```

# Big O Complexity Quiz

Consider the following code. It calculates the number of times a given number can be divided by 2 before becoming less than or equal to 1.

```python
def naive_log2(x):
	count = 0
	while x > 1:
		x /= 2
		count += 1
	return count
```

## Answer
`O(log(n))`

# Big O Complexity Quiz

Consider the following function:

```python
#   halvedSections returns a list of lists. For example,
#	    the input "12" would result in
#	    [
#		    [0 1 2 3 4 5 6 7 8 9 10 11 12]
#			[0 1 2 3 4 5 6]
#			[0 1 2 3]
#			[0 1]
#		]
def halved_sections(n):
    rows = []
    i = n
    while i > 0:
        col = []
        for j in range(i+1):
            col.append(j)
        rows.append(col)
        i //= 2
    return rows
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Which has a specific time complexity of:

`T(n) = O(n + n/2 + n/4 + … 1)`

_Hint: This is a tricky one. You need to take into account the shrinking size of each successive list._

## Answer
`O(2n)`

