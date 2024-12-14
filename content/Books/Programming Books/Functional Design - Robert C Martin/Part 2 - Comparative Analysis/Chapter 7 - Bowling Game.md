---
id: 01JF1NBSXB4A8XTREBX355XXR7
modified: 2024-12-13T23:03:37-05:00
---
## 7

## Bowling Game

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/07_unnum_bowlinggame.jpg)

Now let’s look at another traditional TDD exercise: the Bowling Game kata. What follows is a much-abbreviated version of that kata that appeared in _Clean Craftsmanship_.[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn1a) A related video, _Bowling Game_, is also available. You can access the video by registering at [https://informit.com/functionaldesign](https://informit.com/functionaldesign).

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn1). Robert C. Martin, _Clean Craftsmanship_ (Addison-Wesley, 2021).

### Java Version

We begin, as always, with a test that does nothing, just to prove we can compile and execute:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f066-01)

```
public class BowlingTest {
  @Test
  public void nothing() throws Exception {
  }
}
```

Next, we assert that we can create an instance of the `Game` class:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f066-02)

```
@Test
public void canCreateGame() throws Exception {
  Game g = new Game();
}
```

And then we make that compile and pass by directing the integrated development environment (IDE) to create the missing class:

```
public class Game {
}
```

Next, we see if we can roll one ball:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f066-03)

```
@Test
public void canRoll() throws Exception {
  Game g = new Game();
  g.roll(0);
}
```

And then we make that compile and pass by directing the IDE to create the `roll` function, and we give the argument a reasonable name:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f067-01)

```
public class Game {
  public void roll(int pins) {
  }
}
```

There’s a bit of duplication in the tests already. We should get rid of it. So we factor out the creation of the game into the `setup` function:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f067-02)

```
public class BowlingTest {
  private Game g;

  @Before
  public void setUp() throws Exception {
    g = new Game();
  }
}
```

This makes the first test completely empty. So we delete it. The second test is also pretty useless since it doesn’t assert anything, so we delete it as well.

Next, we want to assert that we can score a game. But to do that we need to roll a complete game:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f067-03)

```
@Test
public void gutterGame() throws Exception {
  for (int i=0; i<20; i++)
    g.roll(0);
  assertEquals(0, g.score());
}

public int score() {
  return 0;
}
```

Next come all ones:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f068-01)

```
@Test
public void allOnes() throws Exception {
  for (int i=0; i<20; i++)
    g.roll(1);
  assertEquals(20, g.score());
}

public class Game {
  private int score;

  public void roll(int pins) {
    score += pins;
  }

  public int score() {
    return score;
  }
}
```

The duplication in the tests can be eliminated by extracting a function called `rollMany`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f068-02)

```
public class BowlingTest {
  private Game g;

  @Before
  public void setUp() throws Exception {
    g = new Game();
  }

  private void rollMany(int n, int pins) {
    for (int i=0; i<n; i++) {
      g.roll(pins);
    }
  }

  @Test
  public void gutterGame() throws Exception {
    rollMany(20, 0);
    assertEquals(0, g.score());
  }

  @Test
  public void allOnes() throws Exception {
    rollMany(20, 1);
    assertEquals(20, g.score());
  }
}
```

OK, next test. One spare, with one extra bonus ball, and all the rest gutter balls:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f069-01)

```
@Test
public void oneSpare() throws Exception {
  rollMany(2, 5);
  g.roll(7);
  rollMany(17, 0);
  assertEquals(24, g.score());
}
```

This test fails, of course. We have to refactor the algorithm in order to get this to pass. We move the computation of the score out of the `roll` method and into the `score` method, and we walk through the `rolls` array two balls (one frame) at a time:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f069-02)

```
public int score() {
  int score = 0;
  int frameIndex = 0;
  for (int frame = 0; frame < 10; frame++) {
    if (isSpare(frameIndex)) {
      score += 10 + rolls[frameIndex + 2];
      frameIndex += 2;
    } else {
      score += rolls[frameIndex] + rolls[frameIndex + 1];
      frameIndex += 2;
    }
  }
  return score;
}

private boolean isSpare(int frameIndex) {
  return rolls[frameIndex] + rolls[frameIndex + 1] == 10;
}
```

One strike is next:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f070-01)

