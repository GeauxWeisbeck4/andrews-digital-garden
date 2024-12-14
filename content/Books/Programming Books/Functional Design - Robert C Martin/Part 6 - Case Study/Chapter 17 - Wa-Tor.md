---
id: 01JF1NPZ6HVPDDAJXVHV3F5SVT
modified: 2024-12-13T23:09:43-05:00
---
## 17

## Wa-Tor

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/28_unnum_wator.jpg)

In the final chapter of this book, you and I are going to play a little game about a little game. The little game our little game will be about is called _Wa-Tor_; a simple little cellular automaton described by A. K. Dewdney in the December 1984 issue of _Scientific American_.[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn1a) The game you and I are going to play is to _pretend_ that Wa-Tor is an enterprise-level application requiring significant effort in architecture and design.

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn1). Alas, _SciAm_, I knew it well. . .

I mean, honestly, I could hack together Wa-Tor in a few hours and walk away happy. But for this chapter, I want us to really think about the issues as though this were a 50 mega line of code (LOC) monster.

So what is Wa-Tor?[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn2a) The Wikipedia article referenced in the footnote should give you all the information you need to understand it in the required depth (which is not much). But essentially, Wa-Tor is a typical predator/prey simulation using fish and sharks. The fish move around randomly and occasionally reproduce. The sharks also move around randomly but will eat a fish if one is adjacent. Sharks will occasionally reproduce if they eat enough fish. Sharks will die if they do not eat a fish before they starve.

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn2). [https://en.wikipedia.org/wiki/Wa-Tor](https://en.wikipedia.org/wiki/Wa-Tor)

The world that the fish and sharks live in has no land; it’s all water. Moreover, the top meets the bottom and the left meets the right, so the world is topologically a torus. Thus, Wa-Tor stands for WAter TORus.

We’ll talk more about the features of the program later. For the moment, what are the architectural and design considerations?

Let’s start with the basics. SRP. Who are the actors—whom do we want to keep separate?

In most large enterprise systems, there are many different actors. But in this little app, there are only two to worry about. There are the user experience (UX) designers, who will undoubtedly change their minds a dozen or so times before they actually like what they see on the screen. And then there are the modelers who will also likely fiddle with the internal shark/fish behavior and might possibly add more animals to the mix.

So we start out with [Figure 17.1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fig01), a very obvious and very traditional partitioning.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c17f001.jpg)

**Figure 17.1.** The obvious and traditional partitioning of Wa-Tor

The `WatorUI` component is lower level[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn3a) than the `WatorModel` component. According to the Dependency Rule, this means that the source code dependencies must cross the architectural boundary pointing toward the `WatorModel`. Because of this, the `WatorUI` will be a plug-in to the `WatorModel`.

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn3). The definition of high and low “level” that I’m using here is “distance from I/O.” See Robert C. Martin, _Clean Architecture_ (Pearson, 2017), p. 183.

There are only two components[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn4a) and one boundary in this partitioning so far. In larger systems, we would see many more boundaries and many more components within each.

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn4). See Martin, _Clean Architecture_, p. 93.

Let’s focus on the model first.[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn5a) What kinds of classes are we going to need?

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn5). [http://wiki.c2.com/?ModelFirst](http://wiki.c2.com/?ModelFirst)

Yes, I said classes. We may be using a functional language, but if you’ve learned anything in this book so far, it is that functional design and OO design are two sides of the same coin.

So, at first blush, I think the object model looks something like [Figure 17.2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fig02).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c17f002.jpg)

**Figure 17.2.** Initial object model of Wa-Tor

The `world` contains a bunch of `cell`s. Each `cell` can process a `tick`[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn6a) of time. I guessed that `cell` is abstract rather than an interface because I expect that there will be concrete functions at this level.

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn6). Dewdney called these _chronons_.

Each `cell` can be `water`, or an `animal` that can `move` and `reproduce`. The two possible subtypes of `animal` are `fish` and `sharks` that can `eat`.

Let’s see if we can code this. No tests yet, because we haven’t defined any behavior:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f290-01)

```
(ns wator.cell)

(defmulti tick ::type)

——————

(ns wator.water
  (:require [wator
             [cell :as cell]]))

(defn make [] {::cell/type ::water})

(defmethod cell/tick ::water [water]
  )

———————

(ns wator.animal)

(defmulti move ::type)
(defmulti reproduce ::type)

(defn tick [animal]
  )

—————

(ns wator.fish
  (:require [wator
             [cell :as cell]
             [animal :as animal]]))

(defn make [] {::cell/type ::fish})

(defmethod cell/tick ::fish [fish]
  (animal/tick fish)
  )

(defmethod animal/move ::fish [fish]
  )

(defmethod animal/reproduce ::fish [fish]
  )

—————

(ns wator.shark
  (:require [wator
             [cell :as cell]
             [animal :as animal]]))

(defmethod cell/tick ::shark [shark]
  (animal/tick shark)
  )

(defmethod animal/move ::shark [shark]
  )

(defmethod animal/reproduce ::shark [shark]
  )

(defn eat [shark]
  )
```

This looks pretty standard. The `cell` module looks like an interface so far. The `water` module implements it trivially. The dangling parentheses are there to remind me that I want to add something to that function.

The `animal` module _does not_ implement `tick`, but it does have a function named `tick` that can be called by its subtypes. I put this in as a guess. It’s a bit of hubris, I suppose; but I have a feeling that it’ll be necessary.[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn7a)

[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn7). Yeah, I know. You Aren’t Gonna Need It (YAGNI). Well, we’ll see.

The `fish` trivially implements both the `cell` and `animal`. This actually looks more like multiple inheritance than the UML diagram. On the other hand, there’s no inheritance anywhere in this code, so. . .

Finally, `shark` also trivially implements both `cell` and `animal` and adds its own `eat` function.

I didn’t code the `world` because I don’t know enough to even start. However, there are a few issues that I think the `world` will have to deal with. We don’t want the `world` to depend upon the GUI, and yet the GUI is going to put a lot of constraints on the `world`. For example, it seems to me that the GUI is going to tell us the size of the `world`. I also think that since the GUI is likely to repaint the screen _N_ times per second, the GUI will define _time_.

But let’s set all that aside for the time being. Enough of this up-front design. Let’s see if we can code some of the behavior.

What is the behavior of `water`? We ask our modelers, and they tell us that a `water` cell will randomly evolve into a `fish` cell if given enough time. Here’s my implementation of that rule:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f293-01)

```
(ns wator.core-spec
  (:require [speclj.core :refer :all]
            [wator
             [cell :as cell]
             [water :as water]
             [fish :as fish]]))

(describe "Wator"
  (with-stubs)
  (context "Water"
    (it "usually remains water"
          (with-redefs [rand (stub :rand {:return 0.0})]
            (let [water (water/make)
                  evolved (cell/tick water)]
              (should= ::water/water (::cell/type evolved)))))


    (it "occasionally evolves into a fish"
      (with-redefs [rand (stub :rand {:return 1.0})]
        (let [water (water/make)
              evolved (cell/tick water)]
          (should= ::fish/fish (::cell/type evolved)))))))

———

(ns wator.water
  (:require [wator
             [cell :as cell]
             [fish :as fish]
             [config :as config]]))

(defn make [] {::cell/type ::water})

(defmethod cell/tick ::water [water]
  (if (> (rand) config/water-evolution-rate)
    (fish/make)
    water))

——————

(ns wator.config)

(def water-evolution-rate 0.99999)
```

