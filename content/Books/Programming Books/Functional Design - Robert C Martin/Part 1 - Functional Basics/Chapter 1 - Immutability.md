---
id: 01JF1N41F3QZYDRZWCHFNP4GW2
title: Chapter 1 - Immutability
modified: 2024-12-13T22:59:22-05:00
---
## 1

## Immutability

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/01_unnum_immutability.jpg)

### What Is Functional Programming?

If you were to ask the average programmer what functional programming is, you might get any of the following answers.

- Programming with functions.
    
- Functions are “first class” elements.
    
- Programming with referential transparency.
    
- A programming style based upon lambda calculus.
    

While these assertions might be true, they are not particularly helpful. I think a better answer is: _Programming without assignment statements_.

Perhaps you don’t think that definition is much better. Perhaps it even frightens you. After all, what do assignment statements have to do with functions; and how can you possibly program without them?

Good questions. Those are the questions that I intend to answer in this chapter.

Consider the following simple C program:

```
int main(int ac, char** av) {
    while(!done())
        doSomething();
}
```

This program is the core loop of virtually every program ever written. It quite literally says: “Do something until you are done.” What’s more, this program has no visible assignment statements. Is it functional? And if so, does that mean every program ever written is functional?

Let’s actually make this function do something. Let’s have it compute the sum of the squares of the first ten integers [1..10]:

```
int n=1;
int sum=0;
int done() {
  return n>10;
}

void doSomething() {
  sum+=n*n;
  ++n;
}

void sumFirstTenSquares() {
    while(!done())
        doSomething();
}
```

This program is not functional because it uses two assignment statements in the `doSomething` function. It’s also just plain ugly with those two global variables. Let’s improve it:

```
int sumFirstTenSquares() {
  int sum=0;
  int i=1;
loop:
  if (i>10)
    return sum;
  sum+=i*i;
  i++;
  goto loop;
}
```

This is better; the two globals have become local variables. But it’s still not functional. Perhaps you are worried about that `goto`. It is there for a good reason. Bear with me as you consider this small modification that uses a worker function to convert the local variables into function arguments:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f005-01)

```
int sumFirstTenSquaresHelper(int sum, int i) {
loop:
  if (i>10)
    return sum;
  sum+=i*i;
  i++;
  goto loop;
}

int sumFirstTenSquares() {
  return sumFirstTenSquaresHelper(0, 1);
}
```

This program is still not functional; but it’s an important _milestone_ that we’ll refer to in a moment. But now, with one last change, something magical happens:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f006-01)

```
int sumFirstTenSquaresHelper(int sum, int i) {
  if (i>10)
    return sum;
  return sumFirstTenSquaresHelper(sum+i*i, i+1);
}

int sumFirstTenSquares() {
  return sumFirstTenSquaresHelper(0, 1);
}
```

All the assignment statements are gone, and this program is functional. It’s also recursive. That’s no accident. If you want to get rid of assignment statements, you _have_ to use recursion. Recursion allows you to replace the assignment of local variables with the _initialization_ of function arguments.

It also burns up a lot of space on the stack. However, there is a little trick we can use to fix that problem.

Notice that the last call to `sumFirstTenSquaresHelper` is also the last use of `sum` and `i` in that function. Holding those two variables on the stack after initializing the two arguments of the recursive call is pointless; they’ll never be used. What if, instead of creating a new stack frame for the recursive call, we simply reused the current stack frame by jumping back to the top of the function with a `goto`, as we did in the _milestone_ program?

This cute little trick is called _tail call optimization (TCO)_ and all functional languages make use of it.[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01fn1a)

[1.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01fn1) In one way or another. The _Java virtual machine (JVM)_ complicates TCO a bit. C, of course, does not do TCO and so all my recursive examples in C will grow the stack.

Notice TCO effectively turns that last program into the _milestone_ program. The last three lines of `sumFirstTenSquaresHelper` in the _milestone_ program are, in effect, the recursive function call. Does that mean the _milestone_ program is functional too? No, it just behaves identically. At the source code level, that program is not functional because it has assignment statements. But if we take one step back and ignore the fact that the local variables changed as opposed to being reinstantiated in a new stack frame, then the program _behaves_ as a functional program.

As we will discover in the next section, that is not a distinction without a difference. In the meantime, just remember when you use recursion to eliminate assignment statements, you are not necessarily wasting lots of space on the stack. The language you are using is almost certainly using TCO.

### The Problem with Assignment

First let’s define what we mean by _assignment_. Assigning a value to a variable _changes_ the original value of the variable to the newly assigned value. It is the change that makes it assignment.

In C we initialize a variable this way:

```
int x=0;
```

But we assign a variable this way:

