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
modified: 2024-10-17T21:46:20-04:00
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
### Answer

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

