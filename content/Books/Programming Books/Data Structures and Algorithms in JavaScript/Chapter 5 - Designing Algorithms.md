---
id: 01JH65N73MVBX5HWT6MB9R69KR
modified: 2025-01-09T13:36:44-05:00
---
## 5DESIGNING ALGORITHMS

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/opener.jpg)

This chapter covers several techniques for designing algorithms. We’ll start with recursion, which solves a problem by breaking it up into one or more simpler cases of the same problem. We’ll also look at _dynamic programming_, which solves a complex problem by solving simpler cases first and storing those solutions to avoid needless recalculations, as well as the _brute-force_ (or _exhaustive_) _search_ strategy, where you find a solution to a problem by systematically trying all possible solutions. Finally, we’ll explore _greedy algorithms_ that apply a heuristic of choosing the best local option at each junction of a problem, with the hope that the given methodology will lead to the

solution. Unlike the other strategies mentioned in this list, greedy algorithms may not always arrive at the best solution.

The strategies explored here are successfully applied to develop algorithms used along with data structures for the implementation of specific abstract data types (ADTs), so focusing on how to design a new solution for any given problem is worthwhile. The techniques covered in this chapter aren’t exhaustive, but they lie below the surface in many of the algorithms that we’ll explore later.

### Recursion

The simplest definition of recursion goes something like this: “A function calls itself over and over, again and again, until it doesn’t_._” In other words, when facing a problem, if it’s small enough, it can be solved without any further recursive calls, but if it isn’t, the function calls itself to solve smaller problems, and out of those solutions, it finds the solution for the original, larger problem.

> NOTE

_For a bit of computer humor, here’s a dictionary definition: “recursion: (n) see recursion.” A common saying is also “In order to understand recursion, you must first understand recursion.”_

As discussed in [Chapter 2](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter2.xhtml), recursion is a key technique in functional programming. Some languages, like Haskell, for instance, don’t even provide common “loops” and instead work exclusively with recursion. In computer science, recursion is all-sufficient for any algorithm, and anything you can do with loops you can also do with recursion. In fact, using recursion is much easier for many algorithms and definitions.

Recursion appears naturally in several areas:

**Mathematics** Definitions such as the factorial of a number or the Fibonacci series are naturally recursive. We’ll explore both later in this chapter.

**Data structures** Many structures are defined in a recursive fashion. For example, as you’ll see in [Chapter 10](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter10.xhtml), a _list_ may either be empty or consist of a special node, the head of the list, followed by another list; another example from [Chapter 13](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter13.xhtml) may be a _tree_, consisting of a parent node called the _root_, connected to any number of trees as its children.

**Procedures** Several algorithms can be expressed logically in a recursive fashion. An example from everyday life is searching your house for an object. You first look in one room, and if you find the object, you’re done; if you don’t find it, you search the rest of the place, applying the same logic. If you have nowhere left to search, you failed.

A recursive function always has two kinds of cases: simple ones that can be solved directly without any recursion and complex ones that need to use the function itself as an aid. The key to solving something recursively is to assume that the problem has already been solved and then code it using the (supposedly) available function. It’s a four-step procedure—which may seem to be circular:

1. Assume you already have a function that solves your problem.

2. Find some simple base cases that you can solve directly without any complications.

3. Figure out how you can solve the original problem by first solving one or more smaller versions of it.

4. Apply your assumed function from step 1 to solve the minor problems of step 3, or if they are small enough, solve them as in step 2.

Let’s look at a couple of recursion techniques to show how to design clear, simple-to-understand algorithms.

#### The Divide-and-Conquer Strategy

As mentioned, the basic idea in recursion is to base the solution of complex cases on the solution of simpler ones. You _divide_ the problem into smaller versions of itself, and _conquer_ it using the solutions to all of them. Often you’ll solve a problem by recursively solving an “only one smaller” version, and that strategy gets its own name, _decrease-and-conquer,_ but it’s still the same idea: reduce the original problem to smaller versions of itself. The only difference is that you solve the big problem by just solving _one_ (smaller) version first. We’ll start with a look at some simpler decrease-and-conquer examples and then move on to the divide-and-conquer strategy.

##### Calculating Factorials

The most often quoted example of a recursive calculation is likely the factorial of a number, _n_!, which is an example of the decrease-and-conquer strategy. The factorial of a non-negative integer number _n_ is defined as follows: for _n_ = 0, 0! = 1, and for _n_ > 0, _n_! = _n_ × (_n_ – 1)!, which is the recursive definition.

This formula comes from a recursive problem; namely, how many ways can you order _n_ books in a row on a shelf? The answer is simple: if no books are on the shelf, there’s only one way—an empty shelf. However, if you have _n_ > 0 books, you can choose any one of them (there are _n_ options), place it at the leftmost empty space on the shelf, and then place the (_n_ – 1) other books in all possible permutations to the right of the book you just placed, which is _n_! = _n_ × (_n_ – 1)! as just described: _n_ ways of choosing the first book multiplied by (_n_ – 1)! ways of ordering the rest.

A quick implementation of the factorial (see also question 5.1) is as follows:

```
const factorial = (n) => {
❶ if (n === 0) {
   return 1;
❷} else {
   return n * factorial(n - 1);
 }
};
```

This code closely follows the definition, with two clear cases: if _n_ is 0 ❶, return 1, and for greater values of _n_ ❷, use recursion. It’s hard to go wrong with a recursive implementation, because the logic matches the definition closely.

##### Searching and Traversing

Let’s look at a few other decrease-and-conquer examples using searching and traversing. In [Chapter 4](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter4.xhtml), we mentioned binary search, which is a way of searching an ordered array. If the array is empty, you’re out of luck; the value you want isn’t there. If the array isn’t empty, check its middle element, and if it’s what you want, you succeed. If the element doesn’t match and is higher than the value you want, search recursively in the left half of the array; otherwise, search its right half.

