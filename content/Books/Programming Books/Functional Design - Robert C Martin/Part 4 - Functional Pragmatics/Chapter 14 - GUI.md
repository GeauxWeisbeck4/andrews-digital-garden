---
id: 01JF1NKEP3N5J5SREB2ENGKPBJ
modified: 2024-12-13T23:07:47-05:00
---
## 14

## GUI

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/18_unnum_gui.jpg)

Over the years, I have used two different GUI frameworks in functional programs. The first is named Quil,[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn1a) and it is based upon the popular Java framework named Processing.[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn2a) The second is SeeSaw,[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn3a) which is based upon the old Java Swing[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn4a) framework.

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn1). [www.quil.info](http://www.quil.info/)

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn2). [https://processing.org](https://processing.org/)

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn3). [https://github.com/clj-commons/seesaw](https://github.com/clj-commons/seesaw)

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn4). [https://en.wikipedia.org/wiki/Swing_(Java)](https://en.wikipedia.org/wiki/Swing_(Java))

Quil is “functional,” which makes it fun and easy to use in a “functional” program. SeeSaw is not functional at all. Indeed, it depends very strongly on mutable state that you must continuously update. This makes it a royal pain to use in a functional program. The difference is startling.

One of the first programs I wrote using Quil was `spacewar`. I’ve mentioned it a few times in this book. If you’d like to see the program in action, you can go to [https://github.com/unclebob/spacewar](https://github.com/unclebob/spacewar) where there is a ClojureScript version you can run in your browser. I did not write `spacewar` to be used in ClojureScript; but Mike Fikes ported it over in a day or so. It actually works better in my browser than it does in native Clojure on my laptop.

### Turtle-Graphics in Quil

Walking through the source code of `spacewar` is beyond the scope of this book. However, there is a simpler Quil program that I wrote awhile back that is the perfect size. It’s `turtle-graphics`.[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn5a)

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn5). [https://github.com/unclebob/turtle-graphics](https://github.com/unclebob/turtle-graphics)

Turtle graphics[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn6a) are a simple set of commands that were invented for the Logo language in the late 1960s. Those commands controlled a robot called a _turtle_. The robot sat on a large piece of paper and had a pen that could be raised and lowered onto the paper. The robot could be told to move forward or backward a certain distance, or to turn a number of degrees left or right.

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn6). [https://en.wikipedia.org/wiki/Turtle_graphics](https://en.wikipedia.org/wiki/Turtle_graphics)

[Figure 14.1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fig01) is a picture of the inventor, Seymour Papert, with one of his turtles.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c14f001.jpg)

**Figure 14.1.** Seymour Papert with one of his turtles[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn7a)

[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn7). Courtesy of MIT Museum.

So, for example, if you’d like to draw a square, you might issue these commands:

```
Pen down
Forward 10
Right 90
Forward 10
Right 90
Forward 10
Right 90
Forward 10
Pen up.
```

The original idea was to introduce children to programming by showing them how to control the turtle to draw interesting shapes. I don’t know how well this worked for children, but it turned out to be pretty useful for programmers who wanted to draw complex designs on the screen. I once used a Logo system with turtle graphics on the Commodore 64 to write a pretty elaborate Lunar Lander game.

Anyway, awhile back, I thought it would be fun to have a turtle graphics system in Clojure so that I could easily investigate some interesting mathematical and geometric puzzles.

My goal was not to create a turtle graphics console on which you would type commands. Instead, I wanted a turtle graphics API that I could use to write graphical functions in Clojure.

So, for example, I wanted to write a program like this:

```
(defn polygon [theta, len, n]
  (pen-down)
  (speed 1000)
  (dotimes [_ n]
    (forward len)
    (right theta)))

(defn turtle-script []
  (polygon 144 400 5))
```

That program draws the picture in [Figure 14.2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fig02). (Notice the little turtle sitting on the left vertex of the star.)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c14f002.jpg)

**Figure 14.2.** A star drawn using turtle graphics

The `turtle-script` function is the entry point for the `turtle-graphics` system. You put your drawing commands into it. In this case, I put a call to the `polygon` function into it.