So, right away we see the “functional” nature of this program.[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn8a) The return value of `tick` is a new `cell`. I don’t know if that `water-evolution-rate` is correct. The modelers haven’t told us what the rate should be. So I just guessed. I expect that they’ll wait until they see how the model behaves and then tell us to change it.

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn8). Almost. The `(rand)` invocation is impure.

So far, I haven’t specified any dynamic types. It seems a bit early for that. But I’m pretty sure it’s coming.

Anyway, let’s see if we can make a `fish` move.

Wait. How do you move a `fish`? Where is the `fish`? Does the `fish` know its location, or is that something the `world` knows?

The `cell`s are arranged in a two-dimensional rectangular Cartesian grid that wraps left to right and top to bottom. So the location of a `cell` is the tuple `[x y]`. The `world` could hold the `cell`s in a two-dimensioned array, or in a map keyed by the position tuple.

I like using maps for things like this, so let’s make a `world` full of `water` cells:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f294-01)

```
(context "world"
  (it "creates a world full of water cells"
    (let [world (world/make 2 2)
          cells (:cells world)
          positions (set (keys cells))]
      (should= #{[0 0] [0 1]
                 [1 0] [1 1]} positions)
      (should (every? #(= ::water/water (::cell/type %))
                      (vals cells))))))

————

(ns wator.world
  (:require [wator
             [water :as water]]))

(defn make [w h]
  (let [locs (for [x (range w) y (range h)] [x y])
        loc-water (interleave locs (repeat (water/make)))
        cells (apply hash-map loc-water)]
    {:cells cells}))
```

Did you catch the use of the lazy list of `water` cells passed into `interleave`? Now we should be able to put a `fish` in the world and move it around. Here’s my first try at a test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f295-01)

```
(context "animal"
  (it "moves"
    (let [fish (fish/make)
          world (-> (world/make 3 3)
                    (world/set-cell [1 1] fish))
          [loc cell] (animal/move fish [1 1] world)]
      (should= cell fish)
      (should (#{[0 0] [0 1] [0 2]
                 [1 0] [1 2]
                 [2 0] [2 1] [2 2]}
               loc)))))
```

This is pretty straightforward. We create a 3-by-3 `world` with a `fish` in the center. Then we move the `fish`. Finally, we make sure it’s still a `fish` and that its destination is one of the neighboring cells.

I made a ton of design decisions while composing this test. Those kinds of decisions are why the last _D_ in _TDD_ often stands for _design_. I’ll walk you through those decisions in a moment, but first let me show you the code that passes this test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f296-01)

```
(ns wator.world
  (:require [wator
             [water :as water]]))

(defn make [w h] . . .)

(defn set-cell [world loc cell]
  (assoc-in world [:cells loc] cell))

——————

(ns wator.animal
  (:require [wator
             [cell :as cell]]))

(defmulti move (fn [animal & args] (::cell/type animal)))
(defmulti reproduce (fn [animal & args] (::cell/type animal)))

(defn tick [animal]
  )

(defn do-move [animal loc world]
  [[0 0] animal])

——————

(ns wator.fish
  (:require [wator
             [cell :as cell]
             [animal :as animal]]))

(defn make [] {::cell/type ::fish})

(defmethod cell/tick ::fish [fish]
  (animal/tick fish)
  )

(defmethod animal/move ::fish [fish loc world]
  (animal/do-move fish loc world))

(defmethod animal/reproduce ::fish [fish]
  )
```

When you see `. . .` in a method body, it means that there has been no change to that method since the last time I presented it.

There’s nothing really astonishing here. I changed the `defmulti` definitions in `animal` to accept multiple arguments, and I created a default `do-move` method in `animal` that the subtypes can call if they like.[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn9a) The implementation of `do-move` is degenerate and is there only to test the test.

[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn9). This is kind of like implementing a method in a base class and allowing subclasses to either override it or not.

So, on to the design decisions that I made while composing this test. My first problem was that an `animal` can’t `move` if it can’t see the `world`. So either every `animal` should hold a reference to the `world`, or the `world` should be a global `atom`, or the `world` should be passed in as an argument to the `move` function. I chose the latter because I feel a kind of mild disdain[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn10a) for abandoning the functional paradigm and falling back on `atom`s and STM.

[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn10). Perhaps that disdain is misplaced, but this IS a book about functional design, so. . .

My next problem was that the `animal` does not know its location. So I need to pass the location of the `animal` into the `move` function along with the `world`.

Finally, and most importantly, I puzzled over what the `move` function should return. At first, I thought it should return the updated `world`. But this creates the following inconsistency problem.

Imagine the update process for the `world`. It begins at location `[0 0]` and walks through the `world` updating each `cell` in turn. Now imagine there is a `fish` at `[0 0]` and that the update moves it to `[0 1]`. But `[0 1]` is the `cell` that the `world` updates next. So that same fish moves _again_. A `fish` should not move twice in a single turn.

So the `move` function cannot update the `world`. Instead, the `world` is going to have to build up a new world from the old world, one cell at a time. I imagine we could do it something like this:[11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn11a)

[11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn11). Remember that `:cells` holds a map, so the `update-cell` function will take `[key val]` pairs and return `[key val]` pairs.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f298-01)

```
(let [new-world-cells (apply hash-map
                             (map update-cell old-world-cells))]. . .)
```

So now let’s actually implement the degenerate `do-move` function. What is the process for moving an `animal`? I think it’s pretty simple. We just get the neighbors of the animal’s location, determine which are valid destinations (i.e., are `water`), and then randomly choose from that list. So `do-move` should look like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f298-02)

```
(defn do-move [animal loc world]
  (let [neighbors (world/neighbors world loc)

        destinations (filter
                       #(water/is?
                         (world/get-cell world %))
                       neighbors)
        new-location (rand-nth destinations)]
    [new-location animal]))
```

Very pretty. We ask the `world` for the `neighbors` of the location, filter out any that aren’t `water`, and then randomly choose one. Cool.

I thought it best to make sure that all the torus math was nicely sequestered within `world`. I didn’t want it leaking out into all the `animal`s:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f299-01)

```
(defn wrap [world [x y]]
  (let [[w h] (::bounds world)]
    [(mod x w) (mod y h)])
  )

(defn neighbors [world loc]
  (let [[x y] loc
        neighbors (for [dx (range -1 2) dy (range -1 2)]
                    (wrap world [(+ x dx) (+ y dy)]))]
    (remove #(= loc %) neighbors))
```

Are you ready for the stuff that’s not pretty? The code above refused to compile, because (are you ready for this?) `water` depends upon `fish` (for the evolution), `fish` depends upon `animal` (for `do-move`), and `animal` depends upon `water`. That’s a dependency cycle, and Clojure _hates_ dependency cycles. See [Figure 17.3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fig03).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c17f003.jpg)

