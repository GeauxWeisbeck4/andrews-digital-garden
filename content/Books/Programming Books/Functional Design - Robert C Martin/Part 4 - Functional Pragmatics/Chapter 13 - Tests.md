---
id: 01JF1NJR8V7MJF6RC1XHNE8P8Q
modified: 2024-12-13T23:07:25-05:00
---
## 13

## Tests

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/17_unnum_tests.jpg)

Throughout this book, you’ve seen many of the unit tests I have written. In virtually every case, I used the TDD[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn1a) discipline of writing my tests and code in a tight loop, with the tests a few seconds ahead of the code.

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn1). I have written a great deal about this discipline in _Clean Craftsmanship_ (Addison-Wesley, 2021), _Clean Code_ (Pearson, 2008), and _Agile Software Development: Principles, Patterns, and Practices_ (Pearson, 2002). There is also a vast amount of information available on the Web. One of the best books on the topic is _Growing Object-Oriented Software, Guided by Tests_ by Steve Freeman and Nat Pryce (Addison-Wesley, 2010).

For the most part, those tests were written using a framework called `speclj`[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn2a) (pronounced “speckle”), written by Micah Martin and others. It is very similar to the RSpec framework that is popular in Ruby.

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn2). [https://github.com/slagyr/speclj](https://github.com/slagyr/speclj)

I have been practicing TDD for well over 20 years now. I’ve used it in Java, C#, C, C++, Ruby, Python, Lua, Clojure, and a variety of other languages. What I have learned in those decades is that the language does not matter to the discipline. The discipline is the same regardless of the language.

The fact that Clojure is a functional language does not change my testing strategy, nor affect my use of the TDD discipline. I write my Clojure programs test-first the way I write my Java programs test-first. The paradigm doesn’t matter. The discipline is universal.

### But What about the REPL?

Lots of functional programmers say they don’t need TDD because they test everything in the REPL. I do lots of experimenting in the REPL too; but in most cases, I encode what I’ve learned into a test. Tests, like diamonds, are forever. Experiments in the REPL aren’t there the morning after.

### What about Mocks?

_Mocking_ is a technique used by TDD practitioners to encapsulate their tests away from large swaths of the system. In effect, they create objects, called _mocks_,[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn3a) that represent those swaths and use the LSP to substitute the mocks in for them.

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn3). They are more formally referred to as _test-doubles_, but in this context, I’ll continue to use the colloquial vernacular.

Since the LSP is viewed as an OO principle, and since mocks in OO languages are based on polymorphic interfaces, it has become something of an urban myth that functional languages do not support mocks.

But as we have seen, the LSP works just as well in a functional language as it does in an OO language, and polymorphic interfaces are generally very easy to create. Thus, the ability to write mocks, in all their various forms, is not at all impeded in a functional language.

As an example, here is a test from my `more-speech`[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn4a) application that employs a couple of mocks:

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn4). [https://github.com/unclebob/more-speech](https://github.com/unclebob/more-speech)

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f185pre01)

```
(it "adds an unrooted article id to a tab"
  (let [message-id 1
        messages {message-id {:tags []}}
        event-context (atom {:text-event-map messages})]
    (reset! ui-context {:event-context event-context})
    (with-redefs [swing-util/add-id-to-tab (stub :add-id-to-tab)
                  swing-util/relaunch (stub :relaunch)]
      (add-article-to-tab 1 "tab" nil)
      (should-have-invoked :relaunch)
      (should-have-invoked :add-id-to-tab
                           {:with ["tab" :selected 1]}))))
```

