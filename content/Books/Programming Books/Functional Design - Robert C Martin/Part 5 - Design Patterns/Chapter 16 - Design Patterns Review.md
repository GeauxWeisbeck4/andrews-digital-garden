---
id: 01JF1NP2GH2FVTWRDPW0VMS9X1
modified: 2024-12-13T23:09:13-05:00
---
## 16

## Design Patterns Review

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/30_unnum_designpatterns.jpg)

A _design pattern_ is a named solution to a common problem in a particular context. Yes, I know, another word salad. So let me tell you a story.

Long ago, in a decade far, far away, I was a prolific writer on a social network called comp.object.[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn2a) In this group, we debated issues of OO design.

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn2). A newsgroup within the vast array of newsgroups transmitted by Network News Transport Protocol (NNTP) over Unix-to-Unix copy (UUCP) and the Internet.

One day someone posed a simple problem and suggested that we all solve it in our own way and then debate the result. The problem was:

_Given a switch and a light, make the switch turn the light on._

The debates raged for months.

The simplest solution was, of course, [Figure 16.1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig01).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f001.jpg)

**Figure 16.1.** The simplest solution for the switch and the light

The `Switch` class[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn3a) calls the `TurnOn` method of the `Light` class.

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn3). Remember, this was an OO forum. Don’t get hung up on the word _class_.

The objection to this was that the `Switch` class could be used to turn on other things like `Fan`s or `Television`s. Therefore, the `Switch` class should not know about the `Light` class. An abstraction should be imposed between the two, as shown in [Figure 16.2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig02).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f002.jpg)

**Figure 16.2.** The Abstract Server

Now the `Switch` class uses an interface named `Switchable`. The `Light` class implements `Switchable`.

This solves the problem. Now we could have any number of devices controlled by the `Switch`. This solution is one of the simplest expressions of the DIP, the OCP, and the LSP. It also has a name. It’s called _Abstract Server_.[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn4a)

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn4). Robert C. Martin, _Agile Software Development: Principles, Patterns, and Practices_ (Pearson, 2002), 318.

If we were on a team discussing how to protect our `Switch` class from being explicitly coupled to our `Light` class, someone on the team could pipe up and say, “We could use an Abstract Server.” If all the team members knew that name and what it implied, they could quickly decide whether that solution was appropriate or not.

That’s a design pattern, a named solution to a problem in a particular context. The value of design patterns is that the names and the solutions are canonical, and therefore, people who are familiar with that canon can understand one another simply by using the name. You say “[Abstract Server](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16lev1sec2)” and I immediately understand that you mean “impose an interface between the client and the server.”

But what about the context part of the design pattern? Well, let’s go back to our team. Someone has just suggested using the Abstract Server pattern. Another team member says, “No, you don’t understand, we don’t own the `Light` class; it’s part of a third-party library, so we can’t alter it to implement an interface.”

So, the context of the problem is that we want to decouple `Switch` from `Light`, but we can’t modify `Light`. So someone else on the team says, “Well, we could use an _Adapter_.”

If you were on the team and didn’t know what the Adapter pattern was, you wouldn’t understand their suggestion. But if you were aware of the design patterns canon, you could swiftly assess the suggestion. Again, the benefit of design patterns is knowing the names and the canonical forms so that you can quickly apply them.

The Adapter pattern looks like [Figure 16.3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig03).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f003.jpg)

**Figure 16.3.** The object form of the Adapter pattern

The `LightAdapter` implements the `Switchable` interface and forwards the `TurnOn` call to the `Light`. Even before this is drawn on the whiteboard, everyone on the team can see it in their minds because they know the design patterns canon. So they all nod in agreement with the idea.

Just as they are about to move on to the next issue, someone on the team says, “Wait, which form of the Adapter should we use?”

It turns out that the canonical name for a design pattern does not necessarily describe a single solution. Some of the patterns have multiple forms. The Adapter is one such pattern. It could look like [Figure 16.3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig03), or it could look like [Figure 16.4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig04).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f004.jpg)

**Figure 16.4.** The class form of the Adapter pattern

The former is called the _object_ form of the Adapter because the `LightAdapter` is its own object. The latter is the _class_ form of the Adapter because the `LightAdapter` is a subclass of `Light`.

The team members debate the two forms for a moment and come to the decision that the class form of the Adapter is sufficient for the moment and will relieve them of the complication of constructing a separate `LightAdapter` object.

### Patterns in Functional Programming

Among the strange rumors we have heard over the years is that design patterns are hacks to get around the problems created by OO languages and that in functional languages they are not necessary.

As you’ll see in the pages that follow, there are indeed aspects of certain design patterns that appear to be workarounds for certain inadequacies in OO languages; but this is hardly applicable to all design patterns. Moreover, even those particular design patterns have a more general form in which they are applicable in functional languages.

### Abstract Server

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/20_unnum_abstractserver.jpg)

So, what does the Abstract Server look like in a functional language?

Consider the `Switch/Light` problem again. Here’s how we might express it in Clojure:

```
(defn turn-on-light []
  ;turn on the bloody light!
  )

(defn engage-switch []
  ;Some other stuff. . .
  (turn-on-light))
```

OK, that’s not rocket science. However, the original problem is immediately evident. Our `engage-switch` function has a direct dependency on `turn-on-light`, which means we can’t use it to turn on a fan or a television or anything else. So, what should we do?

We can use the Abstract Server pattern, of course. All we need to do is insert an abstract interface between the `engage-switch` function and the `turn-on-light` function. We could do that by simply passing a function argument. Let’s call this the _function_ form of the Abstract Server:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f234pre01)

```
(defn engage-switch [turn-on-function]
  ;Some other stuff. . .
  (turn-on-function))
```

That works in the simplest case. But let’s make the problem just a bit more interesting. Let’s say that our `engage-switch` function must turn the light both on and off at various times. Perhaps it’s part of some home security system with special timers for the lights. This changes the original problem to look like this:

```
(defn turn-on-light []
  ;turn on the bloody light!
  )

(defn turn-off-light []
  ;Criminy! just turn it off!
  )

(defn engage-switch []
  ;Some other stuff...
  (turn-on-light)
  ;Some more other stuff...
  (turn-off-light))
```

Now the `engage-switch` function is twice as coupled to the light. We could use the same function form of the Abstract Server, but it’s a bit ugly passing in two arguments. So let’s pass in a single vtable argument. We’ll call this the _vtable_ form of the Abstract Server:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f235-01)

```
(defn make-switchable-light []
  {:on turn-on-light
   :off turn-off-light})

(defn engage-switch [switchable]
  ;Some other stuff...
  ((:on switchable))
  ;Some more other stuff...
  ((:off switchable)))
```

Yeah, that’s actually pretty nice. And since Clojure is a dynamically typed language, we don’t have the problem that an inheritance or implements relationship would cause.

Of course, we could have solved this with the _multi-method_ form of the Abstract Server pattern:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f235-02)

```
(defmulti turn-on :type)
(defmulti turn-off :type)

(defmethod turn-on :light [switchable]
  (turn-on-light))

(defmethod turn-off :light [switchable]
  (turn-off-light))

(defn engage-switch [switchable]
  ;Some other stuff...
  (turn-on switchable)
  ;Some more other stuff...
  (turn-off switchable))
```

I tested this using the following test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f236-01)

```
(describe "switch/light"
  (with-stubs)
  (it "turns light on and off"
    (with-redefs [turn-on-light (stub :turn-on-light)
                  turn-off-light (stub :turn-off-light)]
      (engage-switch {:type :light})
      (should-have-invoked :turn-on-light)
      (should-have-invoked :turn-off-light))))
```

The two stubs mock out the target functions. We invoke the `engage-switch` function with the `{:type :light}` argument. Then we test that the two target functions were, in fact, called.

I’ll leave the _protocol/record_ form of the Abstract Server pattern as an exercise. At this point, it should be clear that the pattern is both applicable and useful in a functional language.

