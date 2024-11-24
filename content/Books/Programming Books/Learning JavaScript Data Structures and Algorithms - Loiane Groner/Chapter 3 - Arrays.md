---
id: 01JCKZKA7VD1TW859TST3JZJPH
title: Chapter 3 - Arrays
modified: 2024-11-13T19:01:43-05:00
---
# 3 Arrays

**Before you begin: Join our book community on Discord**

Give your feedback straight to the author himself and chat to other early readers on our Discord server (find the "learning-javascript-dsa-4e" channel under EARLY ACCESS SUBSCRIPTION).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file0.png)

[https://packt.link/EarlyAccess/](https://packt.link/EarlyAccess/)

An **array** is the simplest memory data structure. For this reason, all programming languages have a built-in array datatype. JavaScript also supports arrays natively, even though its first version was released without array support. In this chapter, we will dive into the array data structure and its capabilities.

An array stores values sequentially that are all the same datatype. Although JavaScript allows us to create arrays with values from different datatypes, we will follow best practices and assume that we cannot do this (most languages do not have this capability).

## Why should we use arrays?

Let's consider that we need to store the average temperature of each month of the year of the city that we live in. We could use something like the following code snippet to store this information:

```
const averageTempJan = 12;
const averageTempFeb = 15;
const averageTempMar = 18;
const averageTempApr = 20;
const averageTempMay = 25;
```

However, this is not the best approach. If we store the temperature for only one year, we could manage 12 variables. However, what if we need to store the average temperature for 50 years? Fortunately, this is why arrays were created, and we can easily represent the same information mentioned earlier as follows:

```
const averageTemp = [12, 15, 18, 20, 25];
// or
averageTemp[0] = 12;
averageTemp[1] = 15;
averageTemp[2] = 18;
averageTemp[3] = 20;
averageTemp[4] = 25;
```

We can also represent the `averageTemp` array graphically:

![Figure 3.1:](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file9.png)

Figure 3.1:

## Creating and initializing arrays

Declaring, creating, and initializing an array in JavaScript is straightforward, as shown in the following example:

```
let daysOfWeek = new Array(); // {1}
daysOfWeek = new Array(7); // {2}
daysOfWeek = new Array('Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'); // {3}
// preferred
daysOfWeek = []; // {4}
daysOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']; // {5}
```

We can:

- Line `{1}`: declare and instantiate a new array using the keyword `new` – this will create an empty array.
- Line `{2}`: create an empty array specifying the _length_ of the array (how many elements we are planning to store in the array).
- Line `{3}`: create and initialize the array, passing the elements directly in the constructor.
- Line `{4}`: create an empty array assigning empty brackets (`[]`). Using the keyword `new` is not considered a best practice, therefore, using brackets is the preferred way.
- Line `{5}`: create and initialize the array using brackets as a best practice.

If we want to know how many elements are in the array (its size), we can use the `length` property. The following code will give an output of `7`:

```
console.log('daysOfWeek.length', daysOfWeek.length); // output: 7
```

## Accessing elements and iterating an array

To access a specific position of the array, we can also use brackets, passing the index of the position we would like to access. For example, let's say we want to output all the elements from the `daysOfWeek` array. To do so, we need to loop the array and print the elements, starting from index 0 as follows:

```
for (let i = 0; i < daysOfWeek.length; i++) {
  console.log(`daysOfWeek[${i}]`, daysOfWeek[i]);
}
```

Let's look at another example. Suppose that we want to find out the first 20 numbers of the _Fibonacci_ sequence. The first two numbers of the Fibonacci sequence are 1 and 2, and each subsequent number is the sum of the previous two numbers:

```
// Fibonacci: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...
const fibonacci = []; // {1}
fibonacci[1] = 1; // {2}
fibonacci[2] = 1; // {3}
// create the fibonacci sequence starting from the 3rd element
for (let i = 3; i < 20; i++) {
  fibonacci[i] = fibonacci[i - 1] + fibonacci[i - 2]; // //{4}
}
// display the fibonacci sequence
for (let i = 1; i < fibonacci.length; i++) { // {5}
  console.log(`fibonacci[${i}]`, fibonacci[i]); // {6}
}
```

The following is the explanation for the preceding code:

1. In line `{1}`, we declared and created an array.
2. In lines `{2}` and `{3}`, we assigned the first two numbers of the Fibonacci sequence to the second and third positions of the array (in JavaScript, the first position of the array is always referenced by 0 (zero), and since as there is no zero in the Fibonacci sequence, we will skip it).
3. Then, all we need to do is create the third to the 20th number of the sequence (as we know the first two numbers already). To do so, we can use a loop and assign the sum of the previous two positions of the array to the current position (line `{4}`, starting from index 3 of the array to the 19th index).
4. Then, to take a look at the output (line `{6}`), we just need to loop the array from its first position to its length (line `{5}`).

> We can use `console.log` to output each index of the array (lines `{5}` and `{6}`), or we can also use `console.log(fibonacci)` to output the array itself.

If you would like to generate more than 20 numbers of the Fibonacci sequence, just change the number 20 to whatever number you like.

### Using the for..in loop

The benefit of using the `for..in` loop, is we do not have to keep track of the length of the array, as the loop will iterate through all the array indexes. The following code achieves the same output as the previous `for` loop.

```
for (const i in fibonacci) {
  console.log(`fibonacci[${i}]`, fibonacci[i]);
}
```

It is another way to write the loop, and you can use the one you feel most comfortable with.

### Using the for…of loop

Another approach, in case you would like to directly extract the values of the array, is to use the `for..of` loop as follows:

```
for (const value of fibonacci) {
  console.log('value', value);
}
```

With this loop, we do not need to access each index of the array to retrieve the value, as the value present in each position can be accessed directly in the loop.

## Adding elements

Adding and removing elements from an array is not that difficult; however, it can be tricky. For the examples we will create in this section, let's consider that we have the following numbers array initialized with numbers from 0 to 9:

```
let numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
```

### Inserting an element at the end of the array

If we want to add a new element to this array (for example, the number 10), all we have to do is reference the last free position of the array and assign a value to it:

```
numbers[numbers.length] = 10;
```

> In JavaScript, an array is a mutable object. We can easily add new elements to it. The object will grow dynamically as we add new elements to it. In many other languages, such as C and Java, we need to determine the size of the array, and if we need to add more elements to the array, we need to create a completely new array; we cannot simply add new elements to it as we need them.

#### Using the push method

The JavaScript API also has a method called `push` that allows us to add new elements to the end of an array. We can add as many elements as we want as arguments to the push method:

```
numbers.push(11); 
numbers.push(12, 13);
```

The output of the numbers array will be the numbers from 0 to 13.

### Inserting an element in the first position

Suppose we need to add a new element to the array (the number `-1`) and would like to insert it in the first position, not the last one. To do so, first we need to free the first position by shifting all the elements to the right. We can loop all the elements of the array, starting from the last position (value of `length` will be the end of the array) and shifting the previous element (`i-1`) to the new position (`i`) to finally assign the new value we want to the first position (index 0). We can create a function to represent this logic or even add a new method directly to the Array prototype, making the `insertAtBeginning` method available to all array instances. The following code represents the logic described here:

```
Array.prototype.insertAtBeginning = function(value) { 
    for (let i = this.length; i >= 0; i--) { 
      this[i] = this[i - 1]; 
    } 
    this[0] = value; 
  };  
numbers.insertAtBeginning(-1);
```

We can represent this action with the following diagram:

![Figure 3.2:](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file10.png)

Figure 3.2:

#### Using the unshift method

The JavaScript Array class also has a method called `unshift`, which inserts the values passed in the method's arguments at the start of the array (the logic behind-the-scenes has the same behavior as the `insertAtBeginning` method):

```
numbers.unshift(-2); 
numbers.unshift(-4, -3); 
```

So, using the `unshift` method, we can add the value -2 and then -3 and -4 to the beginning of the `numbers` array. The output of this array will be the numbers from -4 to 13.

## Removing elements

So far, you have learned how to add elements in the array. Let's look at how we can remove a value from an array.

### Removing an element from the end of the array

To remove a value from the end of an array, we can use the pop method:

```
numbers.pop(); // number 13 is removed
```

The `pop` method also returns the value that is being removed and it returns `undefined` in case no element is being removed (the array is empty). So, if needed, we can also capture the value that is being returned into a variable or into the console instead:

```
console.log('Removed element: ', numbers.pop());
```

The output of our array will be the numbers from -4 to 12 (after removing one number). The length (size) of our array is 17.

> The `push` and `pop` methods allow an array to emulate a basic `stack` data structure, which is the subject of the next chapter.

### Removing an element from the first position

To manually remove a value from the beginning of the array, we can use the following code:

```
for (let i = 0; i < numbers.length; i++) { 
  numbers[i] = numbers[i + 1]; 
} 
```

We can represent the previous code using the following diagram:

![Figure 3.3:](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file11.png)

Figure 3.3:

We shifted all the elements one position to the left. However, the length of the array is still the same (`16`), meaning we still have an extra element in our array (with an `undefined` value). The last time the code inside the loop was executed, `i+1` was a reference to a position that does not exist. In some languages, such as Java, C/C++, or C#, the code would throw an exception, and we would have to end our loop at `numbers.length -1`.

We have only overwritten the array's original values, and we did not really remove the value (as the length of the array is still the same and we have this extra `undefined` element).

To remove the value from the array, we can also create a `removeFromBeginning` method with the logic described in this topic. However, to really remove the element from the array, we need to create a new array and copy all values other than `undefined` values from the original array to the new one and assign the new array to our variable. To do so, we can also create a `reIndex` method as follows:

```
Array.prototype.reIndex = function(myArray) {  
    const newArray = []; 
    for(let i = 0; i < myArray.length; i++ ) { 
        if (myArray[i] !== undefined) { 
            newArray.push(myArray[i]); 
        } 
    } 
    return newArray; 
} 
// remove first position manually and reIndex 
Array.prototype.removeFromBeginning = function() { 
    for (let i = 0; i < this.length; i++) { 
        this[i] = this[i + 1]; 
    } 
    return this.reIndex(this); 
}; 
numbers = numbers.removeFromBeginning();
```

> The preceding code should be used only for educational purposes and should not be used in real projects. To remove the first element from the array, we should always use the `shift` method, which is presented in the next section.

#### Using the shift method

To remove an element from the beginning of the array, we can use the shift method, as follows:

```
numbers.shift(); 
```

If we consider that our array has the values -4 to 12 and a length of 17 after we execute the previous code, the array will contain the values -3 to 12 and have a length of 16.

> The `shift` and `unshift` methods allow an array to emulate a basic `queue` data structure, which is the subject of _Chapter 5, Queues and Deques._

## Adding and removing elements from a specific position

So far, we have learned how to add elements at the end and at the beginning of an array, and we have also learned how to remove elements from the beginning and end of an array. What if we also want to add or remove elements from any position in our array? How can we do this?

We can use the `splice` method to remove an element from an array by specifying the position/index that we would like to delete from and how many elements we would like to remove, as follows:

```
numbers.splice(5,3); 
```

This code will remove three elements, starting from index `5` of our array. This means the elements `numbers[5]`, `numbers[6]`, and `numbers[7]` will be removed from the numbers array. The content of our array will be `-3, -2, -1, 0, 1, 5, 6, 7, 8, 9, 10, 11`, and `12` (as the numbers `2`, `3`, and `4` have been removed).

> As with JavaScript arrays and objects, we can also use the `delete` operator to remove an element from the array, for example, `delete numbers[0]`. However, position `0` of the array will have the value `undefined`, meaning that it would be the same as doing `numbers[0] = undefined` and we would need to re-index the array. For this reason, we should always use the `splice`, `pop`, or `shift` methods to remove elements.

Now, let's say we want to insert numbers 2, 3 and 4 back into the array, starting from position 5. We can again use the splice method to do this:

```
numbers.splice(5, 0, 2, 3, 4); 
```

The first argument of the method is the index we want to remove elements from or insert elements into. The second argument is the number of elements we want to remove (in this case, we do not want to remove any, so we will pass the value 0 (zero)). And from the third argument onward we have the values we would like to insert into the array (the elements 2, 3, and 4). The output will be values from -3 to 12 again.

Finally, let's execute the following code:

```
numbers.splice(5, 3, 2, 3, 4); 
```

The output will be values from -3 to 12. This is because we are removing three elements, starting from the index 5, and we are also adding the elements 2, 3, and 4, starting at index 5.

## Iterator methods

JavaScript also has some built in methods as part of the Array API that are extremely useful in the day-to-day coding tasks. These methods accept a callback function that we can use to manipulate the data in the array as needed.

Let's look at these methods. Consider the following array used as a base for the examples in this section:

```
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
```

### Iterating using the forEach method

If we need the array to be completely iterated no matter what, we can use the `forEach` function. It has the same result as using a `for` loop with the function's code inside it, as follows:

```
numbers.forEach((value, index) => {
  console.log(`numbers[${index}]`, value);
});
```

Most of the times, we are only interested in using the value coming from each position of the array, without having to access each position as the preceding example. Following is a more concise example:

```
numbers.forEach(value => console.log(value));
```

Depending on personal preference you can use this method or the traditional `for` loop. Performance wise, both approaches are _O(n)_, meaning linear time, as it will iterate through all the values of the array.

### Iterating using the every method

The `every` method iterates each element of the array until the function returns `false`. Let's see an example:

```
const isBelowSeven = numbers.every(value => value < 7);
console.log('All values are below 7?:', isBelowSeven); // false
```

The method will iterate through every value within the array until it finds a value equal or bigger than 7. For the preceding example, it returns `false` as we have the value 7 within our array of numbers. If we did not have values bigger or equal to 7, the variable `isBelowSeven` would have the value `true`.

We can rewrite this example using a for loop to understand how it works internally:

```
let isBelowSevenForLoop = true;
for (let i = 0; i < numbers.length; i++) {
  if (numbers[i] >= 7) {
    isBelowSevenForLoop = false;
    break;
  }
}
console.log('All values are below 7?:', isBelowSevenForLoop);
```

The `break` statement will stop the loop at the moment that a value equal or bigger to 7 is found.

By using the `every` method, we can have more concise code to achieve the same result.

### Iterating using the some method

The `some` method has the opposite behavior to the `every` method. However, the `some` method iterates each element of the array until the return of the function is `true`. Here's an example:

```
const isSomeValueBelowSeven = numbers.some(value => value < 7);
console.log('Is any value below 7?:', isSomeValueBelowSeven); // true
```

In this example, the first number of the array is 1, and it will return true right away, stopping the execution of the code.

We can rewrite the preceding code using a for loop `for` better understanding of the logic:

```
let isSomeValueBelowSevenForLoop = false;
for (let i = 0; i < numbers.length; i++) {
  if (numbers[i] < 7) {
    isSomeValueBelowSevenForLoop = true;
    break;
  }
}
```

## Searching an array

The JavaScript API provides a few different methods we can use to search or find elements in an array. Although we will learn how to re-create classic algorithms to search elements in _Chapter 15, Searching and Shuffling Algorithms_, it is always good to know we can use existing APIs without having to write the code ourselves.

Let's take a look at the existing JavaScript methods that allows us to search elements in an array.

### Searching with indexOf, lastIndexOf and includes methods

The methods `indexOf`, `lastIndexOf` and `includes` have a very similar sintax as follows:

- `indexOf(element, fromIndex)`: searches for the `element` starting from the index `fromIndex`, and in case the element exists, returns its index, otherwise returns the value `-1`.
- `includes(element, fromIndex)`: searches for the `element` starting from the index `fromIndex`, and in case the element exists, returns `true`, otherwise returns `false`.

If we try to search for a number in our `numbers` array, let's check if the number 5 exists:

```
console.log('Index of 5:', numbers.indexOf(5)); // 4
console.log('Index of 11:', numbers.indexOf(11)); // -1
console.log('Is 5 included?:', numbers.includes(5)); // true
console.log('Is 11 included?:', numbers.includes(11)); // false
```

If we would like to search in the entire array, we can omit the `fromIndex`, and by default, the index 0 will be used.

The `lastIndexOf` is similar as well, however, it will return the index of the last element found that matches the element we are searching. Think about it as a search from the end of the array towards the beginning of the array instead:

```
console.log('Last index of 5:', numbers.lastIndexOf(5)); // 4
console.log('Last index of 11:', numbers.lastIndexOf(11)); // -1
```

This method is useful when we have duplicate elements in the array.

### Searching with find, findIndex and findLastIndex methods

In real world tasks, we often work with more complex objects. The `find` and `findIndex` methods are especially useful for more complex scenarios, but it does not mean we cannot use them for simpler cases.

Both `find` and `findIndex` methods receive a callback function that will search for an element that satisfies the condition presented in the testing function (callback). Let's start with a simple example: suppose you want to find the first number in the array that has a value below 7. We can use the following code:

```
const firstValueBelowSeven = numbers.find(value => value < 7);
console.log('First value below 7:', firstValueBelowSeven); // 1
```

We are using a callback function that is an arrow function to test every element of the array (`value < 7`), and the first element that returns `true` will be returned. That is why the output is `1`, as it is the first element of the array.

The `findIndex` method is similar, however it will return the index of the element instead of the element itself:

```
console.log('Index: ', numbers.findIndex(value => value < 7)); // 0
```

Likewise, there is also a findLastIndex method, which will return the last index of the element that matches the callback function:

```
console.log('Index of last value below 7:', numbers.findLastIndex(value => value < 7)); // 5
```

In the preceding example, the index 5 is returned because the number 6 is the last element in the array lower than 7.

Now let's check a more complex example, closer to real life. Consider the following array, a collection of books:

```
const books = [
    { id: 1, title: 'The Fellowship of the Ring' },
    { id: 2, title: 'Fourth Wing' },
    { id: 3, title: 'A Court of Thorns and Roses' }
];
```

If we need to find the book with `id` 2, we can use the find method:

```
console.log('Book with id 2:', books.find(book => book.id === 2));
```

It will output `{ id: 2, title: 'Fourth Wing' }`. If we try to find the book "The Hobbit," we will get the output undefined, because this book is not present in the array:

```
console.log(books.find(book => book.title === 'The Hobbit'));
```

Suppose we would like to remove the book with `id` 3 from our array. We can find the index of the book first, and then use the method `splice` to remove the book in the given index:

```
const bookIndex = books.findIndex(book => book.id === 3);
if (bookIndex !== -1) {
    books.splice(bookIndex, 1);
}
```

And of course, it is always good to check if the book was found first (the `bookIndex` is different than -1) before trying to remove the book from the list to avoid any errors in our logic.

### Filtering elements

Let's revisit the following example one more time:

```
const firstValueBelowSeven = numbers.find(value => value < 7);
console.log('First value below 7:', firstValueBelowSeven); // 1
```

The `find` method returns the first element that matches the given condition. What if we would like to know all elements below 7 in the array? That is when the `filter` method comes in handy:

```
const valuesBelowSeven = numbers.filter(value => value < 7);
console.log('Values below 7:', valuesBelowSeven); // [1, 2, 3, 4, 5, 6]
```

The `filter` method returns an array of all matching elements, and the output will be: `[1, 2, 3, 4, 5, 6]`.

## Sorting elements

Throughout this book, you will learn how to write the most used sorting algorithms. However, JavaScript also has a sorting method available which we can use without having to write our own logic whenever we need to sort arrays.

First, let's take our numbers array and put the elements out of order (`[1, 2, 3, ... 10]` is already sorted). To do this, we can apply the `reverse` method, in which the last item will be the first and vice versa, as follows:

```
numbers.reverse();
```

So now, the output for the numbers array will be [`10, 9, 8, 7, 6, 5, 4, 3, 2, 1`]. Then, we can apply the sort method as follows:

```
numbers.sort();
```

However, if we output the array, the result will be [`1, 10, 2, 3, 4, 5, 6, 7, 8, 9`]. This is not ordered correctly. This is because the `sort` method in JavaScript sorts the elements _lexicographically_, and it assumes all the elements are _strings_.

We can also write our own comparison function. As our array has numeric elements, we can write the following code:

```
numbers.sort((a, b) => a - b);
```

This code will return a negative number if `b` is bigger than `a`, a positive number if `a` is bigger than `b`, and 0 (zero) if they are equal. This means that if a negative value is returned, it implies that `a` is smaller than `b`, which is further used by the `sort` function to arrange the elements.

The previous code can be represented by the following code as well:

```
function compareNumbers(a, b) {
    if (a < b) {
      return -1;
    }
    if (a > b) {
      return 1;
    }
    // a must be equal to b
    return 0;
  }
  numbers.sort(compareNumbers);
```

This is because the `sort` function from the JavaScript Array class can receive a parameter called `compareFunction`, which is responsible for sorting the array. In our example, we declared a function that will be responsible for comparing the elements of the array, resulting in an array sorted in ascending order.

### Custom sorting

We can sort an array with any type of object in it, and we can also create a `compareFunction` to compare the elements as required. For example, suppose we have an object, Person, with name and age, and we want to sort the array based on the age of the person. We can use the following code:

```
const friends = [
  { name: 'Frodo', age: 30 },
  { name: 'Violet', age: 18 },
  { name: 'Aelin', age: 20 }
];
const compareFriends = (friendA, friendB => friendA.age - friendB.age;
friends.sort(compareFriends);
console.log('Sorted friends:', friends);
```

In this case, the output from the previous code will be Violet (18), Aelin (20), and Frodo (30).

### Sorting Strings

Suppose we have the following array:

```
let names = ['Ana', 'ana', 'john', 'John'];
console.log(names.sort());
```

What do you think would be the output? The answer is as follows:

```
["Ana", "John", "ana", "john"]
```

Why does `ana` come after `John` when `a` comes first in the alphabet? The answer is because JavaScript compares each character according to its **ASCII** value ([http://www.asciitable.com](http://www.asciitable.com/)).

For example, A, J, a, and j have the decimal ASCII values of A: 65, J: 74, a: 97, and j: 106. Therefore, J has a lower value than a, and because of this, it comes first in the alphabet.

Now, if we pass a function to the `sort` method, which contains the code to ignore the case of the letter, we will have the output [`"Ana", "ana", "john", "John"`], as follows:

```
names = ['Ana', 'ana', 'john', 'John']; // reset the array to its original state
names.sort((a, b) =>  {
  const nameA = a.toLowerCase();
  const nameB = b.toLowerCase();
  if (nameA < nameB) {
    return -1;
  }
  if (nameA > nameB) {
    return 1;
  }
  return 0;
});
```

In this case, the sort function will not have any effect; it will obey the current order of lower and uppercase letters.

If we want lowercase letters to come first in the sorted array, then we need to use the `localeCompare` method:

```
names.sort((a, b) => a.localeCompare(b));
```

The output will be `['ana', 'Ana', 'john', 'John']`.

For accented characters, we can use the `localeCompare` method as well:

```
const names2 = ['Maève', 'Maeve'];
console.log(names2.sort((a, b) => a.localeCompare(b)));
```

The output will be `['Maeve', 'Maève']`.

## Transforming an array

JavaScript also has support to methods that can modify the elements of the array or change its order. We have covered two transformative methods so far: `reverse` and `sort`. Let's learn about other useful methods that can transform the array.

### Mapping values of an array

The `map` method is one of the most used methods in daily coding tasks when using JavaScript or TypeScript. Let's see it in action:

```
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const squaredNumbers = numbers.map(value => value * value);
console.log('Squared numbers:', squaredNumbers);
```

Suppose we would like to find the square of each number in an array. We can use the map method to transform each value within the array and return an array with the results. For our example, the output will be: `[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]`.

We could rewrite the preceding code using a `for` loop to achieve the same result:

```
const squaredNumbersLoop = [];
for (let i = 0; i < numbers.length; i++) {
  squaredNumbersLoop.push(numbers[i] * numbers[i]);
}
```

And this is why the map method is often used, as it saves time when we have to modify all the values within the array.

### Splitting into an array and joining into a string

Imagine we have a CSV file with different names, delimited by a comma, and we would like to have each of these values and added to an array for processing (maybe they need to be persisted in a database by an API). We can use the String `split` method, which will return an array of the values:

```
const namesFromCSV = 'Aelin,Gandalf,Violet,Poppy';
const names = namesFromCSV.split(',');
console.log('Names:', names); // ['Aelin', 'Gandalf', 'Violet', 'Poppy']
```

And if instead of a comma separated file we need to use a semi-colon, we can use the `join` method of the JavaScript array class to output a single string with the array values:

```
const namesCSV = names.join(';');
console.log('Names CSV:', namesCSV); // 'Aelin;Gandalf;Violet;
```

### Using the reduce method for calculations

The `reduce` method is used to calculate a value out of the array. The method receives a callback function with the following arguments: `accumulator` (the result of the calculation), the `element` of the array, the `index` and the `array` itself, and the second argument is the initial value. Usually, the index and the array are not used very often and can be omitted. Let's see a few examples.

The `reduce` method is often used when we want to calculate totals. For example, let's say we would like to know what is the sum of all numbers in a given array:

```
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const sum = numbers.reduce((acc, value) => acc + value, 0); // 55
```

Where `0` is the initial value, and `acc` is the sum. We can rewrite the preceding code using a loop to understand the logic behind it:

```
let sumLoop = 0;
for (let i = 0; i < numbers.length; i++) {
  sumLoop += numbers[i];
}
```

We can also use the `reduce` method to find the minimum or maximum values within an array:

```
const scores = [30, 70, 85, 90, 100];
const highestScore = scores.reduce((max, score) => score > max ? score : max, scores[0]); // 100
```

There is also the `reduceRight` method, which will execute the same logic, however, it will iterate the array from its end to the beginning.

> These methods `map`, `filter`, and `reduce` are the basis of _functional programming_ in JavaScript.

## References for other JavaScript array methods

JavaScript arrays are remarkably interesting because they are powerful and have more capabilities available than primitive arrays in other languages. This means that we do not need to write basic capabilities ourselves, and we can take advantage of these powerful features.

We have already covered many different methods within this chapter. Let's look at other useful methods.

### Using the isArray method

In JavaScript, we can check the type of a variable or object using the `typeof` operator as follows:

```
console.log(typeof 'Learning Data Structures'); // string
console.log(typeof 123); // number
console.log(typeof { id: 1 }); // object
console.log(typeof [1, 2, 3]); // object
```

Note that both the object `{ id: 1}` and the array `[1, 2, 3]` have types as `object`.

But what if we would like to double check the type is array so we can evoke any specific array method? Thankfully, JavaScript also provides a method for that through `Array.isArray`:

```
console.log(Array.isArray([1, 2, 3])); // true
```

This way we can always check in case we receive some data we do not know its type. For example, when working with JavaScript on the front-end, we often receive JSON objects from an API from the server. We can parse the data received into an object, and check if the object received is an array so we can use the methods we learned to find specific information:

```
const jsonString = JSON.stringify('[{"id":1,"title":"The Fellowship of the Ring"},{"id":2,"title":"Fourth Wing"}]');
const dataReceived = JSON.parse(jsonString);
if (Array.isArray(dataReceived)) {
  console.log('It is an array');
  // check if The Fellowship of the Ring is in the array
  const fellowship = dataReceived.find((item) => {
    return item.title === 'The Fellowship of the Ring';
  });  
  if (fellowship) {
    console.log('We received the book we were looking for!');
  } else {
    console.log('We did not receive the book we were looking for!');
  }
}
```

This is helpful to ensure our code will not throw errors and a good practice when handling data structures, so we have not created ourselves within our code.

### Using the from method

The `Array.from` method creates a new array from an existing one. For example, if we want to copy the array `numbers` into a new one, we can use the following code:

```
const numbers = [1, 2, 3, 4, 5];
const numbersCopy = Array.from(numbers);
console.log(numbersCopy); // [1, 2, 3, 4, 5]
```

It is also possible to pass a function so that we can determine which values we want to map. Consider the following code:

```
const evens = Array.from(numbers, x => (x % 2 == 0)); 
console.log(evens); // [false, true, false, true, false]
```

The preceding code created a new array named evens, and a value true if in the original array the number is even, and false otherwise.

It is important to note that the `Array.from()` method creates a new, _shallow_ copy. Let's see another example:

```
const friends = [
  { name: 'Frodo', age: 30 },
  { name: 'Violet', age: 18 },
  { name: 'Aelin', age: 20 }
];
const friendsCopy = Array.from(friends);
```

With the copy done, let's modify the name of the first friend to Sam:

```
friends[0].name = 'Sam';
console.log(friendsCopy[0].name); // Sam
```

The name of the first friend of the copied array also gets updated, so we have to be careful when using this method.

If we need to copy the array, and have different instances of its content there is a workaround that can be used via JSON:

```
const friendsDeepCopy = JSON.parse(JSON.stringify(friends));
friends[0].name = 'Frodo';
console.log(friendsDeepCopy[0].name); // Sam
```

By transforming all the content of the array into a string in JSON format, and then parsing this content back to the array structure, we create brand new data. However, depending on what we need to achieve, there are more robust ways of doing this.

### Using the Array.of method

The `Array.of` method creates a new array from the arguments passed to the method. For example, let's consider the following example:

```
const numbersArray = Array.of(1, 2, 3, 4, 5);
console.log(numbersArray); // [1, 2, 3, 4, 5]
```

The preceding code would be the same as performing the following:

```
const numbersArray = [1, 2, 3, 4, 5];
```

We can also use this method to make a copy of an existing array. The following is an example:

```
let numbersCopy2 = Array.of(...numbersArray);
```

The preceding code is the same as using `Array.from(numbersArray)`. The difference here is that we are using the spread operator. The spread operator (`...`) will spread each of the values of the `numbersArray` into arguments.

### Using the fill method

The `fill` method fills the array with a value. For example, suppose a new game tournament will start and we want to store all the results in an array. As the games are over, we can update each of the results.

```
const tornamentResults = new Array(5).fill('pending');
```

The `tornamentResults` array has the length 5, meaning we have five positions. Each position has been initialized with the value `pending`.

Now, suppose games 1 and 2 were a win. We can also use the `fill` method to populate these two positions by passing the start position (inclusive) and the end position (exclusive):

```
tornamentResults.fill('win', 1, 3);
console.log(tornamentResults);
// ['pending', 'win', 'win', 'pending', 'pending']
```

This method is useful as it provides a compact way to initialize arrays with a single value and it is often faster (in terms of the time we will spend writing the code) than manually looping to fill an array.

### Joining multiple arrays

Consider a scenario where you have different arrays and you need to join all of them into a single array. We could iterate each array and add each element to the final array. Fortunately, JavaScript already has a method that can do this for us, named the concat method, which looks as follows:

```
const zero = 0;
const positiveNumbers = [1, 2, 3];
const negativeNumbers = [-3, -2, -1];
let allNumbers = negativeNumbers.concat(zero, positiveNumbers); 
```

We can pass as many arrays and objects/elements to this array as we desire. The arrays will be concatenated to the specified array in the order that the arguments are passed to the method. In this example, zero will be concatenated to `negativeNumbers`, and then `positiveNumbers` will be concatenated to the resulting array. The output of the numbers array will be the values `[-3, -2, -1, 0, 1, 2, 3]`.

## Two-dimensional arrays

At the beginning of this chapter, we used a temperature measurement example. We will now use this example one more time. Let's consider that we need to measure the temperature hourly for a few days. Now that we already know we can use an array to store the temperatures, we can easily write the following code to store the temperatures over 2 days:

```
let averageTempDay1 = [72, 75, 79, 79, 81, 81]; 
let averageTempDay2 = [81, 79, 75, 75, 73, 72];
```

However, this is not the best approach; we can do better! We can use a **matrix** (a two-dimensional array or an _array of arrays_) to store this information, in which each row will represent the day, and each column will represent an hourly measurement of temperature, as follows:

```
let averageTempMultipleDays = [];
averageTempMultipleDays[0] = [72, 75, 79, 79, 81, 81];
averageTempMultipleDays[1] = [81, 79, 75, 75, 73, 73];
```

JavaScript only supports one-dimensional arrays; it does not support matrices. However, we can implement matrices or any multi-dimensional array using an array of arrays, as in the previous code. The same code can also be written as follows:

```
averageTempMultipleDays = [
  [72, 75, 79, 79, 81, 81],
  [81, 79, 75, 75, 73, 73]
];
```

Or, if you prefer to assign a value for each position separately, we can also rewrite the code as the following snippet:

```
// day 1
averageTemp[0] = [];
averageTemp[0][0] = 72;
averageTemp[0][1] = 75;
averageTemp[0][2] = 79;
averageTemp[0][3] = 79;
averageTemp[0][4] = 81;
averageTemp[0][5] = 81;
// day 2
averageTemp[1] = [];
averageTemp[1][0] = 81;
averageTemp[1][1] = 79;
averageTemp[1][2] = 75;
averageTemp[1][3] = 75;
averageTemp[1][4] = 73;
averageTemp[1][5] = 73;
```

We specified the value of each day and hour separately. We can also represent this two-dimensional array as the following diagram:

![Figure 3.4:](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file12.png)

Figure 3.4:

Each row represents a day, and each column represents the temperature for each hour of the day.

Another way of visualizing a two-dimensional array is thinking about an Excel file (or Google Sheets). We can store any kind of tabular data using a two-dimensional array such as chess board, theater seating and even representing images, where each position of the array can store the color value for each pixel.

### Iterating the elements of two-dimensional arrays

If we want to verify the output of the matrix, we can create a generic function to log its output:

```
function printMultidimensionalArray(myArray) {
  for (let i = 0; i < myArray.length; i++) {
    for (let j = 0; j < myArray[i].length; j++) {
      console.log(myArray[i][j]);
    }
  }
}
```

We need to loop through all the rows and columns. To do this, we need to use a nested `for` loop, in which the variable `i` represents rows, and `j` represents the columns. In this case, each `myMatrix[i]` also represents an array, therefore we also need to iterate each position of `myMatrix[i]` in the nested for loop.

We can output the contents of the `averageTemp` matrix using the following code:

```
printMatrix(averageTemp);
```

> We can also use the `console.table(averageTemp)` statement to output a two-dimensional array. This will provide a more user-friendly output, showing the tabular data format.

## Multi-dimensional arrays

We can also work with multi-dimensional arrays in JavaScript. For example, suppose we need to store the average temperature for multiple days and for multiple locations. We can use a 3D matrix to do so:

- Dimension 1 (`i`): each day
- Dimension 2 (`j`): location
- Dimension 3 (`z`): temperature

Let's say we will only store the last 3 days, for 3 distinct locations and 3 different weather conditions. We can represent a 3 x 3 x 3 matrix with a cube diagram, as follows:

![Figure 3.5:](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file13.png)

Figure 3.5:

We can represent a 3 x 3 matrix, as follows:

```
let averageTempMultipleDaysAndLocation = [];
// day 1
averageTempMultipleDaysAndLocation[0] = [];
averageTempMultipleDaysAndLocation[0][0] = [19, 20, 21]; // location 1
averageTempMultipleDaysAndLocation[0][1] = [20, 22, 23]; // location 2
averageTempMultipleDaysAndLocation[0][2] = [30, 31, 32]; // location 3
// day 2
averageTempMultipleDaysAndLocation[1] = [];
averageTempMultipleDaysAndLocation[1][0] = [21, 22, 23]; // location 1
averageTempMultipleDaysAndLocation[1][1] = [22, 23, 24]; // location 2
averageTempMultipleDaysAndLocation[1][2] = [29, 30, 30]; // location 3
// day 3
averageTempMultipleDaysAndLocation[2] = [];
averageTempMultipleDaysAndLocation[2][0] = [22, 23, 24]; // location 1
averageTempMultipleDaysAndLocation[2][1] = [23, 24, 23]; // location 2
averageTempMultipleDaysAndLocation[2][2] = [30, 31, 31]; // location 3
```

And, if we would like to output the content of this matrix, we will need to iterate each dimension (`i`, `j` and `z`):

```
function printMultidimensionalArray3D(myArray) {
  for (let i = 0; i < myArray.length; i++) {
    for (let j = 0; j < myArray[i].length; j++) {
      for (let z = 0; z < myArray[i][j].length; z++) {
        console.log(myArray[i][j][z]);
      }
    }
  }
}
```

Performance wise, the preceding code is _O(nˆ3)_, cubic time, as we have three nested loops.

We can use 3D matrices to represent medical images such as MRI scans, which is a series of 2D image slides of the body. Each slide is a grid if pixel, and combining these slides, we have a 3D representation of a scanned area of the body. Another usage is visualizing models for a 3D printer, or even video data (each frame is a 2D array of pixels, with the third dimension being time).

If we had a 3 x 3 x 3 x 3 matrix, we would have four nested `for` statements in our code and so on. You will rarely need a four-dimensional array in your career as a developer as it has very specialized use cases such as traffic pattern analysis. Two-dimensional arrays are most common in daily activities that developers will work on most projects.

## The TypedArray class

We can store any datatype in JavaScript arrays. This is because JavaScript arrays are not strongly typed as in other languages such as C and Java.

**TypedArray** was created so that we could work with arrays with a single datatype. Its syntax is `let myArray = new TypedArray(length)`, where `TypedArray` needs to be replaced with one specific class, as defined in the following table:

|   |   |
|---|---|
|**TypedArray**|**Description**|
|`Int8Array`|8-bit two's complement signed integer|
|`Uint8Array`|8-bit unsigned integer|
|`Uint8ClampedArray`|8-bit unsigned integer|
|`Int16Array`|16-bit two's complement signed integer|
|`Uint16Array`|16-bit unsigned integer|
|`Int32Array`|32-bit two's complement signed integer|
|`Uint32Array`|32-bit unsigned integer|
|`Float32Array`|32-bit IEEE floating point number|
|`Float64Array`|64-bit IEEE floating point number|
|`BigInt64Array`|64-bit big integer|
|`BigUint64Array`|64-bit unsigned big integer|

Table 3.1:

The following is an example:

```
const arrayLength = 5;
const int16 = new Int16Array(arrayLength);
for (let i = 0; i < arrayLength; i++) {
  int16[i] = i + 1;
}
console.log(int16);
```

Typed arrays are great for working with WebGL APIs, manipulating bits, and manipulating files, images, and audios. Typed arrays work exactly like simple arrays, and we can also use the same methods and functionalities that we have learned in this chapter.

One practical example of when to use `TypedArray` is when working with **TensorFlow** ([https://www.tensorflow.org](https://www.tensorflow.org/)), which is a library used to create **Machine Learning** models. TensorFlow has the concept of **Tensors**, which is the core data structure of TensorFlow.js. It utilizes `TypedArrays` internally to represent tensor data. This contributes to the efficiency and performance of the library, especially when dealing with large datasets or complex models.

## Arrays in TypeScript

All the source code from this chapter is valid TypeScript code. The difference is that TypeScript will do type checking at compile time to make sure we are only manipulating arrays in which all values have the same datatype.

Let's review one of the previous examples mentioned earlier this chapter:

```
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
```

Due to the type inference, TypeScript understands that the declaration of the numbers array is the same as `const numbers: number[]`. For this reason, we do not need to always declare the variable type explicitly if we initialize it during its declaration.

If we go back to the sorting example of the `friends` array, we can refactor the code to the following in TypeScript:

```
interface Friend {
  name: string;
  age: number;
}
const friends = [
  { name: 'Frodo', age: 30 },
  { name: 'Violet', age: 18 },
  { name: 'Aelin', age: 20 }
];
const compareFriends = (friendA: Friend, friendB: Friend) => {
  return friendA.age - friendB.age;
};
friends.sort(compareFriends);
```

By declaring the `Friend` interface, we make sure the `compareFriend` function receives only objects that have the properties `name` and `age`. The friends array does not have an explicit type, so in this case, if we wanted, we could explicitly declare its type using `const friends: Friend[]`.

In summary, if we want to type our JavaScript variables using TypeScript, we simply need to use `const` or `let variableName: <type>[]` or, when using files with a `.js` extension, we can also have the type checking by adding the comment `// @ts-check` in the first line of the JavaScript file.

At runtime, the output will be exactly the same as if we were using pure JavaScript.

## Creating a simple TODO list using arrays

Arrays is one of the most used data structures in general, it does not matter if we are using JavaScript, .NET, Java, Python, or any other language. This is one of the reasons most languages have native support to this data structure and JavaScript has an excellent API (_Application Programming Interface_) for the `Array` class.

Whenever we access the database, we will get a collection of records back, and we can use arrays to manage the information retrieved from the database. If we are using JavaScript in the frontend, and we make a call to a server API, we usually will get back a collection of records in **JSON** (_JavaScript Object Notation_) format, and we can parse the JSON into an array so we can manage and manipulate the data as needed so we can display it on the screen for the user.

Let's see a simple example of an HTML page using JavaScript where we can create tasks, complete tasks, and remove tasks. Of course, we will use arrays to manage our TODO list:

```
<!-- Path: src/03-array/10-todo-list-example.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Simple TODO List (Array-Based)</title>
</head>
<body>
  <h1>My TODO List</h1>
  <input type="text" id="newTaskInput" placeholder="Add new task">
  <button onclick="addTask()">Add</button>
  <ul id="taskList"></ul>
  <script>
    const taskList = document.getElementById('taskList');
    const newTaskInput = document.getElementById('newTaskInput');
    let tasks = []; // Initialize empty task array
    function addTask() {}
    function renderTasks() {}
    function toggleComplete(index) {}
    function removeTask(index) {}
  </script>
</body>
</html>
```

This HTML code will help us to render a basic TODO application. Once we complete the development of this page, and if we open it in a browser, we will have the following application:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file14.png)

Figure 3.6:

Let's check out the code used to render the task bullet point list:

```
function addTask() {
  const taskText = newTaskInput.value.trim(); // {1}
  if (taskText !== "") {
    tasks.push({ text: taskText, completed: false }); // {2}
    renderTasks(); // Update the displayed list
    newTaskInput.value = ""; // Clear input
  }
}
```

When we click on the **Add** button, the function `addTask` will be called. We will use the `trim` method to remove all the additional spaces at the beginning and the end of the text (`{1}`), and if the text is not empty, we will add it to our array (`{2}`) in the format of an object containing the text and also saying the task is not completed. Then we will evoke the `renderTasks` function and we will clear the input so we can enter more tasks.

Next, let's see the `renderTasks` function:

```
function renderTasks() {
  taskList.innerHTML = ''; // Clear the list
  tasks.forEach((task, index) => { // {3}
    const listItem = document.createElement("li");
    listItem.innerHTML = `
      <input type="checkbox" ${task.completed ? "checked" : ""}
        onchange="toggleComplete(${index})">
      <span class="${task.completed ? "completed" : ""}">
        ${task.text}</span>
      <button onclick="removeTask(${index})">Delete</button>
    `;
    taskList.appendChild(listItem);
  });
}
```

Every time we add a new task or remove one, we will call this `renderTasks` function. First, we will clear the list by rendering an empty space in the screen, then, for each task we have in the array (`{3}`), we will create an element in the HTML list, that contains a checkbox that is checked in case the task is completed, the task text and a button that we can use the remove task, passing the index of the task in the array.

Finally, let's check the `toggleComplete` function (which is called whenever the check or uncheck the checkbox) and the `removeTask` function:

```
function toggleComplete(index) { // Toggle task completion status
  tasks[index].completed = !tasks[index].completed; // {4}
  renderTasks();
}
function removeTask(index) {
  tasks.splice(index, 1); // {5} Remove from array
  renderTasks();
}
```

Both functions receive the `index` of the array as parameter, so we can easily access the task that is being toggled or removed. For the toggle, we can access the array position directly and mark the task as completed or not completed (`{4}`), and to remove the task, we can use the splice method as we learned in this chapter to remove the task from the array (`{5}`), and of course, whenever we make a change, we will render the tasks again.

Arrays are everywhere, hence the importance to master this data structure.

## Exercises

We will resolve a few array exercises from **Hackerrank** using the concepts we learned in this chapter.

### Reversing an array

The first exercise we will resolve the is reverse array problem available at [https://www.hackerrank.com/challenges/arrays-ds/problem](https://www.hackerrank.com/challenges/arrays-ds/problem).

When resolving the problem using JavaScript or TypeScript, we will need to add our logic inside the function `reverseArray(a: number[]): number[] {}`, which receives an array of numbers and it is also expecting an array of numbers to be returned.

The sample input that is given is `[1,4,3,2]` and the expected output is `[2,3,4,1]`.

The logic we need to implement is to reverse the array, meaning the first element will become the last and so on.

The most straightforward solution is using the existing `reverse` method:

```
function reverseArray(a: number[]): number[] {
  return a.reverse();
}
```

This is a solution that passes all the tests and resolves the problem. However, if this exercise is being used in technical interviews, the interviews probably will ask you to try a different solution that does not include using the existing `reverse` method so they can evaluate how you think and communicate the resolution.

A second solution is to manually code reversing the array.

```
function reverseArray2(a: number[]): number[] {
  const result = [];
  for (let i = a.length - 1; i >= 0; i--) {
    result.push(a[i]);
  }
  return result;
}
```

We will create a brand new array, we will iterate the given array starting from the end (since we have to reverse it) until we reach the first index which is 0. Then, for each element, we will add (`push`) to the new array and we can return the `result`. This solution is _O(n)_ as we need to iterate through the length of the array.

Speaking of _Big O notation_, as you might have noticed, we often need to iterate an array. Iterating an array is linear time, and accessing the elements directly is _O(1)_, as we can access any position of the array by accessing its index.

If you prefer to iterate the array from its beginning until its last position, we can use the `unshift` method:

```
function reverseArray3(a: number[]): number[] {
  const result = [];
  for (let i = 0; i < a.length; i++) {
    result.unshift(a[i]);
  }
  return result;
}
```

However, this is one of worst solutions. Given we need to iterate the array, we are talking about _O(n)_ complexity. The `unshift` method also has _O(n)_ complexity as it needs to move all the existing elements already in the array, making this solution _O(nˆ2)_, quadratic time.

Can you think of a solution that does not require iterating through all the array? What if we iterate only half of the array and swap the elements, meaning we swap the first element with the last, the second element with the second last, and so on:

```
function reverseArray4(a: number[]): number[] {
  for (let i = 0; i < a.length / 2; i++) {
    const temp = a[i];
    a[i] = a[a.length - 1 - i];
    a[a.length - 1 - i] = temp;
  }
  return a;
}
```

The loop of this function would run approximately _n/2_ times, where `n` is the length of the array. In Big O notation, this would still be an algorithm of complexity _O(n)_, as we ignore constant factors and lower order terms, however, _n/2_ is better than _n_, so this last solution might be slightly faster.

### Array left rotation

The next exercise we will resolve is the array left rotation available at https://www.hackerrank.com/challenges/array-left-rotation/problem.

When resolving the problem using JavaScript or TypeScript, we will need to add our logic inside the function `function rotLeft(a: number[], d: number): number[] {}` , which receives an array of numbers, a number `d` which is the number of left rotations and it is also expecting an array of numbers to be returned.

The sample input that is given is `[1,2,3,4,5]`, `d` is 2 and the expected output is `[3,4,5,1,2]`.

The logic we need to implement is to remove the first element of the array and add it to the end of the array, doing this `d` times.

Let's check with first solution:

```
function rotLeft(a: number[], d: number): number[] {
  return a.concat(a.splice(0, d));
}
```

We are using the existing methods available in JavaScript, by removing the number of elements we need to rotate using the `splice` method, which returns the array with removed elements. Then, we are concatenating the original array with the array of the removed elements. Basically, we are splitting the original array into two arrays and swapping the order.

The complexity of this solution is _O(n)_ because:

- `a.splice(0, d)`: this operation has a time complexity of _O(n)_ because it needs to shift all the remaining elements of the array after removing the first `d` elements.
- `a.concat()`: this operation also has a time complexity of _O(n)_ because it needs to iterate over all elements in the two arrays (the original array and the spliced array) to create a new array.

Since these operations are performed sequentially (not nested), the time complexities add up, resulting in a total time complexity of _O(n + n) = O(2n)_. However, in Big O notation, we drop the constants, so the final time complexity is _O(n)_.

Another similar solution would be as follows:

```
function rotLeft2(a: number[], d: number): number[] {
  return [...a.slice(d), ...a.slice(0, d)];
}
```

Where we are creating a new array starting from the element at index `d` (`a.slice(d)`), and creating a new array by removing the number of elements we were asked to rotate (`a.slice(0, d)`). The spread operator (`…`) is used to unpack the elements of the two new arrays, and when surrounded by `[]`, we create a new array.

Let's review the complexity of this solution, which is also _O(n)_:

- `a.slice(d)`: this operation has a time complexity of _O(n - d)_ because it needs to create a new array with the elements from index d to the end of the array.
- `a.slice(0, d)`: this operation has a time complexity of _O(d)_ because it needs to create a new array with the first d elements of the array.
- The spread operator (`...`): this operation has a time complexity of _O(n)_ because it needs to iterate over all elements in the two arrays to create a new array.

Since these operations are performed sequentially (not nested), the time complexities add up, resulting in a total time complexity of _O((n - d) + d + n) = O(2n)_. So the final time complexity is _O(n)_.

Again, during an interview, we can be asked to implement a manual solution, so let's review a third possible solution:

```
function rotLeft3(a: number[], d: number): number[] {
  for (let i = 0; i < d; i++) {
    const temp = a[0]; // {1}
    for (let j = 0; j < a.length - 1; j++) {
      a[j] = a[j + 1]; // {2}
    }
    a[a.length - 1] = temp; // {3}
  }
  return a;
}
```

The outer loop will run `d` times as we need to rotate the elements. For each element we need to rotate, we will keep it in a temporary variable (`{1}`). Then, we will iterate the array and move the element in the next position to the current index (`{2}`). And at the end we will move the element we had stored in the temporary variable to the last position of the array (`{3}`). This is remarkably like the algorithm we created to remove an element from the first position. The difference here is we are not removing the element from the beginning of array and throwing it away, we are moving it to the end of the array.

The time complexity of this solution is _O(n*d)_. The first solutions presented are likely to be faster as they are _O(n)_.

## Summary

In this chapter, we covered the most-used data structure: arrays. We learned how to declare, initialize, and assign values as well as add and remove elements. We learned about two-dimensional and multi-dimensional arrays as well as the main methods of an array, which will be particularly useful when we start creating our own algorithms in later chapters.

We also learned how to make sure the array only contains values of the same type by using TypeScript or the TypeScript compile-time checking capability for JavaScript files.

And finally, we resolved a few exercises that can be the topic of technical interviews and reviewed their complexity.

In the next chapter, we will learn about stacks, which can be treated as arrays with a special behavior.