As another example, think about sorting a deck of cards. If the deck is empty, you’re done. Otherwise, you go through the deck looking for the lowest card and remove it from the deck. Then, you sort the rest of the deck and put it on top of the card you put aside.

Finally, let’s consider going through a list of pending tasks. (This is called a _traversal_ of the list.) If the list is empty, you’ve got nothing to do; you’re finished. Otherwise, you take the top task from the list, do it, and then recursively go through the remaining tasks.

##### Considering the Fibonacci Series

For a mathematical divide-and-conquer example, consider the Fibonacci series. This series starts with 0 and 1, and after that, each item is the sum of the two previous ones, so the series goes 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, and so on. (You’ll meet a Fibonacci-based structure in [Chapter 15](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter15.xhtml), and the usage of the series even applies to estimating the complexity of tasks in agile methodologies, so it’s certainly pervasive.)

> NOTE

_For curious readers, this series was named after an Italian mathematician, Leonardo of Pisa, also known as Fibonacci (“filius Bonacci,” or “son of Bonacci”), who is also famous for having introduced Arabic numbers to the Western world. Fibonacci posed (and solved) a question involving the growth of an idealized population of rabbits, but the sequence had already appeared in many other contexts, such as counting patterns of verse._

In order to implement the series recursively, you need to give a proper definition, and that’s not hard: you can say that F0 = 0, F1 = 1, and for _n_ > 1, F_n_ = F_n_–1 + F_n_–2. Given this definition, here’s the code:

```
const fibo = (n) => {
❶ if (n === 0) {
   return 0;
❷} else if (n === 1) {
   return 1;
❸} else {
   return fibo(n - 1) + fibo(n - 2);
 }
};
```

You have two base cases for 0 ❶ and 1 ❷ and a recursive case ❸ for other values. The code is so simple, it can’t go wrong, and testing verifies it. However, it does have a performance defect, which we’ll consider when we discuss dynamic programming later in this chapter.

##### Sorting and Puzzles

Many sorting methods that we’ll explore later in this book, such as merge sort or quick sort, are expressed succinctly in a recursive fashion, but let’s look at another classic example: the Towers of Hanoi puzzle.

This puzzle, invented by French mathematician Édouard Lucas in the 19th century, has three posts: the first has a stack of disks of decreasing size (largest at the bottom, smallest at the top), and the other two posts are empty. To solve the puzzle, you need to move all the disks from the first post to the last one, following two rules: you can move only the top disk from any post at a time (you can’t move two or more disks at once, and you can’t move any disks from the middle of a post), and you can move a disk to another post only if the top disk on that post is larger than the one you’re moving (a larger disk can never go on top of a smaller one). [Figure 5-1](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-1) shows the initial setup; all disks are on the leftmost post, and the goal is to move all the disks to the rightmost post.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-1.jpg)

Figure 5-1: Towers of Hanoi

How do you solve this with the divide-and-conquer strategy? You start by thinking you already have the required function, which might be towers(disks, origin, extra, destination), to move a certain number of disks from the origin post to the destination post using the extra post as an auxiliary, and you may use that function to implement the function itself. The base case is simple: if there are no disks to move, nothing needs to be done; otherwise, you make the moves described previously. The code could be as follows:

```
const towers = (disks, origin, extra, destination) => {
❶ if (disks > 0) {
  ❷ towers(disks - 1, origin, destination, extra);
  ❸ console.log(`Move disk ${disks} from ${origin} to ${destination}`);
  ❹ towers(disks - 1, extra, origin, destination);
  }
};
```

You first test for the base case ❶, because if there are no disks to move, you’re clearly done. Otherwise, recursively move all disks but the bottom one to the extra pole ❷. Having cleared the large disk, move it to the destination pole ❸, and finish by bringing the other disks on top of it ❹.

A call like towers(4, "A", "B", "C") to move four disks from pole A to pole C will produce the following output:

```
Move disk 1 from A to B
Move disk 2 from A to C
Move disk 1 from B to C
Move disk 3 from A to B
Move disk 1 from C to A
Move disk 2 from C to B
Move disk 1 from A to B
Move disk 4 from A to C
Move disk 1 from B to C
Move disk 2 from B to A
Move disk 1 from C to A
Move disk 3 from B to C
Move disk 1 from A to B
Move disk 2 from A to C
Move disk 1 from B to C
```

Using recursion to solve the simpler steps of a puzzle is a clear example of the divide-and-conquer strategy. (See question 5.2 if you ever need to do this puzzle with no computer.)

> NOTE

_There’s a coda to this puzzle. In the original version, monks had to move 64 golden disks from one post to another, and the world would end as soon as they accomplished the task. (In the original puzzle, the temple was in India; who knows how it moved abroad and got to Hanoi?) For n disks, M(n) = 2_n _– 1 moves are required to solve the puzzle, so it’s an algorithm of exponential order; the formula can be verified by noting that M(n) = 2M(n – 1) + 1 and M(0) = 0. At one movement per second, the achievement would require 2__64_ _– 1 seconds, more than 584 billion years, so we’re safe!_

#### The Backtracking Technique

_Backtracking_ is another problem-solving technique that’s usually best implemented in a recursive way. When facing multiple options, choose one and try finding a solution with it. If you succeed, you’re done. If you fail, backtrack to the point where you made the selection and choose a different option. If at some point you run out of options, there definitely is no solution.

##### Finding a Path in a Maze

Finding the way out of a maze (such as the one shown in [Figure 5-2](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-2)) is a classic, ancient problem, and you’ll see it again in [Chapter 17](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter17.xhtml), when working with graphs. It’s also the archetypical example for backtracking, so you’ll use it here. we’ll explore the full algorithms later; this is just the pseudocode.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-2.jpg)

Figure 5-2: A maze to be solved using backtracking

Whenever you get to a junction in the maze with two or more options available, you have to choose one, but obviously, you might end up choosing the wrong way. The idea is to follow the choice: if you get out of the maze, you succeeded, but if not, you backtrack to the last junction and select a different option. If no options are left, you’ll need to backtrack again, and again, until you either find a solution or decide there isn’t one. Here’s the pseudocode for such a recursive algorithm:

```
❶ solveMaze(fromCell, toCell, maze, path=[])
❷ if(fromCell === toCell) {
    return path // success!
  }
❸ mark fromCell as visited
❹ for all nextCell cells adjacent to fromCell {
  ❺ updatedPath = solveMaze(nextCell, toCell, maze, path + fromCell)
    if updatedPath is not null {
      return path
    }
  }
  // All adjacent cells were tried, and failed...
❻ return null   // failure
}
```

The parameters for this function ❶ are the starting point of the path, the final goal, the maze, and the path you’ll with your journey. If you reach the goal ❷, you have succeeded; otherwise ❸, mark the cell as visited so you don’t choose it again in the future and start trying all the available options ❹. If a path is shown again ❺, you’ve succeeded. When all options have been discarded ❻, you know you have to backtrack because you’ve failed. [Figure 5-3](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-3) shows an intermediate position in the search.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-3.jpg)

Figure 5-3: An intermediate step when solving the maze

At position 1, the algorithm had two possible options; it chose the left one and failed, and then it backtracked to choose the other one. (All the cells reached from that incorrect option were left marked.) At position 2, another selection was made; in this case, it chose the option on the right, and the left wasn’t (yet) considered, so the cells in that path remain unmarked. The algorithm currently stands at 3. If it doesn’t find a way out from here, it will backtrack to 2 to try the pending option. Whether it gets from position 3 to the exit quickly depends on the algorithm’s “luck” at choosing the correct option at each junction, but in any case, the algorithm is guaranteed to find a path eventually, if there is one, by recursively backtracking.

##### Solving the Squarest Game on the Beach Puzzle

Let’s apply this technique to the Squarest Game on the Beach puzzle developed by American puzzler Sam Loyd, shown in [Figure 5-4](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-4). In this puzzle, the players throw balls at dolls, and if they manage to knock over dolls whose numbers add up to 50, they win a cigar. (See question 5.3 for a similar puzzle you can also solve with backtracking.)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-4.jpg)

Figure 5-4: Sam Lloyd’s Squarest Game on the Beach puzzle (public domain)

You can implement a recursive backtracking algorithm as follows:

```
❶ const solve = (goal, standing, score = 0, dropped = []) => {
❷ if (score === goal) {
    return dropped;
❸} else if (score > goal || standing.length === 0) {
    return null;
  } else {
  ❹ const chosen = standing[0];
  ❺ const others = standing.slice(1);
  ❻ return (
      solve(goal, others, score + chosen, [...dropped, chosen]) ||
      solve(goal, others, score, dropped)
    );
  }
};
❼ console.log(solve(50, [15, 9, 30, 21, 19, 3, 12, 6, 25, 27]));
```

In the function ❶, goal is the number of points you try to make, and standing represents the available options, an array with the still-standing dolls. The points you have gotten so far will be held in score, and the dolls you knock over go in the dropped array. If you reach the goal exactly, you are finished ❷, and dropped has the list of dolls to drop. If you exceed our goal, or if there are no more dolls to drop ❸, you fail. Otherwise, you pick a doll ❹ (taking the first is simplest in terms of coding), remove it from future consideration ❺, and then attempt to solve the puzzle, including the recently chosen doll. If that fails, you backtrack and try again without including that doll ❻. To find the solution to the puzzle ❼, call solve() providing the goal (50) and list of doll points.

### Dynamic Programming

_Dynamic programming (DP)_ is a technique for solving a problem by first solving other (smaller) problems and storing those results, so they don’t need to be recalculated if they’re needed again. Dynamic programming comes in two flavors: top down, which solves the problem logically by checking whether it’s already been solved before dealing with a subproblem, and _bottom up_, which requires first looking at the smaller subproblems and then solving the original problem. In other words, with top-down DP, you try to solve the original problem directly and then recursively solve smaller problems first, and in bottom-up DP, you start with the simplest problems and move upward, solving harder problems step by step.

This description begs a question: What’s the best way to save previous results? We’ll look at two methods: _memoization_, based on a higher-order function in functional programming and probably best suited for top-down DP, and _tabulation_, based on arrays or matrices, which is typically best for bottom-up DP. Memoization is usually linked to recursive implementations, while tabulation is more useful for straightforward, nonrecursive solutions. The trade-off is that tabulation is probably quicker (not needing recursion) but may solve subproblems that aren’t actually needed, while memoization is slower (because of recursion) but will calculate strictly what’s needed.

#### Calculating Fibonacci Series with Top-Down DP

Let’s return to the Fibonacci numbers discussed earlier in the chapter. Here’s the code:

```
const fibo = (n) => {
  if (n === 0) {
    return 0;
  } else if (n === 1) {
    return 1;
  } else {
    return fibo(n - 1) + fibo(n - 2);
  }
};
```

This is a divide-and-conquer case, but as noted, that implementation has a problem, and we’ll solve it with dynamic programming. The code is clear, simple, and correct, but it can be quite slow. When experimenting with increasingly higher values of _n_, the required time to calculate the _n_th Fibonacci number grows exponentially as shown in [Figure 5-5](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-5). What’s happening?

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-5.jpg)

Figure 5-5: The number of additions needed to calculate Fibonacci numbers recursively grows exponentially.

To understand the problem, consider the calculations involved for fibo(7). [Figure 5-6](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-6) shows all the required calls.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-6.jpg)

Figure 5-6: Required calls to calculate fibo(7)

Clearly too many calls are repeated. Calculating fibo(7) implies summing fibo(6) and fibo(5), but the former is calculated as fibo(5) plus fibo(4), so you are repeating fibo(5). The diagram shows that the calculation of other Fibonacci numbers implies even more repetitions; how many times are fibo(3) or fibo(2) called? (See also question 5.4.) This implementation is exponential in order, so how do you solve it?