**Figure 17.3.** A dependency cycle

OK, take a deep breath. Remember, we’re playing a game here. In a simple application like Wa-Tor, I would not be partitioning these files so ruthlessly. In fact, there’s a good chance I’d just write the whole program in a single file and let the devil have his way with me. But we are pretending that this is a multi-mega-line enterprise application, and so we’re going to be assiduously careful with all these source code dependencies. Right?

So the way we have to solve this is by falling back on something like the old C mechanism of declarations and implementations. See [Figure 17.4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fig04).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c17f004.jpg)

**Figure 17.4.** Breaking the dependency cycle

By splitting `water` such that its `fish` dependency is in `water-imp`, and by making sure that `water-imp` depends upon `water` instead of the other way around (the DIP), the cycle is broken. I also split up `fish` and `shark`[12](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn12a) for consistency. I’ll probably have to split up `animal` pretty soon too.[13](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn13a)

[12](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn12). Actually, just `fish`. I split `shark` on the diagram but not in the code. YAGNI, YAGNI, YAGNI.

[13](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn13). Future Uncle Bob: . . .nope.

So now the code looks like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f301-01)

```
(ns wator.world
  (:require [wator
             [water :as water]]))

(defn make [w h]
  (let [locs (for [x (range w) y (range h)] [x y])
        loc-water (interleave locs (repeat (water/make)))
        cells (apply hash-map loc-water)]
    {::cells cells
     ::bounds [w h]}))

(defn set-cell [world loc cell]
  (assoc-in world [::cells loc] cell))

(defn get-cell [world loc]
  (get-in world [::cells loc]))

; . . .

—————

(ns wator.cell)

(defmulti tick ::type)

—————

(ns wator.water
  (:require [wator
             [cell :as cell]]))

(defn make [] {::cell/type ::water})

(defn is? [cell]
  (= ::water (::cell/type cell)))

——————————

(ns wator.water-imp
  (:require [wator
             [cell :as cell]
             [water :as water]
             [fish :as fish]
             [config :as config]]))

(defmethod cell/tick ::water/water [water]
  (if (> (rand) config/water-evolution-rate)
    (fish/make)
    water))

———————

(ns wator.animal
  (:require [wator
             [world :as world]
             [cell :as cell]
             [water :as water]]))

(defmulti move (fn [animal & args] (::cell/type animal)))
(defmulti reproduce (fn [animal & args] (::cell/type animal)))

(defn tick [animal]
  )

(defn do-move [animal loc world]
  (let [neighbors (world/neighbors world loc)
        destinations (filter #(water/is?
                               (world/get-cell world %))
                             neighbors)
        new-location (rand-nth destinations)]
    [new-location animal]))

————

(ns wator.fish
  (:require [wator
             [cell :as cell]]))
(defn make [] {::cell/type ::fish})

——————

(ns wator.fish-imp
  (:require [wator
             [cell :as cell]
             [animal :as animal]
             [fish :as fish]]))

(defmethod cell/tick ::fish/fish [fish]
  (animal/tick fish)
  )

(defmethod animal/move ::fish/fish [fish loc world]
  (animal/do-move fish loc world))

(defmethod animal/reproduce ::fish/fish [fish]
  )
```

The `shark` isn’t relevant yet, so I didn’t show it.

The criterion for splitting `water` and `fish` is pretty easy to see. Any function that references a file outside of the direct type hierarchy gets put into the `imp` file. Pay special attention to the namespaces and the namespaced keywords. For example, notice that the `defmethod`s in `fish-imp` will still be dispatched on `::fish/fish`.

And just in case you thought I’d forgotten, here are the current tests:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f303-02)

```
(ns wator.core-spec
  (:require [speclj.core :refer :all]
            [wator
             [cell :as cell]
             [water :as water]
             [water-imp]
             [animal :as animal]
             [fish :as fish]
             [fish-imp]
             [world :as world]]))
(describe "Wator"
  (with-stubs)
  (context "Water"
    (it "usually remains water"
      (with-redefs [rand (stub :rand {:return 0.0})]
        (let [water (water/make)
              evolved (cell/tick water)]
          (should= ::water/water (::cell/type evolved)))))

    (it "occasionally evolves into a fish"
      (with-redefs [rand (stub :rand {:return 1.0})]
        (let [water (water/make)
              evolved (cell/tick water)]
          (should= ::fish/fish (::cell/type evolved))))))

  (context "world"
    (it "creates a world full of water cells"
      (let [world (world/make 2 2)
            cells (::world/cells world)
            positions (set (keys cells))]
        (should= #{[0 0] [0 1]
                   [1 0] [1 1]} positions)
        (should (every? #(= ::water/water (::cell/type %))
                        (vals cells)))))

    (it "makes neighbors"
      (let [world (world/make 5 5)]
        (should= [[0 0] [0 1] [0 2]
                  [1 0] [1 2]
                  [2 0] [2 1] [2 2]]
                 (world/neighbors world [1 1]))
        (should= [[4 4] [4 0] [4 1]
                  [0 4] [0 1]
                  [1 4] [1 0] [1 1]]
                 (world/neighbors world [0 0]))
        (should= [[3 3] [3 4] [3 0]
                  [4 3] [4 0]
                  [0 3] [0 4] [0 0]]
                 (world/neighbors world [4 4])))))

  (context "animal"
    (it "moves"
      (let [fish (fish/make)
            world (-> (world/make 3 3)
                      (world/set-cell [1 1] fish))
            [loc cell] (animal/move fish [1 1] world)]
        (should= cell fish)
        (should (#{[0 0] [0 1] [0 2]
                   [1 0] [1 2]
                   [2 0] [2 1] [2 2]}
                 loc))))))
```

Look at the `:require` up in the `ns` statement. Notice that we are requiring the `imp`s but not explicitly using them. Requiring them registers the `defmethod`s that they contain.

OK, now that we can move the `fish`, I’m pretty sure the `shark`s will move too. So next we should try some reproduction. But before we do that, I’m getting (pretend) concerned about the type system for the `world`. Let’s get that set up first:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f305-01)

```
(ns wator.world
  (:require [clojure.spec.alpha :as s]
            [wator
             [cell :as cell]
             [water :as water]]))

(s/def ::location (s/tuple int? int?))
(s/def ::cell #(contains? % ::cell/type))
(s/def ::cells (s/map-of ::location ::cell))
(s/def ::bounds ::location)
(s/def ::world (s/keys :req [::cells ::bounds]))

(defn make [w h]
  {:post [(s/valid? ::world %)]}
  …)
```

OK, that’s better. Now, what do we need for reproduction? The modelers said that a `fish` will reproduce if it is next to a `water` cell and is above a certain age. The two daughter `fish` have their ages reset to zero. Otherwise, the `::age` of a `fish` increases with time.

Here are the tests:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f306-01)