Perhaps you’ve noticed that the `polygon` function does not appear to be functional because it doesn’t produce a return value from its inputs. Instead, it has the side effect of drawing on the screen. Moreover, each of the commands mutates the state of the turtle. So `turtle-graphics` programs are not functional.

And yet, the `turtle-graphics` framework is “functional.” Or rather, it is about as functional as a GUI program can be.[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn8a) After all, the point of a GUI program is to mutate the state of the screen.

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn8). Although you might find this interesting: [https://fsharpforfunandprofit.com/posts/13-ways-of-looking-at-a-turtle/](https://fsharpforfunandprofit.com/posts/13-ways-of-looking-at-a-turtle/).

The `turtle-graphics` framework begins by configuring and invoking Quil:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f204pre01)

```
(defn ^:export -main [& args]
  (q/defsketch turtle-graphics
               :title "Turtle Graphics"
               :size [1000 1000]
               :setup setup
               :update update-state
               :draw draw-state
               :features [:keep-on-top]
               :middleware [m/fun-mode])
  args)
```

I’m not going to do a full tutorial on Quil here, but there are a few things I should point out. Take note of the `:setup`, `:update`, and `:draw` elements. Each points to a function.

The `setup` function will be called once at the start of the program.

The `draw-state` function will be called 60 times a second in order to refresh the screen. Everything that should be on the screen must be drawn by the `draw` function. The screen doesn’t remember anything.

The `update-state` function will be called just before the `draw-state` function. This function is used to change the state of what is being drawn. Think of it as the function that moves the elements of the screen one 60th of a second into the future.

Think of this like a really simple loop:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f204pre02)

```
(loop [state (setup)]
  (draw-state state)
  (recur (update-state state)))
```

If you think of this as a tail recursive loop, then the contents of the screen are the tail recursive values. So even though we are mutating the contents of the screen, we are doing so at the tail of the recursion where the mutation is harmless.[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn9a) So, although not purely functional, it is as “functional” as any TCO[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn10a) system can be.

[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn9). Mostly harmless.

[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14.xhtml#ch14fn10). Remember our discussion about tail call optimization back in [Chapter 1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01).

Here’s my `setup` function:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f205pre01)

```
(defn setup []
  (q/frame-rate 60)
  (q/color-mode :rgb)
  (let [state {:turtle (turtle/make)
               :channel channel}]
    (async/go
      (turtle-script)
      (prn "Turtle script complete"))
    state))
```

This starts out pretty simple. It sets the frame rate to 60fps and the color mode to RGB, and it creates the `state` object that will be passed to `update-state` and `draw-state`.

The `async/go` function starts up a new lightweight thread in which our `turtle-script` will execute.

The `state` object is composed of a `channel` and the `turtle`. We’ll talk about the `channel` later. For the moment, let’s concentrate on the `turtle`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f205pre02)

```
(s/def ::position (s/tuple number? number?))
(s/def ::heading (s/and number? #(<= 0 % 360)))
(s/def ::velocity number?)
(s/def ::distance number?)
(s/def ::omega number?)
(s/def ::angle number?)
(s/def ::weight (s/and pos? number?))
(s/def ::state #{:idle :busy})
(s/def ::pen #{:up :down})
(s/def ::pen-start (s/or :nil nil?
                         :pos (s/tuple number? number?)))
(s/def ::line-start (s/tuple number? number?))
(s/def ::line-end (s/tuple number? number?))
(s/def ::line (s/keys :req-un [::line-start ::line-end]))
(s/def ::lines (s/coll-of ::line))
(s/def ::visible boolean?)
(s/def ::speed (s/and int? pos?))
(s/def ::turtle (s/keys :req-un [::position
                                 ::heading
                                 ::velocity
                                 ::distance
                                 ::omega
                                 ::angle
                                 ::pen
                                 ::weight
                                 ::speed
                                 ::lines
                                 ::visible
                                 ::state]
                        :opt-un [::pen-start]))
(defn make []
  {:post [(s/assert ::turtle %)]}
  {:position [0.0 0.0]
   :heading 0.0
   :velocity 0.0
   :distance 0.0
   :omega 0.0
   :angle 0.0
   :pen :up
   :weight 1
   :speed 5
   :visible true
   :lines []
   :state :idle})
```

