---
id: 01J7SDMDM9G0JXSQ6022ZBPRSJ
title: Chapter Three - Compound Data Types
modified: 2024-09-14T20:14:20-04:00
description: How to create compound data types, or ways to organize multiple pieces of data into a single unit
tags:
  - compound-data-types
  - javascript
  - programming
---
# Chapter Three - Compound Data Types
- ## Arrays
	- Ordered list of values
	- Don't have to be all of the same type, but commonly are
	- Examples: to-do list, collection of numbers representing temperature readings taken at regular intervals 
	- ### Creation and Indexing
		- Here is how to create an array:
```javascript
let primes = [2, 3, 5, 7, 11, 13, 17, 19];
primes;
// (8) [2, 3, 5, 7, 11, 13, 17, 19]
> primes[0]
2
> primes[7]
17
> primes.length
9
> primes[primes.length - 1]
19
> primes[2] = 1
1
>
```
- ### Arrays of Arrays
	- Arrays can contain other arrays and become multidimensional arrays
```javascript
> let ticTacToe = [
... ["", "", ""],
... ["", "", ""],
... ["", "", ""]
... ];
undefined
> ticTacToe;
[ [ '', '', '' ], [ '', '', '' ], [ '', '', '' ] ]
> ticTacToe;
[ [ '', '', '' ], [ '', '', '' ], [ '', '', '' ] ]
> ticTacToe[2][0] = "X";
'X'
> ticTacToe
[ [ '', '', '' ], [ '', '', '' ], [ 'X', '', '' ] ]
> ticTacToe[0][2] = "O"
'O'
> ticTacToe
[ [ '', '', 'O' ], [ '', '', '' ], [ 'X', '', '' ] ]
>
```
- ### Mutations
	- #### Adding and removing elements from an array
		- See below on how to add and remove items to an array. It can be useful to use it in a way like below.
```javascript
> languages.push("Python")
1
> languages.push("Haskel");
2
> languages.push("JavaScript");
3
> languages.push("Rust");
4
> langes;
Uncaught ReferenceError: langes is not defined
> languages;
[ 'Python', 'Haskel', 'JavaScript', 'Rust' ]
> languages.pop();
'Rust'
> languages;
[ 'Python', 'Haskel', 'JavaScript' ]
> let bestLanguage = languages.pop();
undefined
> let message = `My favorite language is ${bestLanguage}.`;
undefined
> message;
'My favorite language is JavaScript.'
> languages;
[ 'Python', 'Haskel' ]
>
```
- ### Combining Arrays
	- The concat method adds arrays together. 
```javascript
> let fish = ["Salmon", "Cod", "Trout"];
undefined
> let mammals = ["Sheep", "Cat", "Tiger"];
undefined
> let animals = fish.concat(mammals);
undefined
> animals
[ 'Salmon', 'Cod', 'Trout', 'Sheep', 'Cat', 'Tiger' ]

> let originals = ["Hope", "Empire", "Jedi"];
undefined
> let prequels = ["Phantom", "Clones", "Sith"];
undefined
> let sequels = ["Awakens", "Last", "Rise"];
undefined
> let starWars = prequels.concat(originals, sequels);
undefined
> starWars;
[
  'Phantom', 'Clones',
  'Sith',    'Hope',
  'Empire',  'Jedi',
  'Awakens', 'Last',
  'Rise'
]
```
### Finding the index of an element in an array
- Use the `indexOf` method. If the element is in the array, it returns the position that the element is in. If it is not in the array it'll return -1.
- If there is multiple instances in the array, it will return the location of the first instance
```javascript
> let sizes = ["Small", "Medium", "Large"];
undefined
> sizes.indexOf("Medium");
1
> sizes.indexOf("Huge");
-1

> let flagOfArgentina = ["Blue", "White", "Blue"];
undefined
> flagOfArgentina.indexOf("Blue");
0
```
- ### Turn array into string
```javascript
> let beatles = ["John", "Paul", "George", "Ringo"];
undefined
> beatles.join();
'John,Paul,George,Ringo'
> beatles.join("");
'JohnPaulGeorgeRingo'
> console.log(beatles.join("&\n"));
John&
Paul&
George&
Ringo
undefined
> [100, true, false, "hi"].join(" - ");
'100 - true - false - hi'
```
- ### Other useful array methods
	- `arr.includes(elem)` - Returns true or false depending on if the element is in the array
	- `arr.reverse()` - Reverses the order of elements in the array. This is a mutating method, so it modifies the original array.
	- `arr.sort()` - Sorts the array elements, modifying the original array. If the elements are strings, they're sorted in alaphabetical order. Otherwise the sorting happens as if the elements were converted to strings
	- `arr.slice(start, end)` - Creates a new array by extracting elements from the original array starting at index start, up to but not including index end.
	- `arr.splie(index, count)` - Removes count elements from the array, starting at index.