```
x=1;
```

In the first case, the variable `x` comes into existence with the value `0`; prior to the initialization, there was no variable `x`. In the second case, the value of `x` is changed to `1`. This may not seem significant, but the implications are profound.

In the first case, we do not know if `x` is actually a variable. It could be a constant. In the second case, there is no doubt. We are varying `x` by assigning it a new value. Thus, we can say that functional programming is programming _without variables_. The values in functional programs _do not vary_.

Why is this desirable? Consider the following:

```
.
//Block A
.
x=1;
.
//Block B
.
```

The _state of the system_ during the execution of `Block A` is different from the state of the system in `Block B`. This means that `Block A` must execute _before_ `Block B`. If the position of the two blocks were swapped, the system would likely not execute correctly.

This is called a _sequential or temporal coupling_—a coupling in time; and it is something you are probably quite familiar with. `Open` must be called before `close`. `New` must be called before `delete`. `Malloc` must be called before `free`. The list of pairs[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01fn2a) like this is endless. And in many ways, they are a bane of our existence.

[2.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01fn2) They are like the Sith; always two there are.

How many times have you forgotten to close a file, or release a block of memory, or close a graphics context, or release a semaphore? How many times have you debugged a pernicious problem only to find that you can fix it by swapping the position of two function calls?

And then there’s garbage collection.

Garbage collection is a horrible[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01fn3a) hack that we have accepted into our languages because we are just so bad at managing temporal couplings. If we were adept at keeping track of allocated memory, we would not depend on some nasty background process to clean up after us. But the sad fact is we are so truly terrible at managing temporal couplings that we celebrate the crutches we build to protect ourselves from them.

[3.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01fn3) And, no, reference counting isn’t any better.

And that doesn’t take into account multiple threads. When two or more threads are competing for the processor, keeping the temporal couplings in the correct order becomes a much more significant challenge. Those threads may get the order correct 99.99 percent of the time; but every once in a great while they may execute in the wrong order and cause all manner of mayhem. We call those situations _race conditions_.

Temporal couplings and race conditions are the natural consequence of programming with variables—of using assignment. Without assignment, there are no temporal couplings and there are no race conditions.[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01fn4a) You cannot have a concurrent update problem if you never update anything. You cannot have an ordering issue within a function if the system state never changes within that function.

[4.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01fn4) We’ll see later that this is not entirely correct. As Spock was fond of saying: “There are always possibilities.”

But perhaps it’s time for a simple example. Here’s our nonfunctional algorithm again; this time without the `goto`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f009-01)

```
1: int sumFirstTenSquaresHelper(int sum, int i) {
2:   while (i<=10) {
3:     sum+=i*i;
4:     i++;
5:   }
6:   return sum;
7: }
```

Now let’s say you’d like to log the progress of the algorithm with a statement like this:

```
   log("i=%d, sum=%d", i, sum);
```

Where would you put that line? There are three possibilities. If you add the `log` statement after line 2 or 4, then the logged data will be correct, and the difference will simply be whether you are logging before or after the computation. If you insert the `log` statement after line 3, then the logged data will be incorrect. That is a temporal coupling—an ordering problem.

Now consider our functional solution, with one interesting cosmetic change:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f010-01)

```
int sumFirstTenSquaresHelper(int sum, int i) {
  return (i>10) ? sum : sumFirstTenSquaresHelper(sum+i*i, i+1);
}
```

There is only one place we can put our `log` statement, and it will log correct data.

### So Why Is It Called Functional?

A function is a mathematical object that maps inputs to outputs. Given _y = f(x)_, there is a value of _y_ for every value of _x_. Nothing else matters to _f_. If you give _x_ to _f_, you will get _y_ every single time. The state of the system in which _f_ executes is irrelevant to _f_.

Or to say that a different way, there are no temporal couplings with _f_. There is no special order in which _f_ must be invoked. If you call _f_ with _x_, you will get _y_ no matter what else may have changed.

Functional programs are true functions in this mathematical sense. If you decompose a functional program into many smaller functions, each of those will also be a true function in the same mathematical sense. This is called _referential transparency_.

A function is referentially transparent if you can always replace the function call with its value. Let’s try that with our functional algorithm for calculating the sum of the squares of the first ten integers:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f011-01)

```
int sumFirstTenSquaresHelper(int sum, int i) {
  return (i>10) ? sum : sumFirstTenSquaresHelper(sum+i*i, i+1);
}

int sumFirstTenSquares() {
  return sumFirstTenSquaresHelper(0, 1);
}
```

When we replace the first call to `sumFirstTenSquaresHelper` with its implementation, it becomes:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f011-02)