```
(it "reproduces"
  (let [fish (-> (fish/make)
                 (animal/set-age config/fish-reproduction-age))
        world (-> (world/make 3 3)
                  (world/set-cell [1 1] fish))
        [loc1 cell1 loc2 cell2] (animal/reproduce
                                  fish [1 1] world)]
    (should= loc1 [1 1])
    (should (fish/is? cell1))
    (should= 0 (animal/age cell1))
    (should (#{[0 0] [0 1] [0 2]
               [1 0] [1 2]
               [2 0] [2 1] [2 2]}
             loc2))
    (should (fish/is? cell2))
    (should= 0 (animal/age cell2))))

(it "doesn't reproduce if there is no room"
  (let [fish (-> (fish/make)
                 (animal/set-age config/fish-reproduction-age))
        world (-> (world/make 1 1)
                  (world/set-cell [0 0] fish))
        failed (animal/reproduce fish [0 0] world)]
    (should-be-nil failed)))

(it "doesn't reproduce if too young"
      (let [fish (-> (fish/make)
                     (animal/set-age
                       (dec config/fish-reproduction-age)))
            world (-> (world/make 3 3)
                      (world/set-cell [1 1] fish))
            failed (animal/reproduce fish [1 1] world)]
        (should-be-nil failed)))
```

Notice that if the `fish` reproduces, the return value contains both daughters. But if something goes wrong, we return `nil`. This is because I reckon that the high-level policy of a `fish` includes something like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f307-01)

```
(if-let [result (animal/reproduce …)]
  result
  (animal/move …))
```

Anyway, here’s the abbreviated code that passes that test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f307-02)

```
(ns wator.animal
  (:require [clojure.spec.alpha :as s]
            [wator
             [world :as world]
             [cell :as cell]
             [water :as water]
             [config :as config]]))

(s/def ::age int?)
(s/def ::animal (s/keys :req [::age]))

(defmulti move (fn [animal & args] (::cell/type animal)))
(defmulti reproduce (fn [animal & args] (::cell/type animal)))
(defmulti make-child ::cell/type)

(defn make []
  {::age 0})

(defn age [animal]
  (::age animal))

(defn set-age [animal age]
  (assoc animal ::age age))

;. . .

(defn do-reproduce [animal loc world]
  (if (>= (age animal) config/fish-reproduction-age)
    (let [neighbors (world/neighbors world loc)
          birth-places (filter #(water/is? (world/get-cell world %))
                               neighbors)]
      (if (empty? birth-places)
        nil
        [loc (set-age animal 0)
         (rand-nth birth-places) (make-child animal)]))
    nil))

————

(ns wator.fish
  (:require [clojure.spec.alpha :as s]
            [wator
             [cell :as cell]
             [animal :as animal]]))

(s/def ::fish (s/and #(= ::fish (::cell/type %))
                     ::animal/animal))
(defn is? [cell]
  (= ::fish (::cell/type cell)))

(defn make []
  {:post [(s/valid? ::fish %)]}
  (merge {::cell/type ::fish}
         (animal/make)))

(defmethod animal/make-child ::fish [fish]
  (make))

——————

(ns wator.fish-imp
  (:require [wator
             [cell :as cell]
             [animal :as animal]
             [fish :as fish]]))

;. . .

(defmethod animal/reproduce ::fish/fish [fish loc world]
  (animal/do-reproduce fish loc world))
```

Again, notice that I am deferring the `fish/reproduce` function to `animal/do-reproduce`. This allows me to specify the common behavior of `reproduce` in `animal` while allowing `fish` to override or augment it. I don’t know if this will be necessary,[14](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn14a) but it’s pretty cheap to add and it eliminates the duplication in `shark` and `fish`.

[14](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn14). Yeah, I know, YAGNI and all that. But rules are meant to be broken.

### Scratch That Itch

I’m getting an itchy feeling that I should have implemented `world/tick` first. I’ve made a lot of decisions about the return values of `move` and `reproduce` based upon what I think `world/tick` is going to need. So let’s switch gears and focus on that before we continue to add more, possibly errant, goop to the `animal`s.

Here’s the first test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f309-01)

```
(it "moves a fish around each tick"
  (let [fish (fish/make)
        small-world (-> (world/make 1 2)
                        (world/set-cell [0 0] fish)
                        (world/tick))
        vacated-cell (world/get-cell small-world [0 0])
        occupied-cell (world/get-cell small-world [0 1])]
    (should (water/is? vacated-cell))
    (should (fish/is? occupied-cell))
    (should= 1 (animal/age occupied-cell))))
```

It’s pretty simple. We make a `small-world` with two cells, one of which is a `fish`. We call `tick` on that `world`, and then we make sure that the `fish` moves to the vacant cell and that it leaves `water` behind.

Next, I wrote a dummy implementation for `tick`, just to see the test pass:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f310-01)

```
(defn tick [world]
  (-> (make 2 1)
        (set-cell [0 0] (water/make))
        (set-cell [0 1] (animal/set-age (fish/make) 1))))
```

Lo and behold, this won’t compile because `world` now depends upon `fish`, which depends upon `animal`, which depends back upon `world`. Sigh. Cyclic dependencies are the bane of source code structures that are thought through poorly.

But we know how to solve this. We simply have to invert a dependency (the DIP) by splitting `world-imp` out of `world`. The UML looks like [Figure 17.5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fig05).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c17f005.jpg)

**Figure 17.5.** Breaking another dependency cycle

The `=0` next to `tick` in the `World` class is my way of indicating that it is an abstract method. So here’s the code:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f311-01)

```
(ns wator.world
  (:require [clojure.spec.alpha :as s]
            [wator
             [cell :as cell]
             [water :as water]]))

(s/def ::location (s/tuple int? int?))
(s/def ::cell #(contains? % ::cell/type))
(s/def ::cells (s/map-of ::location ::cell))
(s/def ::bounds ::location)
(s/def ::world (s/and (s/keys :req [::cells ::bounds])
                      #(= (::type %) ::world)))

(defmulti tick ::type)

(defn make [w h]
  {:post [(s/valid? ::world %)]}
  (let [locs (for [x (range w) y (range h)] [x y])
        loc-water (interleave locs (repeat (water/make)))
        cells (apply hash-map loc-water)]
    {::type ::world
     ::cells cells
     ::bounds [w h]}))

; . . .
—————
(ns wator.world-imp
  (:require [wator
             [world :as world :refer :all]
             [animal :as animal]
             [fish :as fish]
             [water :as water]]))
(defmethod world/tick ::world/world [world]
  (-> (make 2 1)
        (set-cell [0 0] (water/make))
        (set-cell [0 1] (animal/set-age (fish/make) 1))))
```

This passed the test once I added `[world-imp]` to the `:require` list in the test. Take note that `tick` is now a multi-method with only one implementation. That’s the dependency inversion that we needed.

But now I’m bothered by that `water` dependency in `world`. There’s a technical term for how I feel about it. That term is _icky_. That dependency is _wrong_ somehow.

I need a shower. I resolve lots of issues while in the shower.

### Showers Solve Problems

OK, I’m back from my shower, and this is the conversation I had with myself while under the spray.

