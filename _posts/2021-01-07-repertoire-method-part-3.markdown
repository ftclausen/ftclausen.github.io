---
title: "Another repertoire method example from Concrete Mathematics"
date: 2021-01-07 20:16
classes: wide
use_math: true
categories: mathematics
---

This is my third post on the repertoire method. This method seems quite magical (or perhaps that means I don't fully
understand it 😅). In this case I am referring, as is often the case, to the book Concrete Mathematics by Graham,
Patishnick, and Knuth - in particular the example 3, equation 2.40 under "General Methods" for solving a summation.
There they present many ways to find the sum of squares $\unicode{9744}_n$ and what I am attempting to cement my knowledge in is the
repertoire method approach in the aforementioned example. It's frightfully clever.

To reproduce it here:

$$
\begin{align}
R_0 &= \alpha \\
R_n &= R_{n-1} + \beta + \gamma n + \delta n^2 \ \ \ \text{for} \ n > 0 \\
\end{align}
$$

This will have a solution of the general form

$$
R_n = A(n)\alpha + B(n)\beta + C(n)\gamma + D(n)\delta
$$

Conveniently we have already determined $A(n)$, $B(n)$, and $C(n)$ which I wrote about in [this blog
post](http://ftclausen.github.io/mathematics/concrete-mathematics/concrete-mathematics-repertoire-method-section-2.2-the-sequel/). More rigorously we can say that because if we set $\delta$ to $0$ then we end up with what we determined before.

The book then says, rather tersely for me anyway,

> If we now plug in $R_n = n^3$, we find that $n^3$ is the solution when $\alpha = 0$, $\beta = 1$, $\gamma = –3$, $\delta = 3$. Hence
>
>$$
> 3D(n) - 3C(n) + B(n) = n^3
> $$

The process to arrive at that is pretty much the same as the $n^2$ [as discussed
here](http://ftclausen.github.io/mathematics/concrete-mathematics/concrete-mathematics-repertoire-method-section-2.2-the-sequel/).
Which goes something like this:

1 - We start with our recurrence form for which we want to find a closed form
$$
R_n = R_{n-1} + \beta + \gamma n + \delta n^2 \ \ \ \text{for} \ n > 0
$$

As above the closed form will have the below general structure; we need to find $D(n)$ - we already know the rest [from
before](http://ftclausen.github.io/mathematics/concrete-mathematics/concrete-mathematics-repertoire-method-section-2.2-the-sequel/):
$$
R_n = A(n)\alpha + B(n)\beta + C(n)\gamma + D(n)\delta
$$

2a - This is the magic bit: we set the recurrence to $n^3$ and see what $\alpha$, $\beta$, $\gamma$, and $\delta$ need to
   be in order to make the RHS of the recurrence equal to $n^3$

$$
\begin{align}
n^3 &= (n-1)^3 + \alpha + \beta + \gamma n + \delta n^2 \\
    &= n^3 + -3n^2 -1 + \alpha + \beta + \gamma n + \delta n^2 \\
\end{align}
$$

2b - Re-arrange things to make the process clearer

$$
\begin{align}
n^3 &= n^3 + \delta n^2 - 3n^2 + \gamma n + 3n + \beta -1 + \alpha \\
    &= n^3 + n^2(\delta - 3) + n(\gamma + 3) + \beta -1 + \alpha 
\end{align}
$$

2c -  Now we can see what we need to set $\alpha$, $\beta$, $\gamma$, and $\delta$ to in order to make that equation true
i.e. $n^3 = n^3$. Specifically $\alpha = 0$, $\beta = 1$, $\gamma = -3$, and $\delta = 3$

3 - Then we substitute (linearly apply?) the found values for $\alpha$, $\beta$, $\gamma$, and $\delta$ into
   the general form set to n^3.

$$
\begin{align}
n^3 = A(n)\cdot0 + B(n)\cdot1 + C(n)\cdot-3 + D(n)\cdot3 \\
n^3 = B(n) - 3C(n) + 3D(n) \\
3D(n) - 3C(n) + B(n) = n^3 \\
\end{align}
$$

Last re-ordering to match what is in the book at the top of page 45. This will tell us what $D(n)$ needs to be linearly
combined with - namely $3$ - for that general form to be $n^3$.

3 - Express in terms of our target - sum of squares $\unicode{9744}_n$. Now that we have built a repertoire let's see what things have to be set to in order to find $\unicode{9744}_n$. So $\unicode{9744}_n$ equals $\unicode{9744} _{n-1} + n^2$ thus we'll get $\unicode{9744}_n = R_n$ by setting $\alpha
= \beta = \gamma = 0$ and $\delta = 1$. In other words $R_n = A(n)\cdot0 + B(n)\cdot0 + C(n)\cdot0 + D(n)\cdot1$.

So this shows us that $D(n)$ is what we are after - namely - $\unicode{9744}_n$.

4 - Finally substitute in the already known values for $A(n)$, $B(n)$, $C(n)$ and *finally* solving for $D(n)$.

As mentioned above, surprisingly, solving for $D(n)$ gives us our closed form! Let's take a look:

$$
\begin{align}
3D(n) &= n^3 + 3C(n) - B(n) \\
3D(n) &= n^3 - \frac{3(n+1)n}{2} - n \\
3D(n) &= n(n + 1/2)(n + 1) \\
D(n) &= \frac{n(n + 1/2)(n + 1)}{3} \\
\end{align}
$$

So sometimes, in the process of building a repertoire, one of the items actually ends up being The Solution!
