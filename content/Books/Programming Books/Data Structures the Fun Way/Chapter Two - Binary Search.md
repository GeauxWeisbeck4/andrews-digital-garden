---
id: 01J7F9Y856YNWZ1PMQGJX9MFZZ
modified: 2024-09-30T20:25:34-04:00
title: Chapter Two - Binary Search
description: Learn about binary search and how to use it.
tags:
  - data-structures
  - algorithms
  - books
---
- # Binary Search
	- An algorithm for efficiently searching a sorted list.
	- Checks the sorted list for a target value by repeatedly dividing the list in half, determining which of the two halves could contain the target value, discarding the other half.
- ## The Problem
	- We need to find a single item in a list that matches the given target value
	- Formally we can define it as:
		- `Given a set of N data points X = {x1, x2, ...., xN} and a target value x', find a point x = X such that x' = xi, or indicate that no such point exists`
- ## Linear Scan
	- Start with simpler algorithm, teh linear scan, to provide a baseline for comparison. Searches for a target value by testing each value in our list, one after the other, agianst the target value, until the target is found or we reach the end of our list.
	- 