“Creating `water` in `world` is icky. I mean, I just split `world` in two because creating a `fish` led to a cycle. So creating `water` could lead to a cycle too. But wait, this is all about creation. Maybe what I need is a factory! Yeah, an Abstract Factory named `cell-factory`, and it will take opaque tokens like `:fish` and `:water`, and. . . (OH!). . . and `:default-cell`. Yeah, and. . . Wait, why do I need a whole new factory? Why can’t `world` BE the factory? Yeah! That’s the _Factory Method_ pattern. That’s the ticket!”

The UML for this (in [Figure 17.6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fig06)) is revealing.

An architectural boundary just appeared. All dependencies cross it going toward the high-level side, following the Dependency Rule. I may not use this boundary in the actual architecture, but it’s there if I need it.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c17f006.jpg)

**Figure 17.6.** Wa-Tor with the Factory Method pattern

So now the code looks like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f313-01)

```
(ns wator.world
  (:require [clojure.spec.alpha :as s]
            [wator
             [cell :as cell]
             [water :as water]]))

(s/def ::location (s/tuple int? int?))
(s/def ::cell #(contains? % ::cell/type))
(s/def ::cells (s/map-of ::location ::cell))
(s/def ::bounds ::location)
(s/def ::world (s/and (s/keys :req [::cells ::bounds])
                      #(= (::type %) ::world)))

(defmulti tick ::type)
(defmulti make-cell (fn [factory-type cell-type] factory-type))
(defn make [w h]
  {:post [(s/valid? ::world %)]}
  (let [locs (for [x (range w) y (range h)] [x y])
        default-cell (make-cell ::world :default-cell)
        loc-water (interleave locs (repeat default-cell))
        cells (apply hash-map loc-water)]
    {::type ::world
     ::cells cells
     ::bounds [w h]}))
;. . .

———————

(ns wator.world-imp
  (:require [wator
             [world :as world :refer :all]
             [animal :as animal]
             [fish :as fish]
                    [shark :as shark]
             [water :as water]]))

(defmethod world/tick ::world/world [world]
  (-> (make 2 1)
        (set-cell [0 0] (water/make))
        (set-cell [0 1] (animal/set-age (fish/make) 1))))

(defmethod world/make-cell ::world/world [world cell-type]
  (condp = cell-type
    :default-cell (water/make)
    :water (water/make)
    :fish (fish/make)
    :shark (shark/make)))
```

The `factory-type` in `make-cell` is simply passed in as `::world`. That allows the `defmethod ::world/world` to resolve it.

I have high hopes for this change. And please note, this whole change was driven by one test that I made to pass using a dummy implementation in `tick`, reminding us yet again that TDD is a design technique.

OK, now let’s make that dummy implementation fail. Here’s the test that fails:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f315-01)

```
(it "moves a fish around each tick"
  (doseq [scenario
          [{:dimension [2 1] :starting [0 0] :ending [1 0]}
           {:dimension [2 1] :starting [1 0] :ending [0 0]}
           {:dimension [1 2] :starting [0 0] :ending [0 1]}
           {:dimension [1 2] :starting [0 1] :ending [0 0]}]]
    (let [fish (fish/make)
          {:keys [dimension starting ending]} scenario
          [h w] dimension
          small-world (-> (world/make h w)
                          (world/set-cell starting fish)
                          (world/tick))
          vacated-cell (world/get-cell small-world starting)
          occupied-cell (world/get-cell small-world ending)]
      (should (water/is? vacated-cell))
      (should (fish/is? occupied-cell))
      (should= 1 (animal/age occupied-cell)))))
```

I created the four possible 1-by-2 scenarios and made sure the `world` got updated properly after a `tick`.

Making this pass forced me to change the design yet again. The `animal/move`, `animal/reproduce`, and `cell/tick` functions must return a `[from to]` list in which each is a single-element map containing `{loc cell}`. Look at the `world-imp` and you’ll see why:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f315-02)

```
(ns wator.world-imp
  . . .)

(defmethod world/tick ::world/world [world]
  (let [cells (::world/cells world)]
    (loop [locs (keys cells)
           new-cells {}
           moved-into #{}]
      (cond

        (empty? locs)
        (assoc world ::world/cells new-cells)

        (contains? moved-into (first locs))
        (recur (rest locs) new-cells moved-into)

        :else
        (let [loc (first locs)
              cell (get cells loc)
              [from to] (cell/tick cell loc world)
              new-cells (-> new-cells (merge from) (merge to))
              to-loc (first (keys to))]
          (recur (rest locs)
                 new-cells
                 (conj moved-into to-loc)))))))

; . . .
```

It turns out that every operation makes changes to either one or two cells. When an `animal` moves, reproduces, or eats, only two cells are involved. If an `animal` fails to move, or if it starves, only one cell is involved. In the first case the operation will return `[from to]`, and in the second case it will return `[nil to]`. In either case, both `from` and `to` are `merge`d[15](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn15a) into `new-cells`.

[15](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn15). `merge` is well behaved if you merge in a nil.

Notice the `moved-into` argument of the loop. At first, I didn’t have it there, and the tests failed because `world/tick` moved the `fish` to the remaining `water` cell. But then `world/tick` called `cell/tick` on the `water` cell, which replaced itself with `water`. When the `new-cells` were merged in, the `water` overwrote the `fish`.

So `moved-into` is a set of all the `to` cell locations. The `cell/tick` function should not be called on them because they’ve been moved into by a previous `tick`, and so the `animal` there has already been `tick`ed.

Quite a few changes had to be made throughout the structure to get this to work. So my “itch” from a few pages back was correct. It’s a good thing I paid attention to it early enough to make the change doable:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f317-01)

```
(ns wator.cell)

(defmulti tick (fn [cell & args] (::type cell)))

——————

(ns wator.water-imp
  (:require [wator
             [cell :as cell]
             [water :as water]
             [fish :as fish]
             [config :as config]]))

(defmethod cell/tick ::water/water [water loc world]
  (if (> (rand) config/water-evolution-rate)
    [nil {loc (fish/make)}]
    [nil {loc water}]))

————

(ns wator.animal . . .)

;. . .

(defn increment-age [animal]
  (update animal ::age inc))

(defn tick [animal loc world]
  (-> animal
      increment-age
      (move loc world)))

(defn do-move [animal loc world]
  (let [neighbors (world/neighbors world loc)
        destinations (filter #(water/is?
                               (world/get-cell world %))
                             neighbors)
        new-location (if (empty? destinations)
                       loc
                       (rand-nth destinations))]
    (if (= new-location loc)
      [nil {loc animal}]
      [{loc (water/make)} {new-location animal}])))

;. . .

————

(ns wator.fish-imp . . .)

(defmethod cell/tick ::fish/fish [fish loc world]
  (animal/tick fish loc world)
  )

; . . .
```

And, of course, a few of the tests needed to change:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f318-02)

