---
id: 01JF1N5H7DF8XB8ZF4X5SBM8GB
modified: 2024-12-13T23:00:11-05:00
---
## 3

## Recursion and Iteration

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/03_unnum_recursionanditeration.jpg)

In [Chapter 1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01), Immutability, I stated that functional programming makes use of recursion in order to eliminate assignment. In this chapter, we will look at the two different varieties of recursion; one we will call iteration and the other will retain the original name: recursion.

### Iteration

TCO is the remedy for the infinite stack depth implied by infinite recursive loops. However, TCO is only applicable if the recursive call is the very last thing to be executed within the function. Such functions are often called _tail call functions_.

Here is a very traditional implementation of a function to create a list of Fibonacci numbers:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f028-01)

```
(defn fibs-work [n i fs]
  (if (= i n)
    fs
    (fibs-work n (inc i) (conj fs (apply + (take-last 2 fs))))))

(defn fibs [n]
  (cond
    (< n 1) []
    (= n 1) [1]
    :else (fibs-work n 2 [1 1])))
```

This program is written in Clojure, which is a variant of Lisp. You call this function like this:

```
(fibs 15)
```

And it returns an array of the first 15 Fibonacci numbers:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f028-02)

```
[1 1 2 3 5 8 13 21 34 55 89 144 233 377 610]
```

Many programmers experience eyestrain headaches the first few times they look at Lisp, mostly because the parentheses don’t seem to make any sense. So let me give you a very brief tutorial about those parentheses.

#### Very Brief Clojure Tutorial

1. This is a typical function call in C, C++, C#, and Java: `f(x)`;.
    
2. Here is the same function in Lisp: `(f x)`.
    
3. Now you know Lisp. Here ends the tutorial.
    

That’s not much of an exaggeration. The syntax of Lisp is really that simple.

The syntax of Clojure is just a bit more complicated. So let’s take the above program apart, one statement at a time.

First there’s `defn`, which looks like it is being called as a function. Let’s go with that for now. The truth is mostly compatible with that view. So the `defn` “function” defines a new function from its arguments. The functions being defined are named `fibs-work` and `fibs`. The square brackets after the function name enclose the names of the arguments of the function.[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03.xhtml#ch03fn1a) So the `fibs` function takes a single argument named `n`, while the `fibs-work` function takes three arguments named `n`, `i`, and `fs`.

[1.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03.xhtml#ch03fn1) Actually, the square brackets are Clojure syntax for a “vector” (an array). In this case, that vector contains the symbols that represent the arguments.

Following the argument list is the body of the function. So the body of the `fibs` function is a call to the `cond` function. Think of `cond` like a switch statement that returns a value. The `fibs` function returns the value returned by `cond`.

The arguments to `cond` are a set of pairs. The first element in each pair is a predicate, and the second is the value that `cond` will return if that predicate is `true`. The `cond` function walks down the list of pairs until it sees a true predicate, and then it returns the corresponding value.

The predicates are just function calls. The `(< n 1)` predicate simply calls the `<` function with `n` and `1`. It returns `true` if `n` is less than 1. The `(= n 1)` predicate calls the `=` function, which returns `true` if its arguments are equal. The `:else` predicate is considered `true`.

The value returned by `cond` for the `(< n 1)` predicate is `[]`, an empty vector. If `(= n 1)`, then `cond` returns a vector containing 1. Otherwise, `cond` returns the value produced by the `fibs-work` function.

So, the `fibs` function returns `[]` if `n` is less than 1, `[1]` if `n` is equal to 1, and `(fibs-work n 2 [1 1])` in every other case.

Got it? Make sure you do. Go back over it until you do.

The `)))` at the end of the `fibs` function are just the closing parentheses of the `defn`, `cond`, and `fibs-work` function calls. I could have written `fibs` like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f030-01)

```
(defn fibs [n]
  (cond
    (< n 1) []
    (= n 1) [1]
    :else (fibs-work n 2 [1 1])
  )
)
```

Perhaps that makes you feel better. Perhaps that relieves the eyestrain headache you felt coming on. And indeed, many new Lisp programmers use this technique to reduce their parentheses anxiety. That’s certainly what I did a decade and a half ago when I first started learning Clojure.

After a few years, however, it becomes obvious that there is no reason to put trailing parentheses on their own lines, and the technique simply becomes an annoyance. Trust me. You’ll see.

Anyway, that brings us to the heart of the matter, the `fibs-work` function. If you have gotten comfortable with the `fibs` function, you have probably already worked out most of the details of the `fibs-work` function. But let’s go through it step by step just to be sure.

First, the arguments: `[n i fs]`. The `n` argument tells us how many Fibonacci numbers to return. The `i` argument is the index of the next Fibonacci number to compute. The `fs` argument is the current list of Fibonacci numbers.

The `if` function is a lot like the `cond` function. Think of `(if p a b)` as `(cond p a :else b)`. The `if` function takes three arguments. It evaluates the first as a predicate. If the predicate is true, it returns the second argument; otherwise, it returns the third.

So, if `(= i n)`, then we return `fs`. Otherwise… Well, let’s walk through that one carefully.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f031-01)

```
(fibs-work n (inc i) (conj fs (apply + (take-last 2 fs))))
```

This is a recursive call to `fibs-work`, passing in `n` unchanged, `i` incremented by one, and `fs` with a new Fibonacci number appended.

It is the `conj` function that does the appending. It takes two arguments: a vector and the value to append to that vector. Vectors are a kind of list. We’ll talk about them later.

The `take-last` function takes two arguments: a number `n` and a list. It returns a list containing the last `n` elements of the list argument.

