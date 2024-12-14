---
id: 01JF1N659VN3HRR7EBR0WFSJP1
modified: 2024-12-13T23:00:32-05:00
---
## 4

## Laziness

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/04_unnum_laziness.jpg)

Consider the following boldfaced change to our program that calculates a list of Fibonacci numbers:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch04_images.xhtml#f038-01)

```
(declare fib)

(defn fib-w [n]
  (cond
    (< n 1) nil
    (<= n 2) 1
    :else (+ (fib (dec n)) (fib (- n 2)))))

(def fib (memoize fib-w))


(defn lazy-fibs []
  (map fib (rest (range)))
  )
```

The `lazy-fibs` function may look a little strange to you. Let’s walk through it. You already understand the `map` function. The `rest` function takes a list and returns that list without the first element. And that brings us to the `range` function.

The `range` function, as called here, returns a list of integers starting at zero. How many integers, you ask? As many as you need. The `range` function is _lazy_. Or, rather, the range function returns a _lazy_ list.

What is a lazy list? A lazy list is an object that knows how to compute its next value. In Java, C++, and C#, we called such objects _iterators_. A lazy list is an iterator masquerading as a list.

Clojure is friends with lazy lists. Most of the library functions return lazy lists if possible. So, in the above program, `rest` and `map` both return a lazy list. And that means `lazy-fibs` also returns a lazy list.

How would you use `lazy-fibs`? Like so:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch04_images.xhtml#f039-01)

```
(take 10 (lazy-fibs))
returns: (1 1 2 3 5 8 13 21 34 55)
```

The `take` function takes two arguments: a number `n` and a list. It returns a list that contains the first `n` elements of the argument list. Actually, that’s not quite right, but I’ll get to that in a minute.

So, now let’s walk through `lazy-fibs` again. The `range` function returns a lazy list of integers starting at zero. The `rest` function takes that list, drops the first element, and then returns a lazy list of the remaining integers, which in this instance, are the integers starting at one. The `map` function applies each of those integers to the `fib` function returning a lazy list of the Fibonacci numbers starting at `(fib 1)`.

You can have as many Fibonacci numbers as you like, so long as there are no overflows or other machine limitations. So, for example:

```
(nth (lazy-fibs) 50)
returns: 20365011074
```

The `nth` function takes a list and an integer `n` and returns the `n`th element of the list. So this returns the 50th Fibonacci number.

Now consider this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch04_images.xhtml#f039-02)

```
(def list-of-fibs (lazy-fibs))
```

The `def` function (it’s not really a function, but pretend that it is) creates a new symbol and associates it with a value. So the symbol `list-of-fibs` refers to a lazy list of Fibonacci numbers, as you can see from the following:

```
(take 5 list-of-fibs)
returns: (1 1 2 3 5)
```

Now note: When we executed the `def` that created `list-of-fibs`, no Fibonacci numbers were calculated, and no memory was allocated for Fibonacci numbers. The calculations only take place, and the memory is only allocated, as the elements of the list are accessed. Remember, behind the scenes, the lazy lists are really just iterators that know how to calculate their next element. Once that calculation takes place, the memory is allocated and the value is placed into a real list.[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch04.xhtml#ch04fn1a)

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch04.xhtml#ch04fn1). That’s a convenient way to think of it for now. Actually, as we’ll see shortly, the memory is only allocated and the list only grows, if the program needs to hold those values.

It is tempting to think of lazy lists as being infinite. Of course they are not. They are simply unbounded. You can walk through as many items as you like, but that number will always be finite.

### Lazy Accumulation

It should be clear that if you continue to pass lazy lists through functions like `map`, `rest`, and `take` (yes, `take` actually returns a lazy list), you will accumulate a long chain of iterators behind the scenes. Each of those iterators must hold on to the function that calculates its next value. It must also hold on to all the data required for that calculation.

I have written applications that have lists with thousands of elements, each of which holds on to other lists with thousands of other elements; and all these lists are lazy. Now remember, we are _deferring_ calculations. None of the calculations take place until the final results are accessed. So a huge backlog of deferred iterators can get chained through all those lists.

This works fine until you run out of the memory allocated for holding all those deferred iterators. So, from time to time, it might be a good idea to convert your lazy lists into real lists. In Clojure, we do that with the `doall` function:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch04_images.xhtml#f040-01)

```
(def real-list-of-fibs (doall (take 50 (lazy-fibs))))
```

The `doall` function makes `real-list-of-fibs` a real list that occupies memory and contains no deferred iterators. All calculations have been done.

### OK, but Why?

Good question! Laziness is not free. It requires memory and cycles to defer calculations. Then there’s the problem of accumulation that can lead to memory exhaustion.

Yet despite these costs, laziness is a common—if not universal—feature in functional languages. Some languages, like Haskell, are intrinsically lazy. Clojure is not intrinsically lazy, but so many of the library functions are lazy that you cannot easily avoid the laziness. F# and Scala allow laziness, but you must be explicit about it.

Why? Why do all these languages accept the costs of laziness?

Because laziness decouples _what_ you need to do from _how much_ you need to do. You can write a program that creates a lazy sequence without knowing how big a sequence your users are going to want. Your users can determine how much of your sequence they need.

So, for example:

```
(nth (lazy-fibs) 500)
```

returns `22559151616193633087251269503607207204601132491375819058863`➥`8866418474627738686883405015987052796968498626N`

Since `lazy-fibs` puts no limit on the number of Fibonacci numbers it creates, you can ask for as many as you like.

Or, consider this example. I could create a list of 51 integers like this:

```
(range 51)
```

Or like this:

```
(take 51 (range))
```

Notice in the first example, the `51` is far more coupled than in the second. In the first, I have to get that `51` into the `range` function somehow. I might be able to pass it as an argument, but that’s a pretty strong coupling. In the second example, the `range` function doesn’t care at all. That `51` could be way out in some other part of the code, far removed from the call to `range`.

By the way, you might be interested to know that in the `lazy-fibs` example above, `(fib 1)` through `(fib 499)` have likely been garbage-collected. Since I’m not holding on to the list itself, the runtime system is free to dispose of the previously calculated elements. Thus, it would be possible to create and traverse a lazy list with trillions of elements and yet never hold more than one[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch04.xhtml#ch04fn2a) of them in memory at a time.

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch04.xhtml#ch04fn2). Or at least some `n`, where n is small and is the “chunk” size of the lazy engine.

### Coda

There is much more to learn about laziness. My purpose here has been to make you aware of it because it is so common in functional languages. We will be seeing much more of it in the pages to come, but it will almost always be in the background.