```
@Test
public void oneStrike() throws Exception {
  g.roll(10);
  g.roll(2);
  g.roll(3);
  rollMany(16, 0);
  assertEquals(20, g.score());
}
```

Passing it is just a matter of adding the strike condition, and then we refactor a bit:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f070-02)

```
public int score() {
  int score = 0;
  int frameIndex = 0;
  for (int frame = 0; frame < 10; frame++) {
    if (isStrike(frameIndex)) {
      score += 10 + strikeBonus(frameIndex);
      frameIndex++;
    } else if (isSpare(frameIndex)) {
      score += 10 + spareBonus(frameIndex);
      frameIndex += 2;
    } else {
      score += twoBallsInFrame(frameIndex);
      frameIndex += 2;
    }
  }
  return score;
}
```

Lastly, we test for a perfect game:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#f070-02a)

```
@Test
public void perfectGame() throws Exception {
  rollMany(12, 10);
  assertEquals(300, g.score());
}
```

And this passes without change.

### Clojure Version

Things start out quite differently in Clojure. We have no classes to create, and there is no need for a `roll` method. So our first test is the gutter game:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f071-01)

```
(should= 0 (score (repeat2 20 0)))

(defn score [rolls] 0)
```

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn2). The `repeat` function returns a sequence of repeating values. In this case, it is a sequence of 20 zeros.

Followed quickly by all ones:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f071-02)

```
(should= 20 (score (repeat 20 1)))

(defn score [rolls]
  (reduce + rolls))
```

No surprises here. The `reduce`[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn3a) function simply applies the `+` function across the entire list. So our next test is one spare:

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn3). You will want to look this function up. It does much more than this paragraph suggests. But you’ll see that soon enough.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f072-01)

```
(should= 24 (score (concat [5 5 7] (repeat 17 0)))))
```

To make this pass, we go through several steps. The first is to break the `rolls` array up into frames and sum up the frames. At first, we assume that frames have just two rolls:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f072-02)

```
(defn to-frames [rolls]
  (partition4 2 rolls))

(defn add-frame [score frame]
  (+ score (reduce + frame)))

(defn score [rolls]
  (reduce add-frame 0 (to-frames rolls)))
```

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn4). The `partition` function breaks the `rolls` list into a list of pairs. So `[1 2 3 4 5 6]` becomes `[[1 2][3 4][5 6]]`.

Now the `reduce` function has come into its own. It cycles through the pairs of rolls, accumulating them into a score.

This change keeps all the previous tests passing, but it still fails the spare test. To pass that we have to add special processing to the `to-frames` and `add-frame` functions. Our goal is to put all the rolls needed to calculate a frame into the frame data.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f072-03)

```
(defn to-frames [rolls]
  (let [frames (partition 2 rolls)
        possible-bonuses (map #(take 1 %)5 (rest frames))
        possible-bonuses6 (concat7 possible-bonuses [[0]])]
    (map concat frames possible-bonuses)))

(defn add-frame [score frame-and-bonus]
  (let [frame (take 2 frame-and-bonus)]
    (if (= 10 (reduce + frame))
      (+ score (reduce + frame-and-bonus))
      (+ score (reduce + frame)))))

(defn score [rolls]
  (reduce add-frame 0 (to-frames rolls)))
```

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn5). The `#(…)` form creates an anonymous function. The `%` symbol is the argument to that function. You can also use `%n`, where `n` is an integer representing the _n_th argument. So `#(take 1 %)` is a function that returns a list containing the first element of its argument.

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn6). This is not a reassignment, or even a reinitialization. The second `possible-bonuses` value is distinct from the first. Think of it like a local variable in Java hiding a function argument or a member variable of the same name.

[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn7). The `concat` function concatenates lists together. So `(concat [1 2] [3 4])` returns `[1 2 3 4]`.

Look closely at this code. There are lots of little tricks and workarounds in it. Why? Because Clojure is full of lots of lovely, tempting little tools that you can use to get data into _almost_ the form you want, and then use little tricks to maneuver the data into _exactly_ the form you want. If you aren’t careful, those little tricks can start to dominate the code.

So, for example, see if you can figure out why I am passing `[[0]]` into the `concat` function in `to-frames`.[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn8a) As another example, ask yourself why I used `#(take 1 %)` instead of just `first`.[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn9a)

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn8). Since bonuses are based on the next frame, `possible-bonuses` had one too few elements. That would have terminated the final call to `map` one element too early.