### Adapter

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/21_unnum_adapter.jpg)

The Adapter pattern is used whenever you have a client who wants to use a server, but the interface that the client expects and the interface that the server expresses are incompatible.

As an example, let’s suppose that we have the `engage-switch` function from the preceding discussion, but we want to pass it a third-party `:variable-light`. The `turn-on-light` function of the `:variable-light` accepts an argument for the intensity of the light: `0` for off and `100` for full on.

The interface of the `:variable-light` does not match the expectation of the `engage-switch` function. So we need an Adapter.

Perhaps the simplest form of the Adapter might look like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f237-01)

```
(defn turn-on-light [intensity]
  ;Turn it on with intensity.
  )

(defmulti turn-on :type)
(defmulti turn-off :type)

(defmethod turn-on :variable-light [switchable]
  (turn-on-light 100))

(defmethod turn-off :variable-light [switchable]
  (turn-on-light 0))

(defn engage-switch [switchable]
  ;Some other stuff...
  (turn-on switchable)
  ;Some more other stuff...
  (turn-off switchable))
```

I tested this with the following test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f237-02)

```
(describe "Adapter"
  (with-stubs)
  (it "turns light on and off"
    (with-redefs [turn-on-light (stub :turn-on-light)]
      (engage-switch {:type :variable-light})
      (should-have-invoked :turn-on-light {:times 1 :with [100]})
      (should-have-invoked :turn-on-light {:times 1 :with [0]}))))
```

If I were to draw this structure in the UML, I’d likely draw something like [Figure 16.5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig05).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f005.jpg)

**Figure 16.5.** The object form of the Adapter pattern

The `defmulti` functions correspond to the `Switchable` interface. The `{:type :variable-light}` object, coupled to the two `defmethod` functions, corresponds to the `VariableLightAdapter`. The `EngageSwitch` and `VariableLight` “classes” correspond to the two functions that we are trying to adapt.

Perhaps you don’t find this convincing. After all, it’s just a simple little program with a couple of `defmulti` functions. There’s no obvious OO structure like that shown in the UML. So let’s impose that structure by splitting up the source files.

We begin with the `switchable` interface. In the `ns` statement, I used the convention that `turn-on-light` was the overall namespace for the project that contains the `switchable` namespace:

```
(ns turn-on-light.switchable)

(defmulti turn-on :type)
(defmulti turn-off :type)
```

This is a polymorphic interface. Notice that it has no source code dependencies. Also, keep in mind that the `ns` statement in Clojure has the same kind of source file requirement that Java has for classes. The source file and the namespace have to have corresponding names.[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn5a) So, as we move the elements of this code into separate namespaces, we are also moving them into separate source files.

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn5). In particular, the `turn-on-light.switchable` namespace must be in a file named `switchable.clj` within a directory named `turn_on_light`.

Next, let’s see the `engage-switch` and `variable-light` namespaces:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f239-01)

```
(ns turn-on-light.engage-switch
  (:require [turn-on-light.switchable :as s]))

(defn engage-switch [switchable]
  ;Some other stuff...
  (s/turn-on switchable)
  ;Some more other stuff...
  (s/turn-off switchable))

————————————————

(ns turn-on-light.variable-light)

(defn turn-on-light [intensity]
  ;Turn it on with intensity.
  )
```

No real surprises here. The `engage-switch` namespace depends upon the `switchable` interface. The `variable-light` namespace has no outgoing source code dependencies.

The `variable-light-adapter` namespace connects the `switchable` interface to the `variable-light`. Notice the `make-adapter` constructor. The tests will use that:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f239-02)

```
(ns turn-on-light.variable-light-adapter
  (:require [turn-on-light.switchable :as s]
            [turn-on-light.variable-light :as v-l]))
(defn make-adapter []
  {:type :variable-light})

(defmethod s/turn-on :variable-light [switchable]
  (v-l/turn-on-light 100))

(defmethod s/turn-off :variable-light [switchable]
  (v-l/turn-on-light 0))
```

And lastly, the test ties everything together in a nice, neat little ball by depending upon all the concrete namespaces:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f240-01)

```
(ns turn-on-light.turn-on-spec
  (:require [speclj.core :refer :all]
            [turn-on-light.engage-switch :refer :all]
            [turn-on-light.variable-light :as v-l]
            [turn-on-light.variable-light-adapter
              :as v-l-adapter]))

(describe "Adapter"
  (with-stubs)
  (it "turns light on and off"
    (with-redefs [v-l/turn-on-light (stub :turn-on-light)]
      (engage-switch (v-l-adapter/make-adapter))
      (should-have-invoked :turn-on-light
                           {:times 1 :with [100]})
      (should-have-invoked :turn-on-light
                           {:times 1 :with [0]}))))
```

Look through those source code dependencies and compare them to the UML diagram, and you’ll see that they match perfectly.

So which form of the Adapter pattern was this? We might call it the multi-method form; but it is also the object form.

Would it be possible, in Clojure, to build the class form of the Adapter pattern? No, because Clojure does not have inheritance of implementation, and that’s what the class form of the Adapter pattern depends upon.

So, although the Adapter pattern is not language specific, there are forms that are. It would not be possible, for example, to create the multi-method form of the Adapter pattern in Java.

#### Is That Really an Adapter Object?

Perhaps you think that since the only data element in the `variable-light-adapter` is the `:type`, it is not really worthy of being called an object. OK then, here is a different version of the `variable-light-adapter` that you might find more convincing:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f241-01)

```
(ns turn-on-light.variable-light-adapter
  (:require [turn-on-light.switchable :as s]
            [turn-on-light.variable-light :as v-l]))

(defn make-adapter [min-intensity max-intensity]
  {:type :variable-light
   :min-intensity min-intensity
   :max-intensity max-intensity})

(defmethod s/turn-on :variable-light [variable-light]
  (v-l/turn-on-light (:max-intensity variable-light)))

(defmethod s/turn-off :variable-light [variable-light]
  (v-l/turn-on-light (:min-intensity variable-light)))

————————

(ns turn-on-light.turn-on-spec
  (:require [speclj.core :refer :all]
            [turn-on-light.engage-switch :refer :all]
            [turn-on-light.variable-light :as v-l]
            [turn-on-light.variable-light-adapter
              :as v-l-adapter]))
(describe "Adapter"
  (with-stubs)
  (it "turns light on and off"
    (with-redefs [v-l/turn-on-light (stub :turn-on-light)]
      (engage-switch (v-l-adapter/make-adapter 5 90))
      (should-have-invoked :turn-on-light
                           {:times 1 :with [90]})
      (should-have-invoked :turn-on-light
                           {:times 1 :with [5]}))))
```

By now, you should be convinced that this is the Adapter pattern, right out of the GOF[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn6a) book. You should also be expecting that many of the other GOF patterns can be expressed in functional languages like Clojure. And, perhaps more importantly, you should be thinking about namespace/source file structures as part of the design and architecture of functional programs.

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn6). _GOF_ is the affectionate name we gave to the _Design Patterns_ book back in the ‘90s. It stands for “Gang of Four” because there were four authors: Erich Gamma, John Vlissides, Ralph Johnson, and Richard Helm.

### Command

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/22_unnum_command.jpg)

Of all the design patterns in the GOF book, _Command_ is the one that intrigues me the most. Not because it is complicated, but because it is _simple_. Very, very simple.

As an aside, this is also what intrigues me about Clojure. As I said in the introduction to this book, Clojure is semantically rich but syntactically trivial. Well, the Command pattern has the same attributes. Its richness is in its outrageous simplicity.

In C++, we might write the Command pattern as follows:

```
class Command {
  public:
    virtual void execute() = 0;
};
```

That’s it. Just one abstract class (interface) with a single, pure, virtual (abstract) function. So simple. But there are just so many interesting things you can do with this pattern. For a deep dive into this richness, see the corresponding chapter in _Agile Software Development: Principles, Patterns, and Practices_.[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn7a)

