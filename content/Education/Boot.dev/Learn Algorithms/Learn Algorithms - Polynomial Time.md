---
id: 01J63AY3SBTYV3RHGN5FY8D8FF
title: Learn Algorithms - Polynomial Time
modified: 2024-08-24T19:21:09-04:00
description: The polynomial time part of Boot.dev algorithms
tags:
  - boot-dev
  - algorithms
  - python
  - 2024-journal
---
# Polynomial Time


# O(n) - Order "n"

`O(n)` is _very_ common - Any algorithm which has its number of steps grow at the same rate as its input size is classified as `O(n)`

For example, our `find min` algorithm from earlier is `O(n)`:

1. Set `min` to positive infinity.
2. For each number in the list, compare it to `min`. If it is smaller, set `min` to that number.
3. `min` is now set to the smallest number in the list.

The input to the `find min` algorithm is a list of size `n`. Because we loop over each item in the input once, we add one step to our algorithm for each item in our list.

As we use `find min` with larger and larger inputs, the length of time it takes to execute the function grows at a steady linear pace. We can reasonably estimate the time it will take to run, based on a previous measurement:

```
# if we find that...
find_min(10 items) = 2 milliseconds

# ...then we can estimate
find_min(100 items)  =  20 milliseconds
find_min(1000 items) = 200 milliseconds
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Assignment

At Socialytics we now need to display to our users the people who follow them with the _highest_ engagement count. This will help them know which of their followers they should follow back.

Complete the `find_max` function. It should take a list of integers and return the largest value in the list.

The "runtime complexity" (aka Big O) of this function should be `O(n)`