This shows the type specification of the `turtle`, followed by its constructor. Notice that the constructor checks the type as a `:post` condition. The elements of the turtle are mostly self-explanatory. There’s the XY position, the angular heading, the velocity, the up/down state of the pen, the drawing weight of the pen, the visibility state, and so on. The other elements will come to light soon enough.

How do we draw the turtle?

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f207pre01)

```
(defn draw-state [state]
  (q/background 240)
  (q/with-translation
    [500 500]
    (let [{:keys [turtle]} state]
      (turtle/draw turtle))))

——Turtle module——

(defn draw [turtle]
  (when (= :down (:pen turtle))
    (q/stroke 0)
    (q/stroke-weight (:weight turtle))
    (q/line (:pen-start turtle) (:position turtle)))

  (doseq [line (:lines turtle)]
    (q/stroke-weight (:line-weight line))
    (q/line (:line-start line) (:line-end line)))

  (when (:visible turtle)
    (q/stroke-weight 1)
    (let [[x y] (:position turtle)
          heading (q/radians (:heading turtle))
          base-left (- (/ WIDTH 2))
          base-right (/ WIDTH 2)
          tip HEIGHT]
      (q/stroke 0)
      (q/with-translation
        [x y]
        (q/with-rotation
          [heading]
          (q/line 0 base-left 0 base-right)
          (q/line 0 base-left tip 0)
          (q/line 0 base-right tip 0))))))
```

The `draw-state` function, which is called by Quil 60 times each second, sets the background color of the screen to light gray, centers the drawing at (500, 500), and then calls `turtle/draw`, which draws the current line in progress and then all the other lines that were previously drawn. Finally, it draws the turtle itself. Notice how Quil helps with translation and rotation.

So how do we update the turtle state?

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f208pre01)

```
(defn update-state [{:keys [channel] :as state}]
  (let [turtle (:turtle state)
        turtle (turtle/update-turtle turtle)]
    (assoc state :turtle (handle-commands channel turtle))))
```

The `update-state` function calls `turtle/update-turtle`. Then it calls `handle-commands`, and there’s that `channel` again. Let’s look at `update-turtle` first:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f208pre02)

```
(defn update-position
  [{:keys [position velocity heading distance] :as turtle}]
  (let [step (min (q/abs velocity) distance)
        distance (- distance step)
        step (if (neg? velocity) (- step) step)
        radians (q/radians heading)
        [x y] position
        vx (* step (Math/cos radians))
        vy (* step (Math/sin radians))
        position [(+ x vx) (+ y vy)]]
    (assoc turtle :position position
                  :distance distance
                  :velocity (if (zero? distance) 0.0 velocity))))

(defn update-heading [{:keys [heading omega angle] :as turtle}]
  (let [angle-step (min (q/abs omega) angle)
        angle (- angle angle-step)
        angle-step (if (neg? omega) (- angle-step) angle-step)
        heading (mod (+ heading angle-step) 360)]
    (assoc turtle :heading heading
                  :angle angle
                  :omega (if (zero? angle) 0.0 omega))))

(defn make-line [{:keys [pen-start position weight]}]
  {:line-start pen-start
   :line-end position
   :line-weight weight})

(defn update-turtle [turtle]
  {:post [(s/assert ::turtle %)]}
  (if (= :idle (:state turtle))
    turtle
    (let [{:keys [distance
                  state
                  angle
                  lines
                  position
                  pen
                  pen-start] :as turtle}
          (-> turtle
              (update-position)
              (update-heading))
          done? (and (zero? distance)
                     (zero? angle))
          state (if done? :idle state)
          lines (if (and done? (= pen :down))
                  (conj lines (make-line turtle))
                  lines)
          pen-start (if (and done? (= pen :down))
                      position
                      pen-start)]
      (assoc turtle
             :state state
             :lines lines
             :pen-start pen-start))))
```

Notice that `update-turtle` has a `:post` condition that checks the type of the turtle after it has been updated. It’s nice to know that when you update a big structure you haven’t messed up some little part of it.

If the `turtle`’s `:state` is `:idle`, meaning that it is neither moving nor rotating, then we don’t make any changes. Otherwise, we update the position and heading of the `turtle` and then _destructure_ its internals. We are done when the distance and angle remaining in the current animated motion are zero. And if we are done, we set the `:state` to `:idle`.