[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn9). `(take 1 x)` returns a list containing the first element in `x`. `first` returns the first element.

Because of the trickiness in this code, don’t be too concerned if you are struggling to understand it. I struggled too when looking back over it. And so…

When these little tricks proliferate it’s time to rethink the solution. So I refactored the solution into a simple `loop`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f074-01)

```
(defn to-frames [rolls]
  (loop [remaining-rolls rolls
         frames []]
    (cond
      (empty? remaining-rolls)
      frames

      (= 10 (reduce + (take 2 remaining-rolls)))
      (recur (drop 2 remaining-rolls)
             (conj frames (take 3 remaining-rolls)))
      :else
      (recur (drop 2 remaining-rolls)
             (conj frames (take 2 remaining-rolls))))))

(defn add-frames [score frame]
  (+ score (reduce + frame)))

(defn score [rolls]
  (reduce add-frames 0 (to-frames rolls)))
```

This is looking a lot better. Moreover, it’s starting to look a bit like the Java solution. The next test is one strike:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f074-02)

```
(should= 20 (score (concat [10 2 3] (repeat 16 0)))))
```

And we make that pass by adding one more case to the `cond`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f074-03)

```
(defn to-frames [rolls]
  (loop [remaining-rolls rolls
         frames []]
    (cond
      (empty? remaining-rolls)
      frames

      (= 10 (first remaining-rolls))
      (recur (rest remaining-rolls)
             (conj frames (take 3 remaining-rolls)))

      (= 10 (reduce + (take 2 remaining-rolls)))
      (recur (drop 2 remaining-rolls)
             (conj frames (take 3 remaining-rolls)))
      :else
      (recur (drop 2 remaining-rolls)
             (conj frames (take 2 remaining-rolls))))))

(defn add-frames [score frame]
  (+ score (reduce + frame)))

(defn score [rolls]
  (reduce add-frames 0 (to-frames rolls)))
```

Trivial, right? So all that’s left is the perfect game. And if this goes like the Java version, this test should just pass without modification:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f075-01)

```
(should= 300 (score (repeat 12 10))))
```

But it doesn’t! Can you see why? Perhaps the fix will elucidate that for you:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07_images.xhtml#f075-02)

```
(defn score [rolls]
  (reduce add-frames 0 (take 10 (to-frames rolls))))
```

The `to-frames` function happily creates more than ten frames. It just runs to the end of the `rolls` list making as many frames as it can. But a game of bowling is only ten frames.

### Conclusion

There are quite a few interesting differences between the Java and Clojure versions of this problem. First, the Clojure version has no `Game` class. So all the machinations we used to create that class in the Java version simply don’t occur in the Clojure version.

You might think that the loss of the `Game` class is a weakness of the Clojure version. After all, it’s convenient to be able to just create a `Game`, toss it a bunch of rolls, and then get the score. However, the Clojure version has decoupled the accumulation of the rolls from the computation of the score. Those concepts are not bound together in the Clojure version. And that makes me think that the Java version has a subtle violation of the Single Responsibility Principle.[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn10a)

[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch07.xhtml#ch07fn10). See Robert C. Martin, _Clean Architecture_ (Pearson, 2017).

Second, as we tried to solve the one spare case, we saw how the Clojure version got polluted with all those nasty little tricks. This is a real problem with Clojure programs (or perhaps Clojure programmers). It’s just too easy to add one more nasty little trick to get things to work.

Third, the Clojure solution is significantly different from the Java solution. Oh, there are some points of similarity, to be sure. That `cond` structure in the Clojure version is very reminiscent of the `if/else` structure in the Java version. However, those two similar structures produced radically different results. The Java version produced the score. The Clojure version produced a frame that included the bonus balls for spares and strikes.

This is an interesting separation of concerns. It is a fact that computing the score forces both versions to identify all the rolls that impact each frame. However, the Java version does this in situ, whereas the Clojure version nicely separates those two concerns.

Which of these versions is better? The Java version ended up a bit simpler than the Clojure version; but it was also a bit more coupled. The separation of concerns in the Clojure version convinces me that between the two, it would be more flexible and useful.

But, of course, we are only talking about a dozen lines of code.