[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn7). Martin, _Agile Software Development_, p. 181.

In a functional language like Clojure, you might think that this pattern just disappears. After all, if you want to pass a command to some other function, you can just pass the `command` function. You don’t need to make an object out of it, because in functional languages, functions _are_ objects:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f243-01)

```
(ns command.core)

(defn execute []
  )

(defn some-app [command]
  ;Some other stuff. . .
  (command)8
  ;Some more other stuff. . .
  )

———————

(ns command.core-spec
  (:require [speclj.core :refer :all]
            [command.core :refer :all]))

(describe "command"
  (with-stubs)
  (it "executes the command"
    (with-redefs [execute (stub :execute)]
      (some-app execute)
      (should-have-invoked :execute))))
```

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn8). The careful reader will recognize that the command, as it is written, is not a pure (referentially transparent) function. It should be clear, however, that pure functions can be passed in the manner shown.

As you can see, the test passes the `execute` function to `some-app`, and the `some-app` function invokes that command. No big deal.

Now, what if you wanted to create the command with a data element that will get passed as an argument to the execute function? In C++, we’d do that this way (pardon the inline functions):

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f244-01)

```
class CommandWithArgument : public Command {
  public:
    CommandWithArgument(int argument)
    :argument(argument)
    {}

    virtual void execute()
    {theFunctionToExecute(argument);}

  private:
    int argument;

    void theFunctionToExecute(int argument)
    {
      //do something with that argument!
    }
};
```

In Clojure we’d do it like this, once again demonstrating that functions, in functional languages, are actually objects:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f245-01)

```
(describe "command"
  (with-stubs)
  (it "executes the command"
    (with-redefs [execute (stub :execute)]
      (some-app (partial execute :the-argument))
      (should-have-invoked :execute {:with [:the-argument]}))))

—————

(defn execute [argument]
  )

(defn some-app [command]
  ;Some other stuff. . .
  (command)
  ;Some more other stuff. . .
  )
```

#### Undo

One of the more useful variations of the Command pattern can be seen in the following C++ code:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f245-02)

```
class UndoableCommand : public Command {
  public:
    virtual void undo() = 0;
};
```

That `undo()` function opens up so many interesting possibilities.

Long ago, I worked on a GUI application that was an analog of AutoCAD. It was a drawing tool for architectural floor plans, roof plans, property line plans, and so on. The GUI was a typical palette/canvas. Users clicked in the palette to select the function they wanted, such as _Add a Room_, and then they’d click in the canvas for placement and size.

Every click in the palette caused the appropriate derivative of the `UndoableCommand` to be instantiated and executed. The execution managed the mouse/keyboard gestures in the canvas and then made the appropriate modifications to the internal data model. Thus, there was an `UndoableCommand` derivative for every different function that the palette could offer.

When an `UndoableCommand` had finished execution, it was pushed onto the _undo_ stack. Whenever the user clicked on the _undo_ icon in the palette, the `UndoableCommand` on the top of the _undo_ stack was popped off and its `undo` function was called.

As an `UndoableCommand` object executed, it recorded what it did in such a way that the `undo` function could reverse those changes. In C++, that recording was kept in the member variables of the particular `UndoableCommand` object itself:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f246-01)

```
class AddRoomCommand : public UndoableCommand {
  public:
    virtual void execute() {
      // manage canvas events to add room
      // record what was done in theAddedRoom
    }

    virtual void undo() {
      // remove theAddedRoom from the canvas
    }

  private:
    Room* theAddedRoom;
};
```

This is not functional, because the `AddRoomCommand` object is mutable. But in a functional language, we can simply have the `execute` function create a new instance of `UndoableCommand`. Something like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f247-01)

```
(ns command.undoable-command)

(defmulti execute :type)
(defmulti undo :type)

—————

(ns command.add-room-command
  (:require [command.undoable-command :as uc]))

(defn add-room []
  ;stuff that adds rooms to the canvas
  ;and returns the added room
  )

(defn delete-room [room]
  ;stuff that deletes the specified room from the canvas
  )

(defn make-add-room-command []
  {:type :add-room-command})

(defmethod uc/execute :add-room-command [command]
  (assoc (make-add-room-command) :the-added-room (add-room)))

(defmethod uc/undo :add-room-command [command]
  (delete-room (:the-added-room command)))

——————

(ns command.core
  (:require [command.undoable-command :as uc]
            [command.add-room-command :as ar]))

(defn gui-app [actions]
  (loop [actions actions
         undo-list (list)]
    (if (empty? actions)
      :DONE
      (condp = (first actions)
        :add-room-action
        (let [executed-command (uc/execute
                                 (ar/make-add-room-command))]
          (recur (rest actions)
                 (conj undo-list executed-command)))

        :undo-action
        (let [command-to-undo (first undo-list)]
          (uc/undo command-to-undo)
          (recur (rest actions)
                 (rest undo-list)))
        :TILT))))

————————

(ns command.core-spec
  (:require [speclj.core :refer :all]
            [command.core :refer :all]
            [command.add-room-command :as ar]))

(describe "command"
  (with-stubs)
  (it "executes the command"
    (with-redefs [ar/add-room (stub :add-room {:return :a-room})
                  ar/delete-room (stub :delete-room)]
      (gui-app [:add-room-action :undo-action])
      (should-have-invoked :add-room)
      (should-have-invoked :delete-room {:with [:a-room]}))))
```

We create the `undoable-command` interface using `defmulti` functions. We implement that interface in the `add-room-command` namespace, and we simulate the GUI in the `gui-app` function of the `command.core` namespace.

The test stubs out the low-level functions of the `add-room-command` and makes sure they are called correctly. It calls the `gui-app` with a list of `palette-actions`.

The two methods of the `add-room-command` are polymorphically dispatched. That might not seem necessary for the `execute` case, since the `gui-app` has just created the `add-room-command` object. But were we to add more commands to this system, the polymorphic dispatch of `execute` would become more necessary.

The polymorphic dispatch of `undo` is clearly necessary, even in this small example, because by the time the `:undo-action` is received from the palette, we have no idea which command is being undone.

Here, again, we see that as we add complexity to the application, the canonical form of the GOF pattern begins to assert itself. With the single method command, we could get away with using plain old functions (function objects, really). But when the application needed a richer kind of command, we fell back on the GOF style.

### Composite

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/23_unnum_composite.jpg)

_Composite_ continues the theme of semantic richness and syntactic triviality. It is a wonderful example of the old handle/body approach that I first read about in one of Jim Coplien’s books.[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn9a) The structure of the Composite pattern is depicted in the UML in [Figure 16.6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig06).

[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn9). James O. Coplien, _Advanced C++ Programming Styles and Idioms_ (Addison-Wesley, 1991).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f006.jpg)

**Figure 16.6.** The Composite pattern

Our old friend the `Switchable` interface is implemented by our other old friends, the `Light` and the `VariableLight`. The `CompositeSwitchable` also implements `Switchable` and contains a list of other instances of `Switchable`.

The implementation of `TurnOn` and `TurnOff` in the `CompositeSwitchable` simply propagates calls of the same functions to all the instances in the list. Thus, when you call `TurnOn` on an instance of a `CompositeSwitchable`, it will call `TurnOn` on all the `Switchable` instances it contains.

In Java, we might implement `CompositeSwitchable` as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f250-01)

```
public class CompositeSwitchable implements Switchable {
  private List<Switchable> switchables = new ArrayList<>();

  public void addSwitchable(Switchable s) {
    switchables.add(s):
  }

  public void turnOn() {
    for (var s : switchables)
      s.turnOn();
  }

  public void turnOff() {
    for (var s : switchables)
      s.turnOff();
  }
}
```

In a functional language, like Clojure, the temptation is to avoid the Composite pattern and simply use the `map` or `doseq` function, as you can see in the test below:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f251-02)

