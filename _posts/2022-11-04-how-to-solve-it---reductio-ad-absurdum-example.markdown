---
title: "How to solve it - Reductio ad absurdum example"
date: 2022-11-04 15:35
classes: wide
use_math: true
categories: mathematics
---

As before, this is from  George Pólya's ["How to Solve
It"](https://amzn.to/3S8cHr3) book. I am reading this because I
want to sharpen my problem solving skills and, eventually, be able to make derivations like this in Concrete
Mathematics.

In this example they propose the problem of

> Write numbers using each of the ten digits exactly once so that the sum of the numbers is exactly 100

Then the book goes through some attempts where we relax the conditions.

## Relax that they must add up to 100

Here are some examples from the book where they try to get close by relaxing the condition that they need to add up to
100:

$$
19 + 28 + 37 + 46 + 50 = 180
$$

or

$$
19 + 28 + 30 + 7 + 6 + 5 + 4 = 99
$$

close but no cigar.

## Relax that the numbers should be unique

$$
19 + 28 + 31 + 7 + 6 +  5 + 4 = 100
$$

Here we get 100 but we are repeating some numbers. It would seem it is not possibles to satisfy both conditions
(uniqueness and adding up to 100) at once.

We need to _prove that it is impossible to solve both conditions at the same time_.

## Setting about trying to satisfy both conditions

Now we go about being optimistic and try to satisfy both conditions. The book does this by being very clever I thought;
simply pretend both conditions _are_ true even though we suspect they are not.

Some ideas:

- There are ten figurs, 0 to 9, and each one must appear at least once
- The sum of simply those figured is $0 + 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9$ is $45$

Now comes the bit I found confusing:

> Some of these figures denote units and some of them tens

I thought they meant in the above some of single figures. They were all units but then I realised the author meant
_while adding to get 100_ not in the sum to make $45$. Continuing:

> It takes a little sagacity to hit upon the idea that the _sum of the figures denoting tens_ may be of some
importance.

This also caused me some confusion (a common state for me to be in) but using the examples above:


$$
19 + 28 + 30 + 7 + 6 + 5 + 4 = 99
$$

Here the sum of the tens figures are

$$
10 + 20 + 30
$$

From the numbers 19, 28, 30. Another example from above:

$$
19 + 28 + 31 + 7 + 6 +  5 + 4 = 100
$$

Where the tens are, again,

$$
10 + 20 + 30
$$

We can see these give us the _most progress_ to 100 (i.e. to 60). The others (part of the 45) get us the rest of the
way. Now $60 + 45 = 105$ which is obviously too much so we need to subtract the ones we've already used to get to 60.
This, then, finally brings us to:

> It takes a little sagacity to hit upon the idea that the _sum of the figures denoting tens_ may be of some
> importance.
> _In fact, let $t$ stand for this sum_. Then the sum of the remaining figures, denoting units, is $45 - t$. Therefore
the numbers in the set must be

$$
10t + (45 - t) = 100
$$

Breaking that down

- $10t$ - This is the 60 talked about above. Or, $6 * 10$ because as represented here.
- $45 - t$ - This is the remaining units figured

So, altogether, this should add to 100.

## Solving for $t$ - the sum of tens

Solving for $t$ we get

$$
t = \frac{55}{9}
$$

Here we get the absurdum beacause, of course, $t$ should be an integer otherwise it won't add to 100. Thus we can't
satisfy both conditions at once.