```
(ns wator.core-spec . . .)

(describe "Wator"
  (with-stubs)
  (context "Water"
    (it "usually remains water"
      (with-redefs [rand (stub :rand {:return 0.0})]
        (let [water (water/make)
              world (world/make 1 1)
              [from to] (cell/tick water [0 0] world)]
          (should-be-nil from)
          (should (water/is? (get to [0 0])))
          )))
    (it "occasionally evolves into a fish"
      (with-redefs [rand (stub :rand {:return 1.0})]
        (let [water (water/make)
              world (world/make 1 1)
              [from to] (cell/tick water [0 0] world)]
          (should-be-nil from)
          (should (fish/is? (get to [0 0])))))))

;. . .

  (context "animal"
    (it "moves"
      (let [fish (fish/make)
            world (-> (world/make 3 3)
                      (world/set-cell [1 1] fish))
            [from to] (animal/move fish [1 1] world)
            loc (first (keys to))]
        (should (water/is? (get from [1 1])))
        (should (fish/is? (get to loc)))
        (should (#{[0 0] [0 1] [0 2]
                   [1 0] [1 2]
                   [2 0] [2 1] [2 2]}
                 loc))))

    (it "doesn't move if there are no spaces"
      (let [fish (fish/make)
            world (-> (world/make 1 1)
                      (world/set-cell [0 0] fish))
            [from to] (animal/move fish [0 0] world)]
        (should (fish/is? (get to [0 0])))
        (should (nil? from)))
```

There’s another scenario that I think will fail—two `fish` competing for the same spot:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f319-01)

```
(it "move two fish who compete for the same spot"
  (let [fish (fish/make)
        competitive-world (-> (world/make 3 1)
                              (world/set-cell [0 0] fish)
                              (world/set-cell [2 0] fish)
                              (world/tick))
        start-00 (world/get-cell competitive-world [0 0])
        start-20 (world/get-cell competitive-world [2 0])
        end-10 (world/get-cell competitive-world [1 0])]
    (should (fish/is? end-10))
    (should (or (fish/is? start-00)
                (fish/is? start-20)))
    (should (or (water/is? start-00)
                (water/is? start-20)))))
```

A simple 3-by-1 `world` with `fish` at either end. Only one of them can move into the center slot. The other will have to remain where it was. This test fails because the `animal/move` function does not know that a `fish` already moved into the target slot.

Solving this means somehow sending the `moved-into` list to `animal/move`. I hate the idea of adding yet another argument to `animal/move`, so perhaps we can squirrel this information away in the `world` that we pass to `animal/move`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f320-01)

```
(ns wator.world-imp . . .)

(defmethod world/tick ::world/world [world]
  (let [cells (::world/cells world)]
    (loop [locs (keys cells)
           new-cells {}
           moved-into #{}]
      (cond
        (empty? locs)
        (assoc world ::world/cells new-cells)

        (contains? moved-into (first locs))
        (recur (rest locs) new-cells moved-into)

        :else
        (let [loc (first locs)
              cell (get cells loc)
              [from to] (cell/tick
                          cell loc
                          (assoc world :moved-into moved-into))
              new-cells (-> new-cells (merge from) (merge to))
              to-loc (first (keys to))
              to-cell (get to to-loc)
              moved-into (if (water/is? to-cell)
                            moved-into
                            (conj moved-into to-loc))]
          (recur (rest locs) new-cells moved-into))))))

———————

(ns wator.animal . . .)

; . . .

(defn do-move [animal loc world]
  (let [neighbors (world/neighbors world loc)
        moved-into (get world :moved-into #{})
        available-neighbors (remove moved-into neighbors)
        destinations (filter #(water/is?
                               (world/get-cell world %))
                             available-neighbors)
        new-location (if (empty? destinations)
                       loc
                       (rand-nth destinations))]
    (if (= new-location loc)
      [nil {loc animal}]
      [{loc (water/make)} {new-location animal}])))
```

Note that I did not use a namespaced keyword for `:moved-into`. That’s because I consider it to be tramp data that is not really part of the `world` and is just kind of hitching a ride. This feels a little dirty, but it works.[16](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn16a)

[16](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn16). Welcome to real-world engineering trade-offs.

Note that we only put locations into `moved-into` if the `cell` being moved in is not `water`.

### It’s Time to Wildly Reproduce[17](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn17a)

[17](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn17). Ugliness breeds ugliness.

OK, let’s see if we can fill the world with fish:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f322-01)

```
(it "fills the world with reproducing fish"
  (loop [world (-> (world/make 10 10)
                   (world/set-cell [5 5] (fish/make)))
         n 100]
    (if (zero? n)
      (let [cells (-> world ::world/cells vals)
            fishies (filter fish/is? cells)
            fish-count (count fishies)]
        (should (< 50 fish-count)))
      (recur (world/tick world) (dec n)))))
```

Nifty. Create a 10-by-10 `world`. Load it with one `fish`. Send it 100 `tick`s, and make sure there are more than 50 `fish`. I mean, the fish are moving around and reproducing like crazy in there!

Of course, this test fails; but only because we didn’t call `reproduce` in `animal/tick`. So let’s fix that:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f322-02)

```
(defn tick [animal loc world]
  (let [aged-animal (increment-age animal)
        reproduction (reproduce aged-animal loc world)]
    (if reproduction
      reproduction
      (move aged-animal loc world))))
```

Yup. Age the animal, then see if it will reproduce. If not, then move it. Simple. Easy.

Of course, I had to fix the fact that `reproduce` didn’t use our new `[from to]` convention:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f322-03)

```
(defn do-reproduce [animal loc world]
  (if (>= (age animal) config/fish-reproduction-age)
    (let [neighbors (world/neighbors world loc)
          birth-places (filter #(water/is?
                                 (world/get-cell world %))
                               neighbors)]
      (if (empty? birth-places)
        nil
        [{loc (set-age animal 0)}
         {(rand-nth birth-places) (make-child animal)}]))
    nil))
```

And that broke an earlier test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f323-01)

```
(it "reproduces"
  (let [fish (-> (fish/make)
                 (animal/set-age config/fish-reproduction-age))
        world (-> (world/make 3 3)
                  (world/set-cell [1 1] fish))
        [from to] (animal/reproduce fish [1 1] world)
        from-loc (-> from keys first)
        from-cell (-> from vals first)
        to-loc (-> to keys first)
        to-cell (-> to vals first)]
    (should= from-loc [1 1])
    (should (fish/is? from-cell))
    (should= 0 (animal/age from-cell))
    (should (#{[0 0] [0 1] [0 2]
               [1 0] [1 2]
               [2 0] [2 1] [2 2]}
             to-loc))
    (should (fish/is? to-cell))
    (should= 0 (animal/age to-cell))))
```

But with that, the `fish` reproduce like. . . fish. That was pretty easy. I think our design is coming together.

### What about the Sharks?

I’ve neglected the `shark` class so far because its behavior is almost identical to `fish` and is mostly governed by the `animal` abstraction. But now let’s see if we can get `shark` objects to `move` and `reproduce`.

This required me to flesh out the `shark` module and also make one small design change. I used the _Template Method_ pattern to get the reproduction age of an animal. The tests hint at that change:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f324-01)

