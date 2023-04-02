---
title: "Josephus - generalised binary recurrence unfolding"
date: 2023-03-28 16:00
classes: wide
use_math: true
categories: mathematics
---

I have some confusion around the meaning of $b_m$ and $b_j$. I *think* it is as follows:

- $\beta_j$ - In the rewritten 1.15 where $\beta_0 = \beta = -1$ and $\beta_1 = \gamma = 1$
- $\beta_m$ - In the binary representation or expansion. I.e. $(\beta_m \beta_{m-1} ... \beta_1 \beta_0)_2$. As in
    $\beta_m$ is either a $0$ or a $1$.

Note on book context: Page 15, from just before equation 1.15.

# Random Ramblings on Above

About re-using the cyclical bit-shifting "magical solution" to the Josephus problem in a more generalised
context. Firstly, the magical solution they are talking about:

$$
J((b_mb_{m-1}...b_1b_0)_2) = (b_{m-1}...b_1b_0b_m)_2
$$

They then ask

> Does the generalised Josephus recurrence admit of such magic? Sure, why not?

They then go on to, very cleverly, rewrite generalised equation in a new form that allows us to look at it in a
different way. Firstly, the original generalised equation 1.11:

$$
\begin{align}
f(1)    &= \alpha \\
f(2n)   &= 2f(n) + \beta\text{,} \ \ \ \ \ \ \text{for } n \geq 1\text{;} \\
f(2n+1) &= 2f(n) + \gamma\text{,} \ \ \ \ \ \ \text{for } n \geq 1\text{.} \
\end{align}
$$

Then the rewritten one as 1.15

$$
\begin{align}
f(1)      &= \alpha; \\
f(2n + j) &= 2f(n) + \beta, \ \ \ \ \ \ \text{for}\ j=0,1\ \text{and}\ n\geq 1 \\
\end{align}
$$

Now they say

> If we let $\beta_0 = \beta$ and $\beta_1 = \gamma$

Which **I think** means: if we encounter a binary $0$ ($\beta_0)$ in the number then we substitute the $\beta = -1$
_from the generalised form 1.11_ equation. Otherwise, if we encounter a binary $1$ ($\beta_1$) then we substitute the
$\gamma = 1$ also from the generalised equation 1.11.

It is unfortunate that they overload $\beta$  in the generalised equation 1.11, rewritten generalised equation 1.15,
and in the expansion notation $\beta_m$ where $m$ is the position/power of two in the binary number. I think that is the
root of my confusion.

The table presented on page 16 (of the physical book, second edition) is very enlightening in demonstrating the
rewritten (1.15) equation is used:

$$
\begin{array}
n &= &(&1 &1 &0 &0 &1 &0 &0 &)_2 &= 100
\\
\hline
f(n) &= &(&1 &1 &-1 &-1 &1 &-1 &-1 &)_2
\\
\end{array}
$$

Where:

- $\beta = -1$ is applied to $\beta_0$ i.e. binary $0$.
- $\gamma = 1$ is applied to $\beta_1$ i.e. binary $1$.

Applying above wherever we find a binary 1 convert to decimal with the appropriate power of two $m$ as in $2^m$ then
multiply by $1$. Likewise, wherever we find a binary 0, convert to decimal with appropriate power of two $m$ as in $2^m$
then multiply by $-1$. Doing the above leads us to:

$$
+64 +32 -16 -8 +4 -2 -1 = 73
$$

So J(100) = 73
