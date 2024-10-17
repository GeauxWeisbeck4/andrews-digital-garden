---
id: 01JADKR36K74QCGCRJ2VYQNMME
title: Algorithmic Complexity
tags:
  - dsa
  - algorithms
  - algorithmic-complexity
  - programming
  - roadmaps
modified: 2024-10-17T12:08:47-04:00
---
# Algorithmic Complexity

“Algorithmic Complexity” refers to the computing resources needed by an algorithm to solve a problem. These computing resources can be the time taken for program execution (time complexity), or the space used in memory during its execution (space complexity). The aim is to minimize these resources, so an algorithm that takes less time and space is considered more efficient. Complexity is usually expressed using Big O notation, which describes the upper bound of time or space needs, and explains how they grow in relation to the input size. It’s important to analyze and understand the algorithmic complexity to choose or design the most efficient algorithm for a specific use-case.

# Time vs Space Complexity

In the context of algorithmic complexity, “time” refers to the amount of computational time that the algorithm takes to execute, while “space” refers to the amount of memory that the algorithm needs to complete its operation. The time complexity of an algorithm quantifies the amount of time taken by an algorithm to run, as a function of the size of the input to the program. The space complexity of an algorithm quantifies the amount of space or memory taken by an algorithm to run, as a function of the size of the input to the program. It’s important to note that time and space are often at odds with each other; optimizing an algorithm to be quicker often requires taking up more memory, and decreasing memory usage can often make the algorithm slower. This is known as the space-time tradeoff.

Learn more from the following resources:

Free Resources

---

- [articleCheat Sheet](https://www.bigocheatsheet.com/)
- [videoBig O Notation — Calculating Time Complexity](https://www.youtube.com/watch?v=Z0bH0cMY0E8)
- [videoFree Code Camp Big-O Tutorial](https://youtu.be/Mo4vesaut8g?si=1jyb-EkfCLf9PNND)

# How to Calculate Complexity?

The process of calculating algorithmic complexity, often referred to as Big O notation, involves counting the operations or steps an algorithm takes in function of the size of its input. The aim is to identify the worst-case, average-case, and best-case complexity. Generally, the main focus is on the worst-case scenario which represents the maximum number of steps taken by an algorithm. To calculate it, you consider the highest order of size (n) in your algorithm’s steps. For instance, if an algorithm performs a loop 5 times for ‘n’ items, and then does 3 unrelated steps, it has a complexity of O(n), because the linear steps grow faster than constant ones as n increases. Other complexities include O(1) for constant complexity, O(n) for linear complexity, O(n^2) for quadratic complexity, and so on, based on how the steps increase with size.

Learn more from the following resources:

Free Resources

---

- [videoTime & Space Complexity](https://www.youtube.com/watch?v=Z0bH0cMY0E8)
- [videoHow Write and Analyze Algorithm](https://www.youtube.com/watch?v=xGYsEqe9Vl0)