Don’t worry too much about what this test does. Just look down at the `with-redefs` statement. This test mocks the `swing-util/add-id-to-tab` and `swing-util/relaunch` functions to use named stubs. Those stubs are perfect no-ops. They accept any number of arguments and return nothing at all.[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn5a) But they do remember what happened to them.[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn6a) So, down at the bottom, we see that the `:relaunch` stub should have been called, and the `:add-id-to-tab` stub should have been called with three arguments: `"tab"`, `:selected`, and `1`.

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn5). There are ways to get them to return values, but that’s beyond the scope here. Check the `speclj` docs ([https://github.com/slagyr/speclj](https://github.com/slagyr/speclj)) if you are interested.

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn6). Which technically makes them spies.

### Property-Based Testing

One cannot hang out with functional programmers without eventually hearing about QuickCheck and property-based testing. Unfortunately, the topic often arises as a counterargument to TDD. I’m not going to try to support or refute that argument. Instead, I want to show you how very powerful property-based testing is within the TDD discipline.

First of all, what is property-based testing? _Property-based testing_ is a verification and diagnostic technique that employs the random generation of inputs and a very powerful strategy of defect isolation.

Let’s say that I’ve just written a function that computes the prime factors of a given integer:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f186pre01)

```
(defn factors-of [n]
  (loop [factors [] n n divisor 2]
    (if (> n 1)
      (cond
        (> divisor (Math/sqrt n))
        (conj factors n)
        (= 0 (mod n divisor))
        (recur (conj factors divisor)
               (quot n divisor)
               divisor)
        :else
        (recur factors n (inc divisor)))
      factors)))
```

Let’s also say that I wrote this function using TDD. Here are my tests:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f187pre01)

```
(defn power2 [n]
  (apply * (repeat n 2N)))

(describe "factor primes"
  (it "factors 1 -> []"
    (should= [] (factors-of 1)))
  (it "factors 2 -> [2]"
    (should= [2] (factors-of 2)))
  (it "factors 3 -> [3]"
    (should= [3] (factors-of 3)))
  (it "factors 4 -> [2 2]"
    (should= [2 2] (factors-of 4)))
  (it "factors 5 -> [5]"
    (should= [5] (factors-of 5)))
  (it "factors 6 -> [2 3]"
    (should= [2 3] (factors-of 6)))
  (it "factors 7 -> [7]"
    (should= [7] (factors-of 7)))
  (it "factors 8 -> [2 2 2]"
    (should= [2 2 2] (factors-of 8)))
  (it "factors 9 -> [3 3]"
    (should= [3 3] (factors-of 9)))
  (it "factors lots"
    (should= [2 2 3 3 5 7 11 11 13]
             (factors-of (* 2 2 3 3 5 7 11 11 13))))
  (it "factors Euler 3"
    (should= [71 839 1471 6857] (factors-of 600851475143)))

  (it "factors mersenne 2^31-1"
    (should= [2147483647] (factors-of (dec (power2 31))))))
```

Pretty cool, right? But how certain am I that this function actually works? I mean, how do I know that there isn’t some horrible corner case where the function fails unexpectedly?

Of course, I may never be perfectly sure about this; but there are some things I can do to make myself a lot more comfortable. One property of the output is that the product of all the factors will equal the input. So why don’t I generate a thousand random integers and make sure that the prime factors of each multiply together to equal them.

I can do that like so:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f188pre01)

```
(def gen-inputs (gen/large-integer* {:min 1 :max 1E9}))

(declare n)7

(describe "properties"
  (it "multiplies out properly"
    (should-be
      :result
      (tc/quick-check
        1000
        (prop/for-all
          [n gen-inputs]
          (let [factors (factors-of n)]
            (= n (reduce * factors))))))))
```

[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn7). A forward declaration of `n`.

Here I’m using `test.check`,[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn8a) the property-based testing framework in Clojure that mimics the behavior of QuickCheck. The idea is pretty simple. I’ve got a generator up there named `gen-inputs`. It will generate random integers between 1 and a billion. That ought to be a good enough range.

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13.xhtml#ch13fn8). [https://clojure.org/guides/test_check_beginner](https://clojure.org/guides/test_check_beginner)

The test tells QuickCheck to run 1,000 times. For each integer, it calculates the prime factors, multiplies them all together, and makes sure that the product equals the input. Nice.

The `tc/quick-check` function returns a map with the results. The `:result` element of that map will be `true` if all the checks passed; and that’s what the `should-be :result` asserts.

