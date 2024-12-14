---
id: 01JF1NB5DTX36R6SC64W796153
modified: 2024-12-13T23:03:16-05:00
---
## 6

## Prime Factors

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/06_unnum_primefactors.jpg)

Is functional programming better than programming with mutable variables? Let’s do a comparative analysis of some familiar exercises. Here, for example, is the traditional Java derivation of the Prime Factors kata using TDD, roughly as it was presented in [Chapter 2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch02.xhtml#ch02) of _Clean Craftsmanship_.[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06.xhtml#ch06fn3a) A related video, _Prime Factors_, is also available. You can access the video by registering at [https://informit.com/functionaldesign](https://informit.com/functionaldesign).

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06.xhtml#ch06fn3). Martin, _Clean Craftsmanship_, p. 52.

### Java Version

We begin with a simple test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f056-01)

```
public class PrimeFactorsTest {
  @Test
  public void factors() throws Exception {
    assertThat(factorsOf(1), is(empty()));
  }
}
```

And we make it pass in this simple way:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f056-02)

```
private List<Integer> factorsOf(int n) {
  return new ArrayList<>();
}
```

Of course, this passes. So the next most degenerate test is 2:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f056-03)

```
assertThat(factorsOf(2), contains(2));
```

We make this pass with some trivial and obvious code:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f056-04)

```
private List<Integer> factorsOf(int n) {
  ArrayList<Integer> factors = new ArrayList<>();
  if (n>1)
    factors.add(2);
  return factors;
}
```

Next comes 3,

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f057-01)

```
assertThat(factorsOf(3), contains(3));
```

which we make pass by being a bit clever and replacing the `2` with `n`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f057-02)

```
private List<Integer> factorsOf(int n) {
  ArrayList<Integer> factors = new ArrayList<>();
  if (n>1)
    factors.add(n);
  return factors;
}
```

Next comes 4, which is the first time our list will have more than one factor in it:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f057-03)

```
assertThat(factorsOf(4), contains(2, 2));
```

And we make it pass with what appears to be a pretty awful hack:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f057-04)

```
private List<Integer> factorsOf(int n) {
  ArrayList<Integer> factors = new ArrayList<>();
  if (n>1) {
    if (n % 2 == 0) {
       factors.add(2);
      n /= 2;
    }
  }
  if (n>1)
    factors.add(n);
  return factors;
}
```

The next three tests pass without any changes:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f058-01)

```
assertThat(factorsOf(5), contains(5));
assertThat(factorsOf(6), contains(2,3));
assertThat(factorsOf(7), contains(7));
```

The 8 case is the first time we’ve seen more than two elements in the list of factors:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f058-02)

```
assertThat(factorsOf(8), contains(2, 2, 2));
```

And we pass this with the elegant transformation of one of the `if` statements into a `while`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f058-03)

```
private List<Integer> factorsOf(int n) {
  ArrayList<Integer> factors = new ArrayList<>();
  if (n>1) {
    while (n % 2 == 0) {
      factors.add(2);
      n /= 2;
    }
  }
  if (n>1)
    factors.add(n);
  return factors;
}
```

The next test, 9, must also fail because nothing in our solution factors out 3:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f058-04)

```
assertThat(factorsOf(9), contains(3, 3));
```

To solve it, we need to factor out 3’s. We could do that as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f058-05)

```
private List<Integer> factorsOf(int n) {
  ArrayList<Integer> factors = new ArrayList<>();
  if (n>1) {
    while (n % 2 == 0) {
      factors.add(2);
      n /= 2;
    }
    while (n % 3 == 0) {
      factors.add(3);
      n /= 3;
    }
  }
  if (n>1)
    factors.add(n);
  return factors;
}
```

But this is horrific because it implies endless duplication. We can solve that by changing another `if` to a `while`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f059-01)

```
private List<Integer> factorsOf(int n) {
  ArrayList<Integer> factors = new ArrayList<>();
  int divisor = 2;
  while (n>1) {
    while (n % divisor == 0) {
      factors.add(divisor);
      n /= divisor;
    }
    divisor++;
  }
  if (n>1)
    factors.add(n);
  return factors;
}
```

Just a little bit of refactoring and we get this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f059-02)

```
private List<Integer> factorsOf(int n) {
  ArrayList<Integer> factors = new ArrayList<>();

  for (int divisor = 2; n > 1; divisor++)
    for (; n % divisor == 0; n /= divisor)
      factors.add(divisor);
  return factors;
}
```

