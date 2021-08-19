---
title: "Concrete Mathematics Notes: Summation by Parts"
date: 2021-08-17 20:34
classes: wide
use_math: true
categories: mathematics
---

Just a quick note to more verbosely explain some things that took a while to sink in. This post primarily concerns page
55 of Concrete Mathematics (Knuth, Graham, Patashnick) where they derive _Summation by Parts_.

## Applying Difference Operator to Product of Two Functions

This is equation 2.54 and, at first, I did not understand the second step especially the section highlighted in blue

$$
\begin{align}
\Delta(u(x)v(x)) &= u(x+1)v(x+1) - u(x)v(x) \\
                 &= u(x+1)v(x+1) \boldsymbol{\color{blue}{- u(x)v(x+1) + u(x)v(x+1)}}\color{#3d4144} - u(x)v(x) \\
                 &= u(x)\Delta v(x) + v(x+1)\Delta u(x)
\end{align}
$$

My initial reaction was: wtf is the new terms above? Had me stumped for a while I must confess.. Then I realised they
were just cancelling out extras conjured into existence like a
quickly annihilating virtual particle/anti-particle in the quantum vacuum. These "nothing terms" are used for factoring to arrive at our
difference-of-a-product form at 2.54 in the book. So, more verbosely, the factoring step is in blue this time:

$$
\begin{align}
\Delta(u(x)v(x)) &= u(x+1)v(x+1) - u(x)v(x) \\
                 &= u(x+1)v(x+1) - u(x)v(x+1) + u(x)v(x+1) - u(x)v(x) \\
                 &= \boldsymbol{\color{blue}v(x+1)(u(x+1) - u(x)) + u(x)(v(x+1) - v(x))\color{#3d4144}} \\
                 &= v(x+1)\Delta u(x) + u(x)\Delta v(x) \\
                 &= u(x)\Delta v(x) + v(x+1)\Delta u(x)
\end{align}
$$

Last two lines just swapped to end up where the book does. This seemed kind of magical to me but, I suppose, it works
due to the commutative and associative laws of addition/multiplication. I even tried it out with actual numbers and it works :D

## Deriving Actual Summation by Parts

My next moment of confusion came when deriving the actual summation by parts. We start with the compact rule for
difference of a product using the $E$ "nuisance" (quoting the book here :) ):

$$
\Delta(uv) = u\Delta v + Ev\Delta u
$$

Then they say:

> Taking the indefinite sum on both sites of this equation, and rearranging its terms, yields the advertised rule for
summation by parts:

$$
\sum u\Delta v = uv - \sum Ev\Delta u
$$

I get the rearrangement of terms but how $\Delta(uv)$ becomes $uv$ was a bit of a mystery. After the requisite amount of
time overthinking it, it is just literally following through the steps of making $\Delta(uv)$ $-\Delta(uv)$ and shaking
things out. I find it useful to expand everything and go from there:

$$
\begin{align}
-\Delta(uv) &= -\Delta(u(x)v(x)) \\
           &= -(u(x+1)v(x+1) - u(x)v(x)) \\
           &= -u(x+1)v(x+1) + u(x)v(x) \\
           &= -u(x)v(x) \\
           &= -uv \\
\end{align}
$$

Then, in 2.56 in the book, the rearrangment part makes it positive:

$$
\begin{align}
\Delta(uv) &= u\Delta v + Ev\Delta u \\
-u\Delta v &= -\Delta(uv) + Ev\Delta u \\
u\Delta v &= \Delta(uv) - Ev\Delta u
\end{align}
$$

Finally they take the indefinite sum on both sides giving us the summation by parts rule

$$
\sum u\Delta v = \Delta(uv) - \sum Ev\Delta u
$$