There is another property of the prime factors: They should all be prime. So let’s write a function that tests for primality:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f189pre01)

```
(defn is-prime? [n]
  (if (= 2 n)
    true
    (loop [candidates (range 2 (inc (Math/sqrt n)))]
      (if (empty? candidates)
        true
        (if (zero? (rem n (first candidates)))
          false
          (recur (rest candidates)))))))
```

That’s a pretty traditional, if horribly inefficient, algorithm. Inefficient or not, we can use it to write the property test for the primality of all the factors:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f189pre02)

```
(describe "factors"
  (it "they are all prime"
    (should-be
      :result
      (tc/quick-check
        1000
        (prop/for-all
          [n gen-inputs]
          (let [factors (factors-of n)]
            (every? is-prime? factors)))))))
```

OK. So now we know that this function returns a list of integers, each of which is prime, and that when multiplied together equal the input. That’s kind of the definition of prime factors.

So this is nice. I can randomly generate a bunch of inputs and then apply property checks to the outputs.

### A Diagnostic Technique

But I called property-based testing a diagnostic technique, didn’t I? So let’s look at a more interesting example and I’ll show you want I mean.

Remember our Video Store example from the preceding chapter? Let’s do some property-based testing on that.

First of all, remember that we wrote a function called `make-statement-data` that took a `policy` and a `rental-order` and generated the `statement-data` that we then fed into one of our formatters? So here’s the type specification of the `rental-order` using `clojure.spec`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f190pre01)

```
(s/def ::name string?)
(s/def ::customer (s/keys :req-un [name]))
(s/def ::title string?)
(s/def ::type #{:regular :childrens :new-release})
(s/def ::movie (s/keys :req-un [::title ::type]))
(s/def ::days pos-int?)
(s/def ::rental (s/keys :req-un [::days ::movie]))
(s/def ::rentals (s/coll-of ::rental))
(s/def ::rental-order (s/keys :req-un [::customer ::rentals]))
```

That’s not too hard to read. From the bottom up:

- A `:rental-order` is a map with two elements: `:customer` and `:rentals`.
    
- The `:rentals` element is a collection of `:rental` items.
    
- A `:rental` is a map with `:days` and `:movie` elements.
    
- A `:days` element is a positive integer.
    
- A `:movie` element is a map with a `:title` and `:type`.
    
- A `:type` is one of `:regular`, `:childrens`, or `:new-release`.
    
- A `:title` is a string.
    
- A `:customer` is a map with a single `:name` element.
    
- A `:name` is a string.
    

With this type specification in place, we can write a generator that produces rental orders that conform to the type. So first, here are the generators:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f191pre01)

```
(def gen-customer-name
  (gen/such-that not-empty gen/string-alphanumeric))

(def gen-customer
  (gen/fmap (fn [name] {:name name}) gen-customer-name))

(def gen-days (gen/elements (range 1 100)))

(def gen-movie-type
  (gen/elements [:regular :childrens :new-release]))

(def gen-movie
  (gen/fmap (fn [[title type]] {:title title :type type})
            (gen/tuple gen/string-alphanumeric gen-movie-type)))

(def gen-rental
  (gen/fmap (fn [[movie days]] {:movie movie :days days})
            (gen/tuple gen-movie gen-days)))

(def gen-rentals
  (gen/such-that not-empty (gen/vector gen-rental)))

(def gen-rental-order
  (gen/fmap (fn [[customer rentals]]
              {:customer customer :rentals rentals})
            (gen/tuple gen-customer gen-rentals)))

(def gen-policy (gen/elements
                  [(make-normal-policy)
                   (make-buy-two-get-one-free-policy)]))
```

I’m not going to explain the ins and outs of `clojure.check` here, but I will walk through what the generators do.

- `gen-policy` randomly selects one of the two policies.
    
- `gen-rental-order` creates a map from `gen-customer` and `gen-rentals`.
    
- `gen-rentals` creates a vector from `gen-rentals` and ensures that it is not empty.
    
- `gen-rental` creates a map from `gen-movie` and `gen-days`.
    
