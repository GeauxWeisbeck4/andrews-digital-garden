---
id: 01JADH4XDA4S6QAE0XNRGGCV1M
tags:
  - sorting
  - dsa
  - algorithms
  - programiz
  - programming
  - education
  - bubble-sort
modified: 2024-10-22T14:18:42-04:00
title: Bubble Sort
---
# Introduction

Bubble sort works by repeatedly comparing two adjacent elements and swapping them if they are not in the correct order.

Next, we will learn the workings of bubble sort in detail.

# Working of Bubble Sort

Suppose we need to sort a list of numbers in ascending order.

![Unsorted List](https://cdn.programiz.pro/course-images/dsa-with-python/dsa-2.2.1.png)

Figure: Unsorted List

Bubble sort works by comparing two adjacent elements and swapping them if needed.

![Comparfrequentlyap if Needed](https://cdn.programiz.pro/course-images/dsa-with-python/dsa-2.2.2.png)

Figure: Compare and Swap if Needed

So, we compare and swap all elements with their next element:

![Compare Adjacent Elements](https://cdn.programiz.pro/course-images/dsa-with-python/dsa-2.2.3.png)

Figure: Compare Adjacent Elements

After this comparison, the largest element, **16**, will be placed at the last position.

Now, we need to repeat this process up to the second-last element so that the second-largest element is placed at the second-last position.

Let's examine that next.

# Working of Bubble Sort

Let's repeat the process again up to the second-last position.

![Compare Adjacent Elements](https://cdn.programiz.pro/course-images/dsa-with-python/dsa-2.2.4.png)

Figure: Compare Adjacent Elements

Now, **15**—the second-largest—is placed in the second-last position.

If we repeat this process three more times, we will get the sorted list.

![Iterations in Bubble Sort](https://cdn.programiz.pro/course-images/dsa-with-python/dsa-2.2.5.png)

Figure: Iterations in Bubble Sort

This is how bubble sort arranges all elements in a list.

# Thought Process to Implement Bubble Sort

To implement bubble sort in Python:

**1. Compare an element with its next element and swap them if necessary.**

```python
if lst[j] > lst[j + 1]:
    # swap elements       
    lst[j], lst[j + 1] = lst[j + 1], lst[j]
```

**2. Repeat Step 1 for each element in the list using a loop.**

Since we are comparing each element to its next element, we only need to run the loop up to the second-last element.

This is because the last element will not have a next element for comparison.

```python
for j in range(len(lst) - 1):
    if lst[j] > lst[j + 1]:
        lst[j], lst[j + 1] = lst[j + 1], lst[j]
```

**3. Repeat Step 2 for each element in the list.**

Repeat the above steps for each position. With each iteration of the outer loop, one element will settle in its correct position, starting from the largest element.

```python
for i in range(len(lst)):

    for j in range(len(lst) - 1):

        if lst[j] > lst[j + 1]:   
            lst[j], lst[j + 1] = lst[j + 1], lst[j]
```

# Source Code: Bubble Sort

```python
def bubble_sort(lst):    
# outer loop to access each list element    
	for i in range(len(lst)):         
# inner loop to compare list elements        
		for j in range(len(lst) - 1):             
# swap elements if necessary            
			if lst[j] > lst[j + 1]:                
				lst[j], lst[j + 1] = lst[j + 1], lst[j]     
		
	return lst        
	
data_list = [15, 16, 6, 8, 5]
print(f"Unsorted List: {data_list}") 
sorted_list = bubble_sort(data_list) 
print(f"Sorted List: {sorted_list}")
```

Run Code  >>

**Output**

```
Unsorted List: [15, 16, 6, 8, 5]
Sorted List: [5, 6, 8, 15, 16]
```

**Note:** In this program, we compare all consecutive elements during each loop iteration. Hence, we can further optimize this bubble sort, which we will address after a couple of exercises.

##### Practice:

# Bubble Sort

Easy

## Problem Description

Can you write the bubble sort program on your own?

- Create a function named `bubble_sort()` that takes a list as its argument.
- Sort the list in ascending order within the function and return the sorted list.
- Print the sorted list from outside the function.

### Example

```
Test Input1 15 6 8 2 5 9
```

```
Expected Output[1, 2, 5, 6, 8, 9, 15]
```

```python
# Replace ___ with your code

  

  

def bubble_sort(lst):

  

# write your code here

    for i in range(len(lst)):

        for j in range(len(lst) - 1):

            if lst[j] > lst[j + 1]:

                lst[j], lst[j + 1] = lst[j + 1], lst[j]

    return lst

  

  

# take inputs and convert it to a list

  

data_list = list(map(int, input().split()))

  

  

# call the bubble sort function

  

sorted_list = bubble_sort(data_list)

  

  

# print the sorted list

  

print(sorted_list)
```

##### Practice:

# Bubble Sort in Descending Order

Easy

## Problem Description

Write a program to sort the list items in descending order using bubble sort.

- Create a function named `bubble_sort()` that takes a list as its argument.
- Sort the list in descending order within the function and return the sorted list.
- Print the sorted list from outside the function.

**Tip:** In this case, you need to swap elements if the current element is smaller than the next element.

## Answer

```python
# Replace ___ with your code

  

def bubble_sort(lst):

    # write your code here

    for i in range(len(lst)):

        for j in range(len(lst) - 1):

            if lst[j] < lst[j + 1]:

                lst[j + 1], lst[j] = lst[j], lst[j + 1]

    return lst

  
  

# take integer inputs and convert it to a list    

data_list = list(map(int, input().split()))

  

sorted_list = bubble_sort(data_list)

print(sorted_list)
```

# Optimized Bubble Sort Algorithm

Our bubble sort program carries out several unnecessary, redundant comparisons.

We can avoid these redundant tasks and save time and resources by optimizing bubble sort.

# Optimize Inner Loop

This is the bubble sort algorithm we've been using so far:

```python
def bubble_sort(lst):

    for i in range(len(lst)):
        for j in range(len(lst) - 1):           
            if lst[j] < lst[j + 1]:
                lst[j], lst[j + 1] = lst[j + 1], lst[j]
```

If we look at the inner loop, we can notice it always iterates up to the second-last element, making comparisons even for elements that are already in their correct positions.

```python
# inner loop always runs up to the second-last element
for j in range(len(lst) - 1):           
```

However, with every iteration of the outer loop, one element moves to its correct position.

![Sorted Elements in Each Iteration](https://cdn.programiz.pro/course-images/dsa-with-python/dsa-2.2.6.png)

Figure: Sorted Elements in Each Iteration

Hence, the inner loop can reduce its iterations by one with each pass of the outer loop.

```python
# for the first outer loop iteration, i is 0.
# for the second outer iteration, i is 1.
# and so on.
for j in range(len(lst) - 1 - i):
```

# Optimize Loop Logic

In bubble sort, each iteration of the outer loop places one element in its correct position.

However, if no swaps occur during an entire pass of the inner loop, we know that elements are already in their correct positions—the list is sorted.

![No Swaps Occur in a Sorted List](https://cdn.programiz.pro/course-images/dsa-with-python/dsa-2.2.7.png)

Figure: No Swaps Occur in a Sorted List

Here's how we can add this logic to our code:

```python
for i in range(len(lst)):

    # initialize a variable as False
    # to indicate that no swaps have taken place 
    swapped = False

    for j in range(len(lst) - 1 - i):

        if lst[j] > lst[j + 1]:
            lst[j], lst[j+1] = lst[j+1], lst[j]

            # if any element is swapped, 
            # set swapped to True.
            swapped = True

    # if swapped remains False after the inner loop,
    # it means none of the elements were swapped
    # therefore, the list is already sorted, and we can exit.
    if not swapped:
        break
```
# Optimize Outer Loop

Bubble sort places each element in its correct position, starting from the last position and moving towards the start.

When the correct element is placed at the second position, the correct element will be placed at the first position as well.

![List is sorted before the final iteration](https://cdn.programiz.pro/course-images/dsa-with-python/dsa-2.2.8.png)

Figure: List is Sorted Before the Final Iteration

Therefore, we can modify the outer loop from:

```python
for i in range(len(lst)):
```

To:

```python
for i in range(len(lst) - 1):
```

Next, we will integrate all these optimizations to create an optimized bubble sort program.

# Source Code: Optimized Bubble Sort

```python
# Bubble sort in ascending order
def bubble_sort(data):     
	for i in range(len(data) - 1):        
		swapped = False        
		for j in range(len(data) - 1 - i):            
			if data[j] > data[j + 1]:                
				data[j], data[j+1] = data[j+1], data[j]              
				swapped = True        
				if not swapped:            
					break            
	return data 
data_list = [4, 6, 99, 45, 0] 
sorted_list = bubble_sort(data_list)
print(sorted_list)
```

Run Code  >>

**Output**

```
[0, 4, 6, 45, 99]
```

## Question 1:
In an optimized version of bubble sort, what happens if no swaps are made in an iteration?

The algorithm terminates early because it is sorted.


##### Practice:

# Optimized Bubble Sort

Easy

## Problem Description

Write a program to sort the list items in descending order using optimized bubble sort.

- Create a function named `bubble_sort()` that takes a list as an argument.
- Inside the function, sort the list in descending order using optimized bubble sort.
- Print the sorted list outside the function.

### Example

```
Test Input1 15 6 8 2 5 9
```

```
Expected Output[15, 9, 8, 6, 5, 2, 1]
```

## Answer

```python
# Replace ___ with your code

  

def bubble_sort(lst):

    # write your code here

    for i in range(len(lst) - 1):

        swapped = False

        for j in range(len(lst) - 1 - i):

            if lst[j] < lst[j + 1]:

                lst[j], lst[j + 1] = lst[j + 1], lst[j]

                swapped = True

                if not swapped:

                    break

    return lst

  
  
  

# take integer inputs and convert it to a list

data_list = list(map(int, input().split()))

  

sorted_list = bubble_sort(data_list)

print(sorted_list)
```

# Space Complexity

The space complexity of the bubble sort is `O(1)` because it only requires a constant amount of additional space for its variables.

# Applications of Bubble Sort

**When to use bubble sort?**

Use bubble sort for

- Small datasets.
- When simplicity is more important than efficiency.
- When the data is already partially sorted.

**When not to use bubble sort?**

Avoid bubble sort for large datasets and performance-critical applications.

In such cases, sorting algorithms like **merge sort** or **quick sort** are more suitable.