```
(ns composite-example.switchable)

(defmulti turn-on :type)
(defmulti turn-off :type)

—————

(ns composite-example.light
  (:require [composite-example.switchable :as s]))

(defn make-light [] {:type :light})

(defn turn-on-light [])
(defn turn-off-light [])

(defmethod s/turn-on :light [switchable]
  (turn-on-light))

(defmethod s/turn-off :light [switchable]
  (turn-off-light))

———————

(ns composite-example.variable-light
  (:require [composite-example.switchable :as s]))

(defn make-variable-light [] {:type :variable-light})

(defn set-light-intensity [intensity])

(defmethod s/turn-on :variable-light [switchable]
  (set-light-intensity 100))

(defmethod s/turn-off :variable-light [switchable]
  (set-light-intensity 0))

———————————

(ns composite-example.core-spec
  (:require [speclj.core :refer :all]
            [composite-example
             [light :as l]
             [variable-light :as v]
             [switchable :as s]]))

(describe "composite-switchable"
  (with-stubs)
  (it "turns all on"
    (with-redefs
      [l/turn-on-light (stub :turn-on-light)
       v/set-light-intensity (stub :set-light-intensity)]
      (let [switchables [(l/make-light) (v/make-variable-light)]]
        (doseq [s-able switchables] (s/turn-on s-able))
        (should-have-invoked :turn-on-light)
        (should-have-invoked :set-light-intensity
                             {:with [100]})))))
```

This accomplishes the goal of turning on all the lights, but it does so at the expense of externalizing the plurality of the lights. The point of the Composite pattern is to hide that plurality. So let’s use the actual Composite pattern:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f252-02)

```
(ns composite-example.composite-switchable
  (:require [composite-example.switchable :as s]))

(defn make-composite-switchable []
  {:type :composite-switchable
   :switchables []})

(defn add [composite-switchable switchable]
  (update composite-switchable :switchables conj switchable))

(defmethod s/turn-on :composite-switchable [c-switchable]
  (doseq [s-able (:switchables c-switchable)]
    (s/turn-on s-able)))

(defmethod s/turn-off :composite-switchable [c-switchable]
  (doseq [s-able (:switchables c-switchable)]
    (s/turn-off s-able)))

——————

(ns composite-example.core-spec
  (:require [speclj.core :refer :all]
            [composite-example
             [light :as l]
             [variable-light :as v]
             [switchable :as s]
             [composite-switchable :as cs]]))

(describe "composite-switchable"
  (with-stubs)
  (it "turns all on"
    (with-redefs
      [l/turn-on-light (stub :turn-on-light)
       v/set-light-intensity (stub :set-light-intensity)]
      (let [group (-> (cs/make-composite-switchable)
                      (cs/add (l/make-light))
                      (cs/add (v/make-variable-light)))]
        (s/turn-on group)
        (should-have-invoked :turn-on-light)
        (should-have-invoked :set-light-intensity
                             {:with [100]})))))
```

The `composite-switchable` implements the `switchable` interface. The `add` function is functional in that it returns a new `composite-switchable` with the argument added to the `:switchables` list. The `turn-on` and `turn-off` methods use `doseq` to iterate through the `:switchables` list and propagate the appropriate function call. Finally, the test creates the `composite-switchable`, adds a `light` and `variable-light`, and then invokes `turn-on`. And we see both lights turned on appropriately.

#### Functional?

At this point, you might be thinking that this is all well and good for objects that have side effects, like lights and variable lights. Indeed, the entire `switchable` interface is oriented around the side effect of turning something on or off. So is this pattern only for objects with side effects?

Let’s consider a `shape` abstraction that looks like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f254-01)

```
(ns composite-example.shape
  (:require [clojure.spec.alpha :as s]))

(s/def ::type keyword?)
(s/def ::shape-type (s/keys :req [::type]))

(defmulti translate (fn [shape dx dy] (::type shape)))
(defmulti scale (fn [shape factor] (::type shape)))
```

It’s a straightforward interface with two methods: `translate` and `scale`. I also added a type specification for safety’s sake. (This would be a good time to brush up on the double-colon syntax of namespaced keywords.) Every `shape` will be a map that has a `::shape/type` element.

The `circle` and `square` implementations are also pretty straightforward, including their type specifications:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f254-02)

```
(ns composite-example.circle
  (:require [clojure.spec.alpha :as s]
            [composite-example.shape :as shape]))

(s/def ::center (s/tuple number? number?))
(s/def ::radius number?)
(s/def ::circle (s/keys :req [::shape/type
                              ::radius
                              ::center]))

(defn make-circle [center radius]
  {:post [(s/valid? ::circle %)]}
  {::shape/type ::circle
   ::center center
   ::radius radius})

(defmethod shape/translate ::circle [circle dx dy]
  {:pre [(s/valid? ::circle circle)
         (number? dx) (number? dy)]
   :post [(s/valid? ::circle %)]}
  (let [[x y] (::center circle)]
    (assoc circle ::center [(+ x dx) (+ y dy)])))

(defmethod shape/scale ::circle [circle factor]
  {:pre [(s/valid? ::circle circle)
         (number? factor)]
   :post [(s/valid? ::circle %)]}
  (let [radius (::radius circle)]
    (assoc circle ::radius (* radius factor))))

———————

(ns composite-example.square
  (:require [clojure.spec.alpha :as s]
            [composite-example.shape :as shape]))

(s/def ::top-left (s/tuple number? number?))
(s/def ::side number?)
(s/def ::square (s/keys :req [::shape/type
                              ::side
                              ::top-left]))

(defn make-square [top-left side]
  {:post [(s/valid? ::square %)]}
  {::shape/type ::square
   ::top-left top-left
   ::side side})

(defmethod shape/translate ::square [square dx dy]
  {:pre [(s/valid? ::square square)
         (number? dx) (number? dy)]
   :post [(s/assert ::square %)]}
  (let [[x y] (::top-left square)]
    (assoc square ::top-left [(+ x dx) (+ y dy)])))

(defmethod shape/scale ::square [square factor]
  {:pre [(s/valid? ::square square)
         (number? factor)]
   :post [(s/valid? ::square %)]}
  (let [side (::side square)]
    (assoc square ::side (* side factor))))
```

Notice the `:pre` and `:post` conditions on the methods. I’m using these to check the types coming into and going out of the functions. You could rightly be concerned about the runtime penalty of all those checks. I’d either globally disable[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn10a) them, or strategically comment them out once I was happy that my types were being managed properly.

[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn10). There is a compile-time switch that disables all asserts, including `:pre` and `:post`.

Notice that the `translate` and `scale` functions return new `shape` instances. They are fully functional in their behavior.

So, now let’s look at `composite-shape`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f256-01)

```
(ns composite-example.composite-shape
  (:require [clojure.spec.alpha :as s]
            [composite-example.shape :as shape]))

(s/def ::shapes (s/coll-of ::shape/shape-type))
(s/def ::composite-shape (s/keys :req [::shape/type
                                       ::shapes]))

(defn make []
  {:post [(s/assert ::composite-shape %)]}
  {::shape/type ::composite-shape
   ::shapes []})

(defn add [cs shape]
  {:pre [(s/valid? ::composite-shape cs)
         (s/valid? ::shape/shape-type shape)]
   :post [(s/valid? ::composite-shape %)]}
  (update cs ::shapes conj shape))

(defmethod shape/translate ::composite-shape [cs dx dy]
  {:pre [(s/valid? ::composite-shape cs)
         (number? dx) (number? dy)]
   :post [(s/valid? ::composite-shape %)]}
  (let [translated-shapes (map #(shape/translate % dx dy)
                               (::shapes cs))]
    (assoc cs ::shapes translated-shapes)))

(defmethod shape/scale ::composite-shape [cs factor]
  {:pre [(s/valid? ::composite-shape cs)
         (number? factor)]
   :post [(s/valid? ::composite-shape %)]}
  (let [scaled-shapes (map #(shape/scale % factor)
                           (::shapes cs))]
    (assoc cs ::shapes scaled-shapes)))
```