- `gen-movie` creates a map from `gen/string-alphanumeric` and `gen-movie-type`.
    
- `gen-movie-type` selects from among the three types.
    
- `gen-days` selects between integers from 1 to 100.
    
- `gen-customer` creates a map with a name from `gen-customer-name`.
    
- `gen-customer-name` generates a nonempty alphanumeric string.
    

Do you notice an eerie similarity between the type specification and the generator? So do I. Here are a few sample outputs from the generator:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f192pre01)

```
[
 {:customer {:name "5Q"},
  :rentals [{:movie {:title "", :type :new-release}, :days 52}]}

 {:customer {:name "3"},
  :rentals [{:movie {:title "", :type :new-release}, :days 51}]}

 {:customer {:name "XA"},
  :rentals [{:movie {:title "r", :type :regular}, :days 82}
            {:movie {:title "", :type :childrens}, :days 60}]}

 {:customer {:name "4v"},
  :rentals [{:movie {:title "3", :type :childrens}, :days 29}]}

 {:customer {:name "0rT"},
  :rentals [{:movie {:title "", :type :regular}, :days 42}
            {:movie {:title "94Y", :type :regular}, :days 34}
            {:movie {:title "D5", :type :new-release},
                     :days 58}]}

 {:customer {:name "ZFAK"},
  :rentals [{:movie {:title "H8", :type :regular}, :days 92}
            {:movie {:title "d6WS8", :type :regular}, :days 59}
            {:movie {:title "d", :type :regular}, :days 53}
            {:movie {:title "Yj8b7", :type :regular}, :days 58}
            {:movie {:title "Z2q70", :type :childrens},
                     :days 9}]}

 {:customer {:name "njGB0h"},
  :rentals [{:movie {:title "zk3UaE", :type :regular},
                     :days 53}]}

 {:customer {:name "wD"},
  :rentals [{:movie {:title "51L", :type :childrens},
             :days 17}]}

 {:customer {:name "2J5nzN"},
  :rentals [{:movie {:title "", :type :regular}, :days 64}
            {:movie {:title "sA17jv", :type :regular}, :days 85}
            {:movie {:title "27E41n", :type :new-release},
                     :days 85}
            {:movie {:title "Z20", :type :new-release}, :days 68}
            {:movie {:title "8j5B7h6S", :type :regular},
                     :days 76}
            {:movie {:title "vg", :type :childrens}, :days 30}]}

 {:customer {:name "wk"},
  :rentals [{:movie {:title "Kq6wbGG", :type :childrens},
                     :days 43}
            {:movie {:title "3S2DvUwv", :type :childrens},
                     :days 76}
            {:movie {:title "fdGW", :type :childrens}, :days 42}
            {:movie {:title "aS28X3P", :type :childrens},
                     :days 18}
            {:movie {:title "p", :type :childrens}, :days 83}
            {:movie {:title "xgC", :type :regular}, :days 84}
            {:movie {:title "CQoY", :type :childrens}, :days 23}
            {:movie {:title "38jWmKlhq", :type :regular},
                     :days 96}
            {:movie {:title "Liz8T", :type :regular}, :days 56}]}
 ]
```

Just a bunch of random data that conforms nicely to the type of a `rental-order`. But let’s check that:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f194pre01)

```
(describe "Quick check statement policy"
  (it "generates valid rental orders"
    (should-be
      :result
      (tc/quick-check
        100
        (prop/for-all
          [rental-order gen-rental-order]
          (nil?
            (s/explain-data
              ::constructors/rental-order
              rental-order))))))
```

This is a nice little `quick-check` that generates 100 random `rental-order` objects and runs them through the `clojure.spec/explain-data` function. That function makes sure that each rental order conforms to the `::constructors/rental-order` spec that we saw above. If it does, it returns `nil`, which passes the `quick-check`.

Now, does `make-statement-data` create a valid `statement-data` object? Let’s check that using the same strategy as above:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f194pre02)