If we are done and the pen is down, then we add the line in progress to the list of previous lines, and we update the `pen-start` to the current position to prepare for the next line.

Updating the position and heading are simple functions that do the necessary trig calculations to place the turtle in the proper position and orientation. They both use the turtle’s `:velocity` to adjust how big a step they take at each update.

Now on to handling the commands:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f210pre01)

```
(defn handle-commands [channel turtle]
  (loop [turtle turtle]
    (let [command (if (= :idle (:state turtle))
                    (async/poll! channel)
                    nil)]
      (if (nil? command)
        turtle
        (recur (turtle/handle-command turtle command))))))
```

If the turtle is `:idle`, then we are ready for a command. So we poll the `channel`. If there is a command on the `channel`, we process it by calling `turtle/handle-command`, and then repeat until no commands are left on the channel.

Handling each command is pretty straightforward:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f211pre01)

```
(defn pen-down [{:keys [pen position pen-start] :as turtle}]
  (assoc turtle :pen :down
                :pen-start (if (= :up pen) position pen-start)))

(defn pen-up [{:keys [pen lines] :as turtle}]
  (if (= :up pen)
    turtle
    (let [new-line (make-line turtle)
          lines (conj lines new-line)]
      (assoc turtle :pen :up
                    :pen-start nil
                    :lines lines))))

(defn forward [turtle [distance]]
  (assoc turtle :velocity (:speed turtle)
                :distance distance
                :state :busy))

(defn back [turtle [distance]]
  (assoc turtle :velocity (- (:speed turtle))
                :distance distance
                :state :busy))

(defn right [turtle [angle]]
  (assoc turtle :omega (* 2 (:speed turtle))
                :angle angle
                :state :busy))

(defn left [turtle [angle]]
  (assoc turtle :omega (* -2 (:speed turtle))
                :angle angle
                :state :busy))

(defn hide [turtle]
  (assoc turtle :visible false))

(defn show [turtle]
  (assoc turtle :visible true))

(defn weight [turtle [weight]]
  (assoc turtle :weight weight))

(defn speed [turtle [speed]]
  (assoc turtle :speed speed))

(defn handle-command [turtle [cmd & args]]
  (condp = cmd
    :forward (forward turtle args)
    :back (back turtle args)
    :right (right turtle args)
    :left (left turtle args)
    :pen-down (pen-down turtle)
    :pen-up (pen-up turtle)
    :hide (hide turtle)
    :show (show turtle)
    :weight (weight turtle args)
    :speed (speed turtle args)
    :else turtle))
```

We simply translate the command tokens into function calls. Not really rocket science. The command functions manage the state of the turtle. Take for instance, the `forward` command. It sets the `turtle`’s `:state` to `:busy`, sets the turtle’s `:velocity`, and sets the `:distance` it must move before going `:idle` again.

OK, we’re almost done. Now all we need to do is look at the way the `turtle-script` function sends commands to the `channel`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch14_images.xhtml#f212pre01)

```
(def channel (async/chan))
(defn forward [distance] (async/>!! channel [:forward distance]))
(defn back [distance] (async/>!! channel [:back distance]))
(defn right [angle] (async/>!! channel [:right angle]))
(defn left [angle] (async/>!! channel [:left angle]))
(defn pen-up [] (async/>!! channel [:pen-up]))
(defn pen-down [] (async/>!! channel [:pen-down]))
(defn hide [] (async/>!! channel [:hide]))
(defn show [] (async/>!! channel [:show]))
(defn weight [weight] (async/>!! channel [:weight weight]))
(defn speed [speed] (async/>!! channel [:speed speed]))
```

The `async/>!!` function sends its argument to the `channel`. If the `channel` is full, it waits. That really wasn’t very surprising, was it?

And with that, we can put all the turtle graphics commands we like into the `turtle-script` function and watch the turtle dance around the screen drawing our pretty pictures.

You can see this framework in action in the videos at [www.youtube.com/@Cleancoders](http://www.youtube.com/@Cleancoders); specifically, _The Euler Project_, episodes 2.3, 2.2, 5, and 9.