_Memoizing_ is a functional programming technique that may be applied to any pure function (those with no side effects that always return the same results for the same arguments, as discussed in [Chapter 2](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter2.xhtml)). The idea is that when a memoized function is called, it first checks an internal cache to see whether the calculation was already made. If so, it returns the cached value instead of redoing the calculation. If the requested value hasn’t been calculated before, the memoized function does its work, but before returning the result to the caller, it stores it in the internal cache for future reference.

Higher-order functions, such as fast-memoize (from _[https://www.npmjs.com/package/fast-memoize](https://www.npmjs.com/package/fast-memoize)_), are publicly available, but it’s not hard to whip up one yourself:

```
❶ const memoize = (fn) => {
  ❷ const cache = {};
  ❸ return (...args) => {
      ❹ const strX = JSON.stringify(args);
      ❺ return strX in cache
        ❻ ? cache[strX]
        ❼ : (cache[strX] = fn(...args));
    };
};
```

This higher-order function ❶ receives a function as an argument and returns a new one. It uses a closure to maintain a cache of previous calls and calculated values; you use a simple object here ❷, but you could also use a set (see [Chapter 11](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter11.xhtml) for other possible structures). The returned function ❸ first creates a string out of the arguments to the original fn function ❹. If that string is already used as a key in the cache ❺, directly return the previously calculated value from it ❻; otherwise, call the original function, store the returned value in the cache, and return it ❼.

Given memoize(), you can speed up the calculations straightaway with a minor change, wrapping the original function:

```
const fibo = memoize((n) => {
  ...
});
```

If you now try something like fibo(100), the results will be immediate. To understand why, you’ll need fibo(99) and fibo(98), but after calculating fibo(99), the value of fibo(98) will have been calculated, so it won’t be evaluated again. Each possible Fibonacci number between 0 and 100 will be calculated, but only once. The algorithm has become linear instead of exponential just by applying the dynamic programming technique of storing previously calculated values.

#### Line Breaking with Top-Down DP

Let’s look at a practical problem you can solve by applying top-down DP: building a nice-looking web form. Say you want a web page to be able to generate multiple forms onscreen, each with different sets of fields. If you had a fixed number of forms with predetermined, fixed sets of fields, it wouldn’t be a problem. However, in this case, the number of forms grows unpredictably, and fields need to be added or removed, as well as moved around, so you need a more flexible solution. What you need is a “form creator” that takes in a list of fields in a given order and produces a suitable form as output. For instance, a part of the form to be generated might look like the one shown in [Figure 5-7](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-7).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-7.jpg)

Figure 5-7: An example web form

The problem is you want a justified right margin, but the widths of the fields are inconsistent, so you’ll need to break rows and stretch some fields to make everything even. You need to be careful when deciding where to break rows and what fields to put in each row.

> NOTE

_The TeX typesetting system implements the Knuth-Plass algorithm to determine line breaks for paragraphs so they look nice. The problem here is essentially the same, but we’ll use DP to solve it instead._

Consider five fields of widths 7, 2, 5, 3, and 6 (see [Figure 5-8](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-8)). You need to arrange them in rows of width 10.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-8.jpg)

Figure 5-8: Example fields with various widths

You can’t manage with fewer than three rows, and having four or more rows results in too much wasted space (although we’ll need to quantify this concept later). You won’t add white space between fields (as TeX does between words); instead, you’ll expand the fields themselves. First decide how much empty space to leave in each row before expanding blocks or separating words. You have three possible layouts of three rows (see [Figures 5-9](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-9), [5-10](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-10), and [5-11](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-11); blocks in gray represent extra added white space at the end of rows; you’ll have to share that space among all blocks in the same row).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-9.jpg)

Figure 5-9: Field layout 1

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-10.jpg)

Figure 5-10: Field layout 2

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-11.jpg)

Figure 5-11: Field layout 3

What solution is best? Assuming that adding smaller spaces in many rows is better than adding larger spaces in fewer rows, consider the “row cost” to be the square of the added white space for that row, and the total cost will be the sum of all the row costs. (To better understand why squaring is used, imagine you need to add two spaces; putting all of them on the same line would cost 22 = 4, but placing one space in each of two lines would cost 12 + 12 = 2, so squaring the costs before adding them implements a policy that favors smaller spaces.) Given this definition, the cost of the layouts would be 12 + 22 + 42 = 21, 32 + 02 + 42 = 25, and 32 + 32 + 12 = 19, so the third diagram represents the design the algorithm should produce. Let’s program it.

Consider a list of block widths (which in this case would be 7, 2, 5, 3, and 6) and a maximum width (MW) to achieve. The following logic would work: calculate the sum _s_ of all widths, and if _s_ is not greater than MW, the cost is (MW – _s_)2. You can’t make it better by splitting the list into two or more rows. Otherwise, if you have more fields than you can fit in a single row, you can try splitting the list into two fragments in all possible ways and then choose the split that produces the lowest cost.

The following logic does that, but it leaves out the code to distribute white space among the fields in a row, since that’s only needed later. This code finds the cost of the best set of line breaks and where those breaks should be made:

```
const costOfFragment = (p, q) => {
❶ const s = totalWidth(p, q);
  if (s <= MW) {
  ❷ return [(MW - s) ** 2, [q]];
  }

❸ let optimum = Infinity;
❹ let split = [];
❺ for (let r = p; r < q; r++) {
  ❻ const left = costOfFragment(p, r);
    const right = costOfFragment(r + 1, q);
  ❼ const newTry = left[0] + right[0];
  ❽ if (newTry < optimum) {
      optimum = newTry;
      split = [r, . . .right[1]];
    }
  }
❾ return [optimum, split];
};
```