And that algorithm is sufficient to compute the prime factors of any[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06.xhtml#ch06fn4a) integer.

[4.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06.xhtml#ch06fn4) Given enough time and space.

### Clojure Version

OK, so what does this look like in Clojure?

As before, we begin with a simple degenerate test:[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06.xhtml#ch06fn5a)

[5.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06.xhtml#ch06fn5) Using the `speclj` testing framework.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f060-01)

```
(should= [] (prime-factors-of 1))
```

And we make that pass as one might expect, by returning an empty list:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f060-02)

```
(defn prime-factors-of [n] [])
```

The next test follows the Java version pretty closely:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f060-03)

```
(should= [2] (prime-factors-of 2))
```

So does the solution:

```
(defn prime-factors-of [n]
  (if (> n 1) [2] []))
```

And the solution to the third test employs the same clever replacement of `2` by `n`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f060-04)

```
(should= [3] (prime-factors-of 3))

(defn prime-factors-of [n]
  (if (> n 1) [n] []))
```

But with the test for 4, the Clojure and Java solutions begin to diverge:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f061-01)

```
(should= [2 2] (prime-factors-of 4))

(defn prime-factors-of [n]
  (if (> n 1)
    (if (zero? (rem n 2))
      (cons 2 (prime-factors-of (quot n 2)))
      [n])
    []))
```

The solution is recursive. The `cons` function prepends a `2` onto the beginning of the list returned by `prime-factors-of`. Convince yourself that you understand why! The `rem` and `quot` functions are just the integer remainder and quotient operations, respectively.

At this point in the Java program, there was no iteration. The two `if(n>1)` segments were a tantalizing hint of the iteration that was to come, but the solution was still just straight linear logic.

In the functional version, however, we see full-blown recursion. It’s not even tail-called.

The next four tests pass outright, even the test for 8:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f061-02)

```
(should= [5] (prime-factors-of 5))
(should= [2 3] (prime-factors-of 6))
(should= [7] (prime-factors-of 7))
(should= [2 2 2] (prime-factors-of 8))
```

In some ways, this is a shame since it was the test for 8 that caused us to transform an `if` to a `while` in the Java solution. No such elegant transformation takes place in the Clojure solution; though I have to say that the recursion is the better solution—so far.

Next comes the test for 9. And here the Java and Clojure versions face the similar dilemma of duplicated code:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f062-01)

```
(should= [3 3] (prime-factors-of 9))

(defn prime-factors-of [n]
  (if (> n 1)
    (if (zero? (rem n 2))
      (cons 2 (prime-factors-of (quot n 2)))
      (if (zero? (rem n 3))
        (cons 3 (prime-factors-of (quot n 3)))
        [n]))
    []))
```

This solution is not sustainable. It would force us to add the 5, 7, 11, 13… cases all the way up to the maximum prime that our language could hold. But this solution does imply an interesting iterative/recursive solution:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06_images.xhtml#f062-02)

```
(defn prime-factors-of [n]
    (loop [n n 
           divisor 2 
           factors []]
      (if (> n 1)
        (if (zero? (rem n divisor))
          (recur (quot n divisor) divisor (conj factors divisor))
          (recur n (inc divisor) factors))
        factors)))
```

The `loop` function creates a new anonymous function in situ. The `recur` function, when nested inside a `loop` expression, causes the in situ function to be reinvoked with TCO. The arguments to the in situ function are `n`, `divisor`, and `factors`. Each is followed by its initializer. So the `n` within the loop is initialized to the value of `n` outside the loop (the two `n` identifiers are distinct), `divisor` is initialized to `2`, and `factors` is initialized to `[]`.

The recursion in this solution is iterative because the recursive calls are at the tail. Note that the `cons` has been changed to a `conj` because the ordering of the list construction has changed. The `conj` function appends[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06.xhtml#ch06fn6a) to `factors`. Convince yourself that you understand why the ordering has changed!

[6.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch06.xhtml#ch06fn6) In this case because `factors` is a vector.

### Conclusion

There are several things to note about this example. First, the sequence of tests is the same between the Java and Clojure versions. This is significant because it implies that the change to functional programming has little to no impact on the way we express our tests. Tests are somehow more basic, more abstract, or more essential than the programming style.

Second, the solution strategy between the two deviated even before any iteration was required. In Java, the test for 4 did not require iteration; but in Clojure, it caused us to use recursion. This implies that recursion is somehow more semantically essential than standard looping with `while` statements.

Third, the derivation in Java was relatively straightforward; there were few, if any, surprises from one test to the next. But the Clojure derivation took a U-turn once we got to the test for 9. This was because we chose to use non-tail recursion instead of the iterative `loop` construct to solve the test for 4. This implies that, when we have a choice, we should prefer tail-recursive constructs to non-tail recursion.

The end result is an algorithm that is similar to the Java solution but has at least one surprising difference: It is not a doubly nested loop. The Java solution has one loop that increments the divisor and another that repeatedly adds the current divisor as a factor. The Clojure solution replaces that doubly nested loop with two independent recursions.

Which solution is better? The Java solution is a lot faster because Java is a lot faster than Clojure. But otherwise, I see no particular benefit to either. To those who know both languages well, neither is easier than the other to read or understand. Neither is riskier or better structured than the other. From my point of view, it’s a wash. Other than the intrinsic speed of Java, there is no advantage to either style that overrides the other.

However, this is the last example for which the results will be ambiguous. As we proceed from example to example, the differences will become more and more significant.