```
(context "animal"
  (it "moves"
    (doseq [scenario 
             [{:constructor fish/make :tester fish/is?}
              {:constructor shark/make :tester shark/is?}]]
      (let [animal ((:constructor scenario))
            world (-> (world/make 3 3)
                      (world/set-cell [1 1] animal))
            [from to] (animal/move animal [1 1] world)
            loc (first (keys to))]
        (should (water/is? (get from [1 1])))
        (should ((:tester scenario) (get to loc)))
        (should (#{[0 0] [0 1] [0 2]
                   [1 0] [1 2]
                   [2 0] [2 1] [2 2]}
                 loc)))))

  (it "doesn't move if there are no spaces"
    (doseq [scenario 
             [{:constructor fish/make :tester fish/is?}
              {:constructor shark/make :tester shark/is?}]]
      (let [animal ((:constructor scenario))
            world (-> (world/make 1 1)
                      (world/set-cell [0 0] animal))
            [from to] (animal/move animal [0 0] world)]
        (should ((:tester scenario) (get to [0 0])))
        (should (nil? from)))))
  (it "reproduces"
    (doseq [scenario 
             [{:constructor fish/make :tester fish/is?}
              {:constructor shark/make :tester shark/is?}]]
      (let [animal ((:constructor scenario))
            reproduction-age (animal/get-reproduction-age animal)
            animal (animal/set-age animal reproduction-age)
            world (-> (world/make 3 3)
                      (world/set-cell [1 1] animal))
            [from to] (animal/reproduce animal [1 1] world)
            from-loc (-> from keys first)
            from-cell (-> from vals first)
            to-loc (-> to keys first)
            to-cell (-> to vals first)]
        (should= from-loc [1 1])
        (should ((:tester scenario) from-cell))
        (should= 0 (animal/age from-cell))
        (should (#{[0 0] [0 1] [0 2]
                   [1 0] [1 2]
                   [2 0] [2 1] [2 2]}
                 to-loc))
        (should ((:tester scenario) to-cell))
        (should= 0 (animal/age to-cell)))))

  (it "doesn't reproduce if there is no room"
    (doseq [scenario 
             [{:constructor fish/make :tester fish/is?}
              {:constructor shark/make :tester shark/is?}]]
      (let [animal ((:constructor scenario))
            reproduction-age (animal/get-reproduction-age animal)
            animal (animal/set-age animal reproduction-age)
            world (-> (world/make 1 1)
                      (world/set-cell [0 0] animal))
            failed (animal/reproduce animal [0 0] world)]
        (should-be-nil failed))))

  (it "doesn't reproduce if too young"
    (doseq [scenario 
             [{:constructor fish/make :tester fish/is?}
              {:constructor shark/make :tester shark/is?}]]
      (let [animal ((:constructor scenario))
            reproduction-age (animal/get-reproduction-age animal)
            animal (animal/set-age animal (dec reproduction-age))
            world (-> (world/make 3 3)
                      (world/set-cell [1 1] animal))
            failed (animal/reproduce animal [1 1] world)]
        (should-be-nil failed)))))

————————

(ns wator.animal …)

(defmulti move (fn [animal & args] (::cell/type animal)))
(defmulti reproduce (fn [animal & args] (::cell/type animal)))
(defmulti make-child ::cell/type)
(defmulti get-reproduction-age ::cell/type)

; . . .

————————

(ns wator.fish . . .)

(defmethod animal/get-reproduction-age ::fish [fish]
  config/fish-reproduction-age)

; . . .

——————

(ns wator.shark
  (:require [clojure.spec.alpha :as s]
            [wator
             [config :as config]
             [cell :as cell]
             [animal :as animal]]))
(s/def ::shark (s/and #(= ::shark (::cell/type %))
                      ::animal/animal))
(defn is? [cell]
  (= ::shark (::cell/type cell)))

(defn make []
  {:post [(s/valid? ::shark %)]}
  (merge {::cell/type ::shark}
         (animal/make)))

(defmethod animal/make-child ::shark [fish]
  (make))

(defmethod animal/get-reproduction-age ::shark [shark]
  config/shark-reproduction-age)

; . . .
```

So far, with the exception of the reproduction age, the behavior of both the `shark` and `fish` is “inherited” from (actually it is delegated to) `animal`. But the `shark` class has extra constraints that we need to implement now.

The modelers have told us that a `shark` only reproduces if its `:health` is above a certain threshold. The `:health` of a `shark` is increased by eating a `fish`, and it decreases with time. If the `:health` of a `shark` reaches zero, the `shark` starves, leaving behind `water`. When a `shark` reproduces, its `:health` is split between the two daughters.

OK, so let’s test that the `:health` decreases with age:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f327-02)

```
(context "shark"
  (it "starts with some health"
    (let [shark (shark/make)]
      (should= config/shark-starting-health
               (shark/health shark))))

  (it "loses health with time"
    (let [small-world (-> (world/make 1 1)
                          (world/set-cell [0 0] (shark/make)))
          aged-world (world/tick small-world)
          aged-shark (world/get-cell aged-world [0 0])]
      (should= (dec config/shark-starting-health)
               (shark/health aged-shark)))))

—————

(ns wator.shark . . .)

(s/def ::health int?)
(s/def ::shark (s/and #(= ::shark (::cell/type %))
                      ::animal/animal
                      (s/keys :req [::health])))

(defn make []
  {:post [(s/valid? ::shark %)]}
  (merge {::cell/type ::shark
          ::health config/shark-starting-health}
         (animal/make)))

(defn health [shark]
  (::health shark))

(defn decrement-health [shark]
  (update shark ::health dec))

(defmethod cell/tick ::shark [shark loc world]
  (-> shark
      (decrement-health)
      (animal/tick loc world))
  )

; . . .
```

Pretty easy. We just added the `::health` field to the `::shark` spec and `shark/make`, and then we decremented the `::health` in the `tick` function just before delegating the rest of the behavior to the superclass `animal`.

Now let’s test that a `shark` will die when its `::health` goes to zero:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f329-01)

```
(it "dies when health goes to zero"
      (let [sick-shark (-> (shark/make)
                           (shark/set-health 1))
            small-world (-> (world/make 1 1)
                            (world/set-cell [0 0] sick-shark))
            aged-world (world/tick small-world)
            dead-shark (world/get-cell aged-world [0 0])]
        (should (water/is? dead-shark))))

————

(ns wator.shark . . .)

(defmethod cell/tick ::shark [shark loc world]
  (if (= 1 (health shark))
    [nil {loc (water/make)}]
    (-> shark
        (decrement-health)
        (animal/tick loc world))))

; . . .
```

Pretty easy. OK, so now let’s test that sharks will eat when given the opportunity:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f329-02)

```
(it "eats when a fish is adjacent"
  (let [world (-> (world/make 2 1)
                  (world/set-cell [0 0] (fish/make))
                  (world/set-cell [1 0] (shark/make)))
        shark-ate-world (world/tick world)
        full-shark (world/get-cell shark-ate-world [0 0])
        where-shark-was (world/get-cell shark-ate-world [1 0])
        expected-health (+ config/shark-starting-health
                           config/shark-eating-health
                           -1)]
    (should (shark/is? full-shark))
    (should (water/is? where-shark-was))
    (should= expected-health (shark/health full-shark))))
```

