---
title: "Using integrals to find sum of squares closed form"
date: 2021-03-10 20:08
classes: wide
use_math: true
categories: mathematics
---

As with all my post here I'll try to give a more verbose version of what the book covers; specifically how to get the
sum of squares closed form all the way. All the book explains is finding the error in the approximation and the
approximation itself.

This post assumes you have the book to hand although hopefully it's readable on it's own. Any mistakes in this text are
my own.

In Concrete Mathematics (Knuth, Graham, Patashnik) they use integrals to as a stepping stone to find the closed form for
the sum of squares. I am not as eloquent as they are so I'll just quote them here

> In calculus, an integral can be regarded as the area under a curve, and we can approximate this area by adding up the
> areas of long, skinny rectangles that touch the curve. We can also go the other way if a collection of long, skinny
> rectangles is given: Since $\unicode{9744}_n$ is the sum of the areas of rectangles whose sizes are $1 × 1$, $1 × 4$, . . . , $1 × n^2$, it is
> approximately equal to the area under the curve $f(x) = x^2$ between $0$ and $n$.

I am not even going to try and replicate the graph in the book so here is one made with
[mathisfun.com](https://www.mathsisfun.com/calculus/integral-approximation-calculator.html) integral approximation
calculator. The area under this graph is approximately $\unicode{9744}_n$

![approximation x^2](/images/approximation_x_squared.png)

# Part 1 - Examine Error in Approximation $n^3/3$

What confused me is this is not exactly the same as counting the error wedges on the beautiful graph in the book. This is
literally the error in our first pass closed form approximation using the derivative rules.

So we are working out the area under the curve of $x^2$ which, according to the rules of derivatives, is $\int x^2dx =
n^3/3$. Thus we know the sum of squares is approximately $\frac{1}{3}n^3$.

Thus the error in approximation is $E_n = \unicode{9744}_n - \frac{1}{3}n^3$ thus, solving for $\unicode{9744}_n$ we
have, $\color{blue}{\unicode{9744}_n = E_n +\frac{1}{3}n^3}$. We also know what $\unicode{9744}_n = \unicode{9744}_{n-1} + n^2$. Thus 

They very elegantly use this closed form to deduce that $E_n = E_{n-1} + n - \frac{1}{3}$. Thus

$$
\unicode{9744}_n = E_n + \frac{1}{3}n^3 \\
$$

Therefore if we could find the closed form of $E_n$ we're sorted! 