```
(s/def ::customer-name string?)
(s/def ::title string?)
(s/def ::price pos?)
(s/def ::movie (s/keys :req-un [::title ::price]))
(s/def ::movies (s/coll-of ::movie))
(s/def ::owed pos?)
(s/def ::points pos-int?)
(s/def ::statement-data (s/keys :req-un [::customer-name
                                         ::movies
                                         ::owed
                                         ::points]))

(it "produces valid statement data"
  (should-be
    :result
    (tc/quick-check
      100
      (prop/for-all
        [rental-order gen-rental-order
         policy gen-policy]
        (nil?
          (s/explain-data
            ::policy/statement-data
            (make-statement-data policy rental-order)))))))
```

So here we see the `clojure.spec` for the `statement-data`, and the `quick-check` that makes sure that the output of `make-statement-data` conforms to it. Nice.

With all this passing, we can be pretty sure that the generator is generating valid rental orders. So now let’s get on with the property checks.

One property we could check is to make sure that when `make-statement-data` converts a `rental-order` into a `statement-data` the `:owed` member of the `statement-data` object is the sum of all the movies itemized in that object.

The `quick-check` for this might be as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f195pre01)

```
(it "statement data totals are consistent under all policies"
  (should-be
    :result
    (tc/quick-check
      100
      (prop/for-all
        [rental-order gen-rental-order
         policy gen-policy]
        (let [statement-data (make-statement-data
                               policy rental-order)
              prices (map :price (:movies statement-data))
              owed (:owed statement-data)]
          (= owed (reduce + prices)))))))
```

This `quick-check` has a bug in it. Can you spot it?

Here’s the output when I run it:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch13_images.xhtml#f196pre01)

```
{:shrunk
 {:total-nodes-visited 45,
  :depth 14,
  :pass? false,
  :result false,
  :result-data nil,
  :time-shrinking-ms 3,
  :smallest
    [{:customer {:name "0"},
      :rentals [{:movie {:title "", :type :regular}, :days 1}
                {:movie {:title "", :type :regular}, :days 1}
                {:movie {:title "", :type :regular}, :days 1}]}
     {:type
      :video-store.
        buy-two-get-one-free-policy/buy-two-get-one-free}]},
  :failed-after-ms 0,
  :num-tests 7,
  :seed 1672092997135,
  :fail
   [{:customer {:name "4s7u"},
     :rentals
     [{:movie {:title "i7jiVAd", :type :childrens}, :days 85}
      {:movie {:title "7MQM", :type :new-release}, :days 26}
      {:movie {:title "qlS4S", :type :new-release}, :days 99}
      {:movie {:title "X", :type :regular}, :days 87}
      {:movie {:title "w1cRbM", :type :regular}, :days 11}
      {:movie {:title "7Hb41O5", :type :regular}, :days 63}
      {:movie {:title "xWc", :type :childrens}, :days 41}]}
    {:type
     :video-store.
       buy-two-get-one-free-policy/buy-two-get-one-free}],
  :result false,
  :result-data nil,
  :failing-size 6,
  :pass? false}
```

Yes, I know this looks awful; but this is where the real magic of `quick-check` shines through, so bear with me.

First of all, do you see that top element named `:shrunk`? That’s a big clue to what is going on here. When `quick-check` finds an error, it begins hunting for the smallest randomly generated input that continues to produce that error.

So look at the `:fail` element. That’s the `rental-order` that caused the initial failure. Now look at the `:smallest` element within the `:shrunk` element. The `quick-check` function managed to shrink the `rental-order` down while preserving the failure. That’s the smallest `rental-order` that it could find that failed.

And why did it fail? Notice that there are three movies. Notice also that the policy is `buy-two-get-one-free`. Ah, of course, under that policy the sum of the movies is _not_ equal to the `:owed` element.

It’s that shrinking behavior that makes property-based testing a diagnostic technique.

### Functional

So why are tools like `quick-check` not more popular in OO languages? Perhaps it’s because they work best with pure functions. I imagine it’s possible to set up generators and test properties in a mutable system, but it’s likely a lot more complicated than in an immutable system.