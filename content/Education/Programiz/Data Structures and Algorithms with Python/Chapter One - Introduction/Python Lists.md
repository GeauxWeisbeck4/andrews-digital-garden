---
id: 01JADF598YJBWC4EFM8N15HCT6
title: Python Lists
modified: 2024-10-17T11:05:49-04:00
tags:
  - dsa
  - python
  - data-structures
  - algorithms
  - programiz
  - programming
---
# Python Lists

Although we covered lists in our Python course, we will explore more Python operations and methods in this lesson.

This is because many data structures and algorithms make use of Python lists.

We assume that you already know how to create a list, access its elements one by one using a `for` loop, and access elements using an index (including negative indexing).

Now, we'll learn about list slicing in detail, and then we will cover commonly used list methods.

# Slicing a List

We can access a portion of a list using the slicing notation `:`.

Suppose we have a list like this:

```python
numbers = [10, 20, 30, 40, 50, 60 ]
```

We can create a sublist with the items `[30, 40, 50]` from `numbers` using the slicing notation as follows:

```python
numbers = [10, 20, 30, 40, 50, 60] # start: 2# end: 5new_numbers = numbers[2: 5]print(new_numbers)    # [30, 40, 50]
```

Run Code  >>

Here, `numbers[2: 5]` means that the new list's

- first item should be the item at the **2nd** index (of the `numbers` list)
- last item should be the item at the **4th** index

One crucial thing we need to remember about slicing is that the **start index is inclusive**, but the **end index is exclusive**.

Hence, `numbers[2: 5]` means create a new list with items present at index **2** up to **4** (and not **5**).

# List Slicing Examples

```python
numbers = [10, 20, 30, 40, 50, 60] 
# items from index 0 to 3
print(numbers[0: 4])    
# [10, 20, 30, 40] 
# items from index 3 to 4
print(numbers[3: 5])   
# [40, 50] 
# using negative index in slicing 
# items from the fourth item (3rd index)
# to the second-last item 
print(numbers[3: -1])   
# [40, 50]
```

Run Code  >>

Here, `numbers[3: -1]` returns a list with items from index **3** to index **-2** (because the last index **-1** is exclusive and not included in the list).

##### Practice:

# List Slicing

Easy

## Problem Description

Create a program to print a portion of the given list.

Suppose we have a list as follows:

```
animals = ['dog', 'cat', 'guinea pig', 'rat']
```

Use list slicing to extract the elements `cat` and `guinea pig` into a new list. Then, print the new list.

### Example

```
Expected Output['cat', 'guinea pig']
```

### Answer

```python
# Replace ___ with your code
animals = ['dog', 'cat', 'guinea pig', 'rat']
# extract elements using list slicing
new_animals = animals[1: 3]
# print the new list
print(new_animals)
```
# Omit Start and End Index in Slicing

It is possible to omit the start or end index while slicing.

**Omit the Start Index**

If we omit the start index, the slicing starts from the first item.

```python
numbers = [10, 20, 30, 40, 50, 60] # items from first item to third indexprint(numbers[: 4])    # [10, 20, 30, 40]
```

Run Code  >>

**Omit the End Index**

If we omit the end index, the slicing goes up to the last item.

```python
numbers = [10, 20, 30, 40, 50, 60] # items from 3rd index to last itemprint(numbers[3: ])   # [40, 50, 60]
```

Run Code  >>

**What if we omit both the start and end indexes?**

If we omit both the start and end indexes, we get a new list that contains all the items from the original list (from the first item to the last item).

```python
numbers = [10, 20, 30, 40, 50, 60] # omitting both the start and end indexes# items from first to lastprint(numbers[:])   # [10, 20, 30, 40, 50, 60]
```

### Answer

```python
# Replace ___ with your code

  

animals = ['dog', 'cat', 'guinea pig', 'rat']

  

# extract elements using list slicing

new_animals = animals[2: ]

  

# print the new list

print(new_animals)
```

# Repetition with Lists

Suppose, we need to create a list with five elements, all having the value **0**. We can easily achieve this using the `*` operator. For example,

```python
lst = [0] * 5print(lst)  # [0, 0, 0, 0, 0]
```

# List Methods

Now, let's cover a few commonly used list methods. These methods are used throughout the course.

|Method|Description|
|---|---|
|`append()`|add an element to the end of the list|
|`extend()`|add elements of an iterable (list, tuple, etc.) to the end of the list|
|`insert()`|insert an element at the specified index|
|`pop()`|remove an element from the list and return it|
|`reverse()`|reverse the elements of the list|

Let's go through each method one by one.

# List append() and extend()

