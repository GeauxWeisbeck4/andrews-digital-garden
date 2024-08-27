---
id: 01J63AY3SBTYV3RHGN5FY8D8FF
title: Learn Algorithms - Polynomial Time
modified: 2024-08-26T19:54:49-04:00
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


# O(n^2) - Order "n squared"

`O(n^2)` grows in complexity much more rapidly. For small and medium input sizes, these algorithms can still be very useful.

A common way that an algorithm will fall into the `O(n^2)` class is by using a nested loop, where the number of iterations of each loop is equal to the number of items in the input.

## Assignment

Socialytics needs search capabilities! For now, we'll build something slow but simple. Our users want to give us the full name of a fellow Instagrammer, and we need to find them by checking if their first and last names exist in our database.

Complete the `does_name_exist` function. It should loop over all of the first/last name combinations in the `first_names` and `last_names` lists. If it finds a combination that matches the `full_name` it should return `True`. If the full name isn't found, it should return `False` instead.

## Observe

When you run your completed code, notice how each successive call to `does_name_exist` takes quite a bit longer. Assuming the length of `first_names` and `last_names` is the same, each new name doesn't add `n` steps to the algorithm it adds `n^2` steps.

If `does_name_exist(10 first and last names)` takes just `1` second to complete, then we can assume the following timings are roughly accurate:

- `does_name_exist(100 first and last names)` = 100 seconds
- `does_name_exist(1,000 first and last names)` = 10,000 seconds
- `does_name_exist(10,000 first and last names)` = 1,000,000 seconds
```python
def does_name_exist(first_names, last_names, full_name):
    target_first, target_last = full_name.split()
    for first in first_names:
        for last in last_names:
            if first == target_first and last == target_last:
                return True
            
    return False
```
# O(nm)

`O(nm)` is _very_ similar to `O(n^2)`. The difference is that instead of a single input that we care about, there are 2. If `n` and `m` increase at the same rate, then `O(nm)` is effectively the same as `O(n^2)`. However, if `n` or `m` increases faster or slower, then it's useful to track their complexity separately.

## Assignment

Socialytics needs a new tool that allows big brands to see how many of an influencer's followers are loyal to their brand. Complete the `get_avg_brand_followers` function. It takes two inputs:

- `all_handles`: a 2-dimensional list, or "list of lists" of strings representing instagram user handles on a per-influencer basis.
- `brand_name`: a string.

`get_avg_brand_followers` returns the [average](https://en.wikipedia.org/wiki/Arithmetic_mean) number of handles that contain the `brand_name` across all the lists. Each list represents the audience of a single influencer.

### Example input/output

Input:

```py
all_handles = [
    ["cosmofan1010", "cosmogirl", "billjane321"],
    ["cosmokiller", "gr8", "cosmojane3"],
    ["iloveboots", "paperthin"]
]
brand_name = "cosmo"
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Expected output: 1.33 (Because 4 handles contained "cosmo" and there are 3 lists)

## Observe

Regarding Big O, the number of influencers (the number of lists) matters. That's our `n`. However, the average number of followers each influencer has (the length of the lists) is just as important. That's our `m`.
```python
def get_avg_brand_followers(all_handles, brand_name):
    total_brand_handles = 0
    for handles in all_handles:
        total_brand_handles += sum(1 for handle in handles if brand_name in handle)
    if len(all_handles) == 0:
        return 0.0
    else:
        avg_brand_handles = total_brand_handles / len(all_handles)

    return avg_brand_handles
```

# Constants Don't Matter

Big-O notation only describes the theoretical _growth rate_ of algorithms, rather than the actual running time of the algorithm when implemented on a given computer. As such, when doing Big O analysis, we don't let ourselves get bogged down in details.

For example, take a look at the following functions:

```python
def print_names_once(names):
    for name in names:
        print(name)
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

```python
def print_names_twice(names):
    for name in names:
        print(name)
    for name in names:
        print(name)
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

As you would expect, `print_names_once` will take half the time to run as `print_names_twice`. The funny thing about Big O analysis is that _WE DON'T CARE_.

Both functions have the same rate of growth, `O(n)`. You might be tempted to say, "print_names_twice should be `O(2 * n)`" but you would be missing the whole point of Big O.

In Big O analysis we drop all constants because they don't affect the _change_ in the runtime, just the runtime itself.

`O(2 * n)` -> `O(n)`

`O(10 * n^2)` -> `O(n^2)`