We’ve seen this pattern before in the `light`/`variable-light` example. This time, however, the `composite-shape` returns a new `composite-shape` with the new `shape` instances. And so it is functional.

For those of you who are curious, here are the tests I used:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f257-01)

```
(ns composite-example.core-spec
  (:require [speclj.core :refer :all]
            [composite-example
             [square :as square]
             [shape :as shape]
             [circle :as circle]
             [composite-shape :as cs]]))

(describe "square"
  (it "translates"
    (let [s (square/make-square [3 4] 1)
          translated-square (shape/translate s 1 1)]
      (should= [4 5] (::square/top-left translated-square))
      (should= 1 (::square/side translated-square))))

  (it "scales"
    (let [s (square/make-square [1 2] 2)
          scaled-square (shape/scale s 5)]
      (should= [1 2] (::square/top-left scaled-square))
      (should= 10 (::square/side scaled-square)))))

(describe "circle"
  (it "translates"
    (let [c (circle/make-circle [3 4] 10)
          translated-circle (shape/translate c 2 3)]
      (should= [5 7] (::circle/center translated-circle))
      (should= 10 (::circle/radius translated-circle))))

  (it "scales"
    (let [c (circle/make-circle [1 2] 2)
          scaled-circle (shape/scale c 5)]
      (should= [1 2] (::circle/center scaled-circle))
      (should= 10 (::circle/radius scaled-circle)))))

(describe "composite shape"
  (it "translates"
    (let [cs (-> (cs/make)
                 (cs/add (square/make-square [0 0] 1))
                 (cs/add (circle/make-circle [10 10] 10)))
          translated-cs (shape/translate cs 3 4)]
      (should= #{{::shape/type ::square/square
                  ::square/top-left [3 4]
                  ::square/side 1}
                 {::shape/type ::circle/circle
                  ::circle/center [13 14]
                  ::circle/radius 10}}
               (set (::cs/shapes translated-cs)))))

  (it "scales"
    (let [cs (-> (cs/make)
                 (cs/add (square/make-square [0 0] 1))
                 (cs/add (circle/make-circle [10 10] 10)))
          scaled-cs (shape/scale cs 12)]
      (should= #{{::shape/type ::square/square
                  ::square/top-left [0 0]
                  ::square/side 12}
                 {::shape/type ::circle/circle
                  ::circle/center [10 10]
                  ::circle/radius 120}}
               (set (::cs/shapes scaled-cs))))))
```

You may have noticed that as we proceed in these chapters, I’m using more of the nuanced features of Clojure. This is intentional. I expect that as you read this book, you will have a good Clojure reference nearby, so I’m giving you a series of opportunities to look things up and get more familiar with the language.

As we have seen, Composite is yet another GOF pattern that fits well into the functional world. Once we start taking advantage of polymorphic dispatch, with either vtables, multi-methods, or protocol/record structures, the GOF patterns fit right in, more or less as the GOF described them.

### Decorator

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/24_unnum_decorator.jpg)

Yet another of the handle/body patterns is _Decorator_. The Decorator pattern is a way to add functionality to a type model without directly modifying the type model.

For example, let’s continue with our `shape` project. We have a `shape` type model that supports `circle` and `square` subtypes. Within that type model, so long as it conforms to the LSP, we can `translate` and `scale` any of the subtypes of `shape` without knowing the explicit subtype we are manipulating.

Now let’s add a new, optional functionality: a `journaled-shape`. A `journaled-shape` is a `shape` that remembers the operations that have been performed on it since its creation. We want to be able to keep journals on `square`s and `circle`s; but only certain `square`s and `circle`s. We don’t want every `circle` and `square` to be journaled, because the memory and processing penalty is too high.

Now, of course, we could implement this by adding a `:journaled?` flag to the `shape` abstraction and then putting an `if` statement in the `circle` and `square` implementations. But that’s messy. What we really want is a way to add this functionality without changing the `shape` abstraction or any of its subtypes, including `circle`, `square`, and `composite-shape` (the OCP).

Enter the Decorator pattern. The UML looks like [Figure 16.7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig07).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f007.jpg)

**Figure 16.7.** The Decorator pattern

I’ve included the `composite-shape` because it is currently part of the `shape` type model. The `journaled-shape` is the Decorator. The `journaled-shape` derives from `shape` and holds a reference to a `shape`. When `translate` or `scale` is called on a `journaled-shape` it creates an entry in the journal and then delegates the call to the contained `shape`.

Here’s the Clojure implementation:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f261-01)

```
(ns decorator-example.journaled-shape
  (:require [decorator-example.shape :as shape]
            [clojure.spec.alpha :as s]))

(s/def ::journal-entry
       (s/or :translate (s/tuple #{:translate}11 number? number?)
             :scale (s/tuple #{:scale} number?)))
(s/def ::journal (s/coll-of ::journal-entry))
(s/def ::shape ::shape/shape-type)
(s/def ::journaled-shape (s/and
                           (s/keys :req [::shape/type
                                         ::journal
                                         ::shape])
                           #(= ::journaled-shape
                               (::shape/type %))))

(defn make [shape]
  {:post [(s/valid? ::journaled-shape %)]}
  {::shape/type ::journaled-shape
   ::journal []
   ::shape shape})

(defmethod shape/translate ::journaled-shape [js dx dy]
  {:pre [(s/valid? ::journaled-shape js)
         (number? dx) (number? dy)]
   :post [(s/valid? ::journaled-shape %)]}
  (-> js (update ::journal conj [:translate dx dy])
      (assoc ::shape (shape/translate (::shape js) dx dy))))

(defmethod shape/scale ::journaled-shape [js factor]
  {:pre [(s/valid? ::journaled-shape js)
         (number? factor)]
   :post [(s/valid? ::journaled-shape %)]}
  (-> js (update ::journal conj [:scale factor])
      (assoc ::shape (shape/scale (::shape js) factor))))
```

[11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn11). A set can be used as a function that tests for membership.

The `::journaled-shape` object has `::shape` and `::journal` fields. The `::journal` field is a collection of `::journal-entry` tuples that are of the form `[:translate dx dy]` or `[:scale factor]` where `dx`, `dy`, and `factor` are numbers. The `::shape` field must contain a valid `shape`.

The `make` constructor creates a valid `journaled-shape` (as checked by the `:post` condition).

The `translate` and `scale` functions add the appropriate journal entry to the `::journal` and then delegate their respective functions to the `::shape`, returning a new `journaled-shape` with the updated `::journal` and the modified `::shape`.

Here’s the test. I only tested the `journaled-shape` with a `square` because if it works for `square`, it will work for every `shape`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f263-01)

```
(describe "journaled shape decorator"
  (it "journals scale and translate operations"
    (let [jsd (-> (js/make (square/make-square [0 0] 1))
                  (shape/translate 2 3)
                  (shape/scale 5))]
      (should= [[:translate 2 3] [:scale 5]]
               (::js/journal jsd))
      (should= {::shape/type ::square/square
                ::square/top-left [2 3]
                ::square/side 5}
               (::js/shape jsd)))))
```

We make a `journaled-shape` with a `square` in it. We `translate` and `scale` it, and then we make sure the `::journal` has recorded the `translate` and `scale` calls and that the `square` has the translated and scaled values.

Once again, I’ve included the type specifications just to give you a challenge and to demonstrate how they can be used. Frankly, however, I think the tests do an adequate job of checking the types; so in real life, I doubt I would use such detailed type specifications for this kind of small problem. On the other hand, it is kind of nice to see the types all spelled out like that.

In any case, notice that the `journaled-shape` Decorator will work for any `shape`, including a `composite-shape`. So we have effectively added a new functionality to the type model without making any changes to the existing element of that type model. That’s the OCP at work.