The `apply` function takes two arguments: a function and a list. It calls the function with the list as its arguments. So, `(apply + [3 4])` is equivalent to `(+ 3 4)`.

OK, so now you should have a good working grasp of Clojure. There’s more to the language that we’ll encounter as we go along. But for now, let’s get back to the topic of iteration and recursion.

#### Iteration

Notice the recursive call to `fibs-work` is a tail call. The very last thing done by the `fibs-work` function is to call itself. Therefore, the language can employ TCO to eliminate previous stack frames and turn the recursive call into a `goto`, effectively converting the recursion to pure iteration.

So, then, functions that employ tail calls are, for all intents and purposes, iterative.

#### TCO, Clojure, and the JVM

The _Java virtual machine (JVM)_ does not make it easy for languages to employ TCO. Indeed, the code I just showed you does not use TCO and therefore grows the stack throughout the iteration. Thus, in Clojure, we _explicitly_ invoke TCO by using the `recur` function as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f032-01)

```
(defn fibs-work [n i fs]
  (if (= i n)
    fs
    (recur n (inc i) (conj fs (apply + (take-last 2 fs))))))
```

The `recur` function can only be called from a tail position, and it effectively reinvokes the enclosing function without growing the stack.

### Recursion

There is a much more natural and elegant way to write the Fibonacci algorithm using true recursion:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f032-02)

```
(defn fib [n]
  (cond
    (< n 1) nil
    (<= n 2) 1
    :else (+ (fib (dec n)) (fib (- n 2)))))

(defn fibs [n]
  (map fib (range 1 (inc n))))
```

The `fib` function should be self-explanatory by now. After all, _fib(n)_ is just _fib(n−1) + fib(n−2)_. Notice, however, the calls to `fib` are not on the tail of the function. The last thing executed by the `:else` clause is the `+` function. This means we cannot use the `recur` function and that TCO is not possible. This also means that the stack will grow as the algorithm proceeds.

The `range` function takes two arguments, _a_ and _b_, and returns a list of all the integers from _a_ to _b−1_. The `map` function takes two arguments, _f_ and _l_. The _f_ argument must be a function and the _l_ argument must be a list. It calls _f_ with each member of _l_ and returns a list containing the results.

This version of `fib` is extraordinarily inefficient. Consider this execution profile:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f033-01)

```
fib 20 = 6765
"Elapsed time: 1.459277 msecs"
fib 25 = 75025
"Elapsed time: 11.735279 msecs"
fib 30 = 832040
"Elapsed time: 106.490355 msecs"
fib 34 = 5702887
"Elapsed time: 735.689834 msecs"
```

I didn’t bother to analyze the algorithm. But a quick curve fit suggests that the algorithm is `O(n`3`)`. So, as elegant as the implementation appears, it will never do.

We can vastly improve the performance by using iteration as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f033-02)

```
(defn ifib
  ([n a b]
   (if (= 0 n)
     b
     (recur (dec n) b (+ a b))))

  ([n]
   (cond
     (< n 1) nil
     (<= n 2) 1
     :else (ifib (- n 2) 1 1)))
  )
```

The `ifib` function has two overloads: `[n a b]` and `[n]`. Since it is iterative, it does not grow the stack, and it is also much faster than the previous recursive version. Indeed, I believe most of that time was spent in printing rather than true computation.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03.xhtml#f034-01a)

```
ifib 20 = 6765
"Elapsed time: 0.185508 msecs"
ifib 25 = 75025
"Elapsed time: 0.177111 msecs"
ifib 30 = 832040
"Elapsed time: 0.14596 msecs"
ifib 34 = 5702887
"Elapsed time: 0.148221 msecs"
```

Of course, we’ve lost a lot of the expressive power of the recursive algorithm. We can reclaim that by remembering _referential transparency_: In a functional language, functions always return the same values given the same inputs. Thus, it is never necessary to reevaluate a function. Once we have computed the value of `(fib 20)`, we can remember it instead of recomputing it.

We do this by using the `memoize` function as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f034-01)

```
(declare fib)

(defn fib-w [n]
  (cond
    (< n 1) nil
    (<= n 2) 1
    :else (+ (fib (dec n)) (fib (- n 2)))))

(def fib (memoize fib-w))
```

The `declare` function creates an unbound symbol, which can be used by other functions so long as it is bound before its use. I used `declare` in this case because the definition of `fib` comes after `fib-w`, and Clojure wants all names declared or defined before they are used.

The `memoize` function takes an argument _f_, which must be a function, and returns a new function _g_. Calls to _g_ with argument _x_ will call _f_ with _x_ if, and only if, _g_ has never been called with _x_ before. It then remembers those arguments and the return value. Any subsequent call to _g_ with _x_ will return the remembered value.

This version of the algorithm is just as fast as the iterative version because we have short-circuited the vast majority of the recursion without sacrificing the elegance of the algorithm. We pay for that with a little extra memory, but that seems a small price to pay.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch03_images.xhtml#f035-01)

```
fib 20 = 6765
"Elapsed time: 0.168678 msecs"
fib 25 = 75025
"Elapsed time: 0.16232 msecs"
fib 30 = 832040
"Elapsed time: 0.151619 msecs"
fib 34 = 5702887
"Elapsed time: 0.15134 msecs"
```

What we have learned here is that iteration and recursion are very different approaches. Iterative functions must use tail calls to drive the iteration and should use TCO to prevent the growth of the stack. Recursive functions do not use tail calls and therefore will grow the stack. Truly recursive functions can be quite elegant, and memoization can be used to prevent that elegance from significantly affecting performance.

Although Clojure was used as the language in this chapter, the concepts are the same in virtually every other functional language, and could even be implemented in nonfunctional languages, though with a substantial loss of elegance. ;-)