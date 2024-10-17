---
id: 01JADF598YJBWC4EFM8N15HCT6
title: Introduction to DSA
modified: 2024-10-17T10:49:27-04:00
tags:
  - dsa
  - python
  - data-structures
  - algorithms
  - programiz
  - programming
---
# Data Structures

Let's first briefly understand data structures.

A data structure is a way of storing data in an organized manner so that it can be accessed efficiently. We can classify such structures into two types.

**Fundamental Data Structures**

We know about lists, dictionaries, sets, etc., in Python—they are the built-in data structures of the language. Such data structures are called fundamental data structures.

Each of these data structures operates uniquely, and our choice among them depends on our specific requirements. For instance, it's better to use dictionaries if we need to work with **key-value** pairs, even though we can perform the same task using a list.

**Custom Data Structures**

Built-in data structures are not enough for writing complex programs.

Imagine working on a mapping service like Google Maps.

If we rely solely on built-in data structures to create a mapping service, our program won't be efficient.

In such cases, we employ logic to develop custom data structures such as graphs, trees, hash maps, etc.

![Idea Emoji](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAZCAYAAAArK+5dAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAOQSURBVHgBtZbPb9xEFMffzNjx/sruJoqSJUpJS0EiFFWkgOCENuIAhx75H7jAASQqFFXCXLgBFUhccuDEBS5ISESgoAABVSBUipoEKoSESts03Xh3vd61PWPPDG+8bcQhjRdon/Q0s09+34/fPD97Ae6xkbwL9OcvO+1G4bTF7GcVWCcppQxAXtAqXe+0o7VjS+e6/xnQvXTmCYuWPmClyUWwaxbQAmZgioxAp32pQu9iEvuvTpx6+9t/DQgunVmg9vgqqx6fJ84MEFoFAzD6WsUI6ILiN0EOruwmg+D5icW3Lh6kQw8Kaq0JsOJ79sTD86z8ELDCUWCleQwdQcb9uB7F9TjGHgBWfXDGLhRXNjfdsZEB7Z+Xn7bL00ukMAfUnkKvYQVlIKwIFJ1YVaAOxp37gDmzQMuNx+fStDkyYMwZO02LDUasOoqW8CAdBFh4PNhfdEJwTzHGEGRPYkXT+NN+Ias8D6C10bJPEnPXBKumFNfbl+l/OGohCExFrIJb58TWJ2/a+RV87TJKrdK+oNECNXQ9XPWt34Sg76/gnFiczK8AvgGltboM+KSATtAlVpVme63QJcaUwD1HF0OX3JR+Ha62ZS6AuK7Sif5McQ+UHKDH+0KgDTREQIQrzoIMQaU95HVA8ORjsuSm+RWglR6ZWksj7xctWvi891AoGt6xFpkrMwdJmM2CFnuQhu1fU6I+PUjrQAAhLyY8CF9KB1d8nXoo5Gd3ayZ46AOMdRDSwmKuRZBEr0w/6vZHBhibfOzsdzLunZXRX9iCdlaJksNj0UmQASTfwVZ136ksvP7FnXTuCDBWqcGHMmr9JHkHVCaKkAT7kvRBCnP2/lank7x7mMahANJ4bYDde18nCOA+eoCiQQbQAiE8/GjuqWXvMA0LcoyL9IKNwjhIOIFiOBqm2WkAIuFrefm5gMixbnBf9MbKcZXQJHv94rcA4n4omWN7efk074K5hWXv+8vjfmidgrTwJNBaE2D8GdjamaE//jab/m9AVgVXu9eu70G7G4IfBHB15wb4A/HHFBRbebm5R2QsjqW329qDMI5hol6HHkIs5sCX29v8rgDAFBELkKpvmg6JEBBGnLmuq+8KYM/zV0IumvVatd7udCCKIlmtVFbg1rv2MGMwgn21vv77eLXy5/nzPzy3ub0lbra8NxqzR86trq7KvNzcvy23rdlsWkqpY/jVsjc2NrZHzfsbqqLpy8q8XzcAAAAASUVORK5CYII=)

**Note:** Custom data structures are developed using fundamental data structures.

This course will cover all the major data structures, including various types of linked lists, heap, trees, and graphs.

# Algorithms

An algorithm is a step-by-step process to solve a problem.

For example, here is an algorithm to check whether a given number is odd or even.

```
Step 1: Input: A number n
Step 2: If n is perfectly divisible by 2, 
         - Then print "The number is even" 
Step 3: Else
         - Print "The number is odd"
Step 4: End
```

If we implement this process, we will be able to write a program easily to check whether a number is even or odd.

```python
n = 5 if n % 2 == 0:    
	print("The number is even")
else:    
	print("The number is odd")
```

Run Code  >>

This course will cover many commonly used algorithms, including various searching, sorting, pathfinding, and traversal algorithms.