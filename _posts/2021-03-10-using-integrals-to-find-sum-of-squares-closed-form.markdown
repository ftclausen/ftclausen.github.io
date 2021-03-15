---
title: "Using integrals to find sum of squares closed form - part 1"
date: 2021-03-10 20:08
classes: wide
use_math: true
categories: mathematics
---

As with all my posts here I'll try to give a more verbose version of what the book covers; specifically how to get the
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

This post covers the first "sub-method" in _Method 4: Replace sums by integrals_

# Examine Error in Approximation $n^3/3$

What confused me is this is not exactly the same as counting the error wedges on the beautiful graph in the book. This
is literally the error in our first pass closed form approximation using the integration rules where the area under
$x^2$ is $\int x^2dx = n^3/3$. Thus we know the sum of squares is approximately $\frac{1}{3}n^3$.

Thus the error in approximation is $E\_n = \unicode{9744}\_n - \frac{1}{3}n^3$ and therefore $E\_{n-1} = \unicode{9744}\_{n-1} - \frac{(n-1)^3}{3}$. Rearranging we get $\color{green}{\unicode{9744}\_{n-1} = E\_{n-1} + \frac{(n-1)^3}{3} }$

We also know what $\color{red}{\unicode{9744}\_n = \unicode{9744}\_{n-1} + n^2}$. The book then very elegantly puts this all together to find a simple recurrence for $E_n$ as follows

$$
\begin{align}
E_n &= \unicode{9744}_n - \frac13 n^3 = \color{red}\unicode{9744}_{n-1} + n^2 \color{#3d4144} - \frac13 n^3 \\
    &= \color{green}E_{n-1} + \frac{(n-1)^3}{3} \color{#3d4144}+ n^2 - \frac13 n^3 \\
    &= E_{n-1} + n - \frac13
\end{align}
$$

Therefore if we could just find the closed form of $E_n$ we're sorted! 