### Visitor

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/25_unnum_visitor.jpg)

Oh, no! Not the. . . _Visitor_! Yes, we’re going to investigate the much-maligned Visitor pattern. Visitor is not one of the handle/body patterns. It has its own unique structure that, as we’ll see, is complicated by certain language choices.

The purpose of the Visitor pattern is similar to that of the Decorator pattern. We want to add a new function to an existing type model without changing that type model (the OCP). The Decorator is appropriate when the new function is independent of the other subtypes in the type model. Look back at the `journaled-shape` to verify this constraint. The journaling was independent of whether the contained shape was a `circle` or a `square`. The `journaled-shape` Decorator never knew the subtype of the contained `shape`.

We use the Visitor pattern when the function we wish to add is _dependent_ upon the subtypes in the type model.

So, for example, what if we wanted to add a function to our shape abstraction for converting the shape to a string for serialization purposes? We could add a `to-string` function to the `shape` interface. Easy-peasy.

But wait! What if one of our customers wanted the shapes in XML? I suppose we could add a `to-xml` function as well as the `to-string` function.

But, wait again! What if another of our customers wanted the shapes in JSON, and yet another wanted them in YAML, and. . .

At some point, you realize that there is no end to these data formats and that customers are going to continually ask you for more and more and more. And you don’t want to pollute the `shape` interface with all those horrible methods.

The Visitor pattern gives us a way out of this dilemma. The UML looks something like [Figure 16.8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig08).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f008.jpg)

**Figure 16.8.** The Visitor pattern

The first thing I want to point out is the 90-degree rotation of the `Shape` subtypes into methods in the `ShapeVisitor`. Each of the subtypes, `Square` and `Circle`, is the type of the argument of a `visit` function in the `ShapeVisitor`. I call the subtype-to-method transformation a 90-degree rotation because it pleases some neurons in my hindbrain.

We see our `Shape` abstraction and all its subtypes over on the left. On the right, we see the `ShapeVisitor` hierarchy. The pattern adds the `accept` function to the `Shape` interface. That function takes a single argument, which is a `ShapeVisitor`. This violates the OCP, but only once.

In Java, the implementation of the `accept` function is trivial:

```
void accept(ShapeVisitor v) {
  v.visit(this);
}
```

If you’ve never studied the Visitor pattern before, then this might be a little difficult to follow. So take your time and walk through this with me.

Let’s say we want a JSON string for some `Shape` we’ve got. In Java, or C++ or other similar languages, here’s how we’d get it:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f266-01)

```
Shape s = // get a shape without knowing the subtype
ShapeVisitor v = new JsonVisitor();
s.accept(v);
String json = v.getJson();
```

We get a `Shape` object from somewhere. We create the `JsonVisitor`. We pass the `JsonVisitor` to our `Shape` using the `accept` method. The `accept` method polymorphically dispatches to the proper subtype of `Shape`—let’s say it’s a `Square`. The `accept` method of `Square` calls `visit(this)` on the `JsonVisitor`. The type of `this` is `Square`, so the `visit(Square s)` function of the `JsonVisitor` is called. That function generates the JSON string for the `Square` and saves it in a member variable of the `JsonVisitor`. The `getJson()` function returns the contents of that member variable.

You may have to read that over a few times to follow it. This is a technique called _double-dispatch_. The first dispatch deploys to the subtype of the `Shape`, so now we know the type of that subtype. The second dispatch deploys to the proper subtype of the visitor passing along the true type of the subtype.

If you followed all of that, you can see that each of the derivatives of the `ShapeVisitor` is a new “method” of the `Shape` type model, but the only thing we had to add to `Shape` was the `accept` method. So ~(the OCP). You should also now understand why we couldn’t use a Decorator. The new functions depend strongly on the subtypes. You can’t make a JSON string for a `Square` if you don’t know it’s a `Square`.

Now, I told you all that so I could tell you this. All that horrible complexity is there because of a language constraint. Yes, yes. . . this is where all those design pattern naysayers actually do have a point. The Visitor pattern is as complex as it is because of a particular language feature.

What feature is that? _Closed classes_.

#### To Close, or to Clojure?

In languages like C++ and Java, we create classes that are _closed_. What that means is that we cannot add a new method to a class by putting that new method’s declaration in a new source file. If we want to add a new method to a class, in a closed language, we have to open the source file of that class and add the method _within_ the definition of that class.

Clojure does not have this constraint. Neither, to some extent, does C#. Indeed, many languages allow you to add methods to classes without changing the source file that contains the declaration of those classes.

The reason Clojure does not have this constraint is that classes are not a feature of the language. We create them by convention, not by syntax.

So, wait, does that mean we don’t need the Decorator or Visitor pattern in Clojure? No, it doesn’t mean that at all. Indeed, as we saw, we still need the Decorator in its GOF form. How else would you do the `journaled-shape`?

However, the GOF form of the Visitor is not necessary in languages that have open classes. Or rather, some of the details of the GOF form are not necessary.

So let me show you this particular Visitor in Clojure. First, the tests:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f268-01)

```
(ns visitor-example.core-spec
  (:require [speclj.core :refer :all]
            [visitor-example
             [square :as square]
             [json-shape-visitor :as jv]
             [circle :as circle]]))

(describe "shape-visitor"
  (it "makes json square"
    (should= "{\"top-left\": [0,0], \"side\": 1}"
             (jv/to-json (square/make [0 0] 1))))

  (it "makes json circle"
    (should= "{\"center\": [3,4], \"radius\": 1}"
             (jv/to-json (circle/make [3 4] 1)))))
```

This shouldn’t be too surprising; although you should pay special attention to the source code dependencies. This test needs pretty much everything.

Now let’s remember what the `shape` type model looks like. Just to keep things simple, I’ve removed all the `clojure.spec` type specifications:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f268-02)

```
(ns visitor-example.shape)

(defmulti translate (fn [shape dx dy] (::type shape)))
(defmulti scale (fn [shape factor] (::type shape)))

———————

(ns visitor-example.square
  (:require
    [visitor-example.shape :as shape]))

(defn make [top-left side]
  {::shape/type ::square
   ::top-left top-left
   ::side side})

(defmethod shape/translate ::square [square dx dy]
  (let [[x y] (::top-left square)]
    (assoc square ::top-left [(+ x dx) (+ y dy)])))

(defmethod shape/scale ::square [square factor]
  (let [side (::side square)]
    (assoc square ::side (* side factor))))

————————

(ns visitor-example.circle
  (:require
    [visitor-example.shape :as shape]))

(defn make [center radius]
  {::shape/type ::circle
   ::center center
   ::radius radius})

(defmethod shape/translate ::circle [circle dx dy]
  (let [[x y] (::center circle)]
    (assoc circle ::center [(+ x dx) (+ y dy)])))

(defmethod shape/scale ::circle [circle factor]
  (let [radius (::radius circle)]
    (assoc circle ::radius (* radius factor))))
```

That should all look pretty familiar. Now for the `json-shape-visitor`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f269-01)

```
(ns visitor-example.json-shape-visitor
  (:require [visitor-example
             [shape :as shape]
             [circle :as circle]
             [square :as square]]))

(defmulti to-json ::shape/type)

(defmethod to-json ::square/square [square]
  (let [{:keys [::square/top-left12 ::square/side]} square
        [x y] top-left]
    (format "{\"top-left\": [%s,%s], \"side\": %s}" x y side)))

(defmethod to-json ::circle/circle [circle]
  (let [{:keys [::circle/center ::circle/radius]} circle
        [x y] center]
    (format "{\"center\": [%s,%s], \"radius\": %s}" x y radius)))
```

[12](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn12). The namespaced keyword _destructuring_ creates a local var named for the local part of the key—`top-left` in this case.

Look at this carefully. That `defmulti` in the `json-shape-visitor` adds the `to-json` method directly into the `shape` type model. You probably understand it well enough at this point; but do you see _why_ this is a Visitor?

