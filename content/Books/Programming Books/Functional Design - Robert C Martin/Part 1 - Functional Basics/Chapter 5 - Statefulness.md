---
id: 01JF1N6T016W4F3MYNA1ZXRV5X
modified: 2024-12-13T23:00:53-05:00
---
## 5

## Statefulness

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/05_unnum_statefulness.jpg)

In the end, every program ever written is just a form of _y = f(x)_, where _x_ is all the input you give to the program and _y_ is all the output it delivers in response.

This definition is sufficient for all batch jobs. For example, in a payroll system, the input _x_ is all the employee records and timecards and the output _y_ is all the paychecks and reports.

But perhaps this batch definition is too simplistic. After all, in interactive applications, the input you give to the program is often based on the output it just gave you. So perhaps we should think of interactive software systems as:

```
void p(Input x) {
  while (x != DONE)
    x = (getInput(f(x))
}
```

In other words, our program is a loop that computes _y = f(x)_ and then hands _y_ to some source of input that is passed back into _f_ until _f_ finally returns `DONE`.

In some very real sense, the state of this program during each iteration is _x_. If you were debugging some malfunction, you would want to know the value of _x_ and would likely call _x_ the state of the system.

And indeed, in the program above, there is a variable named `x` that holds the state of the system and is updated upon each iteration.

However, we can eliminate that variable by writing the program “functionally” as follows:

```
void p(Input x) {
  if (x!=DONE)
    p(getInput(f(x)));
}
```

Now this program has no variable that is updated to hold the state of the system. Instead, that state is passed as an argument from one invocation of `p` to the next.

A few years ago I wrote a functional program in Clojure that looked very much like this. It was a version of the old computer game _Spacewar!_. You can see (and play) this program at [https://github.com/unclebob/spacewar](https://github.com/unclebob/spacewar). The game is visual and interactive, and it is written in the “functional” style.

The internal state of the `spacewar` program is enormously complex. It consists of the _Enterprise_, dozens of Klingons, hundreds of stars, many dozens of torpedoes, phaser blasts, kinetic projectiles, bases, transports, and a plethora of other entities and attributes. All that complexity is maintained within a single object that I called `world`. And the flow of `spacewar` is, for all intents and purposes:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05_images.xhtml#f045-01)

```
(defn spacewar [world]
  (when (:done? world)
    (System/exit 0))
  (recur (update-world world (get-input world))))
```

In other words, the `spacewar` program is a loop that exits if the :`done?`[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn1a) attribute of the `world` is `true`, and otherwise presents the `world` to the user and gets input that it uses to update the `world`.

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn1). Keywords in Clojure are prefixed with colons. So `:done?` is a keyword, which is just a constant that can be used as an identifier. Often, they are used as keys into hash maps. When used as a function, a keyword behaves like an accessor into a hash map. Thus, `(:done? world)` simply returns the `:done?` element of the `world` hash map.

Here is the actual `update-world` function as it currently exists within `spacewar`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05_images.xhtml#f045-02)

```
(defn update-world [ms world]
  ;{:pre [(valid-world? world)]
  ; :post [(valid-world? %)]}
  (->> world
       (game-won ms)
       (game-over ms)
       (ship/update-ship ms)
       (shots/update-shots ms)
       (explosions/update-explosions ms)
       (clouds/update-clouds ms)
       (klingons/update-klingons ms)
       (bases/update-bases ms)
       (romulans/update-romulans ms)
       (view-frame/update-messages ms)
       (add-messages)
       ))
```

The threading macro (`->>`) simply passes the argument `world` into `game-won`, the output of which gets passed to `game-over`, the output of which gets passed to `ship/update-ship`, and so on. Each of those functions returns an updated version of the `world`.

Note the `ms` argument. It contains the number of milliseconds since the last update and is the primary input to the game as a whole. As an object moves across the screen, its position is updated based upon its velocity vector and the number of milliseconds that have transpired since its position was last updated.