The function finds the best split for a set of blocks from p through q, inclusive, and also returns the list of splits to be made. Assume we have a totalWidth(x,y) function that calculates the width of blocks x through y (you’ll see how to best implement it later). First calculate the width of the whole list of blocks ❶; if it’s less than the available space, you won’t need any splits and you are done. Calculate the cost per the definition and return that a split is done after the q position ❷. If you need a split, set up a search; optimum will be the best possible cost ❸, and split will be the place to split the list ❹. Loop through all possible breaks ❺ and find the costs of fragments p through r and fragments r + 1 through q ❻. The cost of each split is stored ❼, and if it’s better than the previous optimum ❽, r is preferred as the new split. The end result ❾ is the best cost found, together with the list of split points.

[Figure 5-12](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-12) shows how this algorithm would deal with your list of blocks.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-12.jpg)

Figure 5-12: All the possible splits evaluated by the algorithm

Calculating costs, [Figure 5-13](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-13) shows the optimum solution.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-13.jpg)

Figure 5-13: The optimum solution

The cost is shown underneath each block. If a block is split into several blocks, its cost is the sum of the costs of its parts. The highlighted path shows how to achieve the optimum solution: leave 7 on its own in the first row, place 2 and 5 in the second row, and place 3 and 6 in the last row, for a total cost of 19. Running the algorithm produces the following result:

```
❶ const blocks = [7, 2, 5, 3, 6];
const costOfFragment = ...
❷ const result = costOfFragment(0, blocks.length - 1);
❸ console.log(result[0], result[1]);
// 19 [0, 2, 4]
```

You can define the list of block widths ❶, and using costOfFragment(...) ❷ produces the result: the best total cost is 19, and you split lines at positions 0 (just the 7), 2 (the 2 and the 5), and 4 (the 3 and the 6) as expected ❸.

You are done, but if you look closely at [Figure 5-13](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-13), you will notice the same problem as with the Fibonacci calculations: the cost of some blocks is calculated multiple times, for example, (5, 3, 6), (2, 5, 3), and (7, 2). You can apply memoization to avoid this problem, and that produces the needed algorithm:

```
const costOfFragment = memoize((p, q) => {
  ...
});
```

How would the optimized algorithm deal with this example? [Figure 5-14](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-14) shows how little would actually be calculated.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-14.jpg)

Figure 5-14: The optimized calculation reduces work in a significant way.

The grayed-out blocks don’t need recalculation; due to memoization, you just reuse the previously calculated costs. At several places (marked with arrows) no recursion was needed. All in all, the algorithm worked faster, but see question 5.5 for a further optimization.

#### Calculating Fibonacci Series with Bottom-Up DP

Let’s consider DP the other way round, from the bottom up. When working in a top-down fashion, you have to hold off on calculating values until some calculations for lower values are finished. For example, in the section “Calculating Fibonacci Series with Top-Down DP” on page 72, you couldn’t calculate fibo(7) until the calculations for fibo(6) and fibo(5) were done. Using the bottom-up method, you start at the lowest cases and work your way up. To find a Fibonacci number from the bottom up, make the calculations the same way that the series is defined, starting with 0 and 1, and always adding the last two numbers to create the next number in the sequence:

```
const fibo = (n) => {
❶ if (n < 2) {
    return n;
❷} else {
    let a = 0;
    let b = 1;
 ❸ while (n > 1) {
      [a, b] = [b, a + b];
      n--;
    }
 ❹ return b;
 }
};
```

It’s simple code: for 0 or 1 ❶ you need no calculations. For other values ❷, set up a loop starting with a = 0 and b = 1 (a and b represent the two latest numbers in the sequence), and loop enough times ❸ until b becomes the number you are seeking ❹.

You might notice that all the previously calculated numbers aren’t saved, and it’s true, but that’s because you don’t need them for this particular case. The algorithm works in a bottom-up fashion, calculating later numbers by using previous ones; it so happens that to do this, you always need only the two latest numbers, so there’s no need to store all the others.

#### Summing Ranges Recursively with Bottom-Up DP