We create a 2-by-1 `world` with a `shark` next to a `fish`. After one `tick`, the `shark` should be where the `fish` was, and `water` should be where the `shark` was, and the `shark`’s `::health` should have increased.

Getting this to pass forced me to abandon the delegation to `animal/tick` because a `shark` should try to `reproduce` first, then try to `eat` next, and then finally try to `move`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f330-01)

```
(ns wator.shark . . .)

(defn eat [shark loc world]
  (let [neighbors (world/neighbors world loc)
        fishy-neighbors (filter #(fish/is?
                                  (world/get-cell world %))
                                neighbors)]
    (if (empty? fishy-neighbors)
      nil
      [{loc (water/make)}
       {(rand-nth fishy-neighbors) (feed shark)}]))
  )

(defmethod cell/tick ::shark [shark loc world]
  (if (= 1 (health shark))
    [nil {loc (water/make)}]
    (let [aged-shark (-> shark
                         (animal/increment-age)
                         (decrement-health))]
      (if-let [reproduction (animal/reproduce
                              aged-shark loc world)]
        reproduction
        (if-let [eaten (eat aged-shark loc world)]
          eaten
          (animal/move aged-shark loc world))))))
```

All this slipped in with little hassle. We’ve passed through the design bottleneck and are now reaping the benefits.

The modelers told us that a shark will only reproduce if its health is above a threshold. Let’s test that. In fact, let’s make that change first[18](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn18a) and see which tests break:

[18](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn18). TDD VIOLATION! ALERT! ALERT!

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f331-01)

```
(ns wator.shark . . .)

(defmethod animal/reproduce ::shark [shark loc world]
  (if (>= (health shark) config/shark-reproduction-health)
    (animal/do-reproduce shark loc world)
    nil))
```

As expected, the test for animal reproduction fails in the shark scenario. We can address this by putting a little hack in that test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f331-02)

```
(it "reproduces"
  (doseq [scenario [{:constructor fish/make :tester fish/is?}
                    {:constructor
                       #(-> (shark/make)
                            (shark/set-health
                              (inc config/shark-reproduction-
                                   health)))
                     :tester shark/is?}]]

; . . .
```

Yes, that’s a bit ugly, but it does the job. I suppose I should add a test for checking the other side of that threshold:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f331-03)

```
(it "doesn't reproduce if not healthy enough"
  (let [shark (-> (shark/make)
                  (shark/set-health
                    (dec config/shark-reproduction-health))
                  (animal/set-age config/shark-reproduction-age))
        world (-> (world/make 3 3)
                  (world/set-cell [1 1] shark))
        failed (animal/reproduce shark [1 1] world)]
    (should-be-nil failed)))
```

OK. One last thing. The health of the parent shark is split between the two daughter sharks:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f332-01)

```
(it "shares health with both daughters after reproduction"
  (let [initial-health (inc config/shark-reproduction-health)
        pregnant-shark (-> (shark/make)
                           (animal/set-age
                             (inc config/shark-reproduction-age))
                           (shark/set-health initial-health))
        world (-> (world/make 2 1)
                  (world/set-cell [0 0] pregnant-shark))
        new-world (world/tick world)
        daughter1 (world/get-cell new-world [0 0])
        daughter2 (world/get-cell new-world [1 0])
        expected-health (quot (dec initial-health) 2)]
    (should (shark/is? daughter1))
    (should (shark/is? daughter2))
    (should= expected-health (shark/health daughter1))
    (should= expected-health (shark/health daughter2))))
```

Yup. That fails because the expected health isn’t correct. That should be simple to fix:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f332-02)

```
(ns wator.shark . . .)

(defmethod animal/reproduce ::shark [shark loc world]
  (if (< (health shark) config/shark-reproduction-health)
    nil
    (if-let [reproduction (animal/do-reproduce shark loc world)]
      (let [[from to] reproduction
            from-loc (-> from keys first)
            to-loc (-> to keys first)
            daughter-health (quot (health shark) 2)
            from-shark (-> from vals first
                           (set-health daughter-health))
            to-shark (-> to vals first
                         (set-health daughter-health))]
        [{from-loc from-shark} {to-loc to-shark}])
      nil)))
```

And with that, I think the model is complete. Let’s see if we can put a GUI on top of it:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17_images.xhtml#f333-01)

```
(ns wator-gui.main
  (:require [quil.core :as q]
            [quil.middleware :as m]
            [wator
             [world :as world]
             [water :as water]
             [fish :as fish]
             [shark :as shark]
             [world-imp]
             [water-imp]
             [fish-imp]]))

(defn setup []
  (q/frame-rate 60)
  (q/color-mode :rgb)
  (-> (world/make 80 80)
      (world/set-cell [40 40] (fish/make)))
  )

(defn update-state [world]
  (world/tick world))
(defn draw-state [world]
  (q/background 240)
  (let [cells (::world/cells world)]
    (doseq [loc (keys cells)]
      (let [[x y] loc
            cell (get cells loc)
            x (* 12 x)
            y (* 12 y)
            color (cond
                    (water/is? cell) [255 255 255]
                    (fish/is? cell) [0 0 255]
                    (shark/is? cell) [255 0 0])]
        (q/no-stroke)
        (apply q/fill color)
        (q/rect x y 11 11)))))

(declare wator)

(defn ^:export -main [& args]
  (q/defsketch wator
               :title "Wator"
               :size [960 960]
               :setup setup
               :update update-state
               :draw draw-state
               :features [:keep-on-top]
               :middleware [m/fun-mode])

  args)
```

Yeah, that wasn’t too hard. [Figure 17.7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fig07) is a screenshot of the game in progress.

It’s not super-fast; but that’s not a big surprise. There are a bunch of things we could do to speed it up. But never mind that. Look at that GUI code. It depends on the model, yet the model knows nothing of the GUI. And that satisfies our original architectural goal.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c17f007.jpg)

**Figure 17.7.** Screenshot of Wa-Tor in progress

### Conclusion

Wa-Tor is a program that is “functional”[19](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn19a) and object oriented; complete with several OO design patterns right out of the GOF book. Indeed, it was the OO partitioning that helped the design congeal so nicely.

[19](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17fn19). Why the quotes? Because random numbers aren’t referentially transparent, so this program is not purely functional.

The OO partitioning separates and isolates the various data types very nicely, and it provides pleasant locations for the related functions. Any OO programmer would be very comfortable with this.

However, at its heart, this is a data flow model. The `world` flows through the behaviors in the various objects, without any mutation. The plumbing model of functional programming still holds.

Is this a hybrid approach? Have we created an unholy alliance. . . a Frankenstein’s Monster of a program?

I think not. Indeed, I think this combination of approaches is entirely natural and very beneficial. Data is encapsulated and immutable. Behavior is associated with the data it operates on. And yet the data elements flow through the behaviors as opposed to the behaviors iterating over the data.

In the end, I think this is the way software was meant to be.

By the way, you can find all the source code at [https://github.com/unclebob/wator](https://github.com/unclebob/wator).