**The append() method**

The `append()` method adds an element to the end of the list.

```python
currencies = ['Dollar', 'Euro', 'Pound'] # append 'Yen' to the listcurrencies.append('Yen') print(currencies)
```

Run Code  >>

**Output**

```
['Dollar', 'Euro', 'Pound', 'Yen']
```

![Idea Emoji](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAZCAYAAAArK+5dAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAOQSURBVHgBtZbPb9xEFMffzNjx/sruJoqSJUpJS0EiFFWkgOCENuIAhx75H7jAASQqFFXCXLgBFUhccuDEBS5ISESgoAABVSBUipoEKoSESts03Xh3vd61PWPPDG+8bcQhjRdon/Q0s09+34/fPD97Ae6xkbwL9OcvO+1G4bTF7GcVWCcppQxAXtAqXe+0o7VjS+e6/xnQvXTmCYuWPmClyUWwaxbQAmZgioxAp32pQu9iEvuvTpx6+9t/DQgunVmg9vgqqx6fJ84MEFoFAzD6WsUI6ILiN0EOruwmg+D5icW3Lh6kQw8Kaq0JsOJ79sTD86z8ELDCUWCleQwdQcb9uB7F9TjGHgBWfXDGLhRXNjfdsZEB7Z+Xn7bL00ukMAfUnkKvYQVlIKwIFJ1YVaAOxp37gDmzQMuNx+fStDkyYMwZO02LDUasOoqW8CAdBFh4PNhfdEJwTzHGEGRPYkXT+NN+Ias8D6C10bJPEnPXBKumFNfbl+l/OGohCExFrIJb58TWJ2/a+RV87TJKrdK+oNECNXQ9XPWt34Sg76/gnFiczK8AvgGltboM+KSATtAlVpVme63QJcaUwD1HF0OX3JR+Ha62ZS6AuK7Sif5McQ+UHKDH+0KgDTREQIQrzoIMQaU95HVA8ORjsuSm+RWglR6ZWksj7xctWvi891AoGt6xFpkrMwdJmM2CFnuQhu1fU6I+PUjrQAAhLyY8CF9KB1d8nXoo5Gd3ayZ46AOMdRDSwmKuRZBEr0w/6vZHBhibfOzsdzLunZXRX9iCdlaJksNj0UmQASTfwVZ136ksvP7FnXTuCDBWqcGHMmr9JHkHVCaKkAT7kvRBCnP2/lank7x7mMahANJ4bYDde18nCOA+eoCiQQbQAiE8/GjuqWXvMA0LcoyL9IKNwjhIOIFiOBqm2WkAIuFrefm5gMixbnBf9MbKcZXQJHv94rcA4n4omWN7efk074K5hWXv+8vjfmidgrTwJNBaE2D8GdjamaE//jab/m9AVgVXu9eu70G7G4IfBHB15wb4A/HHFBRbebm5R2QsjqW329qDMI5hol6HHkIs5sCX29v8rgDAFBELkKpvmg6JEBBGnLmuq+8KYM/zV0IumvVatd7udCCKIlmtVFbg1rv2MGMwgn21vv77eLXy5/nzPzy3ub0lbra8NxqzR86trq7KvNzcvy23rdlsWkqpY/jVsjc2NrZHzfsbqqLpy8q8XzcAAAAASUVORK5CYII=)

**Note:** The `append()` method does not return any value; it only modifies the original list.

**The extend() method**

The `extend()` method adds all the elements of an iterable (a list, tuple, string, etc.) to the end of the list.

```python
languages = ['French', 'English']languages1 = ['Spanish', 'Portuguese'] # append language1 elements to languageslanguages.extend(languages1) print(languages)
```

Run Code  >>

**Output**

```
['French', 'English', 'Spanish', 'Portuguese']
```

![Idea Emoji](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAZCAYAAAArK+5dAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAOQSURBVHgBtZbPb9xEFMffzNjx/sruJoqSJUpJS0EiFFWkgOCENuIAhx75H7jAASQqFFXCXLgBFUhccuDEBS5ISESgoAABVSBUipoEKoSESts03Xh3vd61PWPPDG+8bcQhjRdon/Q0s09+34/fPD97Ae6xkbwL9OcvO+1G4bTF7GcVWCcppQxAXtAqXe+0o7VjS+e6/xnQvXTmCYuWPmClyUWwaxbQAmZgioxAp32pQu9iEvuvTpx6+9t/DQgunVmg9vgqqx6fJ84MEFoFAzD6WsUI6ILiN0EOruwmg+D5icW3Lh6kQw8Kaq0JsOJ79sTD86z8ELDCUWCleQwdQcb9uB7F9TjGHgBWfXDGLhRXNjfdsZEB7Z+Xn7bL00ukMAfUnkKvYQVlIKwIFJ1YVaAOxp37gDmzQMuNx+fStDkyYMwZO02LDUasOoqW8CAdBFh4PNhfdEJwTzHGEGRPYkXT+NN+Ias8D6C10bJPEnPXBKumFNfbl+l/OGohCExFrIJb58TWJ2/a+RV87TJKrdK+oNECNXQ9XPWt34Sg76/gnFiczK8AvgGltboM+KSATtAlVpVme63QJcaUwD1HF0OX3JR+Ha62ZS6AuK7Sif5McQ+UHKDH+0KgDTREQIQrzoIMQaU95HVA8ORjsuSm+RWglR6ZWksj7xctWvi891AoGt6xFpkrMwdJmM2CFnuQhu1fU6I+PUjrQAAhLyY8CF9KB1d8nXoo5Gd3ayZ46AOMdRDSwmKuRZBEr0w/6vZHBhibfOzsdzLunZXRX9iCdlaJksNj0UmQASTfwVZ136ksvP7FnXTuCDBWqcGHMmr9JHkHVCaKkAT7kvRBCnP2/lank7x7mMahANJ4bYDde18nCOA+eoCiQQbQAiE8/GjuqWXvMA0LcoyL9IKNwjhIOIFiOBqm2WkAIuFrefm5gMixbnBf9MbKcZXQJHv94rcA4n4omWN7efk074K5hWXv+8vjfmidgrTwJNBaE2D8GdjamaE//jab/m9AVgVXu9eu70G7G4IfBHB15wb4A/HHFBRbebm5R2QsjqW329qDMI5hol6HHkIs5sCX29v8rgDAFBELkKpvmg6JEBBGnLmuq+8KYM/zV0IumvVatd7udCCKIlmtVFbg1rv2MGMwgn21vv77eLXy5/nzPzy3ub0lbra8NxqzR86trq7KvNzcvy23rdlsWkqpY/jVsjc2NrZHzfsbqqLpy8q8XzcAAAAASUVORK5CYII=)

**Note:** The `extend()` method also does not return any value; it only modifies the original list.

By the way, we can also use the `+` operator to extend a list in Python. For example,

```python
languages = ['French', 'English']languages1 = ['Spanish', 'Portuguese'] # append language1 elements to languageslanguages = languages + languages1 print(languages)
```

Run Code  >>

**Output**

```
['French', 'English', 'Spanish', 'Portuguese']
```

# List pop()

The `pop()` method removes the item at the specified index. The method also returns the removed item.

```python
prime_numbers = [2, 3, 5, 7] 
# remove the element at index 2
removed_element = prime_numbers.pop(2) 
print(f"Updated List: {prime_numbers}")
print(f"Removed Element: {removed_element}")
```

Run Code  >>

**Output**

```
Updated List: [2, 3, 7]
Removed Element: 5
```

If we do not pass any arguments to the `pop()` method, it removes the last element.

```python
prime_numbers = [2, 3, 5, 7] # remove the last itemprime_numbers.pop() print(f"Updated List: {prime_numbers}")
```

Run Code  >>

**Output**

```
Updated List: [2, 3, 5]
```

![Idea Emoji](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAZCAYAAAArK+5dAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAOQSURBVHgBtZbPb9xEFMffzNjx/sruJoqSJUpJS0EiFFWkgOCENuIAhx75H7jAASQqFFXCXLgBFUhccuDEBS5ISESgoAABVSBUipoEKoSESts03Xh3vd61PWPPDG+8bcQhjRdon/Q0s09+34/fPD97Ae6xkbwL9OcvO+1G4bTF7GcVWCcppQxAXtAqXe+0o7VjS+e6/xnQvXTmCYuWPmClyUWwaxbQAmZgioxAp32pQu9iEvuvTpx6+9t/DQgunVmg9vgqqx6fJ84MEFoFAzD6WsUI6ILiN0EOruwmg+D5icW3Lh6kQw8Kaq0JsOJ79sTD86z8ELDCUWCleQwdQcb9uB7F9TjGHgBWfXDGLhRXNjfdsZEB7Z+Xn7bL00ukMAfUnkKvYQVlIKwIFJ1YVaAOxp37gDmzQMuNx+fStDkyYMwZO02LDUasOoqW8CAdBFh4PNhfdEJwTzHGEGRPYkXT+NN+Ias8D6C10bJPEnPXBKumFNfbl+l/OGohCExFrIJb58TWJ2/a+RV87TJKrdK+oNECNXQ9XPWt34Sg76/gnFiczK8AvgGltboM+KSATtAlVpVme63QJcaUwD1HF0OX3JR+Ha62ZS6AuK7Sif5McQ+UHKDH+0KgDTREQIQrzoIMQaU95HVA8ORjsuSm+RWglR6ZWksj7xctWvi891AoGt6xFpkrMwdJmM2CFnuQhu1fU6I+PUjrQAAhLyY8CF9KB1d8nXoo5Gd3ayZ46AOMdRDSwmKuRZBEr0w/6vZHBhibfOzsdzLunZXRX9iCdlaJksNj0UmQASTfwVZ136ksvP7FnXTuCDBWqcGHMmr9JHkHVCaKkAT7kvRBCnP2/lank7x7mMahANJ4bYDde18nCOA+eoCiQQbQAiE8/GjuqWXvMA0LcoyL9IKNwjhIOIFiOBqm2WkAIuFrefm5gMixbnBf9MbKcZXQJHv94rcA4n4omWN7efk074K5hWXv+8vjfmidgrTwJNBaE2D8GdjamaE//jab/m9AVgVXu9eu70G7G4IfBHB15wb4A/HHFBRbebm5R2QsjqW329qDMI5hol6HHkIs5sCX29v8rgDAFBELkKpvmg6JEBBGnLmuq+8KYM/zV0IumvVatd7udCCKIlmtVFbg1rv2MGMwgn21vv77eLXy5/nzPzy3ub0lbra8NxqzR86trq7KvNzcvy23rdlsWkqpY/jVsjc2NrZHzfsbqqLpy8q8XzcAAAAASUVORK5CYII=)

**Note:** In the above program, we didn't assign the removed value to any variable. So, this data will be lost.

# List insert()

The `insert()` method inserts an element at the specified index.

```python
# create a list of vowels
vowel = ['a', 'e', 'i', 'u'] 
# insert 'o' at index 3 (4th position)
vowel.insert(3, 'o') 
print(vowel)
```

Run Code  >>

**Output**

```
['a', 'e', 'i', 'o', 'u']
```

**Note:** The `insert()` method does not return any value; it only modifies the original list.

# List reverse()

The `reverse()` method reverses the elements of the list.

```python
prime_numbers = [2, 3, 5, 7] # reverse the order of list elementsprime_numbers.reverse() print(prime_numbers) 
```

Run Code  >>

**Output**

```
[7, 5, 3, 2]
```

Note - it doesn't return any value, just modifies the list

# enumerate()

The `for` loop in Python doesn't automatically use indexes.

```python
languages = ['Python', 'Java', 'JavaScript'] for language in languages:    print(language)
```

Run Code  >>

**Output**

```
Python
Java
JavaScript
```

However, if we need to access the index during each iteration of a `for` loop, we can use the `enumerate()` function along with the loop.

First, let's see how `enumerate()` works:

```python
languages = ['Python', 'Java', 'JavaScript'] enumerate_languages = enumerate(languages) # convert enumerate object to listprint(list(enumerate_languages))
```

Run Code  >>

**Output**

```
[(0, 'Python'), (1, 'Java'), (2, 'JavaScript')]
```

As you can see, `enumerate()` adds a counter to our list and returns it.

Now, let's see how we can use it in a `for` loop.

```python
languages = ['Python', 'Java', 'JavaScript'] for index, language in enumerate(languages):    print(index, language)
```

Run Code  >>

**Output**

```
0 Python
1 Java
2 JavaScript
```

Essentially, if we need to work with indexes inside a `for` loop, we can utilize the `enumerate()` function.
## Problem Description

Create a program to perform operations on a list using various list methods.

Suppose we have two lists as follows:

```
animals = ['Dog', 'Cat']
wild_animals = ['Tiger', 'Coyote']
```

Perform these operations on the `animals` list:

- Add `'Raccoon'` to the end of the `animals` list.
- Add all the elements from `wild_animals` to the end of the `animals` list.
- Remove the third element from the `animals` list using the `pop()` method.
- Remove the last element from the `animals` list using the `pop()` method.
- Insert `'Moose'` at the third position in the `animals` list.
- Print the `animals` list.

### Example

```
Expected Output['Dog', 'Cat', 'Moose', 'Tiger']
```
### Answer

```python
# Replace ___ with your code

  

animals = ['Dog', 'Cat']

wild_animals = ['Tiger', 'Coyote']

  

# perform list operations

animals.append('Raccoon')

animals += wild_animals

animals.pop(2)

animals.pop()

animals.insert(2, 'Moose')

  

print(animals)
```