Can you see the 90-degree rotation from subtypes to functions?

Just like the Java version of the Visitor, all the subtypes for the `to-json` operation are gathered into the `json-shape-visitor` module.

If you follow all the source code dependencies and compare them to the UML diagram, you’ll see that they are all there. The only things missing are the `ShapeVisitor` interface and the dual dispatch. Those were just there to get around the fact that languages like C++ and Java have closed classes.

This tells us that the GOF got this pattern a bit wrong. The dual dispatch is ancillary to the Visitor pattern and is only necessary in languages with closed classes.

#### The 90-degree Problem

But wait. That 90-degree rotation has a problem. Whenever you have a module that has methods for each of the subtypes of some type model, that module must be changed whenever the type model is changed. For example, if we were to add a `triangle` to our `shape` hierarchy, our `json-shape-visitor` would need a `::triangle/triangle defmethod` of `to-json`. This violates the OCP.

This is also a problem because it violates the _Dependency Rule_ of _Clean Architecture_[13](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn13a) by forcing higher-level modules to have source code dependencies upon lower-level modules across an architectural boundary.[14](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn14a) This is shown in the UML in [Figure 16.9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig09).

[13](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn13). Robert C. Martin, _Clean Architecture_ (Pearson, 2017), p. 203.

[14](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn14). Martin, _Clean Architecture_, p. 159.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f009.jpg)

**Figure 16.9.** Violation of the Dependency Rule

In general, we want the `shape` implementations to be plug-ins to the `App`. But the `json-shape-visitor` thwarts that because the only way for our `App` to emit JSON is to invoke the `json-shape-visitor`, which depends directly on `circle` and `square`.

In Java, C#, and C++, we can solve this by using an _abstract factory_, which the `App` could use to instantiate the `visitor` object without depending directly upon it.

In Clojure, we have another—and much better—option. We can just separate the interface of the `json-shape-visitor` from its implementation as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f271-01)

```
(ns visitor-example.json-shape-visitor
  (:require [visitor-example
             [shape :as shape]]))
(defmulti to-json ::shape/type)

————————

(ns visitor-example.json-shape-visitor-implementation
  (:require [visitor-example
             [json-shape-visitor :as v]
             [circle :as circle]
             [square :as square]]))

(defmethod v/to-json ::square/square [square]
  (let [{:keys [::square/top-left ::square/side]} square
        [x y] top-left]
    (format "{\"top-left\": [%s,%s], \"side\": %s}" x y side)))

(defmethod v/to-json ::circle/circle [circle]
  (let [{:keys [::circle/center ::circle/radius]} circle
        [x y] center]
    (format "{\"center\": [%s,%s], \"radius\": %s}" x y radius)))
```

The trick to this is to make sure that the `json-shape-visitor-implementation` module is `require`d by `main` so that the `defmethod`s are properly registered with the `defmulti`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f272-01)

```
(ns visitor-example.main
  (:require [visitor-example
             [json-shape-visitor-implementation]]))
```

Typically, `main` is invoked before any part of the application, and thus, the application does not have a source code dependency on `main`.[15](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn15a) Unfortunately, my tests do not have access to a true `main`, so the dependency has to be included:

[15](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn15). Martin, _Clean Architecture_, p. 231.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f272-02)

```
(ns visitor-example.core-spec
  (:require [speclj.core :refer :all]
            [visitor-example
             [square :as square]
             [json-shape-visitor :as jv]
             [circle :as circle]
             [main]]))

(describe "shape-visitor"
  (it "makes json square"
    (should= "{\"top-left\": [0,0], \"side\": 1}"
             (jv/to-json (square/make [0 0] 1))))

  (it "makes json circle"
    (should= "{\"center\": [3,4], \"radius\": 1}"
             (jv/to-json (circle/make [3 4] 1)))))
```

So there it is, a functional, and architecturally competent, Visitor in Clojure. As the UML in [Figure 16.10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig10) shows, all the dependencies cross the architectural boundary pointing to the higher-level (abstract) side of that boundary. Hallelujah!

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f010.jpg)

**Figure 16.10.** Functional and architecturally competent Visitor

So the Visitor pattern is a case where the GOF form was polluted by the language constraints of the day. In 1995, when the GOF book was published, closed classes were considered a necessary attribute of statically typed languages and were therefore almost ubiquitous.

### Abstract Factory

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/26_unnum_abstractfactory.jpg)

The DIP advises us to avoid source code dependencies upon things that are both volatile and concrete. So we create abstract structures and try to route our dependencies upon them. However, when we create instances of objects, we often have to violate that advice; and this can cause architectural difficulties, as shown by the UML in [Figure 16.11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig11).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f011.jpg)

**Figure 16.11.** DIP violation due to creation

The `App` in [Figure 16.11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig11) uses the `Shape` interface. Everything it needs to do can be done through that interface, with one exception. The `App` must create instances of the `Circle` and `Square` derivatives; and that forces the `App` to hang source code dependencies upon the corresponding modules.

We’ve actually seen this situation in our previous examples. Consider, for example, the code from the tests from the `visitor-example` earlier in this chapter. Notice that the test requires source code dependencies upon `square` and `circle` for the sole purpose of calling those `make` functions:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f275-01)

```
(ns visitor-example.core-spec
  (:require [speclj.core :refer :all]
            [visitor-example
             [square :as square]
             [json-shape-visitor :as jv]
             [circle :as circle]]))

(describe "shape-visitor"
  (it "makes json square"
    (should= "{\"top-left\": [0,0], \"side\": 1}"
             (jv/to-json (square/make [0 0] 1))))

  (it "makes json circle"
    (should= "{\"center\": [3,4], \"radius\": 1}"
             (jv/to-json (circle/make [3 4] 1)))))
```

Perhaps this seems a small price to pay. But if, as shown in [Figure 16.12](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig12), we add an architectural boundary to that UML diagram, the true cost becomes clear.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f012.jpg)

**Figure 16.12.** Violation of the Dependency Rule across the architectural boundary

Here we can see that the _Dependency Rule_ of _Clean Architecture_[16](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn16a) has been violated by that `<creates>` dependency. That rule states that all source code dependencies that cross an architectural boundary must point toward the higher-level side of that boundary. The `Circle` and `Square` modules are low-level details that are plug-ins to the `App`. Thus, to preserve the architecture, we need to somehow deal with those `<creates>` dependencies.

[16](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn16). Robert C. Martin, _Clean Architecture_ (Pearson, 2017).

The _Abstract Factory_ pattern provides a good solution. It looks like [Figure 16.13](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig13).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c16f013.jpg)

**Figure 16.13.** Abstract Factory pattern resolves Dependency Rule

All the source code dependencies that cross the boundary now point toward the higher-level side, so the Dependency Rule violation has been resolved. The `Circle` and `Square` can still be independent plug-ins to the `App`. The `App` can still create `Circle` and `Square` instances but indirectly through the `ShapeFactory` interface, which inverts the source code dependency (the DIP).

This is easy to implement in Clojure. All we need is the `shape-factory` interface and its implementation:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f276-01)

```
(ns abstract-factory-example.shape-factory)

(defmulti make-circle
  (fn [factory center radius] (::type factory)))

(defmulti make-square
  (fn [factory top-left side] (::type factory)))

—————

(ns abstract-factory-example.shape-factory-implementation
  (:require [abstract-factory-example
             [shape-factory :as factory]
             [square :as square]
             [circle :as circle]]))

(defn make []
  {::factory/type ::implementation})

(defmethod factory/make-square ::implementation
  [factory top-left side]
  (square/make top-left side))

(defmethod factory/make-circle ::implementation
  [factory center radius]
  (circle/make center radius))
```

And with that, we can write a test that simulates our `App`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f277-01)