I’m showing this to you to give you a glimpse of the complexity being managed by this program. Keep in mind that the `world` is not a mutable variable. Each of those threaded functions into which the `world` is being passed is returning a new version of the `world` and passing it to the next. It is not being held in a variable and being mutated.

Let me give you one more glimpse of the complexity:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05_images.xhtml#f046-01)

```
(s/def ::ship (s/keys :req-un
                      [::x ::y ::warp ::warp-charge
                       ::impulse ::heading ::velocity
                       ::selected-view ::selected-weapon
                       ::selected-engine ::target-bearing
                       ::engine-power-setting
                       ::weapon-number-setting
                       ::weapon-spread-setting
                       ::heading-setting
                       ::antimatter ::core-temp
                       ::dilithium ::shields
                       ::kinetics ::torpedos

                       ::life-support-damage ::hull-damage
                       ::sensor-damage ::impulse-damage
                       ::warp-damage ::weapons-damage
                       ::strat-scale
                       ::destroyed
                       ::corbomite-device-installed]))
```

What you are looking at is a small portion of the type specification of the _Enterprise_, the player’s ship. Clojure provides a mechanism called `clojure.spec` that give us the ability to very specifically design our data structures with even more precision and control than most statically typed languages.

All this complexity of state is managed within the `spacewar` program by passing the `world` from function to function to function, and then recursively passing it back to `spacewar`. The `world` is never held in a variable.

And, the game operates on a large screen at 30 frames per second.

The bottom line here is that there is no level of complexity that demands that we abandon immutability and deviate from the functional style. On the other hand, there are other factors that do, from time to time, make that demand.

### When We MUST Mutate

The `spacewar` program uses a graphical user interface (GUI) framework called _Quil_.[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn2a) This framework allows the programs that use it to be written in a “functional” style. It may not actually be functional in its internals, but from the outside looking in, there need not be any visible mutable state.

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn2). See [www.quil.info](http://www.quil.info/). Quil uses _Processing_ behind the scenes. _Processing_ is a Java framework that is certainly not functional. Quil pretends to be functional by hiding the mutable variables, or at least by not forcing you to mutate those variables.

On the other hand, I am currently writing an application in Clojure named `more-speech`[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn3a) that uses Java’s Swing framework. Swing _is not functional_. Mutable state drips from every appendage of the framework. It is a definitionally mutable object framework.

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn3). [https://github.com/unclebob/more-speech](https://github.com/unclebob/more-speech)

This makes it a challenge to use with Clojure and maintain a “functional” style. To make matters worse, Swing uses a model-view approach, and the models are defined and controlled by Swing. So building an immutable model is virtually impossible.

Swing is not the only framework that forces you into the mutable world. There are many others. So, even if you are determined to use the “functional” style, you must be able to deal with the fact that a large panoply of existing software frameworks will force you out of that style.

Worse, many such frameworks also force you into the multithreaded world. Swing, for example, runs in its own special thread. Programmers should not use that thread for regular processing but must specifically enter that thread when mutating Swing data structures.

This puts the users of such frameworks into the double jeopardy of mutating state from within multiple threads. The dreaded result of that, of course, is race conditions and concurrent update anomalies.

Fortunately, there are functional languages that provide facilities that reduce the problems of mutation and allow the functional style to interface tolerably well with the multithreaded, nonfunctional style.

### Software Transactional Memory (STM)

_STM_ is a set of mechanisms that treat internal memory as though it were a transactional commit/rollback database. The transactions are functions that are protected from concurrent update by a _compare-and-swap_ protocol.

If that was too much of a word salad, perhaps an example would be clarifying.

Let us say that we have an object _o_ and a function _f_ that mutates _o_. So _of_ = _f(o)_ where _of_ is the original _o_ mutated by _f_.

The problem is that _f_ takes time to do its work, and there is a chance that some other thread will interrupt _f_ and apply its own operation _g_ on _o_: _og_ = _g(o)_. When _f_ finally completes, what is the state of _o_? Is it _of_? Or is it _og_? Or have both mutations been applied, giving us _of g_?

The typical concurrent update problem would most often yield _of_, causing the operation of _g_ to be lost. Programmers often resolve this kind of problem by _locking o_ so that _g_ cannot interrupt _f_, and vice versa. The lock forces the interrupting thread to wait until _o_ is unlocked. The problem, however, is that this can lead to the dreaded _deadly embrace_.[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn4a)

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn4). Sometimes known as _deadlock_.