- ## Objects
	- They are like arrays but hold collections of values using keys instead of numeric indices - key-value pair. You can create them using a n object literal which uses a pair of braces to enclose a series of key value pairs
	- If you want to add any characters like a space or special character like $, you must enclose them in quotations
```javascript
> casablanca;
{ title: 'Casablanca', released: 1942, director: 'Michael Curtiz' }
> let obj = { key1: 1, key_2: 2, "key 3": 3, "key#4": 4 };
undefined
> obj;
{ key1: 1, key_2: 2, 'key 3': 3, 'key#4': 4 }
```
- ### Accessing object values
	- enclose them in brackets or use dot notation
```javascript
obj["key 3"]
3
casablanca["title"];
'Casablanca'
> obj.key_2;
2
```
- ### Setting Object Values
```javascript
> let dictionary = {};
undefined
> dictionary.name = "A small rodent";
'A small rodent'
> dictionary["computer mouse"] = "A pointing device for computers";
'A pointing device for computers'
> dictionary;
{
  name: 'A small rodent',
  'computer mouse': 'A pointing device for computers'
}
> dictionary.mouse =
...
> dictionary.mouse = "A furry rodent";
'A furry rodent'
> dictionary;
{
  name: 'A small rodent',
  'computer mouse': 'A pointing device for computers',
  mouse: 'A furry rodent'
}
```
- ### Working with objects
	- #### Getting objects keys
```javascript
> let cats = { "Kiki": "black and white", "Mei": "tabby", "Moona": "gray" };
undefined
> Object.keys(cats);
[ 'Kiki', 'Mei', 'Moona' ]
```
- #### Getting Objects keys and values
```javascript
> let chromosomes = {
... koala: 16,
... snail: 24,
... giraffe: 30,
... cat: 38
... };
undefined
> Object.entries(chromosomes);
[ [ 'koala', 16 ], [ 'snail', 24 ], [ 'giraffe', 30 ], [ 'cat', 38 ] ]
```
- #### Combining objects
```javascript
> let physical = { pages: 208, binding: "Hardcover" };
undefined
> let contents = { genre: "Fiction", subgenre: "Mystery" };
undefined
> let book = {};
undefined
> Object.assign(book, physical, contents);
{
  pages: 208,
  binding: 'Hardcover',
  genre: 'Fiction',
  subgenre: 'Mystery'
}
```
- ## Nesting Objects and Arrays
	- ### Nesting with Literals
```javascript
> let trilogies = [
... {
...   title: "His Dark Materials",
...   author: "Phillip Pullman",
...   books: ["Norhtern Lights", "The Subtle Knife", "The Amber Spyglass"]
... },
... {
...   title: "Broken Earth",
...   author: "N. K. Jemisin",
...   books: ["The Fifth Season", "The Obelisk Gate", "The Stone Sky"]
... }
... ];
undefined
> trilogies[1].books[0];
'The Fifth Season'
```
- ### Nesting with variables
```javascript
> let penny = { name: "Penny", value: 1, weight: 2.5 };
undefined
> let nickel = { name: "Nickel", value: 5, weight: 5 };
undefined
> let dime = { name: "Dime", value: 10, weight: 2.268 };
undefined
> let quarter = { name: "Quarter", value: 25, weight: 5.67 };
undefined
> let change = [quarter, quarter, dime, penny, penny, penny];
undefined
> change[0].value
25
> penny.weight = 2.49
2.49
> change[3].weight;
2.49
> change[4].weight;
2.49
> change[5].weight;
2.49
```
- ### Exploring nested objects in the console
```javascript
> let nested = {
... name: "Outer",
... content: {
...   name: "Middle",
...   content: {
...     name: "Inner",
...     content: "Whoa..."
...   }
... }
... };
undefined
> nested.content.content.content
'Whoa...'
> JSON.stringify(nested);
'{"name":"Outer","content":{"name":"Middle","content":{"name":"Inner","content":"Whoa..."}}}'
> nestedJSON = JSON.stringify(nested, null, 2);
'{\n' +
  '  "name": "Outer",\n' +
  '  "content": {\n' +
  '    "name": "Middle",\n' +
  '    "content": {\n' +
  '      "name": "Inner",\n' +
  '      "content": "Whoa..."\n' +
  '    }\n' +
  '  }\n' +
  '}'
> console.log(nestedJSON);
{
  "name": "Outer",
  "content": {
    "name": "Middle",
    "content": {
      "name": "Inner",
      "content": "Whoa..."
    }
  }
}
undefined
```