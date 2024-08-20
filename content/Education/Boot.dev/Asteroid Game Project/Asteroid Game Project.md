---
id: 01J5NP06FQ3YG8KA8GVEGYHE20
title: Asteroid Game Project
description: Building asteroid with Python
tags:
  - python
  - boot-dev
  - game-development
  - project
modified: 2024-08-19T14:53:00-04:00
---
# Asteroid Game Project
# Build an Asteroids Game

We're going to build a simple video game, based on the classic [Asteroids](https://en.wikipedia.org/wiki/Asteroids_(video_game)). If you've never played before, you can take a look at this (slightly different from our) [version of the game](https://freeasteroids.org/).

![The finished project](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/YmSwzVB.gif)

## Prerequisites

- Python 3.12+ installed (see the [bookbot project](https://www.boot.dev/courses/build-bookbot) for help if you don't already have it)
- Access to a unix-like shell (e.g. zsh or bash)

## Learning Goals

The learning goals of this project are:

1. Introduce you to multi-file Python projects
2. Show you a real-world use case for object-oriented programming
3. To have a ton of fun building a rewarding project!

The goal is _not_ to teach game development or the math required for physics simulations. As a result, there are some places where you will simply copy-paste code that we wrote for you. Don't worry about trying to understand every detail if it's not interesting to you, you will be asked to write the parts that matter most.

## Assignment

To get started, make sure you have the [Boot.dev CLI](https://github.com/bootdotdev/bootdev) installed and working.

**Run and submit the tests using the CLI.**

_The tests just ensure that the CLI is installed and configured correctly._

# Installation

We are going to be using [pygame](https://www.pygame.org/) and a [virtual environment](https://docs.python.org/3/library/venv.html) to develop our game.

## Pygame

Pygame is a module for developing games using Python. It provides simple functions and methods for us to easily draw images within a GUI window and handle user input.

## Virtual Environment (venv)

Virtual environments are Python's way to keep dependencies (e.g. the `pygame` module) separate from other projects on our machine. For example, we need `pygame` version 2 for this project, but another project on your computer might require version 1.

As a best practice, each Python project on your machine should have its own virtual environment to keep them isolated from each other.

## Assignment

1. [ ]  Create a new directory and Git repository for this project somewhere on your computer.
2. [ ]  Create a virtual environment at the top level of your project directory:

```bash
python3 -m venv venv
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

3. [ ]  Activate the virtual environment:

```bash
source venv/bin/activate
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

You should see `(venv)` at the beginning of your terminal prompt, for example, mine is:

```
(venv) wagslane@MacBook-Pro-2 asteroids %
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

4. [ ]  Create a file called `requirements.txt` in the top level of your project directory with the following contents:

```
pygame==2.6.0
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

This tells Python that this project requires `pygame` version 2.6.0.

5. [ ]  Install the requirements:

```bash
pip install -r requirements.txt
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

_[pip](https://pip.pypa.io/en/stable/) is Python's package manager. It will install the `pygame` module into the virtual environment you created._

6. [ ]  Make sure `pygame` is installed:

```bash
python3 -m pygame
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Run and submit the CLI tests.
# Game Loop

Video games are generally built using a [game loop](https://gameprogrammingpatterns.com/game-loop.html). The simplest game loop has 3 steps:

1. Check for player inputs
2. Update the game world
3. Draw the game to the screen

To create a good user experience, these 3 steps need to happen _many times per second_.

## Assignment

1. [ ]  `import pygame` at the top of your `main.py` file. _The [pygame documentaion](https://www.pygame.org/docs/ref/pygame.html) will be _super_ useful throughout this project_.
2. [ ]  [Initialize pygame](https://www.pygame.org/docs/ref/pygame.html#pygame.init) at the beginning of your `main()` function (take a look at the docs for the syntax).
3. [ ]  Ensure our predefined constants (`constants.py`) `SCREEN_WIDTH` and `SCREEN_HEIGHT` are imported at the top of your file:

```python
from constants import *
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

4. [ ]  Use pygame's [`display.set_mode()`](https://www.pygame.org/docs/ref/display.html#pygame.display.set_mode) to get a new GUI window:

```python
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

5. [ ]  Create the game loop. For now, we'll only worry about step 3: _drawing the game onto the screen_.

- Use an [infinte while loop](https://realpython.com/python-while-loop/#infinite-loops) for the game loop. At each iteration, it should:
    - Use the screen's [fill](https://www.pygame.org/docs/ref/surface.html#pygame.Surface.fill) method to fill the screen with a solid "black" color.
    - Use pygame's [`display.flip()`](https://www.pygame.org/docs/ref/display.html#pygame.display.flip) method to refresh the screen.

6. [ ]  It's a good idea to run and test your game frequently as you write code , to make sure it's working as expected:

```bash
python3 main.py
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

You should see a black window open _and stay open_.

7. [ ]  Close the game and kill the program with `Ctrl+C` in the terminal.
8. [ ]  Add the following code to the beginning of each iteration of the game loop:

```python
for event in pygame.event.get():
    if event.type == pygame.QUIT:
        return
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

This will check if the user has closed the window and exit the game loop if they do. _It will make the window's close button work._

9. [ ]  Make a git commit! It's a good idea to commit your progress whenever you get something new working.

With the window painting black and closing properly, you're ready to move on to the next step!

# Sprites

_Note: Throughout this project, we will provide some of the code for you, like the class below. We want you to focus on specific parts and have prewritten the stuff that isn't related to the concepts we're trying to teach._

In pygame, there is a base class called [Sprite](https://www.pygame.org/docs/ref/sprite.html#pygame.sprite.Sprite), to represent visual objects.

## Assignment

In our game, asteroids are _visually_ represented as circles, and the player is a triangle. However, detecting collisions between circles and triangles is _hard_. To avoid this problem, we can cheat a little bit: the player will secretly be a circle.

![The player is secretly a circle.](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/vn3shY7.png)_The red circle won't be visible in the game; only we need to know it exists._

Let's create a `CircleShape` class that inherits from `Sprite` to represent objects in our game that are treated as circles (even if they aren't).

Create a new `circleshape.py` file and paste in the following code:

```python
import pygame

# Base class for game objects
class CircleShape(pygame.sprite.Sprite):
    def __init__(self, x, y, radius):
        # we will be using this later
        if hasattr(self, "containers"):
            super().__init__(self.containers)
        else:
            super().__init__()

        self.position = pygame.Vector2(x, y)
        self.velocity = pygame.Vector2(0, 0)
        self.radius = radius

    def draw(self, screen):
        # sub-classes must override
        pass

    def update(self, dt):
        # sub-classes must override
        pass
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

`CircleShape` extends the `Sprite` class to also store a position, [velocity](https://en.wikipedia.org/wiki/Velocity), and [radius](https://en.wikipedia.org/wiki/Radius).

Later you'll write subclasses of `CircleShape` and override the `draw` and `update` methods with the logic for that particular game object.

**Run the CLI tests from the root of the project.**