In the line-breaking algorithm (see “Line Breaking with Top-Down DP” on [page 74](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#pg_74)), you needed the totalWidth(x,y) function that would add together the widths of values in positions x through y (both inclusive) of an array of block widths. This function needs to be as fast as possible so as not to impact an algorithm’s performance negatively. The trivial version (looping through the array, summing as it goes) has linear _O_(_n_) performance, if _n_ is the number of blocks. However, you can improve upon this, as you’ll see in a couple of alternative implementations, which will focus not only on DP but also on other techniques that you’ve seen in this chapter.

The first algorithm, using a loop to get the sum, is straightforward. Now include another parameter to the function, arr, with the block widths, to make it more general and independent of its caller:

```
const totalWidth1 = (arr, from, to) => {
  let sum = 0;
  for (let i = from; i <= to; i++) {
    sum += arr[i];
  }
  return sum;
};
```

To optimize it using memoization requires only a small change:

```
const totalWidth1 = memoize((arr, from, to) => {
  ...
});
```

This version will be faster if (and this is a big if) you call the function two or more times, with the same arguments. Calling it with different arguments every time will slow it down instead of speeding it up, because of the extra caching work. Suppose you had already calculated the sum of the range from 10 to 20, and now you wanted the sum of the range from 10 to 21. You could add the 21st value to the sum of the range from 10 to 20, with no more work.

This concept is the key to DP: base a problem’s solution on the solution of previous, smaller problems. To implement it, you need to define the sum of a range of values in terms of sums of previous ranges. If you want to calculate the sum of a range of values of array arr consisting of a single element (from p to p), the result is just arr[p]. If you want the sum of values from position 0 to position q (greater than zero), first sum the range from 0 to q-1 and then add arr[q] to that result. Finally, to find the sum of values from position p (greater than zero) to position q (greater than p), find the sum of range 0 to q and subtract the sum of range 0 to p-1.

You can also use memoization to keep track of previously calculated values; the logic is as follows:

```
const totalWidth2 = memoize((arr, from, to) => {
  if (from === to) {
    return arr[from];
  } else if (from === 0) {
    return totalWidth2(arr, 0, to - 1) + arr[to];
  } else {
    return totalWidth2(arr, 0, to) - totalWidth2(arr, 0, from - 1);
  }
});
```

This function works better and does less work. For instance, if you had asked for the sum of range 10 through 20, all sums from 0 to 0, 0 to 1, 0 to 2, and so on, up to 0 to 20 would need to be cached. If you then asked for the range 10 through 21, it would try to calculate the sum of range 0 to 21, which would be done immediately (as the sum of range 0 to 20, plus element 21), and subtract the sum of range 0 to 9 (which was already available). You still have an _O_(_n_) algorithm, but over time, it becomes an _O_(1) process; initial delays become amortized. But you can do even better.

#### Summing Ranges by Precomputing with Bottom-Up DP

Seeing how totalWidth2(...) in the previous section needs the sums of ranges from 0 to all possible other positions, you could use tabulation to precompute all those values, and then all queries would be _O_(1). You can use an internal cache (partial) for those values:

```
const totalWidth3 = ((tab) => {
❶ const partial = [0];
  tab.forEach((v, i) => {
  ❷ partial[i + 1] = partial[i] + v;
  });
❸ return (from, to) => partial[to + 1] – partial[from];
❹})(arr);
```

This is a bit trickier, because you are using a closure for the partial array, initialized in an immediately invoked function expression (IIFE). The precalculation sets up partial[k] to be the sum of the first _k_ elements in the original array, which correctly implies that partial[0] should equal 0 ❶. (You are wasting an extra array place, but that’s irrelevant in comparison with the speedy algorithm that you’ll get.) You also use DP to calculate these partial sums: partial[i+1] is calculated based on the previous calculation of partial[i] ❷. The function you want will calculate the total between two elements ❸ by taking the sum up to the rightmost one (partial[to+1]) and subtracting the sum of elements up to, but not including, the leftmost one (partial[from]), which produces the desired _O_(1) algorithm. The IIFE trickery is done by providing the original array of widths as a parameter ❹. (See question 5.6 for yet another way of doing this work.)

You’ve seen two different ways to use DP in a bottom-up fashion to optimize an algorithm, eventually reaching _O_(1) performance. Given that width calculations are frequently used to calculate line breaks, this is a game-changer for your code’s performance and usability.

### Brute-Force Search

Brute-force algorithms attempt to find a solution to a problem by systematically trying all possible combinations of values. The main issue with this kind of logic is the combinatorial explosion of the number of cases to try. The order of the resulting algorithms usually goes into exponential or factorial classes (as discussed in [Chapter 4](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter4.xhtml)) that makes them potentially impossible to use for even modest-sized input.

We’ll look at a problem in each category, going from worse to worst. Given the resulting order of algorithms in this category, there’s no surprise that we’ll avoid this kind of code in the rest of the book.

#### Detecting Tautologies

In terms of logic, a tautology is a boolean expression that is always true. For instance, if X, Y, and Z are boolean variables, two of the following JavaScript expressions are tautologies:

- X OR Y OR (NOT X AND NOT Y)
- X OR (NOT X AND Y) === X OR Y
- (NOT X) OR (X AND Z) OR (NOT Y) OR (Y AND Z) OR Z

Even for readers well versed in logic and expressions, it may not be immediately obvious which of these expressions are always true.

Recognizing whether a function of _n_ boolean parameters is a tautology potentially requires 2_n_ tests for each possible combination of true/false values, verifying for each one whether the function produces true as its result. Alternatively, you could try to find some combination of arguments that would make it false, and upon finding such a case, you’d know that the function isn’t a tautology. That kind of search would require a logic similar to what you used to solve the Squarest Game on the Beach puzzle.

Using recursion comes in handy: if a function of _n_ variables is a tautology, setting the first variable to false should also be a tautology, and the same would happen if the first variable were set to true. To see whether the original function is a tautology, you need to test a couple of functions with one fewer argument, which leads to a simple implementation:

```
❶ const isTautology = (fn, args = []) => {
❷ if (fn.length === args.length) {
  ❸ const result = !!fn(...args);
  ❹ if (!result) {
      console.log("Failed at", . . .args);
    }
    return result;
  } else {
  ❺ return (
      isTautology(fn, [...args, false]) && isTautology(fn, [...args, true])
    );
  }
};
```

The isTautology() function receives the original function to test, fn, and a list of arguments ❶. The latter will be the combination of values with which you’ll test whether the function is true. If you have the right number of arguments ❷, you evaluate the function ❸, and if it produces a false value ❹, you’ll log the fact and return false, which will short-circuit all future and pending evaluations. If the function returns true, the search will continue. If not enough arguments were provided ❺, you’ll test the function twice: adding a true and adding a false to the list of arguments, so all combinations will be tested eventually.

The following tests the three boolean expressions mentioned earlier:

```
const f = (x, y) => x || y || (!x && !y);
console.log(isTautology(f)); // true

const g = (x, y) => (x || (!x && y)) === (x || y);
console.log(isTautology(g)); // true

const h = (x, y, z) => !x || (x && z) || !y || (y && z) || z;
console.log(isTautology(h)); // false: Failed at true true false
```

The first two functions actually were tautologies, but the last one wasn’t. The search listed at least one case where the failed function evaluates to false.

#### Solving Cryptarithmetic Puzzles

_Cryptarithmetic puzzles_ (also known as _cryptarithms_) are puzzles that provide a mathematical equation where the digits have been replaced by letters of the alphabet. The goal for the solver is to find which letter stands for which digit. Usually no numbers may start with zero, all letters should have different values, and the equation should translate to a phrase that makes sense. [Figure 5-15](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-15) shows an early example of this, which was invented by British writer, puzzlist and mathematician Henry Ernest Dudeney in 1924.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-15.jpg)

Figure 5-15: A classic cryptarithmetic puzzle

You could solve this sort of puzzle with careful analysis (see question 5.7 for another example), but here you will write a solver that goes through all possible combinations of digits, checking whether any work. In this example, given that there are 10 digits, you need to check 10! (3,628,800) combinations, but some puzzles have numeric bases other than 10, so in general, this is an _O_(_n_!) algorithm. A similar example (in terms of its solution) is the traveling salesman problem that provides a list of _n_ cities and the distances between each pair of cities; you need to find the shortest possible route that visits each city only once, returning to the starting city. The solution to this is likewise _O_(_n_!), and the algorithm is similar to the one you’ll see next. (You’ll also see a different type of solution using a greedy algorithm for this problem, later in this chapter.)

What algorithm do we need? The idea is simple: try all combinations of digits from 0123456789 to 9876543210, and check for each one whether the puzzle is solved. (In this case, you’ll use only the first eight digits, but that really doesn’t change anything.) You could design the main logic as follows, assuming that puzzle() is a function to test whether a combination is valid:

```
❶ const solve = (puzzle, digits = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) => {
❷ const d = [...digits].sort();

❸ for (;;) {
  ❹ if (puzzle(...d)) {
      console.log("SOLUTION: ", . . .d);
      return true;
    }
  ❺ // Try generating the next combination of d.
    // If there are no more combinations left, return false.
  }
};
```

The digits parameter ❶ will have the set of digits that you’ll use for the problem; although in this case, 0 to 9 are the possible values, you may as well write code that could be used for cryptarithmetic puzzles in other bases. Make a local copy of the set of digits ❷ to avoid modifying the original argument and to avoid a side effect (as discussed in [Chapter 2](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter2.xhtml)), and sort it to go through the combinations in ascending order. Then set a loop ❸ that will exit when you either find a solution or decide none exists. If the current combination of digits works out ❹, log it and exit; otherwise, generate the next combination of digits ❺ and keep looping, unless you reach the last combination, and then you’ll know that the problem has no solution.

Generating the next permutation of a given set is a well-known algorithm, likely discovered by Indian mathematician Narayana Pandita in the 14th century. Assuming that the current permutation is stored in array d, it requires four steps, in order:

1. Find the rightmost index p such that d[p] < d[p + 1]; if no such p exists, you were already at the last permutation, and the algorithm finishes.

2. Find the rightmost index q such that d[p] < d[q]; d[q] is the least value to the right of d[p]greater than it.

3. Swap the values of d[p] and d[q]; now the values from d[p + 1] to the end of d will be in descending order.

4. Reverse the list of values from d[p + 1] to the end of d.

[Figure 5-16](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-16) shows a working example, starting with permutation 8403976521.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-16.jpg)