```
int sumFirstTenSquares() {
  return (1>10) ? 0 : sumFirstTenSquaresHelper(0+1*1, 1+1);
}
```

When we replace the next function call, it becomes:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f011-03)

```
int sumFirstTenSquares() {
  return
    (1>10) ? 0 :
      (2>10) ? 0+1*1
              : sumFirstTenSquaresHelper((0+1*1)+2*2,
                                         (1+1)+1);
}
```

I think you can see where this is going. Each call to `sumFirstTenSquaresHelper` simply gets replaced with its implementation with the arguments properly replaced.

Notice that you cannot do this simple replacement with the nonfunctional version of the program. Oh, you can unwind the loop if you like; but that’s not the same as simply replacing each function call with its implementation.

So, functional programs are composed of true mathematical, referentially transparent functions. And that’s why this is called functional programming.

### No Change of State?

If there are no variables in functional programs, then functional programs cannot change state. How can we expect a program to be useful if it cannot change state?

The answer is that functional programs compute a new state from an old state, _without changing the old state_. If this sounds confusing, then the following example should clear it up:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f012-01)

```
State system(State s) {
  return isFinal(s) ? s : system(s);
}
```

You can start the `system` in some initial `state`, and it will successively move the `system` from `state` to `state` until the final `state` is reached. The `system` does not change a state variable. Instead, at each iteration, a new `state` is created from the old `state`.

If we turn TCO off and allow the stack to grow with each recursive call, then the stack will contain all the previous states, unchanged. Moreover, the `system` functions as a true function in the mathematical sense. If you call `system` with `state1`, it will return `state2` every single time.

If you look closely at our functional version of `sumFirstTenSquares`, you will see that it uses precisely this approach to the changing of state. There are no variables, and no internal state. Rather, the algorithm moves from the initial state to the final state, one state change at a time.

Of course, our `system` function does not appear to be able to respond to any inputs. It simply starts at some initial `state` and then runs to completion. But with a simple modification we can create a “functional” program that responds quite nicely to input events:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f013-01)

```
State system(State state, Event event) {
  return done(state) ? state : system(state, getEvent());
}
```

Now, the computed next `state` of the `system` is a function of the current `state` and an incoming `event`. And voila! We have created a very traditional finite state machine that can react to events in real time.

Notice the quotes I put around the word _functional_ above. That is because `getEvent` is not referentially transparent. Every time you call it you will get a different result. Thus, you cannot replace the call with its return value. Does this mean that our program is not actually functional?

Strictly speaking, any program that takes input in this manner cannot be purely functional. But this is not a book about purely functional programs. This is a book about functional _programming_. The style of the program above is “functional,” even if the input is not pure; and it is that style we are interested in here.

So here, for your entertainment, is a simple little real-time finite state machine that is written in C and is “functional.” It is the time-honored subway turnstile example. Have fun with it.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01_images.xhtml#f013-02)

```
#include <stdio.h>

typedef enum {locked, unlocked, done} State;
typedef enum {coin, pass, quit} Event;

void lock() {
  printf("Locking.\n");
}

void unlock() {
  printf("Unlocking.\n");
}

void thankyou() {
  printf("Thanking.\n");
}

void alarm() {
  printf("Alarming.\n");
}

Event getEvent() {
  while (1) {
    int c = getchar();
    switch (c) {
      case 'c': return coin;
      case 'p': return pass;
      case 'q': return quit;
    }
  }
}

State turnstileFSM(State s, Event e) {
  switch (s) {
    case locked:
    switch (e) {
      case coin:
      unlock();
      return unlocked;

      case pass:
      alarm();
      return locked;

      case quit:
      return done;
    }

    case unlocked:
    switch (e) {
      case coin:
      thankyou();
      return unlocked;

      case pass:
      lock();
      return locked;

      case quit:
      return done;
    }
    case done:
    return done;
  }
}

State turnstileSystem(State s) {
  return (s==done)? 0
                  : turnstileSystem(
                      turnstileFSM(s, getEvent()));
}

int main(int ac, char** av) {
  turnstileSystem(locked);
  return 0;
}
```

Keep in mind that C does not use TCO, and so the stack will grow until it is exhausted—though that may require quite a few operations in this case.

### Immutability

What all this means is that functional programs contain no variables. Nothing in a functional program changes state. State changes are passed from one invocation of a recursive function to the next, without altering any of the previous states. If those previous states aren’t needed, TCO can optimize them away; but in spirit they all still exist, unchanged, somewhere in a past stack frame.

If there are no variables in a functional program, then the values we name are all _constants_. Once initialized, those constants never go away and never change. In spirit, the entire history of every one of those constants remains intact, unchanged, and immutable.