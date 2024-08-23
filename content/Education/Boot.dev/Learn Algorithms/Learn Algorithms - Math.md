---
id: 01J5VGT43T0AB1TBD6MK99GHVR
title: Learn Algorithms - Math
modified: 2024-08-21T18:59:47-04:00
description: The math section of Boot.dev algorithms course
tags:
  - algorithms
  - mental-health
  - backend
  - boot-dev
  - python
---
# Learn Algorithms - Part Two Math
# Exponents

An [exponent](https://en.wikipedia.org/wiki/Exponentiation) indicates how many times a number is to be multiplied by itself.

For example: 53 = `5 * 5 * 5`

Sometimes exponents are also written using the caret symbol (`^`):

`5^3` = 53

## Exponents in Python

We can calculate exponents in Python using the `**` operator. (Why not the `^` operator? Blame [Fortran](https://softwareengineering.stackexchange.com/a/331392).)

```py
square = 2 ** 2
# square = 4

cube = 2 ** 3
# cube = 8
```
## Assignment

In the social media industry, there is a concept called "spread" - it refers to how much a post spreads due to "reshares" after all of the original author's followers see it. As it turns out, social media posts spread at an _exponential_ rate! We've found that the estimated spread of a social post can be calculated using the following formula:

```
estimated_spread = average_audience_followers * ( num_followers ^ 1.2 )
```
In the formula above, `average_audience_followers` refers to an [average](https://en.wikipedia.org/wiki/Arithmetic_mean) calculated from each number within the `audiences_followers` argument - which is a list containing the individual follower counts of the author's followers. For example, if `audiences_followers = [2, 3, 2, 19]`, then:

- the author has `4` total followers
- each follower has _their own_ `2`, `3`, `2`, and `19` followers, respectively.

Complete the `get_estimated_spread` function by implementing the formula above. The only input is `audiences_followers`, which is a list of the follower counts of all the followers the author has. Return the estimated spread. If the `audiences_followers` list is empty, return 0.
## Answer
```python
def get_estimated_spread(audiences_followers):
    if audiences_followers == []:
        return 0
    else:
        num_followers = int(len(audiences_followers))
        average = sum(audiences_followers) / len(audiences_followers) 
        spread = round(average * (num_followers ** 1.2))
        return spread
```
# Exponents grow very quickly

Exponents grow very large [**very fast**](https://en.wikipedia.org/wiki/Wheat_and_chessboard_problem). Let's take a look at an example.

## Assignment

While the influencers who use our platform are really great at taking selfies, most _aren't_ super great at math. We need to write a tool that predicts an influencer's follower growth over time.

Complete the `get_follower_prediction` function.

- **"fitness" influencers**: follower count **quadruples** each month
- **"cosmetic" influencers**: follower count **triples** each month
- **All other influencers**: follower count **doubles** each month

For example, if a fitness influencer starts with `10` followers, then after `1` month they should have `40` followers. After `2` months, they would have `160` followers; etc.

Use a slightly modified [geometric sequence formula](https://en.wikipedia.org/wiki/Geometric_progression): `total = a1 * r^n`

## Explanation of the formula

_Keep in mind we're using a slightly modified formula for our assignment._
## Answer
```python
def get_follower_prediction(follower_count, influencer_type, num_months):
    if influencer_type == "fitness":
        fitness_followers = follower_count * (4 ** num_months)
        return fitness_followers
    elif influencer_type == "cosmetic":
        cosmetic_followers = follower_count * (3 ** num_months)
        return cosmetic_followers
    else:
        return follower_count * (2 ** num_months)
    
```
# Exponents grow very quickly

Exponents are a huge part of algorithmic analysis.

Exponents grow very large **very fast**. We deal with that fact in some of the following scenarios:

- To slow down attackers in cryptography and security
- To avoid exponential growth in a program's "slowness"
- To avoid storing exponential amounts of information on our servers

![quadratic vs linear](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/VoKRRys.png)

Notice how the doubling formula, `2*x`, results in _linear_ or _straight_ growth.

The quadratic formula, `x^2`, keeps growing _faster and faster_.