---
id: 01J63AY3SBTYV3RHGN5FY8D8FF
title: Learn Algorithms - Polynomial Time
modified: 2024-08-29T13:27:21-04:00
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

# Order 1

`O(1)` means that no matter the size of the input, there is no growth in the runtime of the algorithm. This is also referred to as a "constant time" algorithm.

In Python, the `dictionary` data structure offers the ability to look items up by key, which is an operation that is independent of the size of the dictionary. In other words, dictionary lookups are `O(1)`.

## Assignment

We need to be able to search our Socialytics user base more quickly! Our users are complaining that the search bar is painfully slow. You'll notice that if you run the code in its current state, it will take a _very long time_.

The `find_last_name` function takes `names_dict`, a dictionary of `first_name -> last_name`. It also accepts a `first_name`. If `first_name` is a key in the dictionary, `find_last_name` returns the associated last name. If the key is not found, it returns `None`. Make sure you handle the case where the `first_name` is not in the dictionary!

Write the function so that it runs quickly! It should be `O(1)`.
## Answer
def find_last_name(names_dict, first_name):
    try:
        return names_dict[first_name]
    except:
        return None
        

# Order Log N

`O(log(n))` algorithms are only slightly slower than `O(1)`, but quite a bit faster than `O(n)`. They do grow according to the input size, `n`, but only according to the [log](https://en.wikipedia.org/wiki/Logarithm) of the input.

`O(n)`:

|n|time|
|---|---|
|8|8 ms|
|64|64 ms|
|1024|1024 ms|
|1048576|1048576 ms|

`O(log(n))`:

|n|time|
|---|---|
|8|3 ms|
|64|6 ms|
|1024|10 ms|
|1048576|20 ms|

## Binary Search

![binary search](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/EKzzF4f.png)

A [binary search algorithm](https://en.wikipedia.org/wiki/Binary_search_algorithm) is a common example of an `O(log(n))` algorithm. Binary searches work on a **sorted list** of elements.

### Pseudocode

Given an array of `n` elements **sorted** from least to greatest, and a target value:

- Set low = 0 and high = `n - 1`.
- While low <= high:
    - Set median (the position of the middle element) to `(low + high) // 2`, which is the greatest integer less than or equal to `(low + high) / 2`
    - If `array[median]` == `target`, return `True`
    - Else if `array[median]` < `target`, set low to `median + 1`
    - Otherwise set high to `median - 1`
- Return `False`

Because at each iteration of the search we are halving the size of the search space, the algorithm has a complexity of log2, or `O(log(n))`.

In other words, to add another step to the runtime, we need to double the size of the input. Binary searches are _fast_.

## Assignment

We have a popular Instagrammer using our Socialytics app, and she needs to be able to quickly search for posts from a particular day. This function will be the backbone of her search screen.

Complete the `binary_search` function. It should follow the algorithm as described above.

## Answer
```python
def binary_search(target, arr):
    n = len(arr)
    low = 0
    high = n - 1
    while low <= high:
        median = (low + high) // 2
        if arr[median] == target:
            return True
        elif arr[median] < target:
            low = median + 1
        else:
            high = median - 1

    return False
```

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
What is the most reduced form of Big O complexity?
`O(n)`

# Verifying Solutions

I want to circle back to this idea of "slow to solve, fast to verify".

As it turns out, even when we aren't specifically talking about P and NP, the concept of "slow to solve, fast to verify" is very important in real-world software. As a trivial example, imagine the password on an email account.

When a user inputs a password like:

`p@ssword4Mi`

It's easy to _verify_ if a password matches the one we have saved on file. It's literally as easy as:

```py
should_grant_access = user_input == saved_password
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

The useful bit is that it takes _much_ longer to _guess_ the correct key.

_Note: This password example demonstrates the guess/verify concept well, but when it comes to storing passwords in plain text this example is very insecure. We'll cover how to handle passwords in a production system in a future course._

## Assignment

The influencers who use Socialytics are worried about account security. We've assured them that their passwords are long and strong enough, but they want _data_.

Complete the `get_num_guesses` function. It takes a password length as input and returns the number of guesses required to _ensure_ that a password of that length or shorter is guessed.

### The guessing strategy

We're assuming a [brute-force](https://blog.boot.dev/security/how-do-brute-force-attackers-know-they-found-the-key/) guessing strategy. A guesser needs to guess all possible passwords _up to and including_ the given length to ensure they find the matching one. For example:

```
a
b
c
...
aa
ab
ac
...
ba
bb
bc
...
aaa
aab
aac
...
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Assume that only the 26 lowercase English letters can be used in passwords.

### Example input/output

```
length 1 = 26 guesses
length 2 = 702 guesses
length 3 = 18278 guesses
...
```
## Answer
```python
def get_num_guesses(length):
    total_guesses = 0
    for i in range(1, length + 1):
        total_guesses += 26 ** i

    return total_guesses
```

# Does P Equal NP?

The `P` versus `NP` problem is a [major unsolved problem](https://en.wikipedia.org/wiki/P_versus_NP_problem) in computer science. It asks whether every problem whose solution can be quickly verified (which means it is in `NP`) can also be solved quickly (which would mean it is also in `P`).

The question is "Are all `NP` problems really just `P` problems?"

The answer is, _"we don't know, but probably not"_.

## Why do we care?

Remember `NP-complete` problems?

All problems in `NP` (you know, hard ones like the traveling salesman problem) have been proven to also be solved if we can find a solution to any `NP-Complete` problem.

If a _single_ `NP-complete` problem can be shown to be solved quickly (in polynomial time) then that means that _ALL_ problems in `NP` can be solved in polynomial time.

In other words, if anyone figures out how to solve any `NP-Complete` problem in polynomial time, it means that `P` really does equal `NP`.

# Does P Equal NP?

We don't know if `P = NP`.

We have been unable to prove that `P = NP` because we can't find any polynomial-time solutions to `NP-complete` problems. Additionally, we have been unable to prove that `P != NP`. We _suspect_ `P != NP` just because it has been so difficult to prove that `P = NP`.

That said, it's actually more complicated to prove the negative case. To prove the positive case, that `P = NP`, we simply need to solve an NP-complete problem like TSP in polynomial time. In order to _prove_ the negative case, that `P != NP`, we would need to exhaustively prove that there's _no possible way_ to solve TSP in polynomial time. That's a lot trickier.

# NP-Hard

All `NP-complete` problems are [NP-hard](https://en.wikipedia.org/wiki/NP-hardness), but not all `NP-hard` problems are `NP-complete`. The determining factor between `NP-complete` and `NP-hard` is that not all `NP-hard` problems are in `NP`.

![NP Breakdown](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/p8wXLqA.png)

The image above assumes `P != NP`, which is what we suspect.

## Definition

> A problem is `NP-hard` if _every_ problem in `NP` can be [reduced](https://en.wikipedia.org/wiki/Reduction_(complexity)) into it in polynomial time.

Compare this to the slightly different definition of `NP-complete`:

# Prime Factorization

Let's solve a commonly misunderstood problem in computer science - **finding the prime factors of a number**. Almost all modern cryptography, including your browser's HTTPS encryption, is based on the fact that **prime factorization is slow**. You'll learn why that is later in our Cryptography course.

For now, let's focus on the speed of factorization, and how it relates to the `P` and `NP` classes.

Finding a number's prime factors is an algorithm in the `NP` class.

- When given two primes and their product, all we need to do is a simple multiplication step to _verify_ correctness. (polynomial time)
- Given just a number, finding its prime factors is a much more difficult problem. (exponential time is the best we know of)

The trouble is that no one has formally _proven_ that there is _not_ a polynomial time algorithm for finding prime factors. So, we're technically unsure if the problem is in `P` or if it's `NP-complete`.

Either way, let's build it!

## Prime Factors Algorithm

Given a large number, return a list of all the prime factors.

- `prime_factors(8)` -> [2, 2, 2]
- `prime_factors(10)` -> [2, 5]
- `prime_factors(24)` -> [2, 2, 2, 3]

1. Divide `n` by two as many times as you can do so evenly (no remainder). For each division, append a `2` to the list of prime factors.
2. At this point, `n` must be odd. Start a loop that iterates over all odd numbers from 3 to the square root of `n` _inclusive_. Use [math.sqrt()](https://docs.python.org/3/library/math.html#math.sqrt).
3. For each number `i`, if `n` can be divided evenly by `i`, then divide `n` by `i` and append `i` to the list. Repeat this (nested loop) until `i` can't divide evenly into `n`, then move on to the next `i`.
4. If `n` is still greater than 2 after that loop, it must still be prime, so just append it to the list.
5. Return the list of primes, ordered from least to greatest.

## Assignment

Complete the `prime_factors` function according to the given algorithm. Notice how the algorithm gets much slower as the size of the input (in bits) grows.

_Note: The returned list should only contain ints, no floats._
## Answer


```python
import math


def prime_factors(n):
    pf_list = []
    while n % 2 == 0:
        pf_list.append(2)
        n = n // 2

    for i in range(3, int(math.sqrt(n)) + 1, 2):
        while n % i == 0:
            pf_list.append(i)
            n = n // i

    if n > 2:
        pf_list.append(n)

    return pf_list
```
# Prime Factoring Review

Consider the code from the last assignment:

```python
def prime_factors(n):
    prime_factors = []
    while n % 2 == 0:
        n /= 2
        prime_factors.append(2)
    for i in range(3, int(math.sqrt(n)) + 1, 2):
        while n % i == 0:
            n /= i
            prime_factors.append(i)
    if n > 2:
        prime_factors.append(int(n))
    return prime_factors
```

What class was this a part of? NP
Given two large primes and a product, can we verify that the two primes multiply into the product in polynomial time? Yes

# Prime Factoring Review

## Big O Analysis

Let us denote `n` as the integer input, and `s` as the size of `n` in bits. `s = log2(n)`

Notice that our first loop iterates `log(n)` times and the second loop iterates `sqrt(n)` times. The Big O with respect to `n` is `O(sqrt(n))`! That's fast! That's polynomial complexity which would lead us to believe the problem is in `P`

## Wait!

The problem is that, by definition, when computer scientists talk about this problem, they are talking about the length of `n` in bits. What we will call `s`. For example, the integer `255` only takes `8` bits.

`241` = `11110001` in binary

Since `s = log2(n)`, a complexity of `O(sqrt(n))` is equivalent to `O(sqrt(2^s))`

The complexity in respect to the number of bits is _exponential_.

## Code

```python
def prime_factors(n):
    prime_factors = []
    while n % 2 == 0:
        n /= 2
        prime_factors.append(2)
    for i in range(3, int(math.sqrt(n)) + 1, 2):
        while n % i == 0:
            n /= i
            prime_factors.append(i)
    if n > 2:
        prime_factors.append(int(n))
    return prime_factors
```

What is our algorithm's Big O when n is the input integer, and when s is the size of the input in bits respectively?
`O(sqrt(n))` , `O(sqrt(2^s)` 