Imagine that we have two objects _o_ and _p_ and two functions _f(o, p)_ and _g(p, o)_. These functions lock their arguments before operating on them. Suppose _f_ and _g_ are executing in different threads and _g_ interrupts _f_ just after _f_ locks _o_. Now _g_ locks _p_ but cannot lock _o_ because _o_ is locked by _f_, so _g_ waits. Now _f_ wakes up and tries to lock _p_ but cannot because _p_ is locked by _g_—and nothing can proceed. The functions _f_ and _g_ are in a deadly embrace.

The problem of deadly embrace can be avoided by locking everything in the same order every time. If _f_ and _g_ agree to lock _o_ first and _p_ second, then the embrace cannot happen. However, these agreements are hard to enforce, and as systems get more and more complicated, a correct locking order can be very difficult to divine.

STM solves this problem by _not_ locking, and instead using a commit/rollback technique. Let’s call this technique _swap_. We can enact it with _swap(o, f)_, which will hold the current value of _o_ in _oh_, compute _of_ = _f(o)_, and then, in an _atomic_[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn5a) operation, compare the current value of _o_ with _oh_ and, if they are the same, swap _o_ with _of_. If the compare fails, then the operation is repeated from the beginning and will continue repeating until the compare succeeds.

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn5). Atomic operations cannot be interrupted.

There are several ways to use STM in Clojure, but the simplest is the `atom`. An `atom` is an _atomic_ value that can be altered using the `swap!` function. Here’s an example:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05_images.xhtml#f050-01)

```
(def counter (atom 0))

(defn add-one [x]
  (let [y (inc x)]
    (print (str "(" x ")"))
    y))

(defn increment [n id]
  (dotimes [_ n]
    (print id)
    (swap! counter add-one)))

(defn -main []
  (let [ta (future (increment 10 "a"))
        tx (future (increment 10 "x"))
        _ @ta
        _ @tx]
    (println "\nCounter is: " @counter)))
```

The first line creates the `atom` named `counter`. The `-main` program starts two threads, using `future`, both of which call the `increment` function. The `@ta` and `@tx` expressions wait for the respective threads to complete.

The `add-one` function adds one to its argument, but that `print` function can allow another thread to jump in; and that’s exactly what happens. Here’s an example of the output:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05_images.xhtml#f051-01)

```
a(0)a(1)a(2)a(3)a(4)xa(5)x(5)(6)(6)x(7)(7)a(8)(8)
x(9)(9)a(10)(10)x(11)a(11)(12)(12)a(13)x(13)(14)(14)
x(15)(15)(16)x(17)x(18)x(19)
Counter is:  20
```

At first, thread `a` runs without interruption for a while. But at the fifth increment, the `x` thread jumps in, and the two fight each other. Notice the repeated values as the `swap!` detects the collisions and repeats. Finally, thread `a` finishes and thread `x` experiences no further interruptions. The end count of 20 is correct.

### Life Is Hard, Software Is Harder

It would be nice to live, full time, in a functional world. Multiple threads in a functional world generally do not have race conditions.[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn6a) After all, if you never update, you can’t have concurrent update problems. But all too often we are forced back into the multithreaded, nonfunctional world by frameworks, or legacy code. And when that happens, the mechanisms of STM can help us avoid the worst of an otherwise horrific situation.

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch05.xhtml#ch05fn6). See [Chapter 15](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch15), Concurrency, for when they do.