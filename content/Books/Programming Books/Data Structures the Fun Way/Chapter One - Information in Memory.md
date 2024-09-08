---
id: 01J76Z3W1ZB1RT7D0Z56Y4D8HZ
modified: 2024-09-07T16:41:58-04:00
title: Chapter One - Information in Memory
description: Introduction to how computers store data in memory
tags:
  - data-structures
  - books
  - nostarchpress
  - programming
  - data
---
# Chapter One - Information in Memory
- ## Variables
	- Variables represent the location in memory for a single piece of data.
	- Have a name and type
- ## Composite Data Structures
	- The way a programming language gathers multiple individual variables into a single group - things like struct or objects.
	- A good example right off the cuff is a business card - organized in card folder. Card has email address, physical address, name, business, phone number
	- In their most general form, composite data structures are organized into collections
	- Examples:
```
// Record

CoffeeRecord {
	String: Name
	String: Brand
	Integer: Rating
	Float: Cost_Per_Pound
	Boolean: Is_Dark_Roast
	String: Other_Notes
}

// Collection - how to access

latest_record.name = "Sublime Blend"
```
- ## Array
	- Used to store multiple related values
	- Mechanism to store variables in adjacent and indexable bins
	- Effectively a row of variables - a contiguous block of equal sized bins in the computer's memory and can sit adjacent to arbitrary other information
	- For example, an array's bin can store a value of the given type, such as a number, character, pointer, or even other fixed-size data structures
	- Structure of an array allows you to access any value also known as an element, within the array by specifying its location or index.
	- We reference the value at index i aof array A as A[i]
	- Most programming languages use zero-indexed arrays which means the first value resides at index 0
	- We can compute the ith item of an arry with:
	  `Location(item i) = Location(start of array) + Size of each element X i`
		- `A[5]=16`
	- We can't squeeze things in between arrays like we could a book shelf - it would be more like if we had a row of buildings and we had to add a new building - we'd shift the others over
	- We have to juggle values simply to swap two values in an array. To swap the values at some indices i and j, we need to first assign one of them a temporary variable 
		- Otherwise we would overwrite the value in one of the bins and the two bins would end up with the same value. like example below
```
Temp = A[i]
A[i] = A[j]
A[j] = Temp
```
- ## Insertion Sort
	- The best way to understand the impact of an array's structure has on how it can be used is to examine it in the context of an actual algorithm.
	- Insertion sort is an algorithm that is used to sort the value in an array that can be ordered first.
	- Sort each subset of the array and expanding the sorted range until the entire array is in order. 
	- The algorithm iterates through each element in the unsorted array and moves it down in the correct location of the sorted section.
	- At the start of iteration i, the items in bin 0 through i - 1 are all in sorted order
	- The algorithm then takes the item at index i, find the correct location in the sorted prefix, and inserts it shifting the necessary items down to make room. The sorted prefix has now grown by one - bins 0 through i are in sorted order. We can start at i = 1 by declaring the first element to be our initial sorted prefix
	- Insertion sort isn't necessarily efficient - when inserting elements in the array we could end up shifting around significant portions of the array
		- The worst case is the cost of the algorithm scales everything proportionally or for every item in the list, we shift all the items in front of it. Doubling the size of the array increases the cost by a factor of four
	- For our example, we have the outer loop with iterator i starts at the first unsorted element i = 1 and progresses through each value in the unsorted range
	- The inner loop with iterator j shifts the current value down into the sorted prefix. 
	- At each step, we check the position within the sorted prefix by comparing the current value to the preceding location in the prefix, index j. 
	- If the element at j is larger, the two values are in the wrong order and must be swapped.
	- Since we are storing the current value in a separate variable current, we copy the data from the preceding bin directly
	- The inner loop continues until it shifts the current value to the front of the array or it finds a preceding value that is smaller, wihch indicates the current value is in the correct location of the sorted prfix. We only need to write the current value at the end of the loop when it is in the correct position. The outer loop proceeds to the next unsorted value. 
	- There's an example below with a drawing that visualizes this in Excalidraw
```
// Insertion sort pseudocode

InsertionSort(array: A):
	Integer: N = length(A)
	Integer: i = 1
	WHILE i < N:
		Type: current = A[i]
		Integer: j = i - 1
		WHILE j >= 0 AND A[j] > current:
			A[j + 1] = A[j]
			j = j - 1
		A[j + 1] = current
		i = i + 1
```
- ## Strings
	- Strings are ordered lists of characters that can often be thought of as a special kind of arrays
	- Each bin in the string holds a single character, be that a letter, number, symbol, space, or one of a limited set of special indicators
	- In some programming languages strings are directly implemented as simple arrays of characters. In others, strings may be objects and the string class serves as a wrapper around an array or other data structure holding the characters.
	- The algorithm example below shows checking the equality of two strings. The algorithm starts by comparing the strings' size. If they are not the same length, the algorithm stops there. If they are the same length, the algorithm iterates through each position and compares th erespective letters of each string. WE can terminate the loop as soon as we find a single mismatch. Only if we make it all the way to the end of the strings wihtout a mismatch can we declare the strings equal
```
StringEqual(String: str1, String: str2):
	IF length(str1) != length(str2):
		return False
	Integer: N = length(str1)
	Integer: i = 0
	WHILE i < N AND str[i] == str2[i]:
		i = i + 1
	return i == N
```

- ## Why This Matters
	- Variables and arrays are staples of introductory programming classes and thus might seem less than exciting, but they are important to examine because they provide the very foundations for computer programming and data structres. They also provide the baseline against which to evaluate dynamic data structures and their impact on algorithms. Later on, we'll see how dynamic data structures can offer different trade-offs among efficiency, flexibility, and complexity