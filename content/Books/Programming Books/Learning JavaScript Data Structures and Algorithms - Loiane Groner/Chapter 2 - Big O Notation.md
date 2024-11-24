---
id: 01JCKZHW86C1YSX7566TAX7MEV
title: Chapter 2 - Big O Notation
modified: 2024-11-13T19:00:56-05:00
---
# 2 Big O notation

**Before you begin: Join our book community on Discord**

Give your feedback straight to the author himself and chat to other early readers on our Discord server (find the "learning-javascript-dsa-4e" channel under EARLY ACCESS SUBSCRIPTION).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file0.png)

[https://packt.link/EarlyAccess/](https://packt.link/EarlyAccess/)

In this chapter, we will unlock the power of **Big O notation**, a fundamental tool for analyzing the efficiency of algorithms in terms of both **time complexity** (how runtime scales with input size) and **space complexity** (how memory usage scales). We will explore common time complexities like _O(1)_, _O(log n)_, _O(n)_, and others, along with their real-world implications for choosing the right algorithms and optimizing code. Understanding Big O notation is not only essential for writing scalable and performant software but also for acing technical interviews, as it demonstrates your ability to think critically about algorithmic efficiency. In this chapter we will cover:

- Big O time complexities
- Space complexity
- Calculating the complexity of an algorithm
- Big O notation and tech interviews
- Exercises

## Understanding Big O notation

Big O notation is used to describe and classify the performance or complexity of an algorithm according to how much time it will take for the algorithm to run as the input size grows.

And how do we measure the efficiency of an algorithm? We usually use resources such as CPU (time) usage, memory usage, disk usage, and network usage. When talking about Big O notation, we usually consider CPU (time) usage.

In simpler terms, this notation is a way to describe how the running time of an algorithm grows as the size of the input gets bigger. While the actual time an algorithm takes to run can vary depending on factors like processor speed and available resources, Big O notation allows us to focus on the fundamental steps an algorithm must take. Think of it as measuring the number of operations an algorithm performs relative to the input size.

Imagine you have a stack of papers on your desk. If you need to find a specific document, you will have to search through each paper one by one until you locate it. With a small stack of 10 papers, this would not take long. But if you had 20 papers, the search would likely take twice as long, and with 100 papers, it could take ten times as long!

The tasks that a developer must perform daily include choosing what data structure and algorithms to use to resolve a specific problem. It can be an existing algorithm, or you may have to write your own logic to resolve a business user story. It is important to note that any algorithm can work fine and seem okay for a low volume of data, however, then the volume of the input data increases, an inefficient algorithm will grind to halt and impact the application. Knowing how to measure performance is key to achieving these tasks successfully.

Big O notation is important because it helps us compare different algorithms and choose the most efficient one for a particular task. For instance, if you are searching for a specific product in a large online store, you would not want to use an algorithm that requires looking at every single product. Instead, you would use a more efficient algorithm that only needs to look at a small subset of products.

## Big O time complexities

Big O notation uses capital _O_ to denote upper bound. It signifies that the actual running time could be less than but not greater than what the function expresses. It does not tell us the exact running time of an algorithm. Instead, it tells us how bad things could get as the input size grows large.

Imagine you have a messy room and need to find a specific sock. In the worst case, you have to check each item of clothing one by one (this is like a linear time algorithm). Big O tells you that even if your room gets super messy, you will not need to look at more items than are actually there. You might get lucky and find the sock quickly! The actual time might be much less than the Big O prediction.

When analyzing algorithms, the following classifications of time and space complexities are most encountered:

|   |   |   |
|---|---|---|
|**Notation**|**Name**|**Explanation**|
|O(1)|Constant|The algorithm's runtime or space usage remains the same regardless of the input size (n).|
|O(log(n))|Logarithmic|The algorithm's runtime or space usage grows logarithmically with the input size (n). This means that as the input size doubles, the number of operations or memory usage increases by a constant amount.|
|O(n)|Linear|The algorithm's runtime or space usage grows linearly with the input size (n). This means that as the input size doubles, the number of operations or memory usage also doubles.|
|O(n 2 )|Quadratic|The algorithm's runtime or space usage grows quadratically with the input size (n). This means that as the input size doubles, the number of operations or memory usage quadruples.|
|O(n c )|Polynomial|The algorithm's runtime or space usage grows as a polynomial function of the input size (n). This means that as the input size doubles, the number of operations or memory usage increases by a factor (c) that is a polynomial function of the input size.|
|O(c n )|Exponential|The algorithm's runtime or space usage grows exponentially with the input size (n). This means that as the input size increases, the number of operations or memory usage grows at an increasingly rapid rate.|

Table 2.1: Big O notation classifications of time and space complexities

Let's review each one to understand time complexities in detail.

### O(1): constant time

_O(1)_ signifies that an algorithm's runtime (or sometimes space complexity) remains constant, regardless of the size of the input data. Whether we are dealing with a small input or a massive one, the time it takes to execute the algorithm does not change significantly.

For example, suppose we would like to calculate the number of seconds of a given number of days. We could create the following function to resolve this request:

```
function secondsInDays(numberOfDays) {
  if (numberOfDays <= 0 || !Number.isInteger(numberOfDays)) {
    throw new Error('Invalid number of days');
  }
  return 60 * 60 * 24 * numberOfDays;
}
```

Each minute has 60 seconds, each hour has 60 minutes, and each day has 24 hours.

And we can use `console.log` to see the output of the results passing different numbers of days:

```
console.log(secondsInDays(1)); // 86400
console.log(secondsInDays(10)); // 864000
console.log(secondsInDays(100)); // 8640000
```

If we call this function passing `1` as argument (`secondsinDays(1)`), it will take a few milliseconds for this code to output the results. If we execute the function again passing `10` as argument (`secondsinDays(10)`), it will also take a few milliseconds for the code to output the results.

This `secondsInDays` function has a time complexity of _O(1)_ – constant time. The number of operations it performs (multiplication) is fixed and doesn't change with the input `numberOfDays`. It will take the same amount of time to calculate the result, whether you input 1 day or 1000 days.

_O(1)_ algorithms typically do not involve loops that iterate over the data or recursive calls that multiply operations. They often involve direct access to data, like looking up a value in an array by its index or performing a simple calculation. And while _O(1)_ algorithms are incredibly efficient, they are not always applicable to every problem. Some tasks inherently require processing each item in the input, leading to different time complexities.'

### O(log(n)): logarithmic time

An _O(log n)_ algorithm's runtime (or sometimes space complexity) grows logarithmically with the input size (_n_). This means that each step of the algorithm significantly reduces the problem size, often by dividing it in half or a similar fraction. The larger the input size, the smaller the impact each additional element has on the overall runtime. In other words, as the input size doubles, the runtime increases by a constant amount (for example, only one more step).

Imagine you are playing a "_guess the number_" game. You start with a range of 1 to 64, and with each guess, you cut the possible numbers in half. Let's say your first guess is 30. If it is too high, you now know the number is somewhere between 1 and 29. You have effectively halved the search space! Next, you guess 10 (too low), narrowing the range further to 11 through 29. Your third guess, 20, happens to be correct!

Even if you had started with a much larger range of numbers (like 1 to 1000 or even 1 to 1 million), this halving strategy would still allow you to find the number in a surprisingly small number of guesses – around 7 for 1 to 64, 10 for 1 to 1000, and 20 for 1 to 1 million. This demonstrates the power of logarithmic growth.'

We can say this approach has a time complexity of _O(log(n))_. With each step, the algorithm eliminates a significant portion of the input, making the remaining work much smaller.

A function that has a time complexity of _O(log(n))_ typically halves the problem size with each step. This complexity is often related to divide and conquer algorithms, which we will cover in _Chapter 18, Algorithm Designs and Techniques_.

Logarithmic algorithms are incredibly efficient, especially for large datasets. They are often used in scenarios where you need to quickly search or manipulate sorted data, which we will also cover later in this book.

### O(n): linear time

_O(n)_ signifies that an algorithm's runtime (or sometimes space complexity) grows linearly and proportionally with the input size (_n_). If we double the size of the input data, the algorithm will take approximately twice as long to run. If we triple the input, it will take about three times as long, and so on.

Imagine you have an array of monthly expenses and want to calculate the total amount spent. Here is how we could do it:

```
function calculateTotalExpenses(monthlyExpenses) {
  let total = 0;
  for (let i = 0; i < monthlyExpenses.length; i++) {
    total += monthlyExpenses[i];
  }
  return total;
}
```

The for loop iterates through each element (`monthlyExpense`) in the array adding it to the `total` variable, which is then returned with the amount of the total expenses.

We can use the following code to check the output of this function, passing different parameters:

```
console.log(calculateTotalExpenses([100, 200, 300])); // 600
console.log(calculateTotalExpenses([200, 300, 400, 50])); // 950
console.log(calculateTotalExpenses([30, 40, 50, 100, 50])); //270
```

The number of iterations (and additions to the `total`) directly depends on the size of the array (`monthlyExpenses.length`). If the array has 12 months of expenses, the loop runs 12 times. If it has 24 months, the loop runs 24 times. The runtime increases proportionally to the number of elements in the array.

This is because the function contains a loop that runs _n_ times. Therefore, the time it takes to run this function grows in proportion to the size of the input _n_. If _n_ doubles, the time to run the function approximately doubles as well. For this reason, we can say the preceding function has a complexity of _O(n)_, where in this context, _n_ is the input size.

While _O(n)_ algorithms are not as fast as constant time (_O(1)_) algorithms, they are still considered efficient for many tasks. There are many situations where you need to process every element of the input, making linear time a reasonable expectation.

### O(nˆ2): quadratic time

_O(n²)_ signifies that an algorithm's runtime (or sometimes space complexity) grows quadratically with the input size (_n_). This means that as the input size doubles, the runtime roughly quadruples. If you triple the input, the runtime increases by a factor of nine, and so on. _O(n²)_ algorithms often involve nested loops, where the inner loop iterates _n_ times for each iteration of the outer loop. This results in approximately _n * n_ (or _n²_) operations.

Let's go back to the calculation of expenses example. Suppose you have the following data in a spreadsheet, with each expense by month:

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
|**Month/Expense**|**January**|**February**|**March**|**April**|**May**|**June**|
|Water Utility|100|105|100|115|120|135|
|Power Utility|180|185|185|185|200|210|
|Trash Fees|30|30|30|30|30|30|
|Rent/Mortgage|2000|2000|2000|2000|2000|2000|
|Groceries|600|620|610|600|620|600|
|Hobbies|150|100|130|200|150|100|

Table 2.2: Example of monthly expenses

What if we want to write a function that calculates the total expenses for several months? The code for this function is as follows:

```
function calculateExpensesMatrix(monthlyExpenses) {
  let total = 0;
  for (let i = 0; i < monthlyExpenses.length; i++) {
    for (let j = 0; j < monthlyExpenses[i].length; j++) {
      total += monthlyExpenses[i][j];
    }
  }
  return total;
}
```

The function has two nested loops:

1. The outer loop (`i`) iterates over the rows of the matrix (categories or types of expenses within each month).
2. The inner loop (`j`) iterates over the columns of the matrix (each month) for each row.

Inside the nested loop we simply add the expense to the `total`, which is then returned at the end of the function.

Let's test this function with the data we previous represented:

```
const monthlyExpenses = [
  [100, 105, 100, 115, 120, 135],
  [180, 185, 185, 185, 200, 210],
  [30, 30, 30, 30, 30, 30],
  [2000, 2000, 2000, 2000, 2000, 2000],
  [600, 620, 610, 600, 620, 600],
  [150, 100, 130, 200, 150, 100]
];
console.log('Total expenses: ', calculateExpensesMatrix(monthlyExpenses)); // 18480
```

We can say the preceding function has a complexity of _O(nˆ2)_. This is because the function contains two nested loops. The outer loop will run 6 times (_n_) and the inner loop will also run 6 times as we have 6 months (_m_). We can say the total number of operations is _n * m_. If _n_ and _m_ are similar numbers, we can say _n * n_, hence _nˆ2_.

In Big O notation, we simplify this to the highest order of magnitude, which is _nˆ2_. This means the time complexity of the function grows quadratically (input size squared) with the input size. So, If you have a 12x12 matrix (12 categories of expenses with 12 months each), the inner loop runs 12 times for each of the 12 months, resulting in 144 operations. If we expand the list of expenses and also the number of months, with a matrix 24x24, the number of operations becomes 576 (24 * 24). This is characteristic of an algorithm with _O(nˆ2)_ time complexity.

### O(2^n): exponential time complexity

_O(2^n)_ signifies that an algorithm's runtime (or sometimes space complexity) doubles with each additional unit of input size (_n_). If you add just one more element to the input, the algorithm takes approximately twice as long. If you add two more elements, it takes about four times as long, and so on. The runtime increases exponentially. An algorithm with exponential time complexity does not have satisfactory performance.

A classic example of an algorithm that is _O(2ˆn)_ is when we have brute force that will try all possible combinations of a set of values.

Imagine we want to know how many unique combinations we can have with ice cream toppings or no toppings at all. The available toppings are chocolate sauce, maraschino cherries and rainbow sprinkles.

What are the possible combinations?

Since each topping can be either present or absent, and we have three different toppings, the total number of possible combinations is: 2 * 2 * 2 = 2^3 = 8.

Here is a list of the following combinations:

- No toppings
- Chocolate sauce only
- Maraschino cherries only
- Rainbow sprinkles only
- Chocolate sauce + maraschino cherries
- Chocolate sauce + rainbow sprinkles
- Maraschino cherries + rainbow sprinkles
- Chocolate sauce + maraschino cherries + rainbow sprinkles

If we had 10 toppings to choose from, we would have 2 ^ 10 possible combinations, totaling 1024 different combinations.

Another example of exponential complexity algorithm is the brute force attack to break passwords or PINs. If we have a 4-digit (0-9) code PIN, we have a total of 10ˆ4 combinations, totaling 10000 combinations. If we have passwords using letters only, we will have a total of 26ˆn combinations, where n is the number of letters in the password. If we allow uppercase and lowercase characters in the password, we have a total of 62ˆn combinations. This is one of the reasons it is important to always create long passwords with letters (both uppercase and lowercase), numbers and especial characters, as the number of possible combinations grow exponentially, making it more difficult to break the password by using brute force.

Exponential algorithms are generally considered impractical for large inputs due to their incredibly rapid growth in runtime. They can quickly become infeasible even for moderately sized datasets. It is crucial to find more efficient algorithms whenever possible.

### O(n!): factorial time

_O(n!)_ signifies an algorithm's runtime (or sometimes space complexity) grows incredibly rapidly with the input size (_n_). This growth is even faster than exponential time complexity. An algorithm with factorial time complexity has one of the worst performances.

The factorial of a number _n_ (denoted as _n!_) is calculated as _n * (n-1) * (n-2) , …, * 1_. For example, 4! is 4 * 3 * 2 * 1 = 24 .1 As we can see, factorials get very large very quickly

A classic example of an algorithm that is _O(n!)_ is when we try to find all possible permutations of a set, for example, the letters ABCD as follows:

|   |   |   |   |
|---|---|---|---|
|ABCD|BACD|CABD|DABC|
|ABDC|BADC|CADB|DACB|
|ACBD|BCAD|CBAD|DBAC|
|ACDB|BCDA|CBDA|DBCA|
|ADBC|BDAC|CDAB|DCAB|
|ADCB|BDCA|CDBA|DCBA|

Table 2.3: All permutations of letters ABCD

Algorithms with factorial time complexity are generally considered highly inefficient and should be avoided whenever possible. For many problems that initially seem to require _O(n!)_ solutions, there are often cleverer algorithms with much better time complexities (for example: dynamic programming technique).

> We will cover algorithms with exponential and factorial times in _Chapter 18, Algorithm Designs and Techniques_.

### Comparing complexities

We can create a table with some values to exemplify the cost of the algorithm based on its time complexity and input size, as follows:

|   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|
|**Input Size (n)**|**O(1)**|**O(log (n))**|**O(n)**|**O(n log(n))**|**O(nˆ2)**|**O(2ˆn)**|**O(n!)**|
|10|1|1|10|10|100|1024|3628800|
|20|1|1.30|20|26.02|400|1048576|2.4329E+18|
|50|1|1.69|50|84.94|2500|1.1259E+15|3.04141E+64|
|100|1|2|100|200|10000|1.26765E+30|9.33262E+157|
|500|1|2.69|500|1349.48|250000|3.27339E+150|Very big number|
|1000|1|3|1000|3000|1000000|1.07151E+301|Very big number|
|10000|1|4|10000|40000|100000000|Very big number|Very big number|

Table 2.4: Comparing Big O time complexity based on input size

We can draw a chart based on the information presented in the preceding table to display the cost of different Big O notation complexities as follows:

![Figure 2.1 – Big O Notation complexity chart](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file7.png)

Figure 2.1 – Big O Notation complexity chart

> The preceding chart was also plotted using JavaScript. You can find its source code in the `src/02-bigOnotation` directory of the source code bundle.

When we plot the runtime of algorithms with different time complexities against the input size on a graph, distinct patterns emerge:

- **_O(1) - Constant Time_**: a horizontal line. The runtime remains the same regardless of the input size.
- **_O(log n) - Logarithmic Time_**: a gently rising curve that gradually flattens as the input size increases. Think of it as a slope that gets less and less steep. Each additional input element has a diminishing impact on the overall runtime.
- **_O(n) - Linear Time_**: a straight line with a positive slope. The runtime increases proportionally with the input size. Double the input, and the runtime roughly doubles.
- **_O(n²) - Quadratic Time_**: a curve that starts shallow but becomes increasingly steep. The runtime grows much faster than the input size. Double the input, and the runtime roughly quadruples.
- **_O(2^n) - Exponential Time_**: a curve that initially seems flat but then explodes upwards as the input size increases even slightly. The runtime grows incredibly rapidly.
- **_O(n!) - Factorial Time_**: a curve that rises almost vertically. The runtime becomes astronomically large even for relatively small inputs, quickly becoming impractical to compute.

These visualizations are invaluable tools for understanding the long-term behavior of algorithms as the input size grows. They help us make informed choices about which algorithms are best suited for different scenarios, especially when dealing with large datasets.

## Space complexity

Space complexity refers to the amount of memory (or space) an algorithm uses to solve a problem. It is a measure of how much additional storage the algorithm requires beyond the space occupied by the input data itself.

It is important to understand space complexity as real-world computers have finite memory. If the algorithm's space complexity is too high, it might run out of memory on large datasets. And even if we have plenty of memory, an algorithm with a high space complexity can still be slower due to factors like increased memory access times and cache issues. Also, it is all about tradeoffs. Sometimes, we might choose an algorithm with a slightly higher space complexity if it offers a significant improvement in time complexity. This of course, needs to be reviewed case by case.

Big O notation works for space complexity just like it does for time complexity. It expresses the upper bound of how the algorithm's memory usage grows as the input size increases. Let's review the common Big O space complexities:

- **_O(1) - Constant Space_:** the algorithm uses a fixed amount of memory, regardless of the input size. This is ideal, as the memory usage will not become a bottleneck.
    - For example: swapping two variables.
- **_O(n) - Linear Space_:** the algorithm's memory usage grows linearly with the input size. If we double the input, the memory usage roughly doubles.
    - For example: storing a copy of an input array.
- **_O(log n) - Logarithmic Space_:** the algorithm's memory usage grows logarithmically. This is relatively efficient, especially for large datasets.
    - For example: certain recursive algorithms where the depth of recursion is logarithmic.
- **_O(nˆ2) - Quadratic Space_:** the algorithm's memory usage grows quadratically. This can become a problem for large inputs.
    - For example: storing a multiplication table in a 2D array.
- **_O(2^n) - Exponential Space_:** like the exponential time complexity, this indicates extremely rapid growth in memory usage. It is generally not practical and should be avoided.

## Calculating the complexity of an algorithm

It is also important to understand how to read algorithmic code and identify its complexity in terms of Big O notation. By analyzing the complexity of an algorithm, we can identify potential bottlenecks and focus on improving that specific area.

To determine the cost of a code in terms of **_time complexity_**, we need to review it step by step, and focus on the following points:

- Basic operations such as assignments, bits and math operations, which will usually have constant time (_O(1)_).
- Logarithmic algorithms (_O(log (n))_) typically follow a divide-and-conquer strategy. They break the problem into smaller subproblems and solve them recursively.
- Loops: the number of times a loop runs directly impacts time complexity. Nested loops multiply their effects. So, if we have one loop iterating through the input of size _n_, it will be linear time (_O(n)_), two nested loops (_O(nˆ2)_), and so on.
- Recursions: recursive functions call themselves, potentially leading to exponential time complexity if not carefully designed. We will cover recursion in _Chapter 9, Recursion_.
- Function calls: consider the time complexity of any functions that are called within your code.

And to determine the cost of a code in terms of **space complexity**, we need to review it step by step, and focus on the following points:

- Variables: how much memory do variables used in the algorithm consume? Does the number of variables grow with the input size?
- Data structures: what data structures are being used (arrays, lists, trees, etc.)? How does their size scale with the input?
- Function calls: if the algorithm uses recursion, how many recursive calls are made? Each call adds to the space complexity of the call stack.
- Allocations: are we dynamically allocating memory within the algorithm? How much memory is allocated, and how does it relate to the input size?

Let's see an example of a function that logs the multiplication table of a given number:

```
function multiplicationTable(num, x) {
  let s = '';
  let numberOfAsterisks = num * x;
  for (let i = 1; i <= numberOfAsterisks; i++) {
    s += '*';
  }
  console.log(s);
  for (let i = 1; i <= num; i++) {
    console.log(`Multiplication table for ${i} with x = ${x}`);
    for (let j = 1; j <= x; j++) {
      console.log(`${i} * ${j} = `, i * j);
    }
  }
}
```

Let's break down the time and space complexity of the `multiplicationTable` function using Big O notation. First, let's focus on time complexity:

- **_O(1) operations_**:
    - Assigning variables (`let s = ''` and `let numberOfAsterisks = num * x`)
    - Printing fixed strings (`console.log('Calculating the time complexity of a function')`)
- **_O(n) operations_**:
    - Building the asterisk string: the loop iterates _num * x_ times, and each iteration involves string concatenation, which can be a linear operation depending on the JavaScript implementation.
    - Printing the asterisk string: outputting a string of length _num * x_ takes time proportional to its length.
- **_O(nˆ2) operations_**:
    - Nested loops: the outer loop runs num times, and for each iteration, the inner loop runs _x_ times. This leads to roughly _num * x_ (or _nˆ2_) iterations of the innermost `console.log` statement, where the actual multiplication takes place.

While there are _O(1)_ and _O(n)_ operations in the function, the dominant factor in the time complexity is the nested loop structure, which leads to quadratic time complexity _O(n^2)_. In Big O notation, we simplify this to the highest order of magnitude, which is _n^2_. Therefore, the overall time complexity of the function is _O(n^2)_.

Now let's review the space complexity:

- **_O(1) space_**:
    - Simple variables (`s`, `numberOfAsterisks`, loop counters `i` and `j`) use a fixed amount of memory, regardless of the input values `num` and `x`.
- **_O(n) space_** (_potential_):
    - The string `s` could potentially grow to a size of _num * x_, meaning its space usage is linear in the input size. However, in most implementations, string concatenation is optimized, so this might not be a major concern unless the input values are very large.

So, overall, the space complexity could be considered _O(n)_ due to the potential growth of the asterisk string. However, for practical purposes, the space usage is usually not a significant issue, and we often focus on the _O(n²)_ time complexity as the primary concern for this function.

## Big O notation and tech interviews

During technical interviews for software developer positions, it is common for companies to do a coding test using some services online such as **LeetCode**, **Hackerrank**, and other similar services.

Choosing the correct data structure or algorithm to solve a problem can tell the company some information about how you solve problems that might pop up for you to resolve.

Interviewers might ask you to analyze code and predict how its runtime or memory usage might change under different input sizes. Once you write code to resolve a problem, interviewers might also ask you to pinpoint potential performance problems in your code and if you can identify areas of optimization. Also, different algorithms and data structures have different time complexities, and knowing Big O allows you to make informed decisions about which solution is best suited for a particular problem, considering all the tradeoffs.

During interviews, you can also showcase your velocity in resolving problems and how to optimize them. For example, in case there is any problem involving array search, you can start with a simple algorithm, to demonstrate you can resolve a problem quickly, depending on the criticality, and once the problem is fixed, demonstrate it can be optimized to use a more performative search, if you have more time to resolve the problem.

In each chapter of this book, we will cover some problems pertaining to the chapter topic, and what we can do to further optimize them.

## Exercises

Now that you've explored the fundamentals of time and space complexity with Big O notation, it's time to test your understanding! Analyze the following JavaScript functions and determine their time and space complexities. Experiment with different inputs to see how the functions behave.

**_1_**: determines if the array's size is odd or even:

```
const oddOrEven = (array) => array.length % 2 === 0 ? 'even' : 'odd';
```

**_2_**: calculates and returns the average of an array of numbers:

```
function calculateAverage(array) {
  let sum = 0;
  for (let i = 0; i < array.length; i++) {
    sum += array[i];
  }
  return sum / array.length;
}
```

**_3_**: checks if two arrays have any common values:

```
function hasCommonElements(array1, array2) {
  for (let i = 0; i < array1.length; i++) {
    for (let j = 0; j < array2.length; j++) {
      if (array1[i] === array2[j]) {
        return true;
      }
    }
  }
  return false;
}
```

**_4_**: filters odd numbers from an input array:

```
function getOddNumbers(array) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    if (array[i] % 2 !== 0) {
      result.push(array[i]);
    }
  }
  return result;
}
```

You will find the answers in the source code for this chapter (file `src/02-bigOnotation/03-exercises.js`). Compare your analysis with the provided solutions to solidify your understanding of Big O notation in real-world JavaScript code!

## Summary

In this chapter, we delved into the fundamental concept of Big O notation, a powerful tool for analyzing and expressing the efficiency of algorithms. We explored how to calculate both time complexity (the relationship between input size and runtime) and space complexity (the relationship between input size and memory usage). We also discussed how Big O analysis is a crucial skill for software developers, aiding in algorithm selection, performance optimization, and technical interviews.

In the next chapter, we will dive into our first data structure: the versatile **Array**. We will explore its common operations, analyze their time complexities, and tackle some practical coding challenges.