Figure 5-16: Generating the next permutation

Step 1 sets p to point at 3, because 3 < 9; all other elements from the right (976521) are in descending order. Step 2 sets q to point at 5, the least value to the right of 3 that is greater than it. Step 3 swaps the values pointed by p and q; values to the right of p are again in descending order (976321). Step 4 ends by reversing the values to the right of p, which end in ascending order (123679), and you get the next permutation: 8405123679.

With this logic, you can see the complete version of the cryptarithmetic puzzle solver by adding the code to generate permutations:

```
const solve = (puzzle, digits = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]) => {
  const d = [...digits].sort();

  for (;;) {
    if (puzzle(...d)) {
      console.log("SOLUTION: ", . . .d);
      return true;
    }

    let p = d.length - 2;
    while (p >= 0 && d[p] > d[p + 1]) {
      p--;
    }

    if (p === -1) {
      console.log("No solution found");
      return false;
    }

    let q = d.length - 1;
    while (d[p] > d[q]) {
      q--;
    }

    [d[p], d[q]] = [d[q], d[p]];

    let l = p + 1;
    let r = d.length - 1;
    while (l < r) {
      [d[l], d[r]] = [d[r], d[l]];
      l++;
      r--;
    }
  }
};
```

This code is the same as earlier, with steps 1 through 4 of the permutation algorithm highlighted.

You can now write a function to test whether a given combination of values is actually a solution:

```
❶ const sendMoreMoney = (s, e, n, d, m, o, r, y) => {
❷ if (s === 0 || m === 0) {
    return false;
❸} else {
    const SEND = Number(`${s}${e}${n}${d}`);
    const MORE = Number(`${m}${o}${r}${e}`);
    const MONEY = Number(`${m}${o}${n}${e}${y}`);
    return SEND + MORE === MONEY;
  }
};

❹ solve(sendMoreMoney);
// SOLUTION:  9 5 6 7 1 0 8 2 3 4
// 9567 + 1085=10652
```

The function is called with all 10 digits ❶, but you use only the first 8, ignoring the last 2. If a leading digit is 0 ❷, the solution isn’t valid, so reject that out of hand. If there are no leading zeros ❸, compute the values of the three words (SEND, MORE, and MONEY) and we check whether they fulfill the original equation. Given this function, all you need to do is pass it to the solve() function ❹ and wait (very little) for the solution to appear.

### Greedy Algorithms

Let’s conclude with a set of algorithms that have a rather curious characteristic: they may not always work. The basic definition of an algorithm implies that it’s a well-defined procedure to solve a problem or accomplish some task. _Greedy algorithms_ may (or may not) achieve that.

Sometimes a heuristic is used to describe a way to get a (hopefully not too bad) solution in a faster way by applying some arbitrary choices instead of doing a thorough search. For instance, a chess algorithm could, in principle, always find the best move by considering all possible moves and all possible opponent responses, and all possible responses to those responses, and so on, but that approach grows exponentially and isn’t feasible. The alternative is a heuristic. Reconsidering the chess example, instead of going to the maximum depth, you would stop the search short after a few moves, do a ballpark evaluation of the resulting board positions, and choose the move that leads to the best evaluated move. This method doesn’t _guarantee_ making the best move, but it at least provides some solution.

Greedy algorithms are usually applied to optimization problems. You’ve seen algorithms that use brute force to try all possibilities; greedy algorithms don’t do that. At each step where a decision needs to be made, these algorithms make the best possible choice at the time. On one hand, this approach ensures that the algorithm proceeds quickly without needing to backtrack. On the other hand, the algorithm doesn’t necessarily make the best choice, because it doesn’t look far enough ahead. However, under certain conditions, which you’ll explore in the following sections, these algorithms perform well and are successful.

