---
id: 01J5VGT43T0AB1TBD6MK99GHVR
title: Learn Algorithms - Math
modified: 2024-08-24T18:53:42-04:00
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

# What is a Logarithm?

A logarithm is the _inverse of an exponent_.

24 = 16

log216 = 4

"log216" can be read as "log base 2 of 16", and means "the number of times 2 must be multiplied by itself to equal 16".

"log216" can also be written as `log2(16)`

Some more examples:

- `log2(2) = 1`
- `log2(4) = 2`
- `log2(8) = 3`
- `log2(16) = 4`
- `log2(32) = 5`
- `log10(100) = 2`
- `log10(1000) = 3`
- `log10(10000) = 4`

## Logs (logarithms) in Python

Just `import` the `math` library and use the `math.log()` function.

```py
import math

print(f"Logarithm base 2 of 16 is: {math.log(16, 2)}")
# Logarithm base 2 of 16 is: 4.0
```

## Assignment

We want to be able to give our influencers a simple "influencer score". It will be a small number, like less than 100. This will make it easier to "rank" them against each other in terms of how many people they are "influencing".

Complete the `get_influencer_score` function. The formula for an influencer score is:

> average_engagement_percentage * log2(num_followers)

## Answer
```python
import math


def get_influencer_score(num_followers, average_engagement_percentage):
    return average_engagement_percentage * math.log(num_followers, 2)
```
# Logarithm Review

**Logarithms grow very slowly**

A logarithm is the _inverse of an exponent_.

24 = 16

log216 = 4

"log216" can be read as "log base 2 of 16", and means "the number of 2's to multiply together in order to equal 16".

"log216" can also be written as `log2(16)`

Some more examples:

- `log2(2) = 1`
- `log2(4) = 2`
- `log2(8) = 3`
- `log2(16) = 4`
- `log2(32) = 5`
- `log10(100) = 2`
- `log10(1000) = 3`
- `log10(10000) = 4`
![[Pasted image 20240824164455.png]]

# Factorials

In math, the [factorial](https://en.wikipedia.org/wiki/Factorial) of a positive integer `n`, written `n!`, is the product of all positive integers less than and equal to `n`.

```
5! = 5 * 4 * 3 * 2 * 1
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

The results of a factorial grow _even faster_ than exponentiation!

`n!` grows faster than `2^n`:

||n!|2^n|
|---|---|---|
|2|2|4|
|3|6|8|
|4|24|16|
|5|120|32|
|6|720|64|

## Assignment

Socialytics allows influencers to schedule Instagram posts to be published in the future. We've found that the order in which the posts are published drastically affects their performance. Complete the `num_possible_orders` function. It takes as input the number of posts an influencer has in their schedule and returns the total number of possible orders in which we could publish the posts.

## Tip: Ordering problems have factorial growth

You might wonder why we would _care_ about factorials. They're useful when trying to count how many different orders there are for things.

For example, how many different ways can a deck of cards be arranged?

- The first card could be any of the `52` cards.
- The second card can be any of the `51` remaining cards.
- The third card can be any of the `50` remaining cards, and so on.

This means we have `52` options multiplied by `51` options multiplied by `50` options, and so on.

```
52 * 51 * 50 ... * 2 * 1
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Or in other words, `52!`

## Answer
```python
def num_possible_orders(num_posts):
    if num_posts < 0:
        return 0
    elif num_posts == 0 or num_posts == 1:
        return 1
    else:
        fact = 1
        while(num_posts > 1):
            fact *= num_posts
            num_posts -= 1
        return fact
            
```

# Exponential Decay

In physics, [exponential decay](https://en.wikipedia.org/wiki/Exponential_decay) is a process where a quantity decreases over time at a rate proportional to its current value.

We've found that Instagram influencers tend to lose followers similarly. This means that the more followers you have, the faster you lose them.

## Assignment

Complete the `decayed_followers` function.

It calculates the final value of a quantity after a certain time has passed, given its initial value and a rate of decay. Return the remaining followers.

```
remaining_total = quantity * ( retention_rate ^ time )
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

The `retention_rate` is the opposite of `fraction_lost_daily`. If an influencer lost 0.2 (or 20%) of their followers each day, then the retention rate would be 0.8 (or 80%).

### Example

- `intl_followers = 100`
    
- `fraction_lost_daily = 0.5`
    
- After 0 days: `100` followers
    
- After 1 day: `50` followers
    
- After 2 days: `25` followers
    
- After 3 days: `12.5` followers (rounded down)
    

_Note: This isn't the exact formula shown on Wikipedia, but it's the same idea._