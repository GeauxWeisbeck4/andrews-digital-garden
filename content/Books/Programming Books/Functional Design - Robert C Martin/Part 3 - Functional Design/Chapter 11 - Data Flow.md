---
id: 01JF1NG2EBP2D1ASFKSN0SNP8X
modified: 2024-12-13T23:05:57-05:00
---
## 11

## Data Flow

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/11_unnum_dataflow.jpg)

In [Chapter 9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09), Object-Oriented Programming, I suggested that the design of a functional program is more like plumbing than procedure. There is a definite data flow bias to it. This is because we tend to use `map`, `filter`, and `reduce` to transform the contents of lists into other lists, rather than iterating through the problem one element at a time to produce results.

We can see this bias in many of our previous examples, including the Bowling Game, Gossiping Bus Drivers, and Payroll applications in [Part II](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/part02.xhtml#part02), Comparative Analysis.

As another example, consider this interesting problem from day ten of Advent of Code 2022.[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch11.xhtml#ch11fn1a) The goal was to render pixels on a 6-by-40 screen. The pixels were drawn from left to right, one at a time, based on a clock circuit. Clock cycles were counted starting at 0. If a certain register `x` matched the clock cycle number, then the pixel at the appropriate screen position was turned on; otherwise, it was turned off.

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch11.xhtml#ch11fn1). [https://adventofcode.com/2022/day/10](https://adventofcode.com/2022/day/10)

This is actually quite typical of the way old CRT[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch11.xhtml#ch11fn2a) displays used to work. You had to energize the electron beam at just the right moment as it rastered over the screen. So you matched the bits in the bitmap to the clock that drove that beam. If, according to the clock, the beam was at position 934, and if the 934th bit in the bitmap was set, then you energized the beam for an instant to display that pixel.

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch11.xhtml#ch11fn2). Cathode ray tube. A _cathode ray_ is an electron. CRTs have electron guns that create narrow beams of electrons that are rastered across the screen using regularly changing magnetic fields. The beam strikes phosphors on the screen and makes them glow, thus creating a raster image.

The Advent of Code problem was a bit more interesting. It asked us to simulate a simple processor that had two instructions. The first instruction was `noop`, which took one clock cycle but had no other effect. The other instruction was `addx`, which took an integer argument `n` that it added to the `x` register of the processor. This instruction consumed two clock cycles and only changed the `x` register after both cycles had completed. Pixels on the screen would be visible for a clock cycle if, and only if, at the beginning of that cycle the `x` register matched the clock cycle number.

So if according to the clock, the beam was over screen position 23, and if the `x` register was 23 at the start of cycle 23, then the beam would be energized for that clock cycle.

To complicate matters just a little more, the matching of the `x` register to the clock cycle was widened so that 22, 23, and 24 would match clock cycle 23. In other words, the `x` register specified a window that was three pixels wide. So long as the clock cycle fell within that window, the beam would be energized.

Since the screen is 40 pixels wide and 6 pixels tall, the matching of the clock cycle to `x` is modulus 40.

The task was to execute a set of instructions and produce a list of six strings that were 40 characters each, with `"#"` indicating a pixel that was visible and `"."` indicating one that was not visible.

If you were to write this program in Java, C, Go, C++, C#, or any other procedural/OO language, you might create a loop that iterated one cycle at a time while accumulating the appropriate pixels for each cycle. The loop would consume instructions and modify the `x` register as directed.

Here’s a typical example in Java:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch11_images.xhtml#f119-01)

```
package crt;

public class Crt {
  private int x;
  private String pixels = "";
  private int extraCycles = 0;
  private int cycle = 0;
  private int ic;
  private String[] instructions;
  public Crt(int x) {
    this.x = x;
  }

  public void doCycles(int n, String instructionsLines) {
    instructions = instructionsLines.split("\n");
    ic = 0;
    for (cycle = 0; cycle < n; cycle++) {
      setPixel();
      execute();
    }
  }

  private void execute() {
    if (instructions[ic].equals("noop"))
      ic++;
    else if (instructions[ic].startsWith("addx ")
             && extraCycles == 0) {
      extraCycles = 1;
    }
    else if (instructions[ic].startsWith("addx ")
             && extraCycles == 1) {
      extraCycles = 0;
      x += Integer.parseInt(instructions[ic].substring(5));
      ic++;
    } else
      System.out.println("TILT");
  }

  private void setPixel() {
    int pos = cycle % 40;
    int offset = pos - x;
    if (offset >= -1 && 1 >= offset)
      pixels += "#";
    else
      pixels += ".";
  }

  public String getPixels() {
    return pixels;
  }

  public int getX() {
    return x;
  }
}
```

Notice all the mutated state. Notice how it iterates, cycle by cycle, to populate the pixels. Notice also the funny business of `extraCycles` to account for the fact that `addx` takes two cycles to execute.

Finally, notice that although the program is nicely partitioned into a few smallish functions, those functions are all coupled together by the mutable state variables. That is, of course, the usual situation for methods of a mutable class.

I solved this problem in Clojure today. And the solution I came up with was very different from the Java code above. Remember as you read this to start at the bottom. Clojure programs are always written from the bottom up.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch11_images.xhtml#f121-01)

```
(ns day10-cathode-ray-tube.core
  (:require [clojure.string :as string]))

(defn noop [state]
  (update state :cycles conj (:x state)))

(defn addx [n state]
  (let [{:keys [x cycles]} state]
    (assoc state :x (+ x n)
                 :cycles (vec (concat cycles [x x])))))

(defn execute [state lines]
  (if (empty? lines)
    state
    (let [line (first lines)
          state (if (re-matches #"noop" line)
                  (noop state)
                  (if-let [[_ n] (re-matches
                                   #"addx (-?\d+)" line)]
                    (addx (Integer/parseInt n) state)
                    "TILT"))]3
      (recur state (rest lines)))))

(defn execute-file [file-name]
  (let [lines (string/split-lines (slurp file-name))
        starting-state {:x 1 :cycles []}
        ending-state (execute starting-state lines)]
    (:cycles ending-state)))

(defn render-cycles [cycles]
  (loop [cycles cycles
         screen ""
         t 0]
    (if (empty? cycles)
      (map #(apply str %) (partition 40 40 "" screen))
      (let [x (first cycles)
            offset (- t x)
            pixel? (<= -1 offset 1)
            screen (str screen (if pixel? "#" "."))
            t (mod (inc t) 40)]
        (recur (rest cycles) screen t)))))

(defn print-screen [lines]
  (doseq [line lines]
    (println line))
  true)

(defn -main []
  (-> "input"
      execute-file
      render-cycles
      print-screen))
```

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch11.xhtml#ch11fn3). TILT is my favorite error message. Long ago, pinball machines would put up this message and cancel your game if you physically tilted the machine in order to manipulate the ball.

The `execute-file` function transforms the list of instructions in the named file into a list of resulting `x` values. The `render-cycles` function then transforms the list of `x` values into a list of pixels, which it finally `partition`s into strings of 40 characters.

Notice that there are, of course, no mutable variables. Instead, the `state` value flows through each of the functions as though through a pipeline.

The `state` value begins in `execute-file` and then flows to `execute`, then repeatedly to `noop` or `addx`, and then back to `execute`, and finally back to `execute-file`. At each stage in that flow, a new value of `state` is created from the old without changing the old.

If this seems eerily familiar to you, it should. This is very much like the pipes and filters we have gotten used to in our command-line shells. Data flows into a command from a pipe, is transformed by that command, and then flows out to the next command through a pipe.

Here’s a recent command I’ve been using at the shell:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch11_images.xhtml#f123-01)

```
ls -lh private/messages | cut -c 32-37,57-64
```

It lists the `private/messages` directory and then `cut`s out certain fields. The data flows out of the `ls` command, through the pipe, and then into the `cut` command. This has the same kind of feel as the `state` value flowing through the `execute`, `addx`, and `noop` functions.

As a result of this pipelining, you should notice that my `cathode-ray-tube` program is partitioned into a set of smallish functions that are not coupled to one another by mutable state. Whatever coupling exists is merely the coupling of the data formats that flow from function to function through the pipes.

Finally, notice that there is none of the funny business we saw in the Java program surrounding the two cycles of the `addx` instruction. Instead, the two cycles are neatly accounted for by simply adding two `x` values to the `:cycles` element of the `state`.

Of course, I didn’t have to use the data flow style. I could have created a Clojure algorithm that was much closer to the Java algorithm. But that’s not the way I think about things when I’m writing in a functional language. Instead, I am biased toward data flow solutions.

Some of the newer features in Java and C# lend themselves to the data flow style. But they are wordy and appear to me to be bolted onto the languages in awkward ways. Your mileage may vary; but I find that when I use procedural/OO languages I tend to iterate much more than I tend to plumb.

Or, to say this differently:

_In mutable languages, behaviors flow through objects. In functional languages, objects flow through behaviors._