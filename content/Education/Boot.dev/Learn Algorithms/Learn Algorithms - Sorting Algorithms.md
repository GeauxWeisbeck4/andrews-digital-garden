---
id: 01J6AVG11TD6WQETZF6JHMG5WM
title: Learn Algorithms - Sorting Algorithms
modified: 2024-08-28T19:20:53-04:00
description: How to sort stuff with algorithms
tags:
  - algorithms
  - sorting
  - boot-dev
---
# Learn Algorithms - Sorting Algorithms
# Sorting Algorithms

Sorting is a very common problem in software. Imagine you are organizing books on a shelf; how would you explain it step by step to someone else? Sorting algorithms can be surprisingly tricky!

Fortunately, most programming languages provide their own implementation of sorting. In Python, for example, we can use [`sorted`](https://docs.python.org/3/library/functions.html#sorted):

```python
items = [1, 5, 3]
print(sorted(items)) # [1, 3, 5]
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Assignment

In Socialytics, we want to be able to sort influencers by their overall vanity. We define vanity using this formula:

```
vanity = num_bio_links * 5 + num_selfies
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

### vanity

This function should compute the vanity of an influencer and return the result.

### vanity_sort

This function should return a list of influencers, ordered by their vanity. You can pass a named parameter called `key` to the [sorted](https://docs.python.org/3/library/functions.html#sorted) function to control what the list gets sorted by (hint: you already wrote this function!)

```python
class Influencer:
    def __init__(self, num_selfies, num_bio_links):
        self.num_selfies = num_selfies
        self.num_bio_links = num_bio_links

    def __repr__(self):
        return f"({self.num_selfies}, {self.num_bio_links})"

# dont touch above this line

def vanity(influencer):
    return influencer.num_bio_links * 5 + influencer.num_selfies

def vanity_sort(influencers):
    return sorted(influencers, key=vanity)
```
# Bubble Sort

In this chapter, we are going to be learning about various algorithms to sort data. But wait, if `sorted()` exists, why should we learn bubble sort?

![john f kennedy](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/Jev7rhR.png)

In all seriousness, we're going over these because:

- Hard things make us better engineers
- It helps to see more examples of the Big O runtimes we've studied

Bubble sort is named for the way elements "bubble up" to the top of the list.

## Animated Video

## Description

```
[5, 3, 7, 8, 7] -> [3, 5, 7, 7, 8]
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Bubble sort repeatedly steps through a slice and compares adjacent elements, swapping them if they are out of order. It continues to loop over the slice until the whole list is completely sorted.

Pseudocode for [bubble sort](https://en.wikipedia.org/wiki/Bubble_sort):

```
Procedure bubble_sort(nums):
    Set swapping to True
    Set end to the length of nums
    While swapping is True:
        Set swapping to False
        For i from the 2nd element to end:
            If the (i-1)th element of nums is greater than the ith element:
                Swap the (i-1)th element and the ith element of nums
                Set swapping to True
        Reduce end by one
    Return nums
End Procedure
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Assignment

While our avocado toast influencers were happy with our search functionality, now they want to be able to sort their followers by follower count. Bubble sort is a straightforward sorting algorithm that we can implement quickly, so let's do that!

Complete the `bubble_sort` function according to the described algorithm above.

## Answer
```python
def bubble_sort(nums):
    swapped = True
    n = len(nums)
    for i in range(n):
        for j in range(n - 1 - i):
            if nums[j] > nums[j + 1]:
                nums[j], nums[j + 1] = nums[j + 1], nums[j]
            
        if not swapped:
            break
    return nums
```

# Bubble Sort Big O

Bubble sort implementation reference:

```python
def bubble_sort(nums):
    swapping = True
    end = len(nums)
    while swapping:
        swapping = False
        for i in range(1, end):
            if nums[i - 1] > nums[i]:
                temp = nums[i - 1]
                nums[i - 1] = nums[i]
                nums[i] = temp
                swapping = True
        end -= 1
    return nums
```
# Why Use Bubble Sort?

Bubble sort is famous for how easy it is to write. It's one of the slowest sorting algorithms, but can be useful for a quick script or when the amount of data to be sorted is guaranteed to be small.

## Reference

```python
def bubble_sort(nums):
    swapping = True
    end = len(nums)
    while swapping:
        swapping = False
        for i in range(1, end):
            if nums[i - 1] > nums[i]:
                temp = nums[i - 1]
                nums[i - 1] = nums[i]
                nums[i] = temp
                swapping = True
        end -= 1
    return nums
```

# Merge Sort

Merge sort is a [recursive](https://en.wikipedia.org/wiki/Recursion) sorting algorithm and it's quite a bit faster than bubble sort.

## Divide and Conquer

Merge sort is a [divide and conquer algorithm.](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm):

- Divide: divide the large problem into smaller problems, and recursively solve the smaller problems
- Conquer: Combine the results of the smaller problems to solve the large problem

In merge sort specifically we:

### Divide

- Divide the array into two (equal) halves
- Recursively sort the two halves

### Conquer

- Merge the two halves to form a sorted array

![merge sort gif](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/oMbHRV9.gif)

## Algorithm

The algorithm consists of two separate functions, `merge_sort` and `merge`.

`merge_sort()` divides the input array into two halves, calls itself for the two halves, and then merges the two sorted halves.

The `merge()` function is used for merging two sorted lists back into a single sorted list. At the lowest level of recursion, the two "sorted" lists will each have a length of `1`. Those single element lists will be merged into a sorted list of length two, and we can build from there.

### merge_sort() implementation

Input: A, a list of integers

1. If the length of A is less than 2, it's already sorted so return it
2. Split the input array into two halves down the middle
3. Call merge_sort() twice, once on each half
4. Return the result of calling merge(sorted_left_side, sorted_right_side) on the results of the merge_sort() calls

### merge() implementation

Inputs: A, B. Two lists of integers

1. Create a new "final" list of integers.
2. Set `i` and `j` equal to zero. They will be used to keep track of indexes in the input lists (`A` and `B`).
3. Use a loop to iterate over `A` and `B` at the same time. If an element in `A` is less than or equal to its respective element in `B`, add it to the final list and increment `i`. Otherwise, add the item in `B` to the final list and increment `j`.
4. After comparing all the items, there may be some items left over in either `A` or `B` (if one of the lists is longer than the other). Add those extra items to the final list.
5. Return the final list.

## Assignment

Our users are complaining on Socialytics that when they sort their followers by follower count, it gets really slow if they have more than 1,000 followers. Let's speed it up for them with merge sort.

Complete the `merge_sort` and `merge` functions according to the described algorithms.

## Answer
```python
def merge_sort(nums):
    if len(nums) < 2:
        return nums
    mid = len(nums) // 2
    left_side = nums[:mid]
    right_side = nums[mid:]

    sorted_left_side = merge_sort(left_side)
    sorted_right_side = merge_sort(right_side)

    return merge(sorted_left_side, sorted_right_side)


def merge(first, second):
    final = []
    i = j = 0
    while i < len(first) and j < len(second):
        if first[i] <= second[j]:
            final.append(first[i])
            i += 1
        else:
            final.append(second[j])
            j += 1

    final.extend(first[i:])
    final.extend(second[j:])
    return final
        
```

# Merge Sort Big O

Merge sort implementation reference:

```python
def merge_sort(nums):
    if len(nums) < 2:
        return nums
    first = merge_sort(nums[: len(nums) // 2])
    second = merge_sort(nums[len(nums) // 2 :])
    return merge(first, second)


def merge(first, second):
    final = []
    i = 0
    j = 0
    while i < len(first) and j < len(second):
        if first[i] <= second[j]:
            final.append(first[i])
            i += 1
        else:
            final.append(second[j])
            j += 1
    while i < len(first):
        final.append(first[i])
        i += 1
    while j < len(second):
        final.append(second[j])
        j += 1
    return final
```

# Why Use Merge Sort?

Fast: Merge sort is much faster than bubble sort, being `O(n*log(n))` instead of `O(n^2)`.

Stable: Merge sort is also a [stable sort](https://en.wikipedia.org/wiki/Category:Stable_sorts) which means that values with duplicate keys in the original list will be in the same order in the sorted list.

Extra memory: Most sorting algorithms can be performed using a single copy of the original array. Merge sort requires an extra array in memory to merge the sorted subarrays.

Recursive: Merge sort requires many recursive function calls, and function calls can have significant resource overhead.

## Reference

```python
def merge_sort(nums):
    if len(nums) < 2:
        return nums
    first = merge_sort(nums[: len(nums) // 2])
    second = merge_sort(nums[len(nums) // 2 :])
    return merge(first, second)


def merge(first, second):
    final = []
    i = 0
    j = 0
    while i < len(first) and j < len(second):
        if first[i] <= second[j]:
            final.append(first[i])
            i += 1
        else:
            final.append(second[j])
            j += 1
    while i < len(first):
        final.append(first[i])
        i += 1
    while j < len(second):
        final.append(second[j])
        j += 1
    return final
```

# Insertion Sort

Insertion sort builds a final sorted list one item at a time. It's much less efficient on large lists than more advanced algorithms like quicksort or merge sort.

## Dancing Visual

## Assignment

We have another screen in the Socialytics dashboard where our users want to sort their affiliate deals by revenue. None of our users have more than a couple hundred affiliate deals, so we don't **need** an `n * log(n)` algorithm like merge sort. In fact, `insertion_sort` can be faster than `merge_sort`, and uses less of our server's memory when list sizes are small.

Complete the `insertion_sort` function according to the given algorithm.

## Pseudocode

1. For each index in the input list:
    1. Set a `j` variable to the current index
    2. While `j` is greater than `0` and the element at index `j-1` is greater than the element at index `j`:
        1. Swap the elements at indices `j` and `j-1`
        2. Decrement `j` by `1`
2. Return the list

## Tip

In some languages you need to use a `temp` variable to swap values, but in python you can do that in a single line:

```python
a = 5
b = 3
a, b = b, a
print(a)
# 3
print(b)
# 5
```

## Answer
```python
def insertion_sort(nums):
    for i in range(1, len(nums)):
        j = i
        while j > 0 and nums[j - 1] > nums[j]:
            nums[j - 1], nums[j] = nums[j], nums[j - 1]
            j -= 1
    
    return nums
```
# Insertion Sort Big O

Insertion sort has a Big O of `O(n^2)`, because that is its _worst case_ complexity.

The outer loop of insertion sort executes `n` times, while the inner loop depends on the input. In the worst case (a reverse sorted array) the inner loop executes `n` times as well. In the best case (a sorted array) the inner loop immediately breaks resulting in a total complexity of `O(n)`

## Reference

```python
def insertion_sort(nums):
    for i in range(len(nums)):
        j = i
        while j > 0 and nums[j - 1] > nums[j]:
            nums[j], nums[j - 1] = nums[j - 1], nums[j]
            j -= 1
    return nums
```

# Quick Sort

## Description

Quick sort is an efficient sorting algorithm that's widely used in production sorting implementations. Like merge sort, quick sort is a [divide and conquer algorithm.](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm)

### Divide

- Select a pivot element that will _preferably_ end up close to the center of the sorted pack
- Move everything onto the "greater than" or "less than" side of the pivot
- The pivot is now in its final position
- Recursively repeat the operation on both sides of the pivot

### Conquer

- The array is sorted after all elements have been through the pivot operation

## Visuals

The `quick_sort` algorithm is recursive. And it works in the following way:

- Select a "pivot" element - We'll arbitrarily choose the last element in the list
- Move through all the elements in the list and swap them around until all the numbers less than the pivot are on the left, and the numbers greater than the pivot are on the right
- Move the pivot between the two sections where it belongs
- recursively repeat for both sections

Here is a visual that may help:

![quicksort full](https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)

And a slower one of just the partition function:

![partition](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/QhphN1P.gif)

## Assignment

We now have two sorting algorithms on our Socialytics backend! It is a bit annoying to maintain both in the codebase. Quicksort is fast on large datasets just like merge sort, but is also lighter on memory usage. Let's use quick sort for both follower count and influencer revenue sorting!

Complete the `quick_sort` and `partition` functions according to the given algorithms.

_Note: The process is started with `quick_sort(A, 0, len(A)-1)`_

### Pseudocode: quick_sort(nums, low, high)

1. If `low` is less than `high`:
    1. Partition the input list using the `partition` function
    2. Recursively call `quick_sort` on the left side of the partition
    3. Recursively call `quick_sort` on the right side of the partition

### Pseudocode: partition(nums, low, high)

1. Set `pivot` to the element at index `high`
2. Set `i` to `low`
3. For each index (`j`) from `low` to `high`
    1. If the element at index `j` is less than the `pivot`:
        1. Swap the element at index `i` with the element at index `j`
        2. Increment `i` by `1`
4. Swap the element at index `i` with the element at index `high`
5. Return the index `i`

## Answer
```python
def quick_sort(nums, low, high):
    if low < high:
        pivot_index = partition(nums, low, high)
        quick_sort(nums, low, pivot_index - 1)
        quick_sort(nums, pivot_index + 1, high)


def partition(nums, low, high):
    pivot = nums[high]
    i = low
    for j in range(low, high):
        if nums[j] < pivot:
            nums[i], nums[j] = nums[j], nums[i]
            i += 1
    nums[i], nums[high] = nums[high], nums[i]
    return i
```
# Quick Sort Big O

On average, quicksort has a Big O of `O(n*log(n))`. In the worst case, and assuming we don't take any steps to protect ourselves, it can break down to `O(n^2)`.

`partition()` has a single for-loop that ranges from the lowest index to the highest index in the array. By itself, the `partition()` function is `O(n)`. The overall complexity of quicksort is dependent on _how many times_ `partition()` is called.

In the worst case, the input is already sorted. An already sorted array results in the pivot being the largest or smallest element in the partition each time. When this is the case, `partition()` is called a total of `n` times.

In the best case, the pivot is the middle element of each sublist which results in `log(n)` calls to `partition()`.

## Reference

```python
def quick_sort(nums, low, high):
    if low < high:
        p = partition(nums, low, high)
        quick_sort(nums, low, p - 1)
        quick_sort(nums, p + 1, high)


def partition(nums, low, high):
    pivot = nums[high]
    i = low
    for j in range(low, high):
        if nums[j] < pivot:
            nums[i], nums[j] = nums[j], nums[i]
            i += 1
    nums[i], nums[high] = nums[high], nums[i]
    return i
```
# Why Use Quick Sort?

Pros:

- Very fast in the average case
- In-Place: Saves on memory, doesn't need to do a lot of copying and allocating

Cons:

- More complex implementation
- Typically unstable: changes the relative order of elements with equal keys