#### How to Make Change

How do you make change using the fewest bills and coins? In other words, suppose you have to pay out some amount using today’s US currency: $100, $50, $20, $10, $5, and $1 bills and $0.25 (quarter), $0.10 (dime), $0.05 (nickel), and $0.01 (penny) coins. How would you pay $229.60? You could use so many combinations to reach that amount, but with a greedy algorithm, you would follow this simple rule: at each step, choose as many as possible units of the largest possible denomination, and keep going until you’re done.

The method starts by using two $100 bills, then one $20, a $5, four $1s, two quarters, and a dime. No other solution involves fewer bills and coins. This greedy algorithm is guaranteed to succeed, but it depends on the available denominations. Paying out $16 (the greedy way) in a country that had $9, $8, and $1 bills would end up with one $9 and seven $1 bills, instead of using just two $8 bills. Greedy algorithms may (or may not) succeed depending on the case.

#### The Traveling Salesman Problem

Let’s consider a problem that requires a brute-force search but that a greedy algorithm usually solves quite well. The traveling salesman problem works like this: imagine a salesman has to do a tour, visiting each city on a list once and then returning to the starting point. (In graph terms, this is called a _Hamiltonian cycle_.) Distances (or costs) for traveling between cities is known. What’s the shortest (or cheapest) way to achieve the task?

As is, an algorithm for this problem would require testing all possible permutations of cities (as you did previously for the SEND + MORE = MONEY cryptarithmetic puzzle). If the number of cities grows, the problem becomes intractable because the required time to run the algorithm takes too long.

A greedy algorithm for this problem (which may not find the best solution but performs quickly) would proceed as follows: at each step visit the nearest not-yet-visited city. This method won’t necessarily find the best possible path, and several heuristics may discover an even better one, but under some conditions, the algorithm finds the optimal solution.

#### Minimum Spanning Tree

Let’s wrap up our discussion of greedy algorithms by considering a problem you’ll explore in [Chapter 17](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter17.xhtml). Imagine a cable TV company must provide service to several houses. The company cannot place cables just anywhere and must follow existing roads. Where should it put cables to minimize the total cost?

The solution for this is technically called a _minimum spanning tree_, and Kruskal’s algorithm (which you’ll implement in [Chapter 17](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter17.xhtml)) is a greedy algorithm that solves it and is guaranteed to find the optimal solution. Start by choosing the cheapest segment of road until all houses are connected, and always add the cheapest possible segment that won’t generate a loop; after all, what good would having a closed cable loop be?

You can solve other graph-related problems using greedy algorithms, so this technique may also be a valid one to consider when trying to write code for a specific problem.

### Summary

In this chapter we considered several techniques (recursion, DP, and brute-force and greedy algorithms) that will help you develop algorithms on your own, and they’ll appear again several times in the rest of the book.

In the next chapter, we’ll explore several common problems, such as sorting, selecting, shuffling, sampling, and searching—that’s a lot of alliteration but also a lot of interesting code and plenty of opportunities to study how to write algorithms.

### Questions

**5.1  Factorial in One**

The code for factorial() is totally correct, but it’s seven lines long! Not that it matters (having a long correct function is better than a short incorrect one), but can you write it in a more compact way?

**5.2  Hanoi by Hand**

The recursive algorithm for the Towers of Hanoi is good for computers, but not so much for normal human beings. Can you design a simple algorithm to solve the puzzle that doesn’t involve recursion?

**5.3  Archery Backtracking**

Sam Loyd devised another puzzle (see [Figure 5-17](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-17)) similar to the Squarest Game on the Beach that you solved earlier in this chapter. In this puzzle, you need to get 100 points by aiming arrows at the target. The important difference is that in this puzzle, you can hit a ring two or more times, while in the other problem you could drop a doll only once.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-17.jpg)

Figure 5-17: Another classic puzzle by Sam Loyd, where players must get exactly 100 points with their arrows (public domain)

Can you modify the backtracking algorithm to deal with this variation? And, even with the differences, can you still use the solve() function that you used for the other puzzle to find the solution?

**5.4  Counting Calls**

If you call _C_(_n_), the total number of calls needed to calculate the _n_th Fibonacci number with your recursive implementation, you will see, for example, that _C_(7) = 41. Can you give a recurrence for _C_(_n_) and find an explicit solution for it? Hint: the answer will again involve Fibonacci numbers.

**5.5  Avoid More Work**

When considering how to arrange blocks in rows (in “Line Breaking with Top-Down DP” on [page 74](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#pg_74)) and when considering splits, you analyzed them as (7, 2, 5) and (3, 6) or (7, 2, 5, 3) and (6). However, this wasn’t really needed because blocks 7 + 2 + 5 or 7 + 2 + 5 + 3 couldn’t fit in a line. [Figure 5-18](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-18) shows crossed-out options that an enhanced algorithm wouldn’t have considered.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-18.jpg)

Figure 5-18: An even more efficient way to find row breaks

Can you add this optimization to the code?

**5.6  Reduce for Clarity**

The totalWidth3(...) function (in the section “Summing Ranges by Precomputing with Bottom-Up DP” on [page 81](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#pg_81)) used a common loop to generate the partial array. Can you do the same using .reduce(...) instead?

**5.7  Got GOUT?**

[Figure 5-19](https://learning.oreilly.com/library/view/data-structures-and/9798341620001/xhtml/chapter5.xhtml#fig5-19) shows another well-known cryptarithmetic puzzle; find what each letter stands for, and then the value of GOUT is your answer. You can solve this with the techniques shown earlier in the chapter, or you could try your hand at directly working it out.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9798341620001/files/images/Figure5-19.jpg)

Figure 5-19: A simple cryptarithmetic puzzle with only four letters to find