```
(ns abstract-factory-example.core-spec
  (:require [speclj.core :refer :all]
            [abstract-factory-example
             [shape :as shape]
             [shape-factory :as factory]
             [main :as main]]))

(describe "Shape Factory"
  (before-all (main/init))
  (it "creates a square"
    (let [square (factory/make-square
                   @main/shape-factory
                   [100 100] 10)]
      (should= "Square top-left: [100,100] side: 10"
               (shape/to-string square))))
  (it "creates a circle"
      (let [circle (factory/make-circle
                     @main/shape-factory
                     [100 100] 10)]
        (should= "Circle center: [100,100] radius: 10"
                 (shape/to-string circle)))))
```

The first thing to notice about this test is that it has no source file dependencies on `circle` or `square`. It depends only on the two interfaces: `shape` and `shape-factory`. That was our architectural goal.

But what is that `main` dependency? Do you see the `(before-all (main/init))` line at the start of the test? That tells the test runner to call `(main/init)` before any of the tests. This simulates the `main` module initializing everything before starting the `App`.

Here’s `main`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f278-01)

```
(ns abstract-factory-example.main
  (:require [abstract-factory-example
            [shape-factory-implementation :as imp]]))

(def shape-factory (atom nil))

(defn init[]
  (reset! shape-factory (imp/make)))
```

Oh, HO! We’ve got a global `atom` named `shape-factory`! And that `atom` is being initialized to the `shape-factory-implementation` by the `init` function.

So, looking back at the test, we see that the `make-circle` and `make-square` methods were passing the dereferenced `atom`.

Setting a global like this is a pretty common strategy for dealing with factories. The main program creates the concrete factory implementations and then loads it into a global that everyone can access. In a statically typed language, that global would have the type of the interface `ShapeFactory`. In dynamically typed languages, no such type declaration is required.

#### 90 Degrees Again

Look at that UML diagram in [Figure 16.13](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fig13) again. Do you see the 90-degree rotation in the `ShapeFactory`? You can see it in the `shape-factory` code too. The `ShapeFactory` (and the `shape-factory`) have methods that correspond to the subtypes of `Shape`.

The problem that this caused for Visitor is also present here, although in a slightly different form. Whenever a new subtype of `shape` is added, the `shape-factory` must be modified. That violates the OCP because we must modify a module on the high-level side of the architectural boundary. If the OCP matters at all, it matters most especially across such boundaries. Study that UML diagram until you see what I mean.

We can resolve this problem by replacing the 90-degree rotation with a single method that takes an opaque token. Something like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16_images.xhtml#f279-01)

```
(ns abstract-factory-example.shape-factory)

(defmulti make (fn [factory type & args] (::type factory)))

——————————

(ns abstract-factory-example.shape-factory-implementation
  (:require [abstract-factory-example
             [shape-factory :as factory]
             [square :as square]
             [circle :as circle]]))

(defn make []
  {::factory/type ::implementation})

(defmethod factory/make ::implementation
  [factory type & args]
  (condp = type
    :square (apply square/make args)
    :circle (apply circle/make args)))

————————————————

(ns abstract-factory-example.core-spec
  (:require [speclj.core :refer :all]
            [abstract-factory-example
             [shape :as shape]
             [shape-factory :as factory]
             [main :as main]]))

(describe "Shape Factory"
  (before-all (main/init))
  (it "creates a square"
    (let [square (factory/make
                   @main/shape-factory
                   :square
                   [100 100] 10)]
      (should= "Square top-left: [100,100] side: 10"
               (shape/to-string square))))

  (it "creates a circle"
      (let [circle (factory/make
                     @main/shape-factory
                     :circle
                     [100 100] 10)]
        (should= "Circle center: [100,100] radius: 10"
                 (shape/to-string circle)))))
```

Notice that the argument passed into `shape-factory/make` is opaque. That is, it is not defined by any of the other modules, including—and especially—the `square` and `circle` modules. The `:square` and `:circle` keywords are not namespaced, nor are they declared anywhere. They are simply opaque values that happen to have names. I might as well have used `1` for `square` and `2` for `circle`, or used `"square"` and `"circle"` strings.

This opacity is the key to this solution. If we ever need to add a `triangle` subtype, nothing above the boundary line will have to change (the OCP).

#### Type Safety?

In a statically typed language, like Java, this technique abandons type safety. Opaque values cannot be type safe. There is no way, for example, to use an `enum` in Java to solve this issue.

In Clojure, we aren’t concerned about static type safety, but what about dynamic type specifications? We’re out of luck there too. There is no way to gain an advantage by using `clojure.spec` since all errors, either with or without `clojure.spec`, will be runtime errors.

For example, nothing stops me from calling `shape-factory/make` with `:sqare` (intentionally misspelled). The `condp` in `shape-factory-implementation` will simply throw an exception. If I were to set up some type constraint in `clojure.spec` forcing the `type` argument of `shape-factory/make` to be either `:square` or `:circle`, it would still just throw a runtime exception.

There is no escape from this in any language. Whether in Java, C++, Ruby, Clojure, or C#, if you want to maintain the OCP across architectural boundaries (and you usually do), then at some point across that boundary you are going to have to abandon type safety and rely on runtime exceptions. This is just simply software physics.

### Conclusion

I’ll leave the rest of the GOF patterns, and any other patterns you might be familiar with, as an exercise. By now, I’m pretty sure you understand that functional languages that have facilities similar to Clojure are as OO as Java, C#, Ruby, and Python, and that the patterns described in the GOF book generally apply so long as the constraint of immutability is enforced.

And as for _Singleton_: Just create one.

### Postscript: OO Poison?

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/27_unnum_oopoison.jpg)

I thought it wise to revisit here my hope and goal from the introduction. By now, it should be clear that functional programming and OOP are compatible and mutually beneficial styles.

The design pattern examples that I have presented so far are not unusual. Clojure programmers frequently use `defmulti` and `defmethod` to express polymorphism. They typically use maps to express encapsulated data structures (i.e., objects). They often even build constructors for those objects. They might not realize it, but they are building OO programs.

What might seem unusual to some functional programmers, and even to some Clojure programmers, is the way I have organized the source files and namespaces. That organization is so reminiscent of Java, C++, C#, Ruby, and even Python that it screams “OO” to folks who’d thought that they’d left OO behind many long years ago.

It should be very clear by now that Clojure is every bit as object oriented as Java, C++, C#, Python, and Ruby. Clojure is also as functional as F#, Scala, Elixir, and (dare I say it?) Haskell.

Let’s examine the OO claim just a bit.

Clojure does not have inheritance; but it does have at least three very effective mechanisms of polymorphism. At least two of those mechanisms support open classes.

Clojure does not have `public`/`private`/`protected` modifiers; but it does have namespaced keywords and dynamic type specification, which allows encapsulation to be strongly expressed and dynamically, if not statically, enforced. Clojure also has private functions (created with `defn-`) that can only be seen within the containing source file.

Clojure supports, but does not enforce, a source file and namespace structure that affords the same architectural partitioning we find so familiar in any of the (so-called) enterprise languages.

And so Clojure is an OO/functional[17](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn17a) language. As are, to one extent or another, languages like Scala, Elixir, and F#, to name just a few. And, since that is true, the OO mindset is still a perfectly valid way of modeling applications in those languages.

[17](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16fn17). OOFL? FOOL? Hmm, perhaps we should avoid the acronyms.

We can still describe our functional programs using interfaces and classes, types and subtypes. We can still partition the source files and manage their dependencies in order to create robust, independently deployable and independently developable architectures. Nothing in that regard has changed at all.

What _has_ changed is the extra constraint that functional programming places upon us, which is the elimination, or at least the strong sequestration, of side effects. Our classes and modules will strongly prefer immutable, as opposed to mutable, objects. But they are still objects, and they can still be expressed and organized as classes that implement interfaces.

And that means that the vast majority of the design principles and design patterns that we found so helpful in OO languages still apply, and are